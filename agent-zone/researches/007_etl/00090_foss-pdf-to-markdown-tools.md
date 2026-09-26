# FOSS PDF 轉 Markdown 工具調查

## 摘要

本文調查市面上可用的自由開源（FOSS）工具，能將掃描型 PDF（含紙本掃描影像）轉換為 Markdown 格式。重點關注內建 OCR 支援、表格/數學公式處理能力、安裝方式及商業使用限制，協助使用者根據自身情境選擇最佳工具。

## 總覽

| 工具 | Stars | 授權 | 掃描 PDF OCR | 安裝方式 | 備註 |
|---|---|---|---|---|---|
| **Marker** | ~40k | Apache-2.0（程式碼）+ OpenRail-M（模型權重，商業有限制） | ✅ 內建（surya VLM） | `pip install marker-pdf` | 品質最高，需 GPU |
| **Docling** | ~67k | MIT | ✅ 可插拔 OCR（RapidOCR, EasyOCR, Tesseract, Nemotron, ocrmac） | `pip install docling` | 授權最寬容，OCR 引擎選擇最多 |
| **MinerU** | ~80k | AGPL-3.0 | ✅ 內建（PaddleOCR + VLM） | `uv pip install "mineru>=4.0"` | 最多 Star，四層解析，GPU 最佳 |
| **olmOCR** | ~20k | Apache-2.0 | ✅ VLM 原生全頁 OCR | `pip install olmocr[gpu]` | 需 NVIDIA GPU，專為大規模批次設計 |
| **Surya** | ~21k | Apache-2.0 + OpenRail-M | ✅（它就是 OCR 引擎） | `pip install surya-ocr` | Marker 底層引擎，可單獨使用 |
| **Pix2Text** | ~3k | MIT | ✅（CnOCR/EasyOCR，80+ 語言） | `pip install pix2text` | 開源 Mathpix 替代品，可跑 CPU |
| **Zerox** | ~12k | MIT | ⚠️ 需付費雲端 Vision API | `npm install zerox` / `pip install py-zerox` | 程式碼開源但 OCR 非免費 |
| **OCRmyPDF + Pandoc** | ~35k | MPL-2.0 | ✅（Tesseract，100+ 語言） | `apt install ocrmypdf` | 先加 OCR 層，再轉 Markdown |

## 工具詳述

### 1. Marker（datalab-to/marker）

由 VikParuchuri 發起，現由 datalab-to 維護的 PDF 轉 Markdown 工具，被認為是目前 FOSS 中轉換品質最高的方案之一。

- **掃描 PDF 支援**：✅ 內建 Surya VLM 自動 OCR。自動偵測亂碼或掃描頁面重新辨識。`--force_ocr` 強制 OCR 所有頁面；`--strip_existing_ocr` 重新 OCR 既有文字層。
- **功能特色**：支援 PDF、圖片、PPTX、DOCX、XLSX、HTML、EPUB 輸入；輸出 Markdown、JSON、HTML；保留表格、表單、方程式、行內數學、連結、引用、程式碼區塊；自動移除頁首頁尾。
- **多語言**：透過 surya 支援 90+ 語言。
- **安裝與使用**：`pip install marker-pdf`（需 Python 3.10+、PyTorch）。需 `vllm`（NVIDIA GPU + Docker）或 `llama.cpp`（CPU/Apple Silicon）作為推理後端。CLI：`marker_single file.pdf` 或 `marker folder/`。
- **限制**：模型權重使用修改版 **Open Rail-M 授權** — 研究/個人/年營收 <500 萬美金的新創免費，大型商業使用需付費授權。複雜巢狀表格可能失敗（可加 `--use_llm` 緩解）。需 Docker+GPU 才能發揮最佳表現。[^marker]

### 2. Docling（docling-project/docling）

由 IBM Research Zurich 發起，後捐贈至 LF AI & Data 基金會的文件解析工具。

- **掃描 PDF 支援**：✅ 可插拔 OCR 引擎 — RapidOCR（預設）、EasyOCR、Tesseract CLI、tesserocr、Nemotron-OCR、ocrmac、KServe。可依引擎原生語言代碼或 BCP-47 標籤指定語言。
- **功能特色**：支援 PDF、DOCX、PPTX、XLSX、HTML、EPUB、圖片、音訊、影片、電子郵件等格式；輸出 Markdown、HTML、JSON、DocTags；頁面佈局分析、閱讀順序、表格結構、程式碼、公式辨識；支援圖表理解。
- **生態整合**：提供 LangChain、LlamaIndex、Haystack 整合，內建 MCP Server 與 API Server。
- **安裝與使用**：`pip install docling`（Python 3.10+，macOS/Linux/Windows x86_64/arm64）。CLI：`docling <url-or-path>` → 輸出 `.md`。OCR 引擎以 extra 方式安裝，如 `pip install docling[rapidocr]`。
- **限制**：在 olmocr-bench 基準中得分（50.3%）低於 Marker（76.0%）和 MinerU。預設 RapidOCR 一次只能處理一種語言。[^docling]

### 3. MinerU（opendatalab/MinerU）

Star 數最高的開源 PDF 解析工具，專為 LLM 資料準備設計。

- **掃描 PDF 支援**：✅ 內建 PaddleOCR 管道，搭配 VLM 支援（llama.cpp、vLLM、LMDeploy）。「Basic」層級以上即含 OCR 與模型解析。
- **功能特色**：四層解析（Flash/Basic/Standard/Advanced）；支援 PDF、圖片、Office 文件、EPUB、OFD 等格式；輸出 Markdown、HTML、LaTeX、DOCX、EPUB、PDF；內建 Python SDK、CLI、Gradio WebUI、API。
- **安裝與使用**：`uv pip install -U "mineru>=4.0,<5"`，然後 `mineru-kit parse document.pdf -o document.md --tier standard`。CPU 原生可用，GPU 加速需 `mineru[full]`。
- **限制**：**AGPL-3.0 授權**（強 Copyleft，商業嵌入需注意）。吞吐量較低（約 0.54 頁/秒，Marker 為 2.9 頁/秒）。依賴較重。[^mineru]

### 4. olmOCR（allenai/olmOCR）

AI2（艾倫人工智慧研究所）開發的大規模 PDF 線性化工具，基於 7B Vision-Language Model。

- **掃描 PDF 支援**：✅ 本質上就是 OCR — 7B VLM（Qwen2.5-VL finetune）直接讀取頁面渲染影像，掃描 PDF 是原生使用場景。支援方程式、表格、手寫、多欄、插圖。
- **功能特色**：olmOCR-bench 基準測試套件（7000+ 測試／1400+ 文件）已成為業界標準；大規模批次成本極低（< $200/百萬頁）；支援 S3 叢集處理；v0.4.0 模型得分 82.4。
- **安裝與使用**：`pip install olmocr[gpu]`（需 Python 3.11 conda、NVIDIA GPU ≥12GB、poppler-utils、~30GB 磁碟空間）。CLI：`olmocr ./workspace --markdown --pdfs file.pdf`。
- **限制**：**必須有 GPU**（7B 模型，無實用 CPU 模式）。批次導向設計，單檔使用較重。**不輸出圖片**，僅文字/Markdown。[^olmocr]

### 5. Surya（datalab-to/surya）— OCR 引擎

Marker 底層的 VLM OCR 引擎，650M 參數，可單獨使用。

- **功能特色**：90+ 語言；83.3% olmOCR-bench（3B 參數以下最佳）；約 5 頁/秒（RTX 5090）；輸出含 `<math>`（KaTeX 兼容 LaTeX）和 `<table>` 的 HTML 區塊。
- **安裝**：`pip install surya-ocr`。同樣需 vllm（NVIDIA）或 llama.cpp（CPU/Apple）。模型授權同 Marker（OpenRail-M）。[^surya]

### 6. Pix2Text（breezedeus/Pix2Text）— 開源 Mathpix 替代品

明確定位為「Mathpix 的自由開源替代品」，不需 GPU 即可運行。

- **掃描 PDF 支援**：✅ 使用 CnOCR（中英文）和 EasyOCR（其他語言），80+ 語言。
- **功能特色**：版面分析、表格→Markdown/LaTeX/HTML、數學公式偵測與辨識（MFD/MFR 1.5）；可選閉源 VLM 支援（LiteLLM）；提供免費線上服務（10k 字/天）；macOS 桌面 App。
- **安裝**：`pip install pix2text` 或 `pip install pix2text[multilingual]`。
- **限制**：社群較小（~3.2k stars），高品質掃描/數學表現可能不及 VLM 等級工具。最適合在 CPU 機器上處理數學/圖片為主的文件。[^pix2text]

### 7. Zerox（getomni-ai/zerox）— FOSS 程式碼，付費 OCR

PDF → 圖片 → 送 Vision LLM → 彙總 Markdown。程式碼 MIT 授權。

- **掃描 PDF 支援**：✅ 但**需付費雲端 API 金鑰**（OpenAI GPT-4o、Claude、Gemini）。
- **限制**：無內建 OCR 模型，每頁成本取決於 API 定價。[^zerox]

### 8. 傳統 FOSS 方案：OCRmyPDF + Pandoc

先用 OCRmyPDF 為掃描 PDF 加上 Tesseract OCR 文字層（100+ 語言），再透過 `pdftotext` / `pandoc` 轉 Markdown。

- **安裝**：`apt install ocrmypdf` / `brew install ocrmypdf`。
- **限制**：兩階段處理，流程較繁瑣，且無版面分析（表格、多欄較難處理）。[^ocrmypdf]

## 選擇建議

| 使用情境 | 推薦工具 | 理由 |
|---|---|---|
| 最高品質、混合型 PDF（含掃描+數位原生） | **Marker** | olmOCR-bench 最高分，自動處理掃描頁面 |
| 寬鬆授權、商業嵌入 | **Docling** | MIT 授權，可插拔 OCR 引擎 |
| 最多社群支援、CPU 開箱即用 | **MinerU** | AGPL-3.0，四層解析彈性大 |
| 大規模批次掃描（GPU 叢集） | **olmOCR** | 專為大量文件設計，批次成本極低 |
| 數學/表格為主、無 GPU | **Pix2Text** | Mathpix 免費替代，CPU 可跑 |
| 已有 Tesseract 基礎架構 | **OCRmyPDF + Pandoc** | 傳統成熟方案 |

## 參考資料

[^marker]: datalab-to. (n.d.). *Marker: Convert PDF to markdown quickly and accurately*. Retrieved 2026-09-20, from https://github.com/datalab-to/marker
[^docling]: docling-project. (n.d.). *Docling: Docling: Efficient document understanding and processing*. Retrieved 2026-09-20, from https://github.com/docling-project/docling
[^mineru]: opendatalab. (n.d.). *MinerU: A one-stop, open-source, high-quality data extraction tool*. Retrieved 2026-09-20, from https://github.com/opendatalab/MinerU
[^olmocr]: allenai. (n.d.). *olmOCR: Toolkit for linearizing PDFs for LLM datasets/training*. Retrieved 2026-09-20, from https://github.com/allenai/olmocr
[^surya]: datalab-to. (n.d.). *Surya: OCR, layout analysis, reading order, table recognition in 90+ languages*. Retrieved 2026-09-20, from https://github.com/datalab-to/surya
[^pix2text]: breezedeus. (n.d.). *Pix2Text: A free and open-source Python alternative to Mathpix*. Retrieved 2026-09-20, from https://github.com/breezedeus/Pix2Text
[^zerox]: getomni-ai. (n.d.). *Zerox: Zero-shot PDF to Markdown with vision LLMs*. Retrieved 2026-09-20, from https://github.com/getomni-ai/zerox
[^ocrmypdf]: OCRmyPDF. (n.d.). *OCRmyPDF: Add an OCR text layer to scanned PDFs*. Retrieved 2026-09-20, from https://github.com/ocrmypdf/OCRmyPDF