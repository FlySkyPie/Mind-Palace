# PyTorch 模型「去 CUDA」化與 Vulkan 等通用 GPU API 推論研究

## 摘要

本報告探討如何將依賴 NVIDIA CUDA 的 PyTorch 模型轉移至非 NVIDIA GPU 上執行推論，特別聚焦 Vulkan API 作為通用 GPU 後端的可行性。研究發現：Vulkan 後端在主流 PyTorch 中尚未成熟，真正可用的 Vulkan 實作位於 **ExecuTorch**（PyTorch 的行動端/邊緣裝置推論框架），且以 Android GPU 為主要目標。對於桌面端的非 NVIDIA GPU 推論，目前最務實的方案包括 **ROCm（AMD GPU on Linux）**、**MPS（Apple Silicon）**、**XPU（Intel GPU）** 與 **DirectML（Windows，但已進入維護模式）**。

---

## 1. 背景：何謂「去 CUDA」化

「去 CUDA」（de-CUDA）指的是將原本依賴 NVIDIA CUDA 執行 GPU 加速的 PyTorch 模型，轉移至其他硬體或運算後端執行。這在以下情境特別重要：

- 模型部署於非 NVIDIA GPU 的裝置（AMD、Intel、Apple Silicon、Qualcomm）
- 需要在沒有 GPU 的伺服器上執行 CUDA 檢查點（checkpoint）
- 希望透過通用 GPU API（如 Vulkan、DirectML）降低對特定廠商的依賴

PyTorch 透過統一的 `torch.device` 抽象層來管理不同運算裝置，核心 API 為 `.to(device)` 與 `.cpu()`。[^pytorch-device]

---

## 2. 核心去 CUDA 技術

### 2.1 裝置轉移基本模式

```python
import torch

# 將模型移至 CPU
model.to("cpu")        # 通用寫法
model.cpu()            # 簡寫

# 將張量移至 CPU
tensor = tensor.to("cpu")
tensor = tensor.cpu()
```

### 2.2 載入 CUDA 檢查點至非 CUDA 環境

```python
# map_location 參數是關鍵
checkpoint = torch.load("model_cuda.pth", map_location="cpu")
model.load_state_dict(checkpoint)
```

`map_location="cpu"` 會在載入過程中將所有 CUDA 張量動態映射至 CPU，這是處理 CUDA 檢查點最安全的方式。[^pytorch-serialization]

### 2.3 裝置無關（Device-Agnostic）的寫法

推薦的實務做法是在程式碼開頭偵測可用後端，並統一使用 `torch.device`：

```python
device = torch.device("cpu")

# 依偏好順序檢查可用後端
if torch.backends.mps.is_available():      # Apple Silicon
    device = torch.device("mps")
elif torch.cuda.is_available():
    # 此處涵蓋 NVIDIA CUDA 與 AMD ROCm/HIP
    if torch.version.hip:
        print("使用 AMD ROCm GPU")
    else:
        print("使用 NVIDIA CUDA GPU")
    device = torch.device("cuda")

model = MyModel().to(device)
data = data.to(device)
```

避免使用 `.cuda()` 這種 CUDA 專屬方法，改用 `.to(device)` 以維持裝置中立性。[^pytorch-best-practices]

---

## 3. PyTorch Vulkan 後端現狀

### 3.1 主線 PyTorch 中的 Vulkan

PyTorch 原始碼中確實包含 Vulkan 相關程式碼（位於 `aten/src/ATen/vulkan/`），但其功能極為有限，僅包含基礎的 `Context.cpp` 與 `Context.h`。[^pytorch-vulkan-src] **不存在**可供 Python 層使用的 `torch.device('vulkan')` 或 `torch.backends.vulkan` 模組。[^pytorch-backends]

已知問題[^pytorch-vulkan-issues]：
- Windows 上 `torch.is_vulkan_available()` 回傳 false（Issue #142331）
- 特定操作會導致 segfault（Issue #149941）
- 編譯失敗報告（Issue #156915）
- 桌面 Vulkan 支援的官方功能請求（Issue #160230）標示為「needs research」，無具體時程

**結論：主線 PyTorch 的 Vulkan 後端不可用於生產環境。**

### 3.2 ExecuTorch 中的 Vulkan 後端（ET-VK）

真正的 Vulkan 後端位於 **ExecuTorch**（PyTorch 的行動端/邊緣裝置推論框架）。ET-VK 是一個功能完整的 Vulkan 推論後端。[^et-vk-overview]

主要特性：
- 支援 FP32/FP16 精度
- 支援動態形狀（dynamic shapes）
- 支援量化線性層（quantized linear layers）
- 最低要求 Vulkan 1.1
- 目前以 **Android GPU** 為主要目標，桌面平台支援標示為「實驗性」

使用流程（不同於標準 PyTorch 的 `.to("vulkan")`）：

```python
from executorch.backends.vulkan.partitioner.vulkan_partitioner import VulkanPartitioner
from executorch.exir import to_edge_transform_and_lower

# 導出模型
exported_program = torch.export.export(model, sample_inputs)

# 降低至 Vulkan 後端
etvk_program = to_edge_transform_and_lower(
    exported_program,
    partitioner=[VulkanPartitioner()],
).to_executorch()

# 儲存為裝置端格式
with open("model_vulkan.pte", "wb") as f:
    etvk_program.write_to_file(f)
```

這是一個完全不同的工作流程，無法作為 `.to("cuda")` 的直接替代品。[^et-vk-readme]

---

## 4. Vulkan 以外的通用 GPU API 選項

### 4.1 各後端比較總表

| 後端 | 平台 | 支援 GPU | 狀態 | 使用方式 |
|------|------|----------|------|----------|
| **CPU** | 所有平台 | 任何 CPU | ✅ 正式支援 | `model.to("cpu")` |
| **MPS (Metal)** | macOS | Apple Silicon (M1+) | ✅ 穩定 | `torch.device("mps")`，內建於 PyTorch |
| **ROCm/HIP** | Linux | AMD GPU | ✅ 穩定 | `torch.device("cuda")`，與 CUDA API 相同 |
| **XPU** | Linux, Windows | Intel GPU (Arc, Iris Xe) | ✅ 支援 | `torch.device("xpu")`，Intel oneAPI/SYCL |
| **DirectML** | Windows 10/11 | 任何 DirectX 12 GPU | ⚠️ 維護模式 | `pip install torch-directml` |
| **Vulkan (ExecuTorch)** | 主要 Android | Vulkan 1.1+ GPU | 🟡 開發中 | ExecuTorch 框架限定 |
| **Vulkan (主線 PyTorch)** | - | - | ❌ 不可用 | 無 Python API |
| **OpenCL** | - | - | ❌ 不存在 | 無實作 |

### 4.2 各後端詳述

#### ROCm/HIP — AMD GPU on Linux 的最佳選擇

AMD 開發的開源 GPU 運算平台，安裝 PyTorch 的 ROCm 版本後，使用與 CUDA **完全相同的 API**（`torch.device("cuda")`），程式碼無需修改。可透過 `torch.version.hip` 確認是否執行於 ROCm。[^rocm-docs]

```python
# ROCm 使用與 CUDA 完全相同的 API！
device = torch.device("cuda")
model.to(device)
# 檢查是否為 ROCm
if torch.version.hip:
    print("Running on AMD ROCm")
```

#### MPS (Metal Performance Shaders) — Apple Silicon Mac

內建於 macOS 版 PyTorch，是 Apple Silicon 上最成熟的 GPU 加速方案。[^mps-docs]

```python
if torch.backends.mps.is_available():
    device = torch.device("mps")
```

#### XPU — Intel GPU

Intel 的 SYCL/oneAPI 後端，自 PyTorch 2.5 起穩定支援 Intel Arc 與資料中心 GPU。[^xpu-docs]

```python
if torch.xpu.is_available():
    device = torch.device("xpu")
```

#### DirectML — Windows 跨廠牌 GPU

Microsoft 提供的 DirectX 12 GPU 加速方案，可透過 `pip install torch-directml` 安裝。支援所有 DirectX 12 GPU（AMD、Intel、NVIDIA、Qualcomm），同時支援訓練與推論。[^directml-pypi]

**⚠️ 重要限制**：DirectML 已進入**維護模式**。Microsoft 官方表示不再規劃新功能，僅修復安全性問題。[^directml-repo] 對於 Windows 11 24H2+，Microsoft 建議改用 Windows ML（WinML）。

使用範例：
```python
import torch
import torch_directml

dml = torch_directml.device()
tensor = torch.tensor([1, 2, 3]).to(dml)
```

#### OpenCL — 不存在

PyTorch 中不存在真正的 OpenCL 後端。雖然裝置字串解析中可能出現 "opencl" 字樣，但這僅是歷史遺留的字串匹配，沒有任何 OpenCL 運算子實作或後端程式碼。[^pytorch-opencl-issue]

---

## 5. Vulkan vs CUDA 效能比較

截至研究日期（2026-09-25），**不存在** PyTorch 上 Vulkan 與 CUDA 效能對比的公開基準測試。原因在於 PyTorch 的 Vulkan 後端尚未成熟到足以進行桌面推論的公平對比。Llama.cpp 專案與一般 Vulkan Compute 基準測試顯示 Vulkan 可與廠商專屬 API 競爭，但這些結果無法直接推廣至 PyTorch。[^no-benchmarks]

---

## 6. 建議策略

### 依使用情境推薦

| 使用情境 | 建議方案 | 理由 |
|----------|----------|------|
| Linux + AMD GPU | **ROCm/HIP** | 最成熟，API 與 CUDA 完全相容，無需修改程式碼 |
| macOS + Apple Silicon | **MPS** | 內建支援，穩定可靠 |
| Windows + 任何 GPU | **DirectML** → 考慮移轉至 ONNX Runtime + DirectML | 雖為維護模式，仍是 Windows 上最直接方案 |
| Intel GPU | **XPU** | 官方支援，持續維護 |
| Android 裝置 | **ExecuTorch (ET-VK)** | 唯一可行的 Vulkan 方案 |
| 純 CPU 部署 | **CPU**（搭配 `map_location="cpu"`） | 最簡單，無相容性問題 |
| 跨平台跨廠牌需求 | **ONNX Runtime + 各平台 Execution Provider** | 抽象層最高，可搭配 DirectML/OpenVINO/CoreML 等 |

### 關鍵建議

1. **程式碼改寫為裝置無關（device-agnostic）**：使用 `torch.device()` 與 `.to(device)`，避免 `.cuda()` 與 `.cpu()` 硬編碼。
2. **載入檢查點務必使用 `map_location`**：這是解決 CUDA 檢查點相容性問題最簡單的方法。
3. **Vulkan 不是桌面 PyTorch 的可行選項**：如需 Vulkan 推論，請考慮 ExecuTorch（Android）或使用 Vulkan Compute 直接實作（非 PyTorch）。
4. **考慮替代路線**：PyTorch → ONNX → ONNX Runtime + 目標平台 Execution Provider 是跨平台部署的另一條成熟路徑。

---

## 參考文獻

[^pytorch-device]: PyTorch. (n.d.). `torch.device`. Retrieved 2026-09-25, from https://pytorch.org/docs/stable/tensor_attributes.html#torch.device

[^pytorch-serialization]: PyTorch. (n.d.). Serialization semantics. Retrieved 2026-09-25, from https://pytorch.org/docs/stable/notes/serialization.html

[^pytorch-best-practices]: PyTorch. (n.d.). CUDA semantics. Retrieved 2026-09-25, from https://pytorch.org/docs/stable/notes/cuda.html

[^pytorch-vulkan-src]: PyTorch. (n.d.). `aten/src/ATen/vulkan/`. Retrieved 2026-09-25, from https://github.com/pytorch/pytorch/tree/main/aten/src/ATen/vulkan

[^pytorch-backends]: PyTorch. (n.d.). `torch.backends`. Retrieved 2026-09-25, from https://pytorch.org/docs/stable/backends.html

[^pytorch-vulkan-issues]: PyTorch. (n.d.). Issues labeled "module: vulkan". Retrieved 2026-09-25, from https://github.com/pytorch/pytorch/issues?q=is%3Aissue+vulkan+label%3A%22module%3A+vulkan%22

[^et-vk-overview]: PyTorch. (n.d.). ExecuTorch Vulkan Backend Overview. Retrieved 2026-09-25, from https://github.com/pytorch/executorch/blob/main/docs/source/backends/vulkan/vulkan-overview.md

[^et-vk-readme]: PyTorch. (n.d.). ExecuTorch Vulkan Backend README. Retrieved 2026-09-25, from https://github.com/pytorch/executorch/blob/main/backends/vulkan/README.md

[^rocm-docs]: AMD. (n.d.). ROCm Documentation. Retrieved 2026-09-25, from https://rocm.docs.amd.com/

[^mps-docs]: PyTorch. (n.d.). MPS Backend. Retrieved 2026-09-25, from https://pytorch.org/docs/stable/notes/mps.html

[^xpu-docs]: PyTorch. (n.d.). XPU Backend. Retrieved 2026-09-25, from https://pytorch.org/docs/stable/notes/get_start_xpu.html

[^directml-pypi]: Microsoft. (n.d.). torch-directml. Retrieved 2026-09-25, from https://pypi.org/project/torch-directml/

[^directml-repo]: Microsoft. (n.d.). DirectML. Retrieved 2026-09-25, from https://github.com/microsoft/DirectML

[^pytorch-opencl-issue]: PyTorch. (n.d.). Issue #159745. Retrieved 2026-09-25, from https://github.com/pytorch/pytorch/issues/159745

[^no-benchmarks]: 截至研究日期，未發現 PyTorch Vulkan 與 CUDA 的公開效能對比基準資料。