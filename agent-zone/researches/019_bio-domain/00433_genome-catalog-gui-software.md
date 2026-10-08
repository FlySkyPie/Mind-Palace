# 基因組目錄／庫存管理 GUI 軟體調查

## 概述

本報告調查可用於**索引、庫存、目錄化或瀏覽多個基因組集合**的圖形介面軟體。不同於 IGV、JBrowse、UGENE 等**單一基因組瀏覽器**（著重展示單一基因組的序列與注釋），此處搜尋的軟體提供的是**清單／表格視圖**，讓使用者能查閱、篩選、管理一個基因組集合。

## 推薦軟體

### OpenGenomeBrowser

- **型態**：自架 Web 平台（Python/Django）
- **授權**：原 GPL-3.0，現行版本由 Abrinca GmbH 發布，非嚴格開源
- **網址**：https://opengenomebrowser.github.io/
- **說明**：專為**組織、探索與管理微生物基因組集合**所設計。以模組化資料夾結構儲存每個物件的 `organism.json`（分類、文獻等）與 `genome.json`（定序技術、BUSCO 分數、COG 分類等），Web UI 提供物種清單與基因組清單兩種目錄視圖，並整合比較基因體學工具（BLAST、親緣樹、基因座比較、路徑圖等）[^ogb]。
- **優點**：最吻合「基因組目錄管理器」需求；支援 Docker 部署與使用者權限管理。
- **限制**：新版本授權非完全開源；主要針對微生物基因組。

### GenomeDepot

- **型態**：自架 Web 平台（Django）
- **授權**：GPLv3
- **網址**：https://github.com/aekazakov/genome-depot
- **說明**：由勞倫斯伯克利國家實驗室 ENIGMA 計畫開發，稱為「比較基因體內容管理系統」。可為自有的基因組集合建立網站，內嵌 JBrowse 瀏覽器、BLAST 搜尋、注釋搜尋、比較基因鄰域可視化，並整合 EggNOG、KEGG、GO、Pfam 等注釋管線[^gnd]。
- **優點**：純開源（GPLv3）；已用於 2,205+ 公開細菌基因組與 3,105 個內部基因組。
- **限制**：主要針對微生物基因組。

### NCBI Genome Manager (ngm)

- **型態**：自架 Web 平台（FastAPI/Python）
- **授權**：MIT
- **網址**：https://github.com/mobilome/ncbi-genome-manager
- **說明**：用於管理從 NCBI 下載的區域基因組集合。提供 Web 管理介面，可按分類群批次下載、更新與索引基因組，內建 SQLite 資料庫、基因組清單視圖、BLAST 資料庫建置與 MD5 完整性檢查[^ngm]。
- **優點**：MIT 授權、輕量、專為基因組目錄管理設計。
- **限制**：依賴 NCBI 為資料來源；功能較單純。

### CoGe（Comparative Genomics）

- **型態**：Web 平台（可自架）
- **授權**：開源
- **網址**：https://genomevolution.org/coge/
- **說明**：公有資料庫收錄 22,703 個物種與 60,766 個基因組。**OrganismView** 為主要入口，輸入物種名稱即可瀏覽可用基因組清單與中繼資料；支援 SynMap（共線性點圖）、CoGeBlast、GEvo 等比較工具[^coge]。
- **優點**：公開資料豐富；支援私有／共用基因組；可自架。
- **限制**：公共託管版本是集中式服務；自架需一定技術門檻。

### BIGSdb（BIGSdb-Pasteur / PubMLST）

- **型態**：自架 Web 平台
- **授權**：開源
- **網址**：https://bigsdb.pasteur.fr/
- **說明**：基因體菌株分類與命名平台，用於管理細菌分離株資料庫（如 *Klebsiella*、*Listeria*、*E. coli* 等），每個資料庫是可搜尋的分離株目錄，包含基因組、MLST 型別、抗藥性基因與毒力標記[^bigsdb]。
- **優點**：適合菌株層級的基因組庫存管理；有 API 支援。
- **限制**：主要服務細菌分離株；學習曲線較陡。

### NCBI Genome Database（ncbi.nlm.nih.gov/genome）

- **型態**：公共 Web 入口
- **授權**：免費
- **網址**：https://www.ncbi.nlm.nih.gov/genome/
- **說明**：全球公開基因組的主目錄，提供可搜尋、可篩選的所有基因組清單，包含物種、組裝層級、基因組大小、GC 含量等中繼資料，並連結至下載頁面與 Genome Data Viewer[^ncbig]。
- **優點**：最大最完整的公開基因組資料庫。
- **限制**：不可自架；需網路連線。

### NCBI Datasets

- **型態**：Web GUI + CLI + API
- **授權**：免費
- **網址**：https://www.ncbi.nlm.nih.gov/datasets/
- **說明**：NCBI 的官方工具，用於尋找、瀏覽與下載多個物種的基因組序列、注釋與中繼資料。Web 介面提供基因組表格視圖與篩選功能[^nds]。
- **優點**：同一請求可取得多個基因組與多種檔案格式；CLI 與 API 支援自動化。
- **限制**：不可自架；需網路連線。

### 桌面應用程式

#### NCBI Genome Workbench

- **型態**：桌面 GUI（Windows/macOS/Linux）
- **授權**：免費
- **網址**：https://www.ncbi.nlm.nih.gov/tools/gbench/
- **說明**：NCBI 提供的桌面版基因組操作與可視化軟體，可載入多個基因組、建立專案集合、執行 BLAST、多序列比對、親緣樹分析等。支援向 GenBank 提交完整基因組[^gbench]。

#### Geneious Prime

- **型態**：桌面 GUI
- **授權**：商業（付費）
- **網址**：https://www.geneious.com/
- **說明**：綜合性桌面生物資訊套件，其「文件表格」可作為基因組集合管理器，以清單／表格格式匯入、整理、搜尋、篩選基因組組裝，並整合比對、親緣樹、引子設計等工具[^geneious]。

#### CLC Genomics Workbench（QIAGEN）

- **型態**：桌面 GUI
- **授權**：商業（付費）
- **網址**：https://digitalinsights.qiagen.com/
- **說明**：QIAGEN 的桌面基因體學工作平台，以專案方式組織多個基因組，提供瀏覽器、比較工具與變異分析等功能[^clc]。

## 比較總表

| 軟體 | 型態 | 授權 | 自架 | 目錄清單視圖 | 主要對象 |
|---|---|---|---|---|---|
| **OpenGenomeBrowser** | Web | 原始碼可用（非純開源） | ✅ Docker | ✅ 多層清單 | 微生物 |
| **GenomeDepot** | Web | GPLv3 | ✅ | ✅ 資料庫清單 | 微生物 |
| **NCBI Genome Manager** | Web | MIT | ✅ Docker | ✅ 管理後臺 | 源自 NCBI |
| **CoGe** | Web | 開源 | ✅ | ✅ OrganismView | 泛用（22k+物種） |
| **BIGSdb** | Web | 開源 | ✅ | ✅ 分離株目錄 | 細菌 |
| **NCBI Genome DB** | Web | 免費 | ❌ | ✅ 可搜尋表格 | 泛用（全球） |
| **NCBI Datasets** | Web+CLI | 免費 | ❌ | ✅ 基因組表格 | 泛用 |
| **NCBI Genome Workbench** | 桌面 | 免費 | N/A | ✅ 專案集合 | 泛用 |
| **Geneious Prime** | 桌面 | 商業 | N/A | ✅ 文件表格 | 泛用 |
| **CLC Genomics WB** | 桌面 | 商業 | N/A | ✅ 專案導覽 | 泛用 |

## 結論

若需求是**自架一套 GUI 軟體來管理自己的基因組集合（目錄／庫存功能）**：

1. **OpenGenomeBrowser** — 功能最完整，專為基因組目錄管理設計，但授權非純開源。
2. **GenomeDepot** — 純開源替代方案，提供完整的注釋管線與集合網站功能。
3. **NCBI Genome Manager (ngm)** — 輕量 MIT 授權，適合管理從 NCBI 下載的基因組。
4. **BIGSdb** — 適合菌株層級的微生物基因組庫存管理。

若不需要自架，**NCBI Genome Database** 與 **NCBI Datasets** 是最直接的公共基因組目錄工具。桌面首選則為免費的 **NCBI Genome Workbench** 或付費的 **Geneious Prime**。

## 參考資料

[^ogb]: Roder, T., Oberhaensli, S., Bruggmann, R. (2022). OpenGenomeBrowser: a versatile, dataset-independent web platform for genome data management and comparative genomics. *BMC Genomics*, 23, Article 809. Retrieved 2026-10-03, from https://bmcgenomics.biomedcentral.com/articles/10.1186/s12864-022-09086-3
[^ogb2]: OpenGenomeBrowser. (n.d.). Documentation. Retrieved 2026-10-03, from https://opengenomebrowser.github.io/
[^gnd]: Kazakov, A. E., Korenter, L., La Butte, T., et al. (2026). GenomeDepot—a web-based platform for comparative genomic content management. *Bioinformatics Advances*, 6(1), vbag027. Retrieved 2026-10-03, from https://academic.oup.com/bioinformaticsadvances/article/6/1/vbag027/8474403
[^gnd2]: Kazakov, A. E. (n.d.). GenomeDepot. Retrieved 2026-10-03, from https://github.com/aekazakov/genome-depot
[^ngm]: Mobilome. (n.d.). NCBI Genome Manager. Retrieved 2026-10-03, from https://github.com/mobilome/ncbi-genome-manager
[^coge]: Lyons, E., Pedersen, B., Kane, J., et al. (n.d.). CoGe: Comparative Genomics. Retrieved 2026-10-03, from https://genomevolution.org/coge/
[^bigsdb]: Jolley, K. A., Bray, J. E., Maiden, M. C. J. (2018). Open-access bacterial population genomics: BIGSdb software, the PubMLST.org website, and their applications. *Wellcome Open Research*, 3, 124. Retrieved 2026-10-03, from https://bigsdb.pasteur.fr/
[^ncbig]: National Center for Biotechnology Information. (n.d.). Genome Database. Retrieved 2026-10-03, from https://www.ncbi.nlm.nih.gov/genome/
[^nds]: National Center for Biotechnology Information. (n.d.). NCBI Datasets. Retrieved 2026-10-03, from https://www.ncbi.nlm.nih.gov/datasets/
[^gbench]: National Center for Biotechnology Information. (n.d.). Genome Workbench. Retrieved 2026-10-03, from https://www.ncbi.nlm.nih.gov/tools/gbench/
[^geneious]: Geneious. (n.d.). Geneious Prime. Retrieved 2026-10-03, from https://www.geneious.com/
[^clc]: QIAGEN. (n.d.). CLC Genomics Workbench. Retrieved 2026-10-03, from https://digitalinsights.qiagen.com/