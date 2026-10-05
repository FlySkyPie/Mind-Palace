# Genome Database 軟體解決方案研究報告

> 本報告針對 Genome Database（基因組資料庫）管理之軟體解決方案進行調查，聚焦於 **具備 GUI 圖形介面**、**可直接使用(out-of-box)** 的解決方案。

## 一、背景說明

基因組資料庫管理涵蓋：基因序列儲存/檢索、變異分析、基因註釋瀏覽、NGS 資料視覺化、以及實驗室資料管理(LIMS)。本報告根據部署型態分為三大類：(1) 桌面應用程式、(2) 網頁平台（含 SaaS）、(3) 整合式資料管理平台。

## 二、桌面應用程式（Desktop Applications）

此類別特色為下載安裝即可使用，無需伺服器設定，最符合 out-of-box 需求。

### 2.1 IGV (Integrative Genomics Viewer)

- **類型：** 桌面 GUI（Java）
- **授權：** 開源（MIT License）
- **支援資料：** BAM/CRAM 比對、VCF 變異、CNV、基因表現、甲基化、註釋
- **安裝難度：** ⭐ 極易 — 下載即執行，跨平台（Win/Mac/Linux）
- **特點：** Google Maps 風格導航，可從全基因組連續縮放至單一鹼基對。支援大量 NGS 資料集。另有 IGV-Web 瀏覽器版本。
- **官網：** https://igv.org/[^igv]

### 2.2 UGENE

- **類型：** 桌面 GUI
- **授權：** 開源（GPL v2）
- **支援資料：** DNA/蛋白質序列、多重比對、演化樹、NGS 組裝、BAM、3D 結構
- **安裝難度：** ⭐ 極易 — 提供 Win/Mac/Linux 安裝檔
- **特點：** 300+ 整合工具，包含基因體瀏覽器、圓環質體圖、多重比對編輯器、可視化工作流程設計器。一站式的生物資訊工具套件。
- **官網：** http://ugene.net/[^ugene]

### 2.3 NCBI Genome Workbench

- **類型：** 桌面 GUI
- **授權：** 開源（NCBI Public Domain）
- **支援資料：** GenBank 序列、比對、BLAST 結果、BAM/BED/VCF/GFF、演化樹
- **安裝難度：** ⭐ 易 — 提供 Win/Mac/Linux 安裝檔
- **特點：** NCBI 官方桌面工具，整合序列圖形檢視、比對檢視、演化樹檢視、表格檢視。內建 BLAST、Clustal、MAFFT。
- **官網：** https://www.ncbi.nlm.nih.gov/tools/gbench/[^gbench]

### 2.4 Artemis

- **類型：** 桌面 GUI（Java）
- **授權：** 開源（GPL v3）
- **支援資料：** EMBL/GenBank 條目、FASTA、GFF 註釋、BAM/CRAM
- **安裝難度：** ⭐ 易 — Java 跨平台
- **特點：** Sanger Institute 開發，六框翻譯視覺化，附帶 ACT 比較工具。適合手動基因註釋。
- **官網：** https://sanger-pathogens.github.io/Artemis/[^artemis]

### 2.5 Geneious

- **類型：** 桌面 GUI
- **授權：** ❌ 商業軟體（有免費試用）
- **支援資料：** DNA/蛋白質序列、比對、演化樹、NGS、引子設計、選殖
- **安裝難度：** ⭐ 極易 — 提供 Win/Mac/Linux 安裝檔
- **特點：** 使用友善的分子生物學與序列分析桌面軟體，拖放操作。學術與商業實驗室廣泛使用。
- **官網：** https://www.geneious.com/[^geneious]

### 2.6 SnapGene

- **類型：** 桌面 GUI
- **授權：** ❌ 商業軟體
- **支援資料：** DNA 序列、質體、選殖策略、引子、限制酶
- **安裝難度：** ⭐ 極易 — Win/Mac 安裝檔
- **特點：** 直覺的質體視覺化與選殖模擬軟體，分子生物學日常使用極受歡迎。
- **官網：** https://www.snapgene.com/[^snapgene]

### 2.7 IGB (Integrated Genome Browser)

- **類型：** 桌面 GUI（Java）
- **授權：** 開源
- **支援資料：** 序列、基因模型、比對、微陣列、RNA-Seq、ChIP-seq
- **安裝難度：** ⭐ 易 — Java 跨平台
- **特點：** 具動畫縮放功能，QuickLoad 資料共享系統，可拖曳圖形。
- **官網：** https://bioviz.org/[^igb]

### 2.8 CLC Genomics Workbench（QIAGEN）

- **類型：** 桌面 + 網頁 GUI
- **授權：** ❌ 商業軟體（QIAGEN 出品）
- **支援資料：** NGS（DNA-seq、RNA-seq、ChIP-seq、甲基化）、變異檢測、微生物基因體學
- **安裝難度：** ⭐ 極易 — 桌面安裝檔
- **特點：** 全面的商業基因體分析平台，直覺圖形介面，適合大規模世代隊列分析。
- **官網：** https://digitalinsights.qiagen.com/[^clc]

## 三、網頁平台（Web Platforms）

此類別中，使用公開伺服器者無需任何安裝；自架設則需要中等以上技術能力。

### 3.1 Galaxy

- **類型：** 網頁平台 + 工作流程引擎
- **授權：** 開源（MIT/AFL）
- **支援資料：** 序列、變異、註釋、比對、組裝、代謝體、蛋白質體
- **安裝難度：** ⭐⭐⭐ 中等（自架設）/ ⭐ 不需安裝（使用 public server）
- **特點：** 10,000+ 工具，網頁內拖放建立分析流程，完整 provenance 追蹤。全球 400K+ 使用者。
- **官網：** https://galaxyproject.org/[^galaxy]

### 3.2 JBrowse 2

- **類型：** 網頁/桌面基因體瀏覽器
- **授權：** 開源（Apache 2.0）
- **支援資料：** 序列、GFF3/BED 註釋、BAM/CRAM 比對、VCF 變異
- **安裝難度：** ⭐ 桌面版極易 / ⭐⭐⭐ 網頁版中等
- **特點：** 現代 React 重構，支援線性、環狀、點圖、synteny 四種視圖。桌面版單鍵啟動。
- **官網：** https://jbrowse.org/jb2/[^jbrowse2]

### 3.3 UCSC Genome Browser

- **類型：** 網頁基因體瀏覽器
- **授權：** 學術免費，商業需授權
- **支援資料：** 序列、註釋、mRNA 比對、重複序列、基因表現、SNP
- **安裝難度：** ⭐ 公開站台不需安裝 / ⭐⭐⭐⭐ 自架設複雜
- **特點：** 最經典的基因體瀏覽器，豐富的註釋 tracks（track hub 系統），可上傳自訂資料。
- **官網：** https://genome.ucsc.edu/[^ucsc]

### 3.4 Ensembl

- **類型：** 網頁基因體入口
- **授權：** 開源（Apache）
- **支援資料：** 序列、基因註釋、變異、比較基因體學、調控資料、200+ 物種
- **安裝難度：** ⭐ 公開站台不需安裝 / ⭐⭐⭐⭐⭐ 自架設極複雜
- **特點：** EMBL-EBI 維護，內建 VEP（Variant Effect Predictor）、BioMart、REST API。
- **官網：** https://www.ensembl.org/[^ensembl]

### 3.5 InterMine

- **類型：** 網頁資料倉儲系統
- **授權：** 開源（LGPL v3）
- **支援資料：** 基因、蛋白質、GO 詞條、交互作用、GFF3、FASTA
- **安裝難度：** ⭐⭐⭐ 中高（需 PostgreSQL、Java、Tomcat）
- **特點：** 整合多來源生物資料，提供 QueryBuilder、全文搜尋、模板查詢。驅動 FlyMine、HumanMine 等主要模型資料庫。
- **官網：** https://intermine.org/[^intermine]

### 3.6 IGV-Web

- **類型：** 網頁基因體瀏覽器（IGV 的瀏覽器版本）
- **授權：** 開源（MIT）
- **支援資料：** 同桌面 IGV（BAM、VCF、註釋等）
- **安裝難度：** ⭐ 不需安裝 — 瀏覽器直接使用
- **特點：** 在瀏覽器中提供桌面級視覺化，無需安裝任何軟體。
- **官網：** https://igv.org/app/[^igv]

### 3.7 HiGlass

- **類型：** 網頁基因交互地圖瀏覽器
- **授權：** 開源（MIT）
- **支援資料：** Hi-C 接觸矩陣、2D 基因體資料
- **安裝難度：** ⭐⭐⭐ 中等（伺服器）/ ⭐ 公開測試站不需安裝
- **特點：** 專為基因體交互地圖設計，多視圖同步導航，瓷磚式渲染。
- **官網：** https://higlass.io/[^higlass]

### 3.8 NCBI Genome Data Viewer (GDV)

- **類型：** 網頁基因體瀏覽器
- **授權：** 免費（NCBI 公開服務）
- **支援資料：** RefSeq 基因體組裝、註釋、比對、變異
- **安裝難度：** ⭐ 不需安裝
- **特點：** NCBI 官方基因體瀏覽器，與 NCBI 資料庫深度整合。支援 eukaryotic RefSeq 基因體。
- **官網：** https://www.ncbi.nlm.nih.gov/genome/gdv/[^gdv]

### 3.9 WashU Epigenome Browser

- **類型：** 網頁基因體瀏覽器
- **授權：** 開源
- **支援資料：** 基因體資料（ChIP-seq、RNA-seq、甲基化、Hi-C）
- **安裝難度：** ⭐ 公開站台不需安裝 / ⭐⭐⭐ 自架設中等
- **特點：** 專注基因體資料的視覺化、整合與分析。現代 React 版本。
- **官網：** https://epigenomegateway.wustl.edu/[^washu]

### 3.10 OpenGenomeBrowser

- **類型：** 網頁微生物比較基因體平台
- **授權：** ⚠️ 原開源（GPL-3.0），現商業化
- **支援資料：** 基因體序列、註釋、演化樹、生化路徑、BLAST
- **安裝難度：** ⭐⭐⭐ 中等（Docker）
- **特點：** 自架式比較基因體學平台，互動演化樹、基因座比較、路徑瀏覽。適合微生物基因體。
- **官網：** https://opengenomebrowser.github.io/[^ogb]

## 四、整合式資料管理平台（Integrated Data Management Platforms）

此類別適合需要 LIMS、ELN、樣品追蹤、資料整合的實驗室環境。

### 4.1 Benchling

- **類型：** 雲端 SaaS（ELN + LIMS + 分子生物學）
- **授權：** ❌ 商業
- **支援資料：** 序列、構築體、引子、質體、樣品、檢測資料、庫存
- **安裝難度：** ⭐ 不需安裝（雲端 SaaS）
- **特點：** 整合電子實驗筆記本、分子生物學設計、庫存管理、AI 輔助分析。200,000+ 科學家使用（Moderna、Gilead 等）。
- **官網：** https://www.benchling.com/[^benchling]

### 4.2 LabKey Server

- **類型：** 伺服器（LIMS + 資料管理）
- **授權：** ⚠️ 核心開源（open-core），進階功能商業
- **支援資料：** 生醫研究資料、樣品、檢測結果、ELISA、質譜、NGS
- **安裝難度：** ⭐⭐⭐ 中等（伺服器安裝）
- **特點：** 整合、分析、分享複雜生醫資料的平台。500+ 實驗室使用（NIH、Fred Hutch、Takeda）。
- **官網：** https://www.labkey.com/[^labkey]

### 4.3 Apollo

- **類型：** 網頁協作基因註釋編輯器
- **授權：** 開源（BSD）
- **支援資料：** 基因體註釋（基因、pseudogene、ncRNA、重複區域）、GO 詞條
- **安裝難度：** ⭐⭐⭐⭐ 複雜（需 Java、Grails、資料庫、Tomcat）
- **特點：** 即時協作基因體註釋，多人同步編輯，JBrowse 2 外掛。
- **官網：** https://genomearchitect.readthedocs.io/[^apollo]

## 五、比較總表

### 5.1 最易使用（Out-of-Box 排序）

| 排名 | 軟體 | 類型 | 開源? | GUI | 安裝 |
|:---:|:---|:---|:---:|:---:|:---:|
| 1 | **IGV** | 桌面瀏覽器 | ✅ | 極佳 | 下載即用 |
| 2 | **UGENE** | 桌面工具套件 | ✅ | 極佳 | 下載即用 |
| 3 | **IGV-Web** | 網頁瀏覽器 | ✅ | 極佳 | 不需安裝 |
| 4 | **NCBI GDV** | 網頁瀏覽器 | ✅ | 佳 | 不需安裝 |
| 5 | **UCSC Browser** | 網頁瀏覽器 | ⚠️ | 極佳 | 不需安裝 |
| 6 | **Ensembl** | 網頁入口 | ✅ | 佳 | 不需安裝 |
| 7 | **Galaxy (public)** | 網頁平台 | ✅ | 極佳 | 不需安裝 |
| 8 | **Geneious** | 桌面工具 | ❌ | 極佳 | 下載即用 |
| 9 | **SnapGene** | 桌面分子生物 | ❌ | 極佳 | 下載即用 |
| 10 | **IGB** | 桌面瀏覽器 | ✅ | 佳 | 下載即用 |

### 5.2 使用場景建議

| 使用場景 | 推薦解決方案 |
|:---|:---|
| 快速瀏覽基因體註釋 | UCSC Genome Browser、Ensembl（不需安裝） |
| 瀏覽自己的 NGS 資料（BAM/VCF） | IGV（桌面，極易安裝） |
| 一站式中大型分析平台 | Galaxy（公開伺服器免費使用） |
| 完整的生物資訊桌面工具套件 | UGENE（免費）、Geneious（商業） |
| 質體選殖與分子生物學日常 | SnapGene（商業） |
| 實驗室資料管理（LIMS + 資料） | Benchling（SaaS）或 LabKey（自架） |
| 協作基因註釋 | Apollo |
| 基因交互地圖（Hi-C） | HiGlass |
| 微生物比較基因體學 | OpenGenomeBrowser |
| 整合多來源生物資料查詢 | InterMine |

## 六、結論

對於 **out-of-box 且具 GUI** 的需求，建議分層考量：

1. **零安裝需求：** UCSC Genome Browser、Ensembl、Galaxy（usegalaxy.org）、IGV-Web — 瀏覽器即可使用。
2. **桌面下載即用（最符合 out-of-box）：** IGV（NGS 視覺化）、UGENE（全方位生物資訊工具）、NCBI Genome Workbench。
3. **需要資料管理 + 協作：** Benchling（SaaS 雲端，零維護）、LabKey Server（需伺服器但功能完整）。
4. **商業首選（支援完善）：** Geneious、CLC Genomics Workbench、SnapGene。

## 參考文獻

[^igv]: IGV Team. (n.d.). Integrative Genomics Viewer. Retrieved 2026-10-03, from https://igv.org/
[^ugene]: Unipro. (n.d.). UGENE — Integrated Bioinformatics Toolkit. Retrieved 2026-10-03, from http://ugene.net/
[^gbench]: NCBI. (n.d.). Genome Workbench. Retrieved 2026-10-03, from https://www.ncbi.nlm.nih.gov/tools/gbench/
[^artemis]: Sanger Institute. (n.d.). Artemis Genome Viewer. Retrieved 2026-10-03, from https://sanger-pathogens.github.io/Artemis/
[^geneious]: Geneious. (n.d.). Geneious Prime. Retrieved 2026-10-03, from https://www.geneious.com/
[^snapgene]: SnapGene. (n.d.). SnapGene. Retrieved 2026-10-03, from https://www.snapgene.com/
[^igb]: Loraine Lab. (n.d.). Integrated Genome Browser. Retrieved 2026-10-03, from https://bioviz.org/
[^clc]: QIAGEN. (n.d.). CLC Genomics Workbench. Retrieved 2026-10-03, from https://digitalinsights.qiagen.com/
[^galaxy]: Galaxy Project. (n.d.). Galaxy. Retrieved 2026-10-03, from https://galaxyproject.org/
[^jbrowse2]: JBrowse. (n.d.). JBrowse 2. Retrieved 2026-10-03, from https://jbrowse.org/jb2/
[^ucsc]: UCSC Genome Browser. (n.d.). UCSC Genome Browser. Retrieved 2026-10-03, from https://genome.ucsc.edu/
[^ensembl]: EMBL-EBI. (n.d.). Ensembl Genome Browser. Retrieved 2026-10-03, from https://www.ensembl.org/
[^intermine]: InterMine. (n.d.). InterMine. Retrieved 2026-10-03, from https://intermine.org/
[^higlass]: HiGlass. (n.d.). HiGlass. Retrieved 2026-10-03, from https://higlass.io/
[^gdv]: NCBI. (n.d.). Genome Data Viewer. Retrieved 2026-10-03, from https://www.ncbi.nlm.nih.gov/genome/gdv/
[^washu]: WashU Epigenome Browser. (n.d.). Epigenome Gateway. Retrieved 2026-10-03, from https://epigenomegateway.wustl.edu/
[^ogb]: OpenGenomeBrowser. (n.d.). OpenGenomeBrowser. Retrieved 2026-10-03, from https://opengenomebrowser.github.io/
[^benchling]: Benchling. (n.d.). Benchling. Retrieved 2026-10-03, from https://www.benchling.com/
[^labkey]: LabKey. (n.d.). LabKey Server. Retrieved 2026-10-03, from https://www.labkey.com/
[^apollo]: GMOD. (n.d.). Apollo Genome Annotation Editor. Retrieved 2026-10-03, from https://genomearchitect.readthedocs.io/