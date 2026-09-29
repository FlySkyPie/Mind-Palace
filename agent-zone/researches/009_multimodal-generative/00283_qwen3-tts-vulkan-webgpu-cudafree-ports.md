# Qwen3-TTS 無 CUDA 加速後端調查：Vulkan / WebGPU

## 摘要

調查 Qwen3-TTS（原專案 [QwenLM/Qwen3-TTS](https://github.com/QwenLM/Qwen3-TTS)）是否有移植或分支版本可以不依賴 NVIDIA CUDA，並使用 Vulkan 或 WebGPU 進行 GPU 加速。結果顯示存在多種方案：**qwentts.cpp**（GGML/GGUF 移植，支援 Vulkan 後端）、**NVIDIA NVIGI**（專有 SDK，提供 Vulkan / D3D12 後端）、**LunaVox**（C++ 引擎，支援 Vulkan + DML）。但**WebGPU 後端尚不存在**。此外，ROCm（AMD）、OpenVINO（Intel）、MLX（Apple Silicon）等非 CUDA 方案均已驗證可行。

## Vulkan 後端

### qwentts.cpp（GGML/GGUF 移植）⭐ 推薦

[ServeurpersoCom/qwentts.cpp](https://github.com/ServeurpersoCom/qwentts.cpp) 是一個純 C++17 / GGML 的 Qwen3-TTS 12 Hz 移植，支援多種後端，包括 **Vulkan**。使用者可透過環境變數 `GGML_BACKEND=Vulkan0` 強制使用 Vulkan，適用於 AMD、Intel、NVIDIA 任一 GPU。專案提供 `buildvulkan.sh` / `buildvulkan.cmd` 建置腳本，並附帶三種 CLI 工具：`qwen-tts`（文字轉 WAV）、`qwen-codec`（WAV ↔ RVQ 編碼）、`tts-server`（OpenAI 相容 API）。模型權重以 GGUF 格式提供（F32、BF16、Q8_0、Q4_K_M），支援 0.6B 與 1.7B 參數版本，涵蓋 base、customvoice、voicedesign 模式。[^qwenttscpp]

### NVIDIA NVIGI（專有 SDK）

NVIDIA ACE In-Game Inference (NVIGI) Qwen3 TTS Plugin Pack 提供預編譯的 Qwen3-TTS 推理（Qwen 0.6B Base Talker，Q4_K_M 量化），支援三種後端：CUDA、Vulkan、D3D12。[^nvigi] 但此為 **Windows 10/11 專用**的專有 SDK，定位於遊戲內整合，非通用目的開源方案。

### LunaVox（Vulkan + DML）

[Lux-Luna/LunaVox](https://github.com/Lux-Luna/LunaVox) 使用自訂 llama.cpp 封裝進行 LLM 序列預測，搭配 ONNX Runtime 進行音訊解碼。支援 Vulkan + DML 後端，測試數據顯示 TTFB 194 ms、RTF 0.152，相較 PyTorch 基線加速 33.33 倍。[^lunavox]

## WebGPU 後端

**目前不存在**任何針對 Qwen3-TTS 的 WebGPU（瀏覽器端 GPU 加速）實作。搜尋結果中未發現 WebAssembly 移植、WebGPU 推理引擎或瀏覽器端運行的 Qwen3-TTS 方案。網路上雖有宣稱「如何在瀏覽器以 WebGPU 執行 Qwen3-TTS」的文章，但經查證為自動生成的低品質 / SEO 內容，未提供可運作的實作。[^webgpu]

## 其他無 CUDA 加速方案

| 方案 | 適用硬體 | 專案 | 狀態 |
|---|---|---|---|
| **ROCm** | AMD GPU（RX 7000、Strix Halo、Radeon 780M 等） | [BoredYama/Qwen3-TTS-ROCm](https://github.com/BoredYama/Qwen3-TTS-ROCm) | ✅ 可用 |
| **ROCm + Docker** | AMD GPU | [sizeak/qwen-tts-rocm](https://github.com/sizeak/qwen-tts-rocm) | ✅ 可用 |
| **ROCm + 一鍵安裝** | AMD GPU（Windows） | [trevortai/Qwen3-TTS-AMD](https://github.com/trevortai/Qwen3-TTS-AMD) | ✅ 可用 |
| **ROCm + OpenAI API** | AMD Strix Halo | [justinram11/qwen3-tts-rocm](https://github.com/justinram11/qwen3-tts-rocm) | ✅ 可用 |
| **OpenVINO** | Intel GPU / NPU / CPU | [wangtong10086/qwen3-tts-openvino](https://github.com/wangtong10086/qwen3-tts-openvino) | ✅ 可用 |
| **IPEX（XPU）** | Intel Arc GPU | [rvo-ar/qwen3-tts-intel-xpu](https://github.com/rvo-ar/qwen3-tts-intel-xpu) | ✅ 可用 |
| **MLX + Metal** | Apple Silicon（M1–M4） | [andreisuslov/qwen-tts](https://github.com/andreisuslov/qwen-tts)（PyPI） | ✅ 可用 |
| **GGML / Metal** | Apple Silicon | [andimarafioti/faster-qwen3-tts](https://github.com/andimarafioti/faster-qwen3-tts) | ✅ 實驗性 |
| **DirectML** | Windows GPU | — | ❌ 不存在 |

## 結論

若目標是**不使用 CUDA 且以 Vulkan 加速**，推薦使用 **qwentts.cpp**（GGML/GGUF 移植），其支援 Vulkan 後端且跨平台（Linux / Windows）。若僅需在 AMD GPU 上執行，ROCm 方案更直接。**WebGPU 加速目前無任何可用移植**。

[^qwenttscpp]: ServeurpersoCom. (n.d.). qwentts.cpp: C++17/GGML port of Qwen3-TTS 12 Hz. Retrieved 2026-09-26, from https://github.com/ServeurpersoCom/qwentts.cpp
[^nvigi]: NVIDIA. (n.d.). NVIDIA ACE In-Game Inference Qwen3 TTS Plugin Pack. Retrieved 2026-09-26, from https://docs.nvidia.com/ace-for-games/qwen-tts/1.0/getting-started.html
[^lunavox]: Lux-Luna. (n.d.). LunaVox: C++ inference engine for Qwen3-TTS. Retrieved 2026-09-26, from https://github.com/Lux-Luna/LunaVox
[^webgpu]: WebML Community. (n.d.). Qwen3 WebGPU Space (Qwen3 language model only). Retrieved 2026-09-26, from https://huggingface.co/spaces/webml-community/qwen3-webgpu