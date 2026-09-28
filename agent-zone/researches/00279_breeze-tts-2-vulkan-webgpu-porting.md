# Breeze-TTS-2 非 CUDA 加速方案調查：Vulkan 與 WebGPU 支援現況

## 摘要

本報告調查 Breeze-TTS-2 模型能否在不依賴 NVIDIA CUDA 的情況下執行，並重點探討 Vulkan 與 WebGPU 作為加速後端的可行性。結果顯示：Vulkan 支援已透過第三方 C++/GGUF 移植專案實現，但 WebGPU 支援目前不存在。

## 1. Breeze-TTS-2 簡介

Breeze-TTS-2 是由 BreezeBlue 團隊開發的開源雙語（英文/中文）文字轉語音模型，於 2026 年 8 月 25 日發布，參數量約 3.5B，在 Artificial Analysis TTS 排行榜上居開放權重模型首位[^breeze-hf]。其架構包含四個階段：

- **Text Encoder**（基於 T5Gemma2）
- **Backbone**（基於 Qwen3）
- **Depth Decoder**（每音訊幀 15 個循序步驟）
- **Vocoder / Codec**（基於 Qwen3-TTS 音訊編解碼器）

官方實作使用 PyTorch，並深度依賴 CUDA Graphs 進行圖形編譯加速[^breeze-gh]。

## 2. 官方實作的 CUDA 依賴性

官方版本的執行環境明確要求「具備 CUDA 能力的 NVIDIA GPU」，程式碼中大量使用 `torch.cuda`，Docker 建置目標亦針對特定 NVIDIA 架構（sm90 給 H100，sm80 給 A100）[^breeze-gh]。因此官方版無法在純 CPU 或非 NVIDIA GPU 上執行。

## 3. Vulkan 加速支援

**已存在**。第三方專案 [HoppouAI/Breeze-TTS-2.cpp][^cpp-gh] 提供了完整的 C++/GGUF 移植，以 ggml 為底層，預設啟用 Vulkan 後端：

- 可在 NVIDIA、AMD、Intel GPU 上執行
- 當無 GPU 可用時自動回退至 CPU
- 可透過 `-DBREEZE_VULKAN=OFF` 編譯純 CPU 版本
- 提供量化權重（GGUF 格式），可在 RTX 3060 上以 Q8_0 達到約 1.2 倍即時率
- 支援語音設計、語音複製、語音方向控制、實驗性語音轉換與行內語音事件
- 附帶 CLI、串流 HTTP + WebSocket 伺服器、Web UI 以及 C API

需注意其 GGUF 權重**不與 llama.cpp 相容**，必須使用 Breeze-TTS-2.cpp 執行環境[^cpp-readme]。

## 4. WebGPU 加速支援

**不存在**。截至調查日期（2026-09-26），未發現任何基於 WebGPU 的 Breeze-TTS-2 實作。沒有任何瀏覽器端推論移植、ONNX Runtime WebGPU Execution Provider 設定，或 WebGPU 專屬分支的證據。

雖然 ONNX Runtime 通用性地支援 [WebGPU Execution Provider][^onnx-webgpu]，但尚無人將 Breeze-TTS-2 匯出為 ONNX 格式或為其建置 WebGPU 後端。

## 5. 其他非 CUDA 選項

| 方案 | 狀態 | 說明 |
|------|------|------|
| **ggml + Vulkan** | ✅ 可用 | HoppouAI/Breeze-TTS-2.cpp，支援 Vulkan 或 CPU 回退 |
| **MLX（Apple Silicon）** | ✅ 可用 | [mlx-community/Breeze-TTS-2-mlx][^mlx-hf] 與 [mlx-audio][^mlx-audio] 提供 bf16 / 8-bit / 4-bit 量化 |
| **llama.cpp** | ❌ 不相容 | 架構差異（多階段 TTS 管線 vs 標準 Transformer LLM）導致 GGUF 不相容 |
| **ONNX Runtime** | ❌ 不存在 | 無 ONNX 匯出或轉換腳本 |
| **DirectML** | ❌ 不存在 | 無 DirectML 執行提供者實作 |

## 6. 結論

若需在不使用 CUDA 的情況下執行 Breeze-TTS-2，最可行的方案是 **HoppouAI/Breeze-TTS-2.cpp**（使用 Vulkan 加速或純 CPU），或在 Apple Silicon 上使用 **MLX** 移植版。WebGPU 與 DirectML 支援目前尚不存在，需要社群從零開始建置。

[^breeze-hf]: BreezeBlue. (2026). *Breeze-TTS-2*. Hugging Face. Retrieved 2026-09-26, from https://huggingface.co/BreezeBlue/Breeze-TTS-2
[^breeze-gh]: BreezeBlue AI. (2026). *breeze-tts* (Apache 2.0). GitHub. Retrieved 2026-09-26, from https://github.com/breezeblue-ai/breeze-tts
[^cpp-gh]: HoppouAI. (2026). *Breeze-TTS-2.cpp* (GGUF/C++ reimplementation). GitHub. Retrieved 2026-09-26, from https://github.com/HoppouAI/Breeze-TTS-2.cpp
[^cpp-readme]: HoppouAI. (n.d.). Breeze-TTS-2.cpp README — GGUF note. Retrieved 2026-09-26, from https://huggingface.co/HoppouAI/Breeze-TTS-2.cpp/blob/main/README.md
[^onnx-webgpu]: ONNX Runtime. (n.d.). WebGPU Execution Provider. Retrieved 2026-09-26, from https://onnxruntime.ai/docs/execution-providers/WebGPU-ExecutionProvider.html
[^mlx-hf]: mlx-community. (2026). *Breeze-TTS-2-mlx*. Hugging Face. Retrieved 2026-09-26, from https://huggingface.co/mlx-community/Breeze-TTS-2-mlx
[^mlx-audio]: Blaizzy. (n.d.). mlx-audio — Breeze-TTS model docs. Retrieved 2026-09-26, from https://blaizzy.github.io/mlx-audio/models/tts/breeze-tts/