# 數位化 DNA/RNA/染色體/基因資料瀏覽 GUI 軟體調查報告

## 摘要

本報告調查市面上具備**圖形使用者介面 (GUI)** 之基因體瀏覽軟體，涵蓋桌面應用程式及網頁平台，用以瀏覽、視覺化、探索數位化之 DNA、RNA、染色體與基因資料。依據部署型態分為三大類：桌面原生應用程式、網頁瀏覽器（含公開站台與自架方案）、以及整合式分析平台。

---

## 1. 桌面原生應用程式 (Desktop Native Applications)

此類別軟體需下載安裝至本機電腦，特色為效能佳、可離線使用、可直接讀取使用者本機資料。

### 1.1 IGV (Integrative Genomics Viewer)

- **開發者：** Broad Institute & UC San Diego
- **授權：** 開源（MIT License）
- **語言/平台：** Java；跨平台（Linux / macOS / Windows）
- **官網：** https://igv.org/ [^igv]

**功能特色：**
- Google Maps 風格導航，從全基因組連續縮放至單一鹼基對解析度
- 支援比對讀取 (BAM/CRAM)、變異 (VCF)、拷貝數 (CNV)、基因表現、甲基化、基因註釋
- 多解析度檔案格式支援即時探索任意大小資料集
- 可同時視覺化數百至數千樣本
- 支援本地、遠端及雲端資料載入（TCGA、1000 Genomes、ENCODE）
- 另有 IGV-Web 瀏覽器版本及 igv.js JavaScript 嵌入函式庫

**適用場景：** 癌症基因體學、變異審閱、NGS 資料探索、整合分析

### 1.2 JBrowse 2 (Desktop 版)

- **開發者：** Evolutionary Software Foundation / GMOD（UC Berkeley）
- **授權：** 開源（Apache License 2.0）
- **語言/平台：** JavaScript (React) + Electron；跨平台
- **官網：** https://jbrowse.org/jb2/ [^jbrowse2]

**功能特色：**
- 模組化架構：線性視圖、環狀視圖、共線性 (synteny) 視圖、點圖 (dot plot)
- 桌面版以 Electron 封裝，一鍵啟動
- 支援多 GB 等級基因組與深度定序資料
- 外掛系統，可擴充功能
- 支援結構變異視覺化（VCF、PacBio HiFi pileups）
- 編譯預載基因組（hg38、hg19、mm39）搭配 UCSC tracks

**適用場景：** 自訂基因體瀏覽器部署、結構變異分析、比較基因體學

### 1.3 UGENE

- **開發者：** Unipro
- **授權：** 開源（GPL v2.0）
- **語言/平台：** C++；跨平台（Linux / macOS / Windows）
- **官網：** http://ugene.net/ [^ugene]

**功能特色：**
- 整合式生物資訊工具套件（300+ 工具）
- 基因體瀏覽器 + 環狀質體圖 + 多重序列比對編輯器 + 3D 結構檢視器（PDB/MMDB）
- 可視化工作流程設計器（Workflow Designer）
- GPU 加速（NVIDIA CUDA、ATI Stream）
- 內建 PCR 引子設計（Primer3）、限制酶搜尋（REBASE）
- 組裝瀏覽器 (BAM viewer)

**適用場景：** 全方位生物資訊分析、分子選殖設計、教學、小型至中型基因組分析

### 1.4 NCBI Genome Workbench

- **開發者：** NCBI, U.S. National Library of Medicine
- **授權：** 開源（NCBI Public Domain License）
- **語言/平台：** C++；跨平台（Linux / macOS / Windows）
- **官網：** https://www.ncbi.nlm.nih.gov/tools/gbench/ [^gbench]

**功能特色：**
- 序列圖形檢視、比對檢視、演化樹檢視、表格檢視
- 內建 BLAST、Clustal、Kalign、MAFFT
- 多重比對顯示模式（Alignment Span、Summary、Cross Align、Dot Matrix）
- Broadcasting 功能：跨多個檢視共享選取物件
- 以專案為基礎的資料組織

**適用場景：** 深度序列分析、系統發生分析、NCBI 資料庫整合

### 1.5 Artemis + ACT

- **開發者：** Wellcome Trust Sanger Institute
- **授權：** 開源（GPL v3.0）
- **語言/平台：** Java；跨平台
- **官網：** https://sanger-pathogens.github.io/Artemis/ [^artemis]

**功能特色：**
- 基因體檢視器與註釋工具，六框翻譯顯示
- **ACT (Artemis Comparison Tool)** — 成對基因體比較、共線性分析
- **BamView** — 獨立 BAM/CRAM 檢視器
- **DNAPlotter** — 圓形/線性 DNA 地圖生成
- 專為微生物及小型真核基因體設計
- 輕量、啟動快速

**適用場景：** 基因體註釋、比較基因體學、微生物基因體分析、手動策展

### 1.6 IGB (Integrated Genome Browser)

- **開發者：** Loraine Lab（原 Affymetrix）
- **授權：** 開源
- **語言/平台：** Java（Genoviz SDK）；跨平台
- **官網：** https://bioviz.org/ [^igb]

**功能特色：**
- 獨特動畫縮放（smooth/continuous zoom）
- 可拖曳圖形疊加於參考註釋
- Edge-matching：跨不同 track 標示相同邊界
- QuickLoad 資料共享系統
- Intron-trimming sliced view（摺疊大內含子以清晰顯示外顯子範圍）
- 可透過 HTTP 請求驅動（Web-controls）

**適用場景：** 替代性剪接分析、基因表現調控、表觀遺傳修飾

### 1.7 Tablet

- **開發者：** The James Hutton Institute
- **授權：** 開源（BSD 2-Clause）
- **語言/平台：** Java；跨平台
- **官網：** https://ics.hutton.ac.uk/tablet/ [^tablet]

**功能特色：**
- 輕量高效能 NGS 組裝與比對視覺化
- 支援 packed/stacked 讀取顯示
- 成對末端 (paired-end) 視覺化（適用 SAM/BAM）
- CIGAR 支援（插入、刪除、剪切事件）
- 全 contig 概覽含覆蓋率資訊
- 按名稱或子序列搜尋讀取

**適用場景：** NGS 比對檢視、組裝驗證、變異驗證

### 1.8 VX Genome Viewer（新興工具）

- **開發者：** Arnaroo 社群
- **授權：** CC-BY-NC-ND 4.0（非商業免費）；MCP/API 規格為 MIT
- **語言/平台：** D/LDC 編譯；跨平台（Linux / macOS / Windows）
- **版本：** v0.9.0（約 2025-2026 公開預覽）
- **官網：** https://github.com/Arnaroo/VX/ [^vx]

**功能特色：**
- GPU 加速 OpenGL 渲染，順暢縮放平移
- 非同步背景執行緒 BAM 載入（視口永不阻塞）
- 單一二進位檔約 6-19 MB，無執行時期依賴
- 支援 FASTA、GTF/GFF、BAM、BigWig、BigBed、VCF、.cool/.mcool（Hi-C）
- 內建 43 項分析（Peak calling、RPKM/TPM、GC content、CpG、motif search、TAD boundaries、Hi-C contact maps、alignment QC、Ts/Tv）
- 內建螢幕錄影/GIF 輸出
- **MCP (Model Context Protocol) Server** — AI agent 可直接觀察、驅動、分析

**適用場景：** AI 輔助基因體分析、高效能視覺化、Hi-C 瀏覽

### 1.9 商業桌面軟體

#### VarSeq + GenomeBrowse（Golden Helix）

- 桌面變異分析 + 基因體瀏覽器；Windows / macOS / Linux
- 自動註釋 ClinVar、gnomAD、OMIM、CADD、SIFT、PolyPhen2
- 即時過濾鏈、ACMG 自動分類（VSClinical）
- GenomeBrowse 可獨立免費使用（學術）
- 官網：https://www.goldenhelix.com/ [^varseq]

#### Alamut Visual Plus（SOPHiA GENETICS）

- 桌面基因體瀏覽器與變異解讀工具；Windows
- 55+ 資料庫註釋（ClinVar、dbSNP、COSMIC、gnomAD、Mastermind）
- ACMG 分類、Missense/splicing predictors（REVEL、AlphaMissense、PolyPhen-2、SIFT）
- BAM/VCF/BED/Sanger 檔案檢視器；HGVS 命名法合規
- v1.13（2025）活躍更新
- 官網：https://www.sophiagenetics.com/ [^alamut]

#### DNASTAR Lasergene（含 GenVision Pro）

- 綜合分子生物學桌面套件；Windows / macOS
- GenVision Pro 基因體瀏覽器 — 多達 50 個人類外觀組同時比較
- 結構變異檢測（split-read analysis）、單倍型分析（diploid phasing）
- Lasergene 18 於 2024 年 10 月發布
- 官網：https://www.dnastar.com/ [^lasergene]

#### SEQUENCE Pilot（JSI Medical Systems）

- 桌面 GUI（Linux / Windows Server）；CE-IVD 認證
- NGS + Sanger + MLPA 全自動分析
- 強調本地端處理：NO CLOUD - STAY IN CONTROL OF YOUR DATA
- 官網：https://www.jsi-medisys.de/ [^seqpilot]

#### NextGENe（SoftGenetics / Dotmatics）

- Windows-only 桌面 GUI；點擊操作無需腳本
- SNP/Indel、結構變異、CNV、融合基因、de novo 組裝
- 轉錄體分析（替代性剪接、表現量）、ChIP-Seq、miRNA、總體基因體學
- 官網：https://www.softgenetics.com/ [^nextgene]

#### Geneious Prime

- 桌面 GUI（Windows / macOS / Linux）；❌ 商業軟體
- 直覺拖放操作的分子生物學與序列分析
- 學術與商業實驗室廣泛使用
- 官網：https://www.geneious.com/ [^geneious]

#### SnapGene

- 桌面 GUI（Windows / macOS）；❌ 商業軟體
- 質體視覺化與選殖模擬
- 分子生物學日常使用極受歡迎
- 官網：https://www.snapgene.com/ [^snapgene]

---

## 2. 網頁基因體瀏覽器（Web Genome Browsers）

此類別可再分為「公開站台免安裝直接使用」與「自架設需要伺服器」兩種。

### 2.1 UCSC Genome Browser

- **開發者：** UC Santa Cruz Genomics Institute
- **授權：** 學術免費，商業需授權
- **官網：** https://genome.ucsc.edu/ [^ucsc]

**功能特色：**
- 最廣泛使用的基因體瀏覽器，180+ 基因體組裝來自 100+ 物種
- 數百筆預載註釋 track（11 大類）加上自訂 track（BED/WIG/GFF/BigWig/BigBed）
- Track Hub 系統共享大規模遠端資料
- 工具：BLAT、In-Silico PCR、Table Browser、LiftOver、REST API、Variant Annotation Integrator
- 全球核心生物資料資源 (Global Core Biodata Resource)
- 出版品質圖像輸出 (PDF/SVG)

**適用場景：** 參考基因體探索、比較基因體學、變異解讀、教學、出版圖稿

### 2.2 Ensembl Genome Browser

- **開發者：** EMBL-EBI & Wellcome Trust Sanger Institute
- **授權：** 開源（Apache 2.0）
- **官網：** https://www.ensembl.org/ [^ensembl]

**功能特色：**
- 1999 年上線，首個基因體瀏覽器
- 脊椎動物 271+ 物種 + Ensembl Genomes（植物、真菌、細菌、原生生物，50,000+ 基因體）
- 業界標準 VEP (Variant Effect Predictor)
- 進階比較基因體學（同源關係、基因樹、全基因體比對）
- BioMart 資料提取介面、REST API、Perl API

**適用場景：** 脊椎動物基因體學、比較基因體學、變異效應預測

### 2.3 NCBI Genome Data Viewer (GDV)

- **開發者：** National Center for Biotechnology Information
- **授權：** 免費公開服務
- **官網：** https://www.ncbi.nlm.nih.gov/genome/gdv/ [^gdv]

**功能特色：**
- 730+ 真核生物 RefSeq 基因體組裝
- Widget 架構：搜尋、BLAST 結果、資料上傳、顯示控制間互通
- 整合 GEO (Gene Expression Omnibus) 資料集
- 簡潔現代化介面

**適用場景：** 快速 RefSeq 基因體查閱、NCBI 資料庫探索

### 2.4 WashU Epigenome Browser

- **開發者：** Wang Lab — Washington University in St. Louis
- **授權：** 開源
- **官網：** https://epigenomegateway.wustl.edu/ [^washu]

**功能特色：**
- 專注表觀基因體資料視覺化
- 預載公共資料：4DN、ENCODE、Roadmap Epigenomics、TCGA
- VR (Virtual Reality) 3D 染色質結構視覺化
- Live Browsing 即時協作
- Undo/Redo 瀏覽歷史、Docker 部署

**適用場景：** 表觀基因體學（ChIP-seq、DNase-seq、Hi-C、甲基化）

### 2.5 HiGlass

- **開發者：** Harvard Medical School
- **授權：** 開源（MIT License）
- **官網：** https://higlass.io/ [^higlass]

**功能特色：**
- 專為 Hi-C 與接觸矩陣視覺化設計
- 多視圖同步導航
- 連續縮放與平移
- 瓷磚式渲染處理極大資料集

**適用場景：** 3D 基因體架構、Hi-C 資料探索、染色質互動分析

---

## 3. 整合式分析平台

### 3.1 Galaxy

- **開發者：** Galaxy Project（Penn State / Johns Hopkins）
- **授權：** 開源（MIT/AFL）
- **官網：** https://galaxyproject.org/ [^galaxy]

**功能特色：**
- 10,000+ 分析工具的網頁平台
- 拖放建立分析流程、完整 provenance 追蹤
- 全球 400K+ 使用者
- 公開伺服器免費使用（usegalaxy.org）；亦可自架

**適用場景：** 一站式 NGS 分析平台、可重現性分析、教學

### 3.2 InterMine

- **開發者：** InterMine 社群（University of Cambridge 等）
- **授權：** 開源（LGPL v3）
- **官網：** https://intermine.org/ [^intermine]

**功能特色：**
- 整合多來源生物資料的資料倉儲系統
- QueryBuilder、全文搜尋、模板查詢
- 驅動 FlyMine、HumanMine、MouseMine、YeastMine 等

**適用場景：** 跨資料庫生物資料整合查詢

### 3.3 Apollo

- **開發者：** GMOD
- **授權：** 開源（BSD）
- **官網：** https://genomearchitect.readthedocs.io/ [^apollo]

**功能特色：**
- 即時協作基因體註釋編輯器
- JBrowse 2 外掛形式
- 歷史追蹤、GO 詞條、PubMed ID 整合

**適用場景：** 社群基因體註釋專案、協同策展

---

## 4. 比較總表

| 軟體 | 類型 | 開源 | GUI 品質 | 安裝難度 | 主要資料類型 | 活躍開發 |
|:---|:---|:---:|:---:|:---:|:---|:---:|
| **IGV** | 桌面 | ✅ | 極佳 | ⭐下載即用 | BAM/VCF/Wiggle/註釋 | ✅ |
| **JBrowse 2** | 桌面+網頁 | ✅ | 極佳 | ⭐⭐低 | GFF/BAM/VCF/BigWig | ✅ |
| **UGENE** | 桌面 | ✅ | 極佳 | ⭐下載即用 | 序列/BAM/3D/質體 | ✅ |
| **VX Genome Viewer** | 桌面 | ⚠️非商業 | 極佳(GPU) | ⭐下載即用 | BAM/VCF/Hi-C | ✅(新) |
| **Artemis** | 桌面 | ✅ | 中等 | ⭐下載即用 | EMBL/GenBank/BAM | ⚠️ |
| **Tablet** | 桌面 | ✅ | 中等 | ⭐下載即用 | SAM/BAM/ACE | ⚠️ |
| **IGB** | 桌面 | ✅ | 佳 | ⭐下載即用 | 序列/BAM/GFF | ⚠️ |
| **NCBI GBench** | 桌面 | ✅ | 中等 | ⭐下載即用 | NCBI/BAM/VCF/GFF | ⚠️ |
| **UCSC** | 網頁 | ⚠️學術免費 | 極佳 | ⭐不需安裝 | 註釋/BigWig/BigBed/BAM | ✅ |
| **Ensembl** | 網頁 | ✅ | 佳 | ⭐不需安裝 | 註釋/VCF/BAM | ✅ |
| **NCBI GDV** | 網頁 | ✅ | 佳 | ⭐不需安裝 | RefSeq/GFF/BED/VCF | ✅ |
| **Galaxy** | 網頁平台 | ✅ | 極佳 | ⭐不需安裝 | 多種 NGS 資料 | ✅ |
| **WashU EpiBrowser** | 網頁 | ✅ | 佳 | ⭐不需安裝 | 表觀基因體 | ✅ |
| **HiGlass** | 網頁 | ✅ | 佳 | ⭐不需安裝 | Hi-C 接觸矩陣 | ✅ |

---

## 5. 使用場景建議

| 使用場景 | 推薦軟體 | 理由 |
|:---|:---|:---|
| 快速瀏覽參考基因體註釋 | UCSC / Ensembl | 不需安裝，預載豐富資料 |
| 瀏覽自己的 NGS 比對資料 (BAM/VCF) | IGV（桌面） | 效能最佳，支援格式最廣 |
| 全功能生物資訊桌面工具 | UGENE | 免費開源，300+ 工具一包搞定 |
| AI 輔助 + 高效能 GPU 瀏覽 | VX Genome Viewer | 新興工具，MCP 整合，流暢度最佳 |
| 基因註釋與比較基因體學 | Artemis + ACT | 業界標準註釋工具，輕量 |
| 表觀基因體資料探索 | WashU Epigenome Browser | 預載大量公開資料，VR 功能獨特 |
| Hi-C / 3D 染色質互動 | HiGlass | 同級最佳 Hi-C 視覺化 |
| 一站式分析平台（含工作流程） | Galaxy（public） | 免費公開伺服器，10,000+ 工具 |
| 協作基因體註釋 | Apollo | 即時同步，分散團隊適用 |
| 商業首選（支援完善） | Geneious / Lasergene / VarSeq | 使用友善，技術支援完備 |
| 結構變異分析 | JBrowse 2 / IGV | 現代 SV 視覺化，PacBio HiFi 支援 |
| 已安裝 Java 環境的輕量方案 | IGB / Tablet | 啟動快速，學習曲線低 |

---

## 6. 結論

針對「具 GUI 可瀏覽數位化 DNA/RNA/染色體/基因資料」的需求：

1. **零安裝（網頁瀏覽器即可）：** UCSC Genome Browser、Ensembl、NCBI GDV、Galaxy（usegalaxy.org）、IGV-Web
2. **桌面下載即用（最佳離線體驗）：** IGV（NGS 標準）、UGENE（全方位工具套件）、JBrowse 2 Desktop、VX Genome Viewer（GPU 加速新星）
3. **微生物基因體專用：** Artemis + ACT（註釋 + 比較）
4. **Hi-C/3D 基因體架構：** HiGlass、WashU Epigenome Browser
5. **商業首選：** Geneious Prime、DNASTAR Lasergene、Alamut Visual Plus、VarSeq + GenomeBrowse

---

## 參考文獻

[^igv]: IGV Team. (n.d.). Integrative Genomics Viewer. Retrieved 2026-10-03, from https://igv.org/
[^jbrowse2]: GMOD. (n.d.). JBrowse 2. Retrieved 2026-10-03, from https://jbrowse.org/jb2/
[^ugene]: Unipro. (n.d.). UGENE — Integrated Bioinformatics Toolkit. Retrieved 2026-10-03, from http://ugene.net/
[^gbench]: NCBI. (n.d.). Genome Workbench. Retrieved 2026-10-03, from https://www.ncbi.nlm.nih.gov/tools/gbench/
[^artemis]: Wellcome Sanger Institute. (n.d.). Artemis Genome Viewer & Annotation Tool. Retrieved 2026-10-03, from https://sanger-pathogens.github.io/Artemis/
[^igb]: Loraine Lab. (n.d.). Integrated Genome Browser. Retrieved 2026-10-03, from https://bioviz.org/
[^tablet]: The James Hutton Institute. (n.d.). Tablet — Graphical Viewer for NGS Assemblies. Retrieved 2026-10-03, from https://ics.hutton.ac.uk/tablet/
[^vx]: Arnaroo. (2025-2026). VX Genome Viewer. Retrieved 2026-10-03, from https://github.com/Arnaroo/VX/
[^varseq]: Golden Helix. (n.d.). VarSeq & GenomeBrowse. Retrieved 2026-10-03, from https://www.goldenhelix.com/
[^alamut]: SOPHiA GENETICS. (n.d.). Alamut Visual Plus. Retrieved 2026-10-03, from https://www.sophiagenetics.com/
[^lasergene]: DNASTAR. (n.d.). Lasergene. Retrieved 2026-10-03, from https://www.dnastar.com/
[^seqpilot]: JSI Medical Systems. (n.d.). SEQUENCE Pilot. Retrieved 2026-10-03, from https://www.jsi-medisys.de/
[^nextgene]: SoftGenetics / Dotmatics. (n.d.). NextGENe. Retrieved 2026-10-03, from https://www.softgenetics.com/
[^geneious]: Geneious. (n.d.). Geneious Prime. Retrieved 2026-10-03, from https://www.geneious.com/
[^snapgene]: SnapGene. (n.d.). SnapGene. Retrieved 2026-10-03, from https://www.snapgene.com/
[^ucsc]: UCSC Genome Browser. (n.d.). UCSC Genome Browser. Retrieved 2026-10-03, from https://genome.ucsc.edu/
[^ensembl]: EMBL-EBI. (n.d.). Ensembl Genome Browser. Retrieved 2026-10-03, from https://www.ensembl.org/
[^gdv]: NCBI. (n.d.). Genome Data Viewer. Retrieved 2026-10-03, from https://www.ncbi.nlm.nih.gov/genome/gdv/
[^washu]: Wang Lab, Washington University in St. Louis. (n.d.). WashU Epigenome Browser. Retrieved 2026-10-03, from https://epigenomegateway.wustl.edu/
[^higlass]: Harvard Medical School. (n.d.). HiGlass. Retrieved 2026-10-03, from https://higlass.io/
[^galaxy]: Galaxy Project. (n.d.). Galaxy. Retrieved 2026-10-03, from https://galaxyproject.org/
[^intermine]: InterMine. (n.d.). InterMine. Retrieved 2026-10-03, from https://intermine.org/
[^apollo]: GMOD. (n.d.). Apollo Genome Annotation Editor. Retrieved 2026-10-03, from https://genomearchitect.readthedocs.io/