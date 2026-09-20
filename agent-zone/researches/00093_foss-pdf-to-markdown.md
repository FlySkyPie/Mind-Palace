# 學術論文 PDF 轉 Markdown 之 FOSS 工具調查

## 概述

本報告調查學術論文 PDF 轉換為 Markdown 格式的開源軟體（FOSS）工具，涵蓋 AI/ML 型工具與傳統非機器學習方案，並針對學術論文常見需求進行比較，包括：數學方程式、引用文獻、表格、圖片、多欄佈局等。

---

## AI/ML 型工具

### 1. Marker（datalab-to/marker）— ⭐ ~40k stars

- **類型：** CLI + Python API + Streamlit GUI（`marker_gui`）+ API server
- **授權條款：** Apache 2.0（程式碼）；modified OpenRAIL-M（模型權重，個人學術研究及 $5M 以下新創免費）
- **支援輸出：** Markdown、JSON、HTML、chunks
- **特色：**
  - 處理雙欄佈局、方程式 → LaTeX（`$$` fence）、行內數學、表格（文字層重建，VLM 備援）、圖片與標題、圖片萃取、註腳上標、章節標題、頁首/頁尾移除、參考文獻
  - 支援 GPU/CPU/MPS；可選 `--use_llm` 混合模式（Gemini/Claude/OpenAI/Ollama）提高準確度
- **優勢：** olmocr-bench 得分 76.0%（數位原生文件 83.5%），超越 MinerU 和 docling；吞吐量約 2.9 pg/s（均衡模式），CPU 無 OCR 模式可達 23.7 pg/s；選擇性呼叫 VLM 以保持速度[^marker]
- **劣勢：** 模型權重有商業使用限制；需要 vLLM（GPU）或 llama.cpp（CPU）後端，設定較重；複雜巢狀表格/表單可能失敗

### 2. MinerU（opendatalab/MinerU）— ⭐ ~80k stars（最受歡迎）

- **類型：** CLI（`mineru-kit parse`）+ Python SDK + Gradio WebUI（`mineru-kit webui`）+ API server
- **授權條款：** MinerU Open Source License（基於 Apache 2.0 的附加條件授權）
- **支援輸出：** Markdown、HTML、LaTeX、DOCX、EPUB、PDF、Structured Content、Content List V1/V2
- **特色：**
  - 4 層解析模式（Flash/Basic/Standard/Advanced）
  - 支援多種輸入格式：PDF、掃描圖片、DOC/DOCX、PPTX、XLSX、RTF、ODF、EPUB、OFD、HTML、CSV
  - 佈局分析、OCR、公式辨識 → LaTeX、表格辨識、閱讀順序、圖片處理、穩定頁面/區塊引用定位（`doc:id/tier/page/block`）
- **優勢：** 複雜佈局與中文文件表現最佳；ONNX/llama.cpp 預配置開箱即用；文檔庫 + agent-read/search 工作流程；社群龐大[^mineru]
- **劣勢：** 非純 Apache 2.0 授權；比 marker 更重；VLM 模式需要 vLLM/LMDeploy 加速；olmocr-bench 得分 72.7% 略低於 marker

### 3. Docling（docling-project/docling，IBM）— ⭐ ~67k stars

- **類型：** CLI（`docling <url|file>`）+ Python API + API server（docling-serve）+ MCP server
- **授權條款：** MIT（程式碼），模型授權因模型而異
- **支援輸出：** Markdown、HTML、DocTags、lossless JSON
- **特色：**
  - 極廣泛的格式支援：PDF、DOCX、PPTX、XLSX、HTML、EPUB、LaTeX、圖片、音訊/影片（ASR）、Email、XBRL、JATS XML、USPTO 專利
  - 進階 PDF 理解（頁面佈局、閱讀順序、表格結構、程式碼、公式、圖片分類）
  - 可選 VLM 管道（GraniteDocling）；圖表理解功能
  - 整合 LangChain、LlamaIndex、CrewAI、Haystack
- **優勢：** 最廣泛的格式覆蓋；統一的 `DoclingDocument` 模型；強大的生態整合；LF AI & Data 託管專案[^docling]
- **劣勢：** 第三方比較中 olmocr-bench 得分 50.3%（預設管道）低於 marker/MinerU；中繼資料萃取（標題、作者、參考文獻）仍標示為「即將推出」

### 4. olmOCR（allenai/olmocr）— ⭐ ~20k stars

- **類型：** CLI batch pipeline（workspace 基礎）+ Docker + web demo
- **授權條款：** Apache 2.0
- **特色：**
  - 7B 參數 VLM 直接頁面圖片 → Markdown
  - 處理方程式、表格、手寫字、複雜排版；自動移除頁首/頁尾
  - 多欄佈局與嵌入式區塊的自然閱讀順序
  - FP8 量化支援；內建 benchmark harness（olmOCR-bench）
- **優勢：** 極高準確度（olmocr-bench v0.4.0 得分 82.4%）；專為百萬級語料庫轉換設計（< $200/M 頁）；支援遠端/OpenAI 相容推論商[^olmocr]
- **劣勢：** 需要 NVIDIA GPU（≥12 GB VRAM），無實用 CPU 模式；安裝重量級（~30GB Docker 映像）；單檔互動使用不如 marker/MinerU 方便

### 5. Nougat（facebookresearch/nougat）— ⭐ ~10k stars

- **類型：** CLI（`nougat file.pdf -o out/`）+ API + HuggingFace Space demo
- **授權條款：** MIT（程式碼）；**CC-BY-NC**（模型權重——禁止商業使用）
- **支援輸出：** `.mmd`（Mathpix-Markdown），含 LaTeX 數學與 LaTeX 表格
- **特色：** 專為學術論文設計（訓練於 arXiv/PMC）；內建錯誤偵測啟發式
- **優勢：** 對 arxiv 風格論文表現優異；學術 PDF→Markdown 研究的參考模型；CLI 簡潔[^nougat]
- **劣勢：** **模型權重非商業授權**；僅支援英文/拉丁語系（中文/俄文/日文失敗）；掃描/低品質頁面產生 `[MISSING_PAGE]`；2023 年技術相對老舊；無圖片/OCR 萃取

### 6. Surya（datalab-to/surya）— ⭐ ~21k stars

- **類型：** CLI（`surya_ocr`, `surya_layout`, `surya_table`, `surya_detect`）+ Python API + Streamlit GUI（`surya_gui`）
- **授權條款：** Apache 2.0（程式碼）；modified OpenRAIL-M（權重）
- **特色：** 650M 參數 VLM，執行 OCR、佈局分析（表格、圖片、標題、方程式、註腳、參考文獻、標題）、閱讀順序、表格辨識（行+列，輸出 HTML）；支援 90+ 語言；數學回傳為 KaTeX 相容的 LaTeX（`<math>` 標籤內）
- **優勢：** olmocr-bench 得分 83.3%（3B 參數以下最佳，帕雷托最優）；RTX 5090 上 5 pg/s；多語言表現佳（91 語言平均 87.2%）；模型小巧（0.65B）[^surya]
- **劣勢：** 元件模型——本身非完整文件→Markdown 轉換器（需搭配 Marker）；需要 vLLM/llama.cpp 推論伺服器；權重有商業限制

### 7. DeepSeek-OCR（deepseek-ai）— ⭐ ~24k stars

- **類型：** CLI/inference scripts（vLLM, transformers）；prompt 驅動
- **授權條款：** MIT
- **特色：** 視覺語言「Contexts Optical Compression」模型；文件→Markdown、圖片解析、OCR；原生解析度最高 1280×1280；A100-40G 上約 2500 tokens/s
- **優勢：** MIT 授權（寬鬆）；olmocr-bench 得分 75.7%；透過 vLLM 輕量推論[^deepseekocr]
- **劣勢：** 模型/權重導向，非完整即用管道；需要 NVIDIA GPU + CUDA 設定

### 8. Zerox（getomni-ai/zerox）— ⭐ ~12k stars

- **類型：** Python + Node SDKs；使用視覺 LLM（GPT-4o, Claude, Gemini, Azure, Bedrock）
- **授權條款：** MIT
- **特色：** PDF → 頁面圖片 → 視覺 LLM → 聚合 Markdown；`maintain_format` 支援跨頁表格連續性；並行處理、方向校正、schema 基礎資料萃取
- **優勢：** 極簡；MIT 授權；對特殊佈局/表格表現出色（使用 frontier VLM）；支援任何 LiteLLM 後端（含本地 Ollama）[^zerox]
- **劣勢：** **需要 API 金鑰/外部 LLM 成本**（非本地模型）；非確定性輸出；每頁比本地模型慢且貴

### 9. GOT-OCR2.0（Ucas-HaoranWei）— ⭐ ~8k stars

- **類型：** CLI demos + HF Space；社群 GUI
- **授權條款：** Apache 2.0（程式碼）；CC-BY-NC 4.0（資料）
- **特色：** 統一端到端 OCR-2.0 模型；純文字 OCR、格式化 OCR（含結構的 Markdown）、細粒度/框選 OCR、多裁切多頁 OCR
- **優勢：** 對學術/LaTeX 密集內容表現佳（基於 Vary + Qwen，中英文）；發布時 HF 熱門榜第一[^gotocr]
- **劣勢：** 研究導向程式碼（腳本而非打磨過的 CLI）；資料授權 CC-BY-NC；需要 GPU

---

## 傳統 / 非 ML 工具

### 10. Pandoc（jgm/pandoc）— ⭐ ~46k stars

- **類型：** CLI + library；WebAssembly 線上 demo（GUI）
- **授權條款：** GPL v2+
- **特色：** 讀取 **LaTeX、HTML、JATS XML、BibTeX/BibLaTeX/CSL-JSON** 並輸出 Markdown——當論文**原始碼**可用時（非原始 PDF），產生近乎完美的 Markdown，含原生 LaTeX 數學、引用文獻（透過 citeproc）、註腳、表格、章節結構
- **優勢：** 結構化原始碼轉換的黃金標準（如 arXiv .tex 或 JATS XML）；原生處理引用文獻/參考文獻；無限格式矩陣[^pandoc]
- **劣勢：** **無法直接讀取 PDF**——需搭配 PDF 文字萃取器（如 pdftotext），會失去佈局/方程式/表格

### 11. PyMuPDF / PyMuPDF4LLM（pymupdf）— ⭐ ~11k stars

- **類型：** Python library + CLI 單行 API（`to_markdown()`）；demo web app
- **授權條款：** AGPL v3
- **特色：** 結構感知的 PDF Markdown：多欄閱讀順序、表格偵測→GFM pipe tables 或 HTML tables、字體大小/TOC 標題偵測、粗體/斜體/程式碼、圖片與向量圖形萃取、頁首/頁尾移除、混合選擇性 OCR（Tesseract）、分塊輸出含中繼資料、LlamaIndex/LangChain 整合
- **優勢：** 無需 GPU/雲端，極快（MuPDF C 引擎）；比 VLM 方案便宜 10–250 倍；完全離線可用；每月下載量超過 5000 萬次[^pymupdf]
- **劣勢：** **AGPL**（Copyleft，商業嵌入需注意）；**無方程式/數學理解**（數學輸出為亂碼或圖片）；未建模標題/參考文獻語義；OCR 需 Tesseract

### 12. pdfplumber（jsvine）— ⭐ ~11k stars

- **類型：** Python library + CLI（CSV/JSON/text dumps）
- **授權條款：** MIT
- **特色：** 細緻的字元/線條/矩形/曲線/圖片物件資料；可設定的 **表格萃取**（線條/文字策略、視覺除錯）；表單值；版面保留文字
- **優勢：** MIT 授權；機器生成 PDF 的細粒度表格萃取最佳選擇；適合 building 自訂管道[^pdfplumber]
- **劣勢：** 非 Markdown 轉換器（需自行組裝 Markdown）；無 OCR、無數學、無語義結構（章節/圖片/引用文獻）

### 13. GROBID（grobidOrg/grobid）— ⭐ ~5k stars

- **類型：** Java REST web service（Docker）+ 多語言 clients（Python/Java/Node/Go）；batch CLI
- **授權條款：** Apache 2.0
- **特色：** ML（CRF + 可選深度學習）萃取學術 PDF 的**標題中繼資料（標題/作者/機構）、完整參考文獻列表（F1 ~87–90%）、引用上下文解析、全文結構**（段落、章節標題、圖表、註腳標記）→ TEI XML；DOI/PMID 透過 CrossRef 完善；PDF 座標輸出；用於 ResearchGate、Semantic Scholar、HAL、scite.ai、CERN 等生產環境
- **優勢：** 學術 PDF 引用文獻/參考文獻的參考工具——在「參考文獻」需求上無可比擬；可擴展（每節點 ~10 PDFs/s）；Apache 2.0[^grobid]
- **劣勢：** 輸出為 TEI/XML，**非 Markdown**（需透過 XSLT/pandoc 轉換）；不處理方程式/數學 OCR；需要 Java/JDK 21；圖片僅定位不描述

---

## 比較總結

| 工具 | 授權條款 | olmocr-bench | 方程式 | 參考文獻 | 表格 | GPU 需求 | GUI |
|------|---------|:-----------:|:-----:|:-------:|:---:|:-------:|:---:|
| **Marker** | Apache 2.0 + OpenRAIL-M | 76.0% | ✅ LaTeX | ✅ | ✅ | 選擇性 | ✅ |
| **MinerU** | Apache 2.0（附加條件） | 72.7% | ✅ LaTeX | ✅ | ✅ | 選擇性 | ✅ |
| **Docling** | MIT | 50.3% | ✅ | ❌（即將推出） | ✅ | 選擇性 | ❌ |
| **olmOCR** | Apache 2.0 | **82.4%** | ✅ | ❌ | ✅ | 必備 | ✅ |
| **Nougat** | MIT + CC-BY-NC | — | ✅ LaTeX | ❌ | ✅ LaTeX | 選擇性 | ✅ |
| **Surya** | Apache 2.0 + OpenRAIL-M | **83.3%** | ✅ KaTeX | ✅ | ✅ | 選擇性 | ✅ |
| **DeepSeek-OCR** | MIT | 75.7% | ✅ | ❌ | ✅ | 必備 | ❌ |
| **Pandoc** | GPL v2+ | — | ✅ 原生 | ✅ 原生 | ✅ 原生 | 無 | ✅ |
| **PyMuPDF** | AGPL v3 | — | ❌ | ❌ | ✅ GFM/HTML | 無 | ✅ |
| **GROBID** | Apache 2.0 | — | ❌ | ✅ F1~90% | ❌ | 無 | ❌ |
| **pdfplumber** | MIT | — | ❌ | ❌ | ✅ 最佳 | 無 | ❌ |

---

## 建議

### 最佳組合（學術論文）

1. **數位原生 PDF（arxiv 風格）：** Marker（均衡模式）或 MinerU（Standard 模式）——品質與速度平衡最佳
2. **需要最高準確度（有 GPU）：** olmOCR 或 Surya（搭配 Marker 管道）——olmocr-bench 得分最高
3. **需要參考文獻萃取：** GROBID → TEI/XML → Pandoc 轉 Markdown——引用文獻萃取無可取代
4. **無 GPU/離線環境：** PyMuPDF4LLM——極快但數學方程式會遺失
5. **原始 .tex/JATS 可用時：** Pandoc——直接轉換近乎完美
6. **中文文件為主：** MinerU——中文文件表現最佳[^mineru]

---

[^marker]: Datalab. (n.d.). Marker: Convert PDF to Markdown Quickly and Accurately. Retrieved 2026-09-20, from https://github.com/datalab-to/marker

[^mineru]: OpenDatalab. (n.d.). MinerU: A One-Stop Open-Source Data Extraction Tool. Retrieved 2026-09-20, from https://github.com/opendatalab/MinerU

[^docling]: Docling Project. (n.d.). Docling: Document Understanding and Conversion. Retrieved 2026-09-20, from https://github.com/docling-project/docling

[^olmocr]: AI2. (n.d.). olmOCR: Open Language Model OCR. Retrieved 2026-09-20, from https://github.com/allenai/olmocr

[^nougat]: Facebook Research. (n.d.). Nougat: Neural Optical Understanding for Academic Documents. Retrieved 2026-09-20, from https://github.com/facebookresearch/nougat

[^surya]: Datalab. (n.d.). Surya: Multilingual OCR, Layout Analysis, Table Recognition. Retrieved 2026-09-20, from https://github.com/datalab-to/surya

[^deepseekocr]: DeepSeek. (n.d.). DeepSeek-OCR. Retrieved 2026-09-20, from https://github.com/deepseek-ai/DeepSeek-OCR

[^zerox]: OmniAI. (n.d.). Zerox: OCR-as-a-Service Using Vision LLMs. Retrieved 2026-09-20, from https://github.com/getomni-ai/zerox

[^gotocr]: Ucas-HaoranWei. (n.d.). GOT-OCR2.0: OCR-2.0 Model. Retrieved 2026-09-20, from https://github.com/Ucas-HaoranWei/GOT-OCR2.0

[^pandoc]: Pandoc. (n.d.). Pandoc: A Universal Document Converter. Retrieved 2026-09-20, from https://github.com/jgm/pandoc

[^pymupdf]: PyMuPDF. (n.d.). PyMuPDF: Render PDF, OpenXPS, EPUB, MOBI for Python. Retrieved 2026-09-20, from https://github.com/pymupdf/PyMuPDF

[^pdfplumber]: Singer-Vine, J. (n.d.). pdfplumber: Plumb a PDF for Detailed Info About Text, Tables, and Images. Retrieved 2026-09-20, from https://github.com/jsvine/pdfplumber

[^grobid]: Grobid. (n.d.). GROBID: Machine Learning for Document Parsing. Retrieved 2026-09-20, from https://github.com/grobidOrg/grobid