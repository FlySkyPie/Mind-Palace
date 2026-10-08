# DNA 數據儲存與庫存管理軟體調查

## 摘要

本報告調查目前用於儲存、管理、盤點 DNA 序列資料的主要軟體系統，涵蓋三大範疇：(1) 國際公共序列資料庫、(2) 實驗室資訊管理系統 (LIMS) 與生物樣本庫管理軟體、(3) DNA 條碼資料管理工具。

---

## 1. 國際公共 DNA 序列資料庫

### 1.1 International Nucleotide Sequence Database Collaboration (INSDC)

INSDC 是三大國際核酸序列資料庫的合作框架，成員每日交換資料，確保任一資料庫的提交可在其他兩個資料庫取得。三者共用 DDBJ/ENA/GenBank Feature Table 資料標準、持久性登錄號 (accession identifiers) 及開放取用原則。[^insdc]

### 1.2 GenBank（NCBI，美國）

由美國國家生物技術資訊中心 (NCBI) 維護的開放取用核酸序列資料庫，自 1982 年營運至今。截至 2026 年 4 月第 271 版，已收錄 **53.9 兆個鹼基對**、**62.7 億筆序列記錄**，涵蓋超過 58.1 萬個正式描述物種。提供 BLAST 比對、Entrez 檢索、E-utilities 程式化存取等功能。[^genbank]

### 1.3 European Nucleotide Archive (ENA，EMBL-EBI，歐洲)

歐洲生物資訊研究所 (EMBL-EBI) 維護的歐洲 INSDC 節點，涵蓋原始定序讀取 (透過 European Read Archive)、組裝序列及註解基因組。提供 REST API 進行程式化存取，並與 UniProt、Ensembl 等 EBI 資源整合。[^ena]

### 1.4 DNA Data Bank of Japan (DDBJ，NIG，日本)

日本國立遺傳學研究所 (NIG) 維護的亞洲 INSDC 節點，提供 DRA (DDBJ Read Archive) 儲存原始定序資料，並提供多種分析與檢索工具。[^ddbj]

### 1.5 NCBI RefSeq

NCBI 維護的策展非冗餘參考序列集（基因組、轉錄本、蛋白質），提供比對研究的穩定基準線。不同於 GenBank 的檔案性質，RefSeq 每個自然生物分子僅保留一條記錄。[^refseq]

### 1.6 NCBI Sequence Read Archive (SRA)

2009 年上線的公開高通量定序原始資料儲存庫，收錄來自 Illumina、PacBio 等平台的短序列讀取 (<1,000 bp)，目前超過 900 萬筆記錄、12+ PB 資料，為 INSDC 成員，每日與 ENA 及 DDBJ 同步。[^sra]

---

## 2. 實驗室資訊管理系統 (LIMS) 與生物樣本庫管理軟體

### 2.1 商業 LIMS

| 軟體名稱 | 開發者 | 主要特色 |
|----------|--------|----------|
| STARLIMS | STARLIMS Corp. | 試管譜系追蹤、冷凍庫庫存管理、法規遵循 (21 CFR Part 11, GxP)、條碼支援 |
| LabVantage LIMS | LabVantage Solutions | 分裝流程與譜系查詢、多專案/多冷凍庫管理、無需大量編碼即可設定 |
| LabWare LIMS | LabWare, Inc. | 母-子樣本關係建模、庫存位置追蹤、實驗室事件整合 |
| LabCollector Biobank | AgileBio | 條碼驅動的生物樣本庫管理、冷凍庫地圖視覺化、模組化附加元件 (ELN、試劑管理) |
| Freezerworks | ATGC Labs | 冷凍庫地圖視覺化、入庫與庫存追蹤、條碼就緒儲存位置、審計軌跡 |
| Thermo Scientific SampleManager | Thermo Fisher Scientific | 樣本登記與搬遷流程、加強庫存管理、儀器整合 |
| NorayBanks | NorayBio (西班牙) | 樣本生命週期管理、實驗室工作流程自動化、合規追蹤、研究人員請求工作流程 |

### 2.2 開放原始碼 LIMS

| 軟體名稱 | 開發者 | 主要特色 |
|----------|--------|----------|
| OpenSpecimen | Krishagni Solutions | 專為生物樣本庫打造的開放原始碼平台，100+ 生物樣本庫使用於 20+ 國家；涵蓋完整樣本生命週期、智慧冷凍庫儲存管理、條碼標籤產生與列印、REDCap/EPIC 整合。[^openspecimen] |
| MISO | Sanger Institute 社群 | 專為 NGS 定序中心設計的開放原始碼 LIMS；模組化架構，支援多種 NGS 平台。[^miso] |
| Sequencescape | Wellcome Sanger Institute | 網頁版 LIMS，管理超過 500 萬樣本與 180 萬件實驗器皿；支援工單追蹤、冷凍庫管理、API 整合。[^sequencescape] |
| iSkyLIMS | BU-ISCIII (西班牙) | 基因組學 LIMS，從文庫製備到資料產出進行樣本管理；連接溼實驗室與生物資訊分析。[^iskylims] |
| SENAITE | SENAITE 社群 | 企業級開放原始碼 LIMS，涵蓋從樣本收件到報告發布的完整分析流程；內建品質合規管理、自動化分析證明書、完整審計軌跡。[^senaite] |
| LImBuS | Aberystwyth University (英國) | 生物樣本庫資訊管理系統，支援樣本分裝與衍生物建立、捐贈者與知情同意管理、加密文件管理、儲存 rack 批次管理、條碼產生。[^limbus] |
| Baobab LIMS | Baobab 社群 | 基於 Plone (Python) 的生物樣本全生命週期追蹤 LIMS。[^baobab] |
| Pangu LIMS | Jianwei Zhang | 網頁版基因組資料管理 LIMS，支援 DNA 定序、基因表現、基因分型。[^pangu] |
| eLabFTW | Nicolas Carpi 社群 | 電子實驗筆記本 (ELN) 兼實驗室資源管理器，提供區塊鏈與可信時間戳確保資料完整性。[^elabftw] |


### 2.3 LabKey Server

LabKey Software 開發的開放原始碼核心平台，具備生物樣本庫管理模組，涵蓋從收集到廢棄的完整生命週期追蹤、可自訂的冷凍庫地圖與儲存位置、審計就緒的監管鏈、與 REDCap 及 EPIC 的整合。[^labkey]

### 2.4 Biobank (CBSR)

加拿大生物樣本儲存庫 (CBSR) 開發的開放原始碼 Java 應用程式，支援多使用者同時處理樣本、護理師/技術員/研究員/管理員角色管理、第 4 版為網頁版架構、支援 2D DataMatrix 條碼掃描。[^cbsr]

---

## 3. DNA 條碼 (DNA Barcoding) 資料管理工具

### 3.1 BOLD (Barcode of Life Data System)

全球最大的雲端 DNA 條碼資料庫、分析與發布平台，由加拿大圭爾夫大學生物多樣性基因組學中心 (Centre for Biodiversity Genomics) 開發，2005 年上線，2024 年推出第 5 版。收錄超過 31.8 萬個正式描述物種的條碼序列（約 890 萬個樣本）。提供資料入口、教育入口、BIN 註冊表（條碼索引號，用於推定物種）以及資料收集分析工作台。[^bold]

### 3.2 BarKeeper

德國明斯特大學 Kai Müller 與 Sarah Wiechers 開發的開放原始碼網頁框架，用於組裝、分析與管理 DNA 條碼資料及後設資料。不受限於特定標記或分類群，同時支援 Sanger 定序與高通量定序 (HTS) 資料。採用 Docker 部署，內建 SATIVA/MAFFT 錯誤標記分析及 OpenStreetMap 位置標記。[^barkeeper]

### 3.3 CodonCode Aligner

CodonCode Corporation (美國) 開發的商用 DNA 序列組裝、比對、編輯與分析軟體，特別支援 DNA 條碼工作流程。支援 ABI、AB1、SCF、FASTA、FASTQ、GenBank 等多種格式，內建 Sanger 色譜圖編輯、突變偵測、虛擬選殖、引子設計、RFLP 分析、親緣樹建構、亞硫酸鹽定序甲基化分析等功能。[^codoncode]

---

## 4. 結論

DNA 數據儲存與庫存管理軟體生態系極為成熟，涵蓋從大規模公共序列資料庫（GenBank、ENA、DDBJ）到實驗室層級的 LIMS（OpenSpecimen、STARLIMS、Sequencescape）以及專門的條碼資料管理平台（BOLD、BarKeeper）。開放原始碼選項豐富，可滿足不同規模與預算的研究機構需求。

---

[^insdc]: International Nucleotide Sequence Database Collaboration. (n.d.). *About INSDC*. Retrieved 2026-10-01, from https://www.insdc.org/
[^genbank]: National Center for Biotechnology Information. (n.d.). *GenBank Overview*. Retrieved 2026-10-01, from https://www.ncbi.nlm.nih.gov/genbank/
[^ena]: European Bioinformatics Institute. (n.d.). *European Nucleotide Archive*. Retrieved 2026-10-01, from https://www.ebi.ac.uk/ena/browser/home
[^ddbj]: National Institute of Genetics. (n.d.). *DNA Data Bank of Japan*. Retrieved 2026-10-01, from https://www.ddbj.nig.ac.jp/
[^refseq]: National Center for Biotechnology Information. (n.d.). *RefSeq: NCBI Reference Sequence Database*. Retrieved 2026-10-01, from https://www.ncbi.nlm.nih.gov/refseq/
[^sra]: National Center for Biotechnology Information. (n.d.). *Sequence Read Archive (SRA)*. Retrieved 2026-10-01, from https://www.ncbi.nlm.nih.gov/sra
[^openspecimen]: Krishagni Solutions. (n.d.). *OpenSpecimen - Biobank Management Software*. Retrieved 2026-10-01, from https://www.openspecimen.org/
[^miso]: MISO LIMS Project. (n.d.). *MISO: Modular and Integrated Sequencing Operations*. Retrieved 2026-10-01, from https://github.com/miso-lims/miso-lims
[^sequencescape]: Wellcome Sanger Institute. (n.d.). *Sequencescape*. Retrieved 2026-10-01, from https://github.com/sanger/sequencescape
[^iskylims]: BU-ISCIII. (n.d.). *iSkyLIMS*. Retrieved 2026-10-01, from https://github.com/BU-ISCIII/iskylims
[^senaite]: SENAITE Community. (n.d.). *SENAITE LIMS*. Retrieved 2026-10-01, from https://github.com/senaite/senaite.core
[^limbus]: Aberystwyth Systems Biology. (n.d.). *LImBuS: Laboratory Information Management System for Biobanks*. Retrieved 2026-10-01, from https://github.com/AberystwythSystemsBiology/limbus
[^baobab]: Baobab LIMS Community. (n.d.). *Baobab LIMS*. Retrieved 2026-10-01, from https://github.com/BaobabLims/baobab.lims
[^pangu]: Zhang, J. (n.d.). *Pangu LIMS*. Retrieved 2026-10-01, from https://github.com/Jianwei-Zhang/LIMS
[^elabftw]: Carpi, N. (n.d.). *eLabFTW: Electronic Lab Notebook*. Retrieved 2026-10-01, from https://github.com/elabftw/elabftw
[^labkey]: LabKey Software. (n.d.). *LabKey Server Biobank Software*. Retrieved 2026-10-01, from https://www.labkey.com/solutions/biobank-software/
[^cbsr]: Canadian BioSample Repository. (n.d.). *Biobank*. Retrieved 2026-10-01, from https://github.com/CBSR-Biobank/biobank
[^bold]: Centre for Biodiversity Genomics, University of Guelph. (n.d.). *Barcode of Life Data System (BOLD)*. Retrieved 2026-10-01, from https://portal.boldsystems.org/about
[^barkeeper]: Müller, K. & Wiechers, S. (n.d.). *BarKeeper*. Retrieved 2026-10-01, from https://github.com/DNA-BarKeeper/barkeeper
[^codoncode]: CodonCode Corporation. (n.d.). *CodonCode Aligner Features*. Retrieved 2026-10-01, from https://www.codoncode.com/aligner/features.htm