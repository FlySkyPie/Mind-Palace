# BlueMagpie-TTS 的 ONNX / GGUF 版本調查

## 摘要

針對開源繁體中文（台灣華語）語音合成模型 **BlueMagpie-TTS**（擁有者為 OpenFormosa，Hugging Face 帳號 `OpenFormosa/BlueMagpie-TTS`），本報告調查其在 ONNX 與 GGUF 兩種推論格式上的社群轉換情形。

結論如下：

- **GGUF：存在**。社群使用者 `BricksDisplay`（亦以 `hans00` 名義活動）已發布轉換版本 `BricksDisplay/BlueMagpie-TTS-GGUF`，並由 `mybigday` 的 `codec.cpp` 與 `llama.rn` 兩個 C/C++ 推論引擎支援。
- **ONNX：不存在**。截至 2026-10-23，Hugging Face Hub 與 GitHub 上均查無 BlueMagpie-TTS 的 ONNX 轉換版本。

## 原始模型背景

BlueMagpie-TTS 是一個台灣華語文字轉語音（TTS）模型，支援中文與中英夾雜（code-switching）的語音合成，輸出取樣率為 48 kHz[^orig-hf]。原始模型倉庫採 `license: other` 授權，聲明僅供研究與評估使用，且要求不得在未釐清權利與同意的情況下再散布權重或生成語音[^orig-hf]。

原始發布之檔案格式為 PyTorch 權重（`pytorch_model.bin`、`audiovae.pth`）與 `tokenizer.json`、`config.json` 等設定檔，並未包含 ONNX 或 GGUF 檔案[^orig-files]。對應的 GitHub 程式碼倉庫 `OpenFormosa/BlueMagpie-TTS` 為 Python 專案，約 69 顆星[^orig-gh]。

## GGUF 版本（存在）

### 倉庫位置與檔案

社群使用者 BricksDisplay 於 Hugging Face 發布倉庫 [`BricksDisplay/BlueMagpie-TTS-GGUF`](https://huggingface.co/BricksDisplay/BlueMagpie-TTS-GGUF)（建立於 2026-07-02，約 3 個讚、當月 578 次下載）[^gguf-hf][^gguf-api]。其設計採「兩個檔案構成一個模型」的形式，類似 llama.cpp 的多模態投影器（projector）架構[^gguf-hf]：

- **backbone 檔案**：文字到語音的語言模型主幹，由（已修補的）llama.cpp 執行
- **codec 檔案**：音訊解碼器／聲碼器（vocoder）、說話者編碼器，以及兩者之間的轉接層，由 codec.cpp 執行

倉庫內含以下檔案[^gguf-api]：

| 檔案 | 大小 | 說明 |
| --- | --- | --- |
| `BlueMagpie-AudioVAE-q8_0.gguf` | 約 1.09 GB | codec 檔案（每個組合都需搭配一個 backbone） |
| `BlueMagpie-Barbet-1B-f16.gguf` | 約 2.18 GB | backbone，FP16，品質最佳 |
| `BlueMagpie-Barbet-1B-q8_0.gguf` | 約 1.16 GB | backbone，Q8_0，官方推薦 |
| `BlueMagpie-Barbet-1B-q6_k.gguf` | 約 899 MB | backbone，Q6_K |
| `BlueMagpie-Barbet-1B-q5_k_m.gguf` | 約 793 MB | backbone，Q5_K_M |
| `BlueMagpie-Barbet-1B-q4_k_m.gguf` | 約 693 MB | backbone，Q4_K_M，體積最小 |

推薦組合（backbone + codec）為 `q8_0` 版，合計約 2.3 GB；最小組合約 1.8 GB；最佳品質組合約 3.3 GB[^gguf-hf]。

### 架構與相容性注意事項

此 backbone 採用 **Barbet** 架構（Mamba2 加上注意力機制），上游原版 llama.cpp 尚未支援。因此使用者**必須使用**由 codec.cpp 或 llama.rn 一併建置的（已套用修補的）llama.cpp，**不能使用原版 llama.cpp**[^gguf-hf]。這點對自行整合者非常重要，否則模型無法載入。

### 支援的執行環境

1. **codec.cpp（C/C++ 命令列）**：由組織 `mybigday` 維護，MIT 授權，約 11 顆星，最近更新於 2026-10-23[^codeccpp]。提供 `tts-cli` 工具，支援 CPU 與 GPU（CUDA、Vulkan、Metal）[^gguf-hf]。
2. **codec.cpp 的 Python 套件 `codec_cpp`**：`pip install` 後可用 `codec_cpp.TTS` 類別執行同一套流程，會自行建置 bundled 的 llama.cpp，因此需要 CMake 與 C++ 編譯器[^gguf-hf][^codeccpp]。
3. **llama.rn（React Native）**：同一組織的 llama.cpp React Native 綁定，MIT 授權，約 1,061 顆星，可在 iOS 與 Android 載入同一組 GGUF 檔案[^llamarn][^gguf-hf]。

換言之，GGUF 版已可在桌上型（CLI／Python）與行動裝置（React Native）離線執行。

## ONNX 版本（不存在）

經多方查證，截至 2026-10-23 並無 BlueMagpie-TTS 的 ONNX 版本：

- Hugging Face Hub 以 `BlueMagpie` 搜尋僅得兩個模型：原始模型 `OpenFormosa/BlueMagpie-TTS` 與 GGUF 版 `BricksDisplay/BlueMagpie-TTS-GGUF`，皆無 ONNX 檔案[^gguf-api][^orig-files]。
- Hugging Face Hub 以 `bluemagpie` 加上 `onnx` 篩選，回傳空結果[^hf-onnx-search]。
- 原始模型倉庫檔案清單僅含 PyTorch 權重與設定檔，無 `.onnx`[^orig-files]。
- GitHub 以 `BlueMagpie` 搜尋倉庫，結果中與本模型相關者為原始專案、ComfyUI 節點、API 服務、網頁 demo 等，並無任何 ONNX 匯出專案[^gh-search]。

值得注意的是，同一發布者 BricksDisplay 曾為其他 TTS 模型（例如 `chatterbox-multilingual-ONNX-q4`、`ellie-Bert-VITS2-small-onnx`）製作 ONNX 版本，但目前尚未針對 BlueMagpie-TTS 提供 ONNX 轉換[^gguf-hf]。這顯示技術上並非不可能，而是尚無人完成。

## 對照總表

| 格式 | 是否存在 | 倉庫／來源 | 維護者 | 執行引擎 |
| --- | --- | --- | --- | --- |
| GGUF | 是 | `BricksDisplay/BlueMagpie-TTS-GGUF` | BricksDisplay（hans00） | codec.cpp、llama.rn |
| ONNX | 否 | 無 | 無 | 無 |

## 給使用者的實務建議

- 若目標是**離線、跨裝置、低資源推論**，GGUF 版是當前唯一選擇，推薦 `BlueMagpie-Barbet-1B-q8_0.gguf` 搭配 `BlueMagpie-AudioVAE-q8_0.gguf`[^gguf-hf]。
- 由於 backbone 需要 Barbet（Mamba2 + attention）修補，**切勿**以原版 llama.cpp 載入，否則會失敗；請從 codec.cpp 或 llama.rn 取得對應的建置[^gguf-hf]。
- 若必須使用 ONNX（例如部署到 ONNX Runtime、TensorRT 或特定嵌入式推論框架），目前**沒有現成檔案**，需自行由 PyTorch 權重匯出，或向模型作者與社群提出需求。
- 任何再利用皆須遵守原始模型 `license: other` 的研究與評估限制[^orig-hf][^gguf-hf]。

## 參考文獻

[^orig-hf]: OpenFormosa. (n.d.). *OpenFormosa/BlueMagpie-TTS* [Model card]. Hugging Face. Retrieved 2026-10-23, from https://huggingface.co/OpenFormosa/BlueMagpie-TTS

[^orig-files]: Hugging Face. (n.d.). *Files and versions — OpenFormosa/BlueMagpie-TTS* [Repository file listing]. Retrieved 2026-10-23, from https://huggingface.co/OpenFormosa/BlueMagpie-TTS/tree/main

[^orig-gh]: OpenFormosa. (n.d.). *BlueMagpie-TTS* [GitHub repository]. GitHub. Retrieved 2026-10-23, from https://github.com/OpenFormosa/BlueMagpie-TTS

[^gguf-hf]: BricksDisplay. (n.d.). *BricksDisplay/BlueMagpie-TTS-GGUF* [Model card]. Hugging Face. Retrieved 2026-10-23, from https://huggingface.co/BricksDisplay/BlueMagpie-TTS-GGUF

[^gguf-api]: Hugging Face. (n.d.). *Models API search results for "BlueMagpie"* [Application programming interface]. Retrieved 2026-10-23, from https://huggingface.co/api/models?search=BlueMagpie&full=true

[^codeccpp]: mybigday. (n.d.). *codec.cpp: Neural audio codec inference in C/C++* [GitHub repository]. GitHub. Retrieved 2026-10-23, from https://github.com/mybigday/codec.cpp

[^llamarn]: mybigday. (n.d.). *llama.rn: React Native binding of llama.cpp* [GitHub repository]. GitHub. Retrieved 2026-10-23, from https://github.com/mybigday/llama.rn

[^hf-onnx-search]: Hugging Face. (n.d.). *Models API search results for "bluemagpie" filtered by "onnx"* [Application programming interface]. Retrieved 2026-10-23, from https://huggingface.co/api/models?search=bluemagpie&filter=onnx

[^gh-search]: GitHub. (n.d.). *Repository search results for "BlueMagpie"* [Application programming interface]. Retrieved 2026-10-23, from https://api.github.com/search/repositories?q=BlueMagpie
