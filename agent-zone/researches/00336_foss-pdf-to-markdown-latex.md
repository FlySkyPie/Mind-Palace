# FOSS 學術論文 PDF 轉換工具研究 — Markdown / LaTeX

學術論文 PDF 轉換爲 Markdown 或 LaTeX 原始碼是許多研究者的常見需求。本報告系統性整理了目前已知的開放原始碼（FOSS）解決方案，並依其輸出格式、技術路線、適用場景進行分類評估。

## 工具一覽表

| 工具 | 類型 | 輸出格式 | ⭐ GitHub | 數學公式 | 表格 | 多欄 | 掃描 PDF | 需 GPU？ | 授權 |
|---|---|---|---|---|---|---|---|---|---|
| **Marker** | CLI/Python 函式庫 | Markdown, JSON, HTML | 40.2k | ✅ 佳 | ✅ 佳 | ✅ 佳 | ✅（OCR） | 建議 GPU | Apache 2.0 |
| **Nougat** (Meta) | CLI/Python 函式庫 | Mathpix Markdown (`.mmd`) | 10.1k | ✅✅ 極佳 | ✅ 佳 | ✅ 佳 | ✅（OCR-native） | 建議 GPU | MIT |
| **GROBID** | Server/API | TEI XML | 5.2k | ⚠️ 部分 | ⚠️ 部分 | ✅ 佳 | ✅ | 否 | Apache 2.0 |
| **Pix2Text** | Python 函式庫 | Markdown (含 LaTeX) | 3.3k | ✅✅ 極佳 | ✅✅ 極佳 | ✅ 佳 | ✅（OCR-native） | 建議 GPU | Apache 2.0 |
| **MinerU** | CLI/Python 函式庫 | Markdown, JSON | 81k | ✅ 佳 | ✅✅ 極佳 | ✅✅ 極佳 | ✅（OCR-native） | 建議 GPU | Apache 2.0 |
| **Docling** (IBM) | CLI/Python 函式庫 | Markdown, JSON | 68.3k | ⚠️ 基礎 | ✅ 佳 | ✅ 佳 | ✅（OCR） | 可選 | Apache 2.0 |
| **Pandoc** | CLI | Markdown, LaTeX, HTML 等 | 46.5k | ❌ 差 | ❌ 差 | ❌ 差 | ❌ 無法 | 否 | GPL-2.0 |
| **PyMuPDF4LLM** | Python 函式庫 | Markdown | 2.2k | ❌ 無 | ⚠️ 基礎 | ⚠️ 基礎 | ❌ 無法 | 否 | AGPL-3.0 |
| **LaTeXML** | CLI/Perl 函式庫 | XML, HTML, MathML, ePub | 1.3k | ✅✅ 極佳 | ✅ 佳 | ✅ 佳 | N/A（LaTeX→XML） | 否 | GPL |
| **TeX4ht** | CLI | HTML, XML, MathML, ODF | — | ✅✅ 極佳 | ✅ 佳 | ✅ 佳 | N/A（LaTeX→HTML） | 否 | GPL |
| **pdf2latex** (emsquid) | CLI (Rust) | LaTeX (`.tex`) | 19 | ⚠️ 基礎 | ⚠️ 基礎 | ⚠️ 基礎 | ⚠️ 基礎 | 否 | 未指定 |
| **pdf2tex** (p3nGu1nZz) | CLI/Python | LaTeX | 6 | ✅ 佳 | ⚠️ 部分 | ⚠️ 基礎 | ✅（OCR） | 建議 GPU | 未指定 |
| **pdf-craft** | CLI | Markdown, EPUB | 6.3k | ✅ 佳 | ✅ 佳 | ✅ 佳 | ✅✅ 極佳 | 建議 GPU | Apache 2.0 |
| **pix2tex/LaTeX-OCR** | Python 函式庫/GUI | LaTeX（公式限定） | 16.6k | ✅✅ 極佳 | N/A | N/A | N/A（圖片→LaTeX） | 建議 GPU | MIT |

## 輸出 Markdown 的工具

### Marker ⭐ 40.2k

Marker 是目前最受關注的 PDF→Markdown 轉換工具之一，採用 Surya OCR 搭配版面檢測（Layout Detection）與表格重建的 pipeline 架構[^marker]。

- **安裝與使用**: `pip install marker-pdf`，指令 `marker_single paper.pdf output/`
- **數學公式**: ✅ 佳。平衡模式（balanced mode）會自動將行內公式轉為 LaTeX。可搭配 `--use_llm` 標記獲得最複雜方程式的較佳結果。在 olmocr-bench 的 arXiv math 類別中達到 83.9%（平衡模式）[^marker-bench]。
- **表格**: ✅ 佳。多欄表格透過文字層搭配 CPU 啟發式演算法重建；信心度低的表格會回落到 VLM。
- **多欄**: ✅ 佳。版面檢測（rf-detector 或 VLM）決定閱讀順序，在 olmocr-bench 多欄類別得分 76.6%。
- **圖形**: ✅ 會提取圖片並以相對路徑保留。
- **已知限制**: 極複雜的巢狀表格可能失敗；最佳表現需要 GPU；公式精確度在超複雜方程式上仍不如專用 API 方案。
- **適用場景**: 大量批次轉換、離線本地處理。該分類中的「瑞士刀」級工具。

### Nougat ⭐ 10.1k (Meta / Facebook Research)

Nougat 是一個視覺 Transformer 模型，專為學術 PDF 的 OCR 設計，輸出 **Mathpix Markdown (.mmd)** 格式——使用 LaTeX 來表示數學和表格的輕量級標記語言[^nougat]。

- **安裝與使用**: `pip install nougat-ocr`，指令 `nougat paper.pdf -o output/`
- **數學公式**: ✅✅ **極佳**——這是 Nougat 的核心優勢。採用 Donut 架構（vision encoder-decoder）將 PDF 頁面作為圖像讀取，直接生成 LaTeX 數學表達式。
- **表格**: ✅ 佳。使用 LaTeX table 語法輸出。
- **多欄**: ✅ 佳。視覺 Transformer 方法自然地處理多欄排版。
- **已知限制**: 速度明顯慢於文字提取工具；實際使用需要 GPU；記憶體消耗大；輸出格式 (`.mmd`) 為較小眾的 Mathpix 相容 Markdown 方言；模型主要用英文 arXiv 論文訓練，其他領域表現可能較差。
- **適用場景**: 數學密集的學術論文，公式忠實度為首要考量。

### Pix2Text ⭐ 3.3k (P2T)

Pix2Text 是一個開源的 Python 工具，旨在成爲 **Mathpix 的免費替代方案**。它能辨識版面、表格、數學公式（LaTeX）和文字，全部轉換為 Markdown[^pix2text]。

- **安裝與使用**: `pip install pix2text`
- **數學公式**: ✅✅ **極佳**。專用 MFR（Math Formula Recognition）模型將公式轉為 LaTeX。支援 80+ 種語言。
- **表格**: ✅✅ **極佳**。專用表格辨識模型能處理合併儲存格、複雜表頭、巢狀表格，轉換為標準 Markdown 表格。
- **多欄**: ✅ 佳。版面分析模型處理複雜排版。
- **適用場景**: 需要類似 Mathpix 體驗且可離線的使用者；包含混合文字、公式與表格的圖像。

### MinerU ⭐ 81k

MinerU 由上海 AI Lab（OpenDataLab）開發，採用多模型融合（版面檢測 + 公式辨識 + 表格重建）的 high-fidelity 文件提取工具[^mineru]。

- **安裝與使用**: `pip install mineru`，指令 `mineru -p paper.pdf -o ./output`
- **數學公式**: ✅ 佳。公式辨識整合於 pipeline 中，在高度特殊化的方程式上品質可能不一致。
- **表格**: ✅✅ **極佳**——最突出的功能。必要時在 Markdown 中嵌入 HTML 表格。能處理非常複雜的表格結構。
- **多欄**: ✅✅ **極佳**——恢復多欄文件閱讀順序的最佳工具之一。
- **中日韓（CJK）**: ✅✅ **同類最佳**，對 CJK 學術內容的支援無可比擬。
- **適用場景**: CJK 文件、版面複雜的學術論文、大規模資料集建立。

### Docling ⭐ 68.3k (IBM Research)

Docling 是一個企業級文件理解工具，能將 PDF、DOCX、PPTX、HTML 和圖像統一解析為 Markdown/JSON 輸出[^docling]。

- **安裝與使用**: `pip install docling`，指令 `docling paper.pdf`
- **數學公式**: ⚠️ **基礎**——公式處理不如 Marker 或 MinerU 成熟。
- **表格**: ✅ 佳。表格提取能力優秀，結構保持良好。
- **多欄**: ✅ 佳。版面分析能偵測欄位並決定閱讀順序。
- **適用場景**: 企業工作流程、RAG pipeline（LlamaIndex/LangChain 整合）、多格式文件處理。

### PyMuPDF4LLM ⭐ 2.2k

PyMuPDF4LLM 是 PyMuPDF 團隊開發的輕量級 Python 函式庫，專爲 LLM 友善的 Markdown 提取而設計。純文字提取——無 ML 模型、無 GPU[^pymupdf4llm]。

- **安裝與使用**: `pip install pymupdf4llm`，Python 中呼叫 `pymupdf4llm.to_markdown("paper.pdf")`
- **數學公式**: ❌ **無**——公式會變成亂碼文字。
- **表格**: ⚠️ 對結構良好的 PDF 有基本表格偵測能力。
- **多欄**: ❌ 無法處理——閱讀順序會錯亂。
- **掃描 PDF**: ❌ **完全無效**——無 OCR 能力。
- **適用場景**: **只能處理原生（數位原生）簡單 PDF**；無 GPU 環境下的快速提取。

### pdf-craft ⭐ 6.3k

pdf-craft 專爲將**掃描書本**轉換為 Markdown 或 EPUB 而設計，使用 DeepSeek OCR，完全離線運行[^pdfcraft]。

- **數學公式**: ✅ 佳。DeepSeek OCR 對公式處理良好。
- **表格**: ✅ 佳。表格結構保留。
- **掃描 PDF**: ✅✅ **爲此而生**。
- **適用場景**: 實體書本數位化、圖書館/檔案專案、完全離線轉換。

### Pandoc ⭐ 46.5k

Pandoc 是通用文件轉換器，但在學術 PDF→Markdown 轉換上**經常被誤解**[^pandoc]。

- **數學公式**: ❌ **差**。Pandoc 不做 OCR——它直接讀取 PDF 文字流，因此數學符號會變成亂碼。
- **表格**: ❌ 無法從 PDF 中辨識表格結構。
- **多欄**: ❌ 閱讀順序錯亂。
- **掃描 PDF**: ❌ **完全無法處理**。
- **根本原因**: Pandoc 底層使用 `pdftotext`，僅提取原始文字，完全不具版面/數學感知能力。
- **適用場景**: **學術 PDF 轉換請不要使用 Pandoc**。它在 DOCX→Markdown、LaTeX→Markdown 或 Markdown→PDF 方向表現優秀。

## 輸出 LaTeX 原始碼的工具

### LaTeXML ⭐ 1.3k

LaTeXML 將 **LaTeX 原始碼** 轉換為 XML/HTML/MathML/ePub。注意：**輸入是 `.tex` 原始碼，不是 PDF**[^latexmld]。

- **安裝**: `cpan install LaTeXML` 或套件管理器，指令 `latexml paper.tex > paper.xml`
- **數學公式**: ✅✅ **極佳**。保留 LaTeX 數學語義，可輸出 Presentation + Content MathML。
- **表格**: ✅ 佳。
- **多欄**: ✅ 佳（透過 LaTeX 套件）。
- **已知限制**: **不接受 PDF 作為輸入**。部分 LaTeX 套件未完整支援。
- **適用場景**: 將現有 LaTeX 原始碼轉換爲網頁格式，保留數學語義。

### TeX4ht (make4ht) — TeX Live 內建

TeX4ht 將 TeX/LaTeX 文件轉換為 HTML、XML、MathML、OpenDocument 等格式。它**實際運行 TeX** 來處理文件（不同於 LaTeXML 的解析方式）[^tex4ht]。

- **安裝**: 包含在 TeX Live/MikTeX 中，指令 `make4ht paper.tex`
- **數學公式**: ✅✅ **極佳**。透過 MathML 完整支援。
- **表格**: ✅ 佳。支援多數 LaTeX 表格套件。
- **已知限制**: **不接受 PDF 作為輸入**。自訂套件的設定可能複雜。
- **適用場景**: 將含大量自訂套件的 LaTeX 原始碼轉換為 HTML。

### GROBID ⭐ 5.2k

GROBID 是一個機器學習函式庫，能從 PDF 中提取結構化 TEI XML——包含標題、作者、所屬單位、參考文獻/引用、全文段落層級結構、圖表與公式[^grobid]。

- **安裝與使用**: Docker/Java 伺服器，REST API 介面
- **輸出格式**: TEI XML（需後處理轉為 LaTeX 或 Markdown）
- **數學公式**: ⚠️ **部分**。GROBID 提取公式位置，但**不做完整的數學內容 OCR**。
- **表格**: ⚠️ 部分。提取表格結構和座標。
- **多欄**: ✅ 佳。理解閱讀順序與章節層級。
- **引用/參考文獻**: ✅✅ **同類最佳**——解析書目參考文獻、引用上下文、DOI 提取。
- **適用場景**: **元數據提取**（作者、參考文獻、引用、章節）——而非完整視覺重建。

### pdf2latex ⭐ 19 (emsquid — Rust)

一個 Rust 寫成的 CLI 工具，旨在將 PDF 轉回 LaTeX 原始碼[^pdf2latex-emsquid]。

- **使用**: 需 Rust toolchain 從原始碼編譯，指令 `pdf2latex paper.pdf -o paper.tex`
- **數學公式**: ⚠️ **基礎**。依賴基於字型的文字提取與字元辨識。
- **適用場景**: 排版簡單的 PDF。目前尚不適合複雜學術論文。

### pdf2tex ⭐ 6 (p3nGu1nZz — Python)

基於 RAG（Retrieval Augmented Generation）的 PDF→LaTeX 轉換模組，專為大型文件設計，使用 PyMuPDF、Nougat 和 PaddleOCR 的多路徑處理[^pdf2tex-pypi]。

- **安裝**: `pip install pdf2tex`
- **數學公式**: ✅ **佳**——聲稱使用神經網路方程式辨識達到 95%+ 準確率。
- **掃描 PDF**: ✅ 透過 PaddleOCR 處理。
- **大型文件**: ✅ 專為 2000+ 頁文件設計。
- **適用場景**: 含大量數學內容的大型學術文件。

### pix2tex / LaTeX-OCR ⭐ 16.6k (lukas-blecher)

一個基於學習的系統，接受**數學公式的圖像**，回傳對應的 **LaTeX 程式碼**。並非全文件轉換工具——專門用於公式 OCR[^pix2tex-latex-ocr]。

- **安裝**: `pip install "pix2tex[gui]"`，提供 GUI 和 CLI 介面
- **數學公式**: ✅✅ **極佳**——使用 ViT（Vision Transformer）encoder-decoder 架構。
- **表格/圖形/程式碼**: **不適用**——公式限定。
- **適用場景**: 從 PDF 螢幕截圖或裁剪區域中提取特定方程式。

## 使用情境建議

| 使用情境 | 最佳工具 |
|---|---|
| **批量轉換 arXiv 論文為 Markdown** | **Marker** ——平衡模式，準確率高，支援批次處理 |
| **數學密集論文，公式忠實度最優先** | **Nougat**（本地）或 **Pix2Text**（Mathpix 替代） |
| **複雜表格 + 多欄排版論文** | **MinerU** ——表格和多欄排版最強 |
| **中日韓（CJK）學術論文** | **MinerU** ——CJK 支援無人能及 |
| **企業 RAG pipeline，多格式文件** | **Docling** ——LlamaIndex/LangChain 整合 |
| **掃描書本數位化為 Markdown/EPUB** | **pdf-craft** ——專爲掃描書本設計 |
| **從原生 PDF 快速提取文字** | **PyMuPDF4LLM** ——輕量、無 GPU、純 CPU |
| **將現有 LaTeX 原始碼轉為 HTML** | **LaTeXML** ——語義 XML/MathML 輸出 |
| **從學術 PDF 提取參考文獻/引用** | **GROBID** ——元數據提取最佳 |
| **從圖像中提取單一公式** | **pix2tex/LaTeX-OCR** ——隔離公式圖像 |
| **通用文件轉換（非 PDF 輸入）** | **Pandoc** ——DOCX/LaTeX→Markdown 表現優異 |
| **PDF 轉換為 LaTeX 原始碼** | **pdf2tex**（PyPI）——RAG 多路徑提取 |

## 關鍵發現

1. **Pandoc 無法真正「讀取」PDF**：這是最常見的誤解。Pandoc 不具備 OCR 或版面感知能力，對學術 PDF 轉換完全不合適。
2. **真正的 PDF→Markdown 需要深度學習**：目前效果最好的工具（Marker、Nougat、MinerU、Pix2Text）都依賴視覺模型來理解版面與數學符號，這意味著 GPU 是實際需求。
3. **PDF→LaTeX 仍是更困難的問題**：相比 Markdown，直接輸出 LaTeX 的工具較少且成熟度較低。pdf2tex（PyPI）是目前最有潛力的選擇，但社群仍小。
4. **GROBID 是「結構化提取」而非「視覺重建」**：如果目標是提取參考文獻、引用、章節標題等元數據，GROBID 是最佳選擇；但無法還原視覺排版。
5. **LaTeXML 和 TeX4ht 不接受 PDF 輸入**：它們處理的是 `.tex` 原始碼，適合從 LaTeX 轉為網頁格式的場景，而非從 PDF 逆轉。

## 參考資料

[^marker]: Datalab-to. (n.d.). Marker — Convert PDF to Markdown quickly and accurately. Retrieved 2026-10-01, from https://github.com/datalab-to/marker
[^marker-bench]: Datalab-to. (n.d.). Marker benchmarks on olmocr-bench. Retrieved 2026-10-01, from https://github.com/datalab-to/marker?tab=readme-ov-file#benchmarks
[^nougat]: Facebook Research. (n.d.). Nougat — Neural Optical Understanding for Academic Documents. Retrieved 2026-10-01, from https://github.com/facebookresearch/nougat
[^pix2text]: breezedeus. (n.d.). Pix2Text — An open-source Python tool to recognize layouts, tables, math formulas and text in images. Retrieved 2026-10-01, from https://github.com/breezedeus/pix2text
[^mineru]: OpenDataLab. (n.d.). MinerU — A high-fidelity document extraction tool. Retrieved 2026-10-01, from https://github.com/opendatalab/MinerU
[^docling]: docling-project. (n.d.). Docling — Document understanding tool by IBM Research. Retrieved 2026-10-01, from https://github.com/docling-project/docling
[^pymupdf4llm]: PyMuPDF. (n.d.). PyMuPDF4LLM — Extract PDF text as LLM-friendly Markdown. Retrieved 2026-10-01, from https://github.com/pymupdf/pymupdf4llm
[^pdfcraft]: oomol-lab. (n.d.). pdf-craft — Convert scanned books to Markdown or EPUB locally. Retrieved 2026-10-01, from https://github.com/oomol-lab/pdf-craft
[^pandoc]: Pandoc. (n.d.). Pandoc — A universal document converter. Retrieved 2026-10-01, from https://pandoc.org
[^latexmld]: NIST. (n.d.). LaTeXML — A LaTeX to XML/HTML/MathML converter. Retrieved 2026-10-01, from https://math.nist.gov/~BMiller/LaTeXML/
[^tex4ht]: TeX Users Group. (n.d.). TeX4ht — TeX to HTML converter. Retrieved 2026-10-01, from https://tug.org/tex4ht/
[^grobid]: Grobid Organization. (n.d.). GROBID — Machine learning for parsing raw PDF documents into structured TEI XML. Retrieved 2026-10-01, from https://github.com/grobidOrg/grobid
[^pdf2latex-emsquid]: emsquid. (n.d.). pdf2latex — Convert PDF to LaTeX (Rust). Retrieved 2026-10-01, from https://github.com/emsquid/pdf2latex
[^pdf2tex-pypi]: p3nGu1nZz. (n.d.). pdf2tex — RAG-based PDF to LaTeX conversion module. Retrieved 2026-10-01, from https://pypi.org/project/pdf2tex/
[^pix2tex-latex-ocr]: lukas-blecher. (n.d.). LaTeX-OCR — pix2tex: Learning-based LaTeX OCR system. Retrieved 2026-10-01, from https://github.com/lukas-blecher/LaTeX-OCR