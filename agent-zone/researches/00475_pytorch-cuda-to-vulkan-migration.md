# PyTorch CUDA 實作遷移至 Vulkan / 其他 GPU 的多種途徑

## 摘要

本報告整理將「以 PyTorch + CUDA 撰寫的實作」遷移到 **Vulkan**（或其他非 NVIDIA GPU 平台）的多種可行途徑。由於 CUDA 僅能在 NVIDIA GPU 上執行，若要將實作帶到 AMD、Intel、Apple Silicon 或任何支援 Vulkan 的 GPU，需選擇一種遷移/重構策略。本報告依「改動幅度」與「技術路線」區分為四大類：**零程式碼改動（透過官方/廠商後端）**、**ONNX 匯出路線**、**編譯器路線 (ML Compiler)**、以及 **自訂 Vulkan 後端/卷積核心**。每條路線都附上成熟度、實作難易、代表性專案與 GitHub star 數。

---

## 背景：為什麼要「去 CUDA」(deCUDA)

CUDA 是 NVIDIA 專有、僅適用於 NVIDIA GPU 的平行計算平台。Vulkan 則是由 Khronos Group 維護的跨廠牌開放圖形/計算 API，可在 AMD、Intel、NVIDIA、Apple Silicon（透過 MoltenVK）等裝置上運作[^khronos]。因此「去 CUDA 化」的核心目標是讓既有 PyTorch 實作可以脫離 NVIDIA 的 CUDA 依賴，改利用 Vulkan 或其他開放標準（OpenCL、HIP、SYCL）在多種 GPU 上執行。

需要先說明一個關鍵現實：**PyTorch 官方上游的 Vulkan 後端已被廢棄**。從 PyTorch 1.7 起加入的 `USE_VULKAN` 實驗性後端（主要針對 Android 行動 GPU），目前官方文件已標註「no longer maintained」並建議改用 ExecuTorch Vulkan Delegate[^pyvulkan]. 因此當前的遷移選擇集中在：廠商後端、ONNX 匯出、ML 編譯器、與社群自製後端四大類。

---

## 路線一：零/低程式碼改動 — 透過廠商或官方後端

這類方案不改寫模型邏輯，只改變 PyTorch 的「裝置」與「後端」，改動幅度最小，適合希望保留訓練能力的使用者。

### 1. AMD HIP / hipify（CUDA → HIP 移植）

AMD 的 ROCm 生態提供 **HIPIFY** 工具，可自動將 CUDA 原始碼轉換為可攜式的 HIP C++[^hipify]. HIP（Heterogeneous-computing Interface for Portability）程式碼既能編譯到 AMD GPU（透過 ROCm），也能編譯回 NVIDIA CUDA，形成雙平台方案[^rocmofficial]. 對 PyTorch 而言，官方就是用它把手動 PyTorch 的 AMD 版本移植出來。

- **成熟度**：非常成熟，是 CUDA → AMD 的標準遷移路徑。
- **訓練？** 是；**推論？** 是。
- **與 Vulkan 的關聯**：HIP 跑在 ROCm 而非 Vulkan 之上，但支援 ROCm 的 AMD GPU 通常也支援 Vulkan。
- **缺陷**：消費級 AMD GPU（gfx1150 等）的 ROCm 支援可能不穩定，有時需要架構偽裝 (arch spoofing)。
- **代表專案**：ROCm/HIPIFY，約 **731** stars[^hipify]。

### 2. PyTorch XPU / SYCL（Intel 路徑）

自 PyTorch 2.4–2.5 起，Intel GPU 透過 **XPU 後端**（`torch.xpu`）獲得官方支援，建構在 Intel 的 oneAPI DPC++ 編譯器之上，而 DPC++ 是 SYCL 標準的實作[^xpu]. SYCL 是 Khronos 的開放標準（與 Vulkan 同一標準組織），理論上可作為 Vulkan 後端，在 Intel 上則透過自家的 GPU 驅動堆疊執行。

- **成熟度**：對 Intel 資料中心 GPU（Max 系列）與客戶端 GPU（Arc、內顯）已是 **production-ready**，支援訓練與推論。
- **訓練？** 是；**推論？** 是。
- **改動**：`.to("xpu")` 即可，Python 端幾乎不用改。
- **代表專案**：Intel LLVM/DPC++，約 **1,500** stars[^intel].

### 3. Microsoft DirectML（DirectX 12，不直接是 Vulkan）

DirectML 是微軟的 DirectX 12 機器學習加速函式庫，可在所有 DirectX 12 相容 GPU（AMD、Intel、NVIDIA、Qualcomm）上執行[^dml]. PyTorch 端透過 `torch-directml` 外掛以 `.to("dml")` 呼叫[^pydml]. 

- **成熟度**：目前為 **maintenance mode**——不再新增功能，僅安全更新。微軟建議 Windows 11 24H2+ 改用 Windows ML（WinML）。
- **訓練？** 是；**推論？** 是。
- **與 Vulkan 的關聯**：DirectX 12 可經由相容層（如 DXVK）在 Vulkan 上執行，但原生上是 D3D12，並非直接跑在 Vulkan。
- **代表專案**：DirectML，約 **2,600** stars[^dml].

### 4. pytorch_dlprim（OpenCL 後端）

`pytorch_dlprim`（也稱 `pytorch_ocl`）是 PyTorch 的 out-of-tree **OpenCL 後端**，透過 `PrivateUse1` 註冊 `ocl` 裝置，支援在 AMD、NVIDIA、Intel GPU 上訓練[^dlprim]. 使用 DLPrimitives 函式庫提供最佳化的 GEMM 與卷積 kernel（含 Winograd）[^dlprims].

- **成熟度**：早期但可用，已在 20+ 個 torchvision 網路上驗證正向+反向，支援訓練與推論。
- **性能**：在 AMD GPU 約為原生 CUDA 的 50–70%。
- **改動**：`device='ocl:0'` 即可。
- **代表專案**：pytorch_dlprim，約 **401** stars[^dlprim].

> 註：OpenCL 與 Vulkan 都是 Khronos 的開放計算標準。如果你的目標 GPU 支援 OpenCL 而非 Vulkan，這條路線也很適用；兩者在「跨廠牌開放後端」的定位上是替代關係。

---

## 路線二：ONNX 匯出路線（推論為主）

將 PyTorch 模型匯出為 ONNX，再用支援 Vulkan/其他 GPU 的推論引擎執行。適用於「只要推論、不需訓練」的情境。

### 5. ONNX Runtime WebGPU Execution Provider（經由 Dawn 走 Vulkan）

ONNX Runtime 提供 **WebGPU Execution Provider**，在原生平台上使用 Google 的 **Dawn**（WebGPU 實作）分派到底層的 **Vulkan**、D3D12 或 Metal[^ortwebgpu]. 流程是：PyTorch 模型 → ONNX → ONNX Runtime + WebGPU EP。

- **成熟度**：積極開發中（外掛 EP，透過 `onnxruntime-ep-webgpu` PyPI 套件發行），跨平台：Windows（D3D12/Vulkan）、Linux（Vulkan）、Android（Vulkan）、macOS（Metal）[^ortwebgpu].
- **訓練？** 否；**推論？** 是。
- **注意**：ONNX Runtime **沒有官方的 Vulkan Execution Provider**——官方清單（CUDA、TensorRT、DirectML、ROCm、OpenVINO、CoreML 等）不含 Vulkan[^ortissue], 只有 WebGPU EP 可以「間接經由 Dawn」使用 Vulkan。
- **代表專案**：ONNX Runtime，約 **22,000** stars[^ort].

### 6. onnx-vulkan-rs（社群 Rust Vulkan EP 外掛）

這是一個以 Rust 撰寫的 **Vulkan Execution Provider** 外掛，可用於 ONNX Runtime，也提供獨立的純 Rust 引擎（內建自己的 ONNX parser，不需 ORT）[^ortvkrs]. 透過 `ash` + `gpu-allocator` + `naga`（WGSL→SPIR-V）呼叫 Vulkan compute。

- **效能**：宣稱 int4 LLM 達約 160 tok/s（gemma3-1b）、約 71 tok/s（qwen2.5-VL），在 Parakeet encoder 上比 ORT CPU EP 快 7.94 倍[^ortvkrs].
- **成熟度**：非常年輕（約 35 commits），但積極開發；支援協同矩陣（tensor core）kernel。
- **需求**：外掛 EP API 需 ORT ≥ 1.23。
- **代表專案**：automataIA/onnx-vulkan-rs，約 **7** stars[^ortvkrs].

### 7. GGML / llama.cpp Vulkan 後端

若實作是 LLM 推論導向，可考慮將模型轉為 **GGUF** 格式並使用 **llama.cpp**（及其底層張量函式庫 ggml）內建的 **Vulkan 後端**[^llamaCpp]. ggml 的 Vulkan 後端是正式支援的後端之一，與 CUDA、HIP、Metal、SYCL、OpenCL 並列[^llamaCpp].

- **成熟度**：**production-grade**，llama.cpp 是最廣用的在地 LLM 推論引擎，Vulkan 後端積極維護。
- **訓練？** 否；**推論？** 是（GGUF 量化 LLM 推論）。
- **注意**：這是替換整個執行引擎，不是 PyTorch 執行期替代；不適合訓練。
- **代表專案**：ggml-org/llama.cpp，約 **130,700** stars[^llamaCpp].

---

## 路線三：ML 編譯器路線（自動產出 SPIR-V）

透過編譯器框架直接將 PyTorch 模型編譯成 Vulkan 的 **SPIR-V shader**，自動化程度高，適合需要最佳化 kernel 的部署情境。

### 8. Apache TVM Vulkan Runtime

Apache TVM 是開放 ML 編譯器，可將 PyTorch、TensorFlow、ONNX 模型編譯到多種後端，其中 **Vulkan 後端**會為每個計算 kernel 產生最佳化的 SPIR-V shader[^tvm]. 使用方式：`from_pytorch()` 匯入，target 指定 `"vulkan"`，TVM 即產生 Vulkan runtime module[^tvm].

- **成熟度**：成熟（Apache top-level 專案），Vulkan 後端已存在多年。
- **支援**：Vulkan 1.1+、SPIR-V codegen、subgroup 運算、push constants、storage buffers、float16/float64/int8/int16/int64[^tvm].
- **訓練？** 否（以部署為主）；**推論？** 是。
- **代表專案**：apache/tvm，約 **13,800** stars[^tvm].

### 9. IREE（OpenXLA）Vulkan/SPIR-V 後端

IREE 是以 MLIR 為基礎的編譯器與執行期，其 **Vulkan/SPIR-V 後端**是一等公民[^iree]. 可經由 StableHLO 從 PyTorch/TensorFlow/JAX 匯入模型，編譯出適用於行動/邊緣裝置的精簡 runtime。AMD 曾用它進行 MLPerf 基準（SDXL）[^iree].

- **成熟度**：**production-grade**，2024 年加入 LF AI & Data Foundation 為 sandbox 專案[^iree].
- **代表專案**：iree-org/iree，約 **4,000** stars[^iree].

> 補充：目前 **Triton 沒有 → SPIR-V 後端**。PyTorch `torch.compile` 的 Triton 目前只產生 CUDA PTX 與 AMD ROCm，沒有 Vulkan/SPIR-V 產出[^triton]. 若想從 PyTorch 自動走到 Vulkan SPIR-V，IREE（經 torch-mlir/StableHLO）與 TVM 是可行的編譯器路線。

---

## 路線四：自訂 Vulkan 後端 / 自寫 SPIR-V kernel（DIY）

若需要最大掌控度、或需要訓練能力，可透過 PyTorch 的 `PrivateUse1` dispatch 機制自建 Vulkan 後端，或手寫 SPIR-V compute shader。這是社群在做的方向。

### 10. torchvulkan（社群 `PrivateUse1` Vulkan 後端）

`torchvulkan` 以 PyTorch 的 **PrivateUse1** dispatch key 註冊一個 `"vulkan"` 裝置，kernel 以 **Slang** 語言撰寫並在建置期編譯為 SPIR-V，使用 VMA 與 volk[^torchvulkan]. 支援在 AMD、Intel、NVIDIA、Apple Silicon（MoltenVK）上執行，未實作的運算子會 fallback 到 CPU[^torchvulkan].

- **成熟度**：**早期實驗型**，原生運算子集合較小。
- **訓練？** 是（支援 forward+backward）；**推論？** 是。
- **需求**：PyTorch ≥ 2.10、Vulkan 1.3，有預建 wheel，`pip install torchvulkan`[^torchvulkan].
- **改動**：`.to("vulkan")` 即可。
- **代表專案**：ZainSharief/torchvulkan，約 **6** stars[^torchvulkan].

### 11. torch-vulkan（SPIR-V kernel，AMD RDNA 導向）

另一個社群 `PrivateUse1` 後端，使用**手寫 GLSL→SPIR-V** compute shader，建構在 **Kompute** 之上，專門支援 GGUF 量化的 LLM 推論於 AMD RDNA 3.5（不需 ROCm）[^torchvk]. 支援約 10 個運算子（mm、addmm、Q4_K matmul、relu、gelu、softmax、layer_norm、embedding、attention）[^torchvk].

- **成熟度**：非常早期、**僅推論**（無 autograd 訓練）。
- **代表專案**：Peterc3-dev/torch-vulkan，約 **2** stars[^torchvk].

### 12. 直接手寫 SPIR-V kernel（DIY 最大掌控）

最底層的做法是自行撰寫 **SPIR-V compute shader**（從 GLSL/Slang 編譯），透過 Vulkan API 直接 dispatch，再以 `torch.utils.cpp_extension` 建置 PyTorch 的 C++ extension 來管理 buffer 與 dispatch[^kompute]. 

- **成熟度**：完全自訂，取決於實作能力。
- **輔助框架**：**Kompute**（約 **2,600** stars，Linux 基金會支援、穩定 v0.7.0，包裝 Vulkan 樣板）[^kompute]；**Slang**（shader 語言，可編譯為 SPIR-V）[^slang].

### 13. WebGPU（以 Vulkan 為後端）作為替代路徑

若目標是「盡可能跨平台」，可考慮直接採用 **WebGPU**（其原生就是建立在 Vulkan/Metal/D3D12 之上），而不必直接碰 Vulkan API。Rust 的 **Burn** ML 框架即因 WGPU 的跨平台性而選擇 WGPU 而非原生 Vulkan[^burn]. 此路線適合瀏覽器 / WASM 部署，或想要高層抽象的使用者。

---

## 路線比較總表

| 途徑 | 技術後端 | 訓練 | 推論 | 成熟度 | 改動幅度 | GitHub stars |
|---|---|---|---|---|---|---|
| AMD HIP / hipify | ROCm | 是 | 是 | 成熟 | 中（移植程式碼） | ~731[^hipify] |
| PyTorch XPU / SYCL | Intel/oneAPI | 是 | 是 | Production | 低 `.to("xpu")` | ~1,500[^intel] |
| DirectML | DirectX 12 | 是 | 是 | 維護模式 | 低 `.to("dml")` | ~2,600[^dml] |
| pytorch_dlprim | OpenCL | 是 | 是 | 早期 | 低 `.to("ocl")` | ~401[^dlprim] |
| ORT WebGPU EP | Vulkan(via Dawn) | 否 | 是 | 積極開發 | 中（ONNX 匯出） | ~22,000[^ort] |
| onnx-vulkan-rs | Vulkan | 否 | 是 | 實驗性 | 中（ONNX 匯出） | ~7[^ortvkrs] |
| llama.cpp ggml | Vulkan | 否 | 是 | Production | 高（GGUF 匯入） | ~130,700[^llamaCpp] |
| Apache TVM | Vulkan/SPIR-V | 否 | 是 | 成熟 | 中（TVM 編譯） | ~13,800[^tvm] |
| IREE | Vulkan/SPIR-V | 否 | 是 | Production | 中（編譯器） | ~4,000[^iree] |
| torchvulkan | Vulkan | 是 | 是 | 早期實驗 | 低 `.to("vulkan")` | ~6[^torchvulkan] |
| torch-vulkan | Vulkan | 有限 | 是 | 非常早期 | 低 `.to("vulkan")` | ~2[^torchvk] |
| 手寫 SPIR-V | Vulkan | DIY | DIY | 自訂 | 高 | N/A |
| WebGPU/Burn | Vulkan(via WGPU) | 是 | 是 | 積極開發 | 中 | —[^burn] |

---

## 選擇建議（依使用情境）

- **要保留訓練、目標是 AMD GPU**：走 **HIP/hipify**（效能最佳）。
- **要保留訓練、目標是 Intel GPU**：官方 **PyTorch XPU**。
- **要保留訓練、最大可攜性（任何支援 Vulkan 的 GPU）**：`torchvulkan` 或自訂 PrivateUse1 後端。
- **只要推論、要跨平台**：**ONNX Runtime WebGPU EP**（經 Dawn 走 Vulkan）。
- **LLM 推論**：**llama.cpp** 的 Vulkan 後端。
- **要自動最佳化 kernel**：**IREE** 或 **TVM**（編譯到 SPIR-V）。
- **Windows、任何 GPU、只要推論**：**DirectML**（注意為維護模式）。

---

## 重要注意事項與來源反思

1. **PyTorch 官方 Vulkan 後端已廢棄**，現行官方路徑是 ExecuTorch ET-VK（主要針對 Android 行動 GPU，桌面為實驗性）[^pyvulkan].
2. **ONNX Runtime 沒有官方 Vulkan EP**，只有 WebGPU EP 可間接經由 Dawn 使用 Vulkan[^ortissue].
3. **Triton（`torch.compile`）沒有 → SPIR-V 後端**，需透過 IREE/TVM 編譯器路線[^triton].
4. **star 數會隨時變動**，本報告數據為 2026-10-09 時點。社群專案（torchvulkan、torch-vulkan、onnx-vulkan-rs）star 數極低（<10），屬於「niche 專案」，因主題冷門仍值得列入，但採用以需求謹慎評估其成熟度。
5. 各來源以官方文件與專案 GitHub 為主，輔以 Khronos 官方教學，信譽與時效性較佳；社群小專案的效能數據多為作者自測，未經獨立複核，引用時宜謹慎。

---

## 參考資料

[^khronos]: Khronos Group. (n.d.). Vulkan Overview. Retrieved 2026-10-09, from https://www.khronos.org/vulkan/
[^pyvulkan]: PyTorch. (n.d.). Vulkan Backend User Workflow (Deprecated). Retrieved 2026-10-09, from https://docs.pytorch.org/tutorials/unstable/vulkan_workflow.html
[^hipify]: AMD ROCm. (n.d.). HIPIFY. Retrieved 2026-10-09, from https://github.com/ROCm/HIPIFY
[^rocmofficial]: AMD. (n.d.). HIPIFY documentation. Retrieved 2026-10-09, from https://rocm.docs.amd.com/projects/HIPIFY/en/latest/
[^xpu]: PyTorch. (n.d.). Intel GPU Support Getting Started. Retrieved 2026-10-09, from https://docs.pytorch.org/docs/2.14/notes/get_start_xpu.html
[^intel]: Intel. (n.d.). Intel LLVM / DPC++. Retrieved 2026-10-09, from https://github.com/intel/llvm
[^dml]: Microsoft. (n.d.). DirectML. Retrieved 2026-10-09, from https://github.com/microsoft/DirectML
[^pydml]: Microsoft. (n.d.). PyTorch with DirectML. Retrieved 2026-10-09, from https://learn.microsoft.com/en-us/windows/ai/directml/pytorch-windows
[^dlprim]: Beilis, A. (n.d.). pytorch_dlprim (OpenCL backend for PyTorch). Retrieved 2026-10-09, from https://github.com/artyom-beilis/pytorch_dlprim
[^dlprims]: Beilis, A. (n.d.). DLPrimitives. Retrieved 2026-10-09, from https://github.com/artyom-beilis/dlprimitives
[^ortwebgpu]: Microsoft. (n.d.). ONNX Runtime WebGPU Execution Provider. Retrieved 2026-10-09, from https://onnxruntime.ai/docs/execution-providers/WebGPU-ExecutionProvider.html
[^ortissue]: Microsoft. (n.d.). Submit Vulkan Execution Provider feature request. Retrieved 2026-10-09, from https://github.com/microsoft/onnxruntime/issues/21917
[^ort]: Microsoft. (n.d.). onnxruntime. Retrieved 2026-10-09, from https://github.com/microsoft/onnxruntime
[^ortvkrs]: automataIA. (n.d.). onnx-vulkan-rs. Retrieved 2026-10-09, from https://github.com/automataIA/onnx-vulkan-rs
[^llamaCpp]: ggml-org. (n.d.). llama.cpp. Retrieved 2026-10-09, from https://github.com/ggml-org/llama.cpp
[^tvm]: Apache Software Foundation. (n.d.). Vulkan Runtime — Apache TVM. Retrieved 2026-10-09, from https://tvm.apache.org/docs/arch/runtimes/vulkan.html
[^tvmrepo]: Apache Software Foundation. (n.d.). apache/tvm. Retrieved 2026-10-09, from https://github.com/apache/tvm
[^iree]: IREE project. (n.d.). GPU - Vulkan - IREE. Retrieved 2026-10-09, from https://iree.dev/guides/deployment-configurations/gpu-vulkan/
[^ireerepo]: iree-org. (n.d.). iree. Retrieved 2026-10-09, from https://github.com/iree-org/iree
[^triton]: PyTorch. (n.d.). torch.compile for Vulkan backend discussion. Retrieved 2026-10-09, from https://discuss.pytorch.org/t/torch-compile-for-vulkan-backend/177480
[^torchvulkan]: Sharief, Z. (n.d.). torchvulkan. Retrieved 2026-10-09, from https://github.com/ZainSharief/torchvulkan
[^torchvk]: Peterc3-dev. (n.d.). torch-vulkan. Retrieved 2026-10-09, from https://github.com/Peterc3-dev/torch-vulkan
[^kompute]: Kompute Project. (n.d.). Kompute. Retrieved 2026-10-09, from https://github.com/KomputeProject/kompute
[^slang]: Slang. (n.d.). Slang shader language. Retrieved 2026-10-09, from https://github.com/shader-slang/slang
[^burn]: Burn. (n.d.). Cross-platform GPU backend. Retrieved 2026-10-09, from https://burn.dev/blog/cross-platform-gpu-backend/
