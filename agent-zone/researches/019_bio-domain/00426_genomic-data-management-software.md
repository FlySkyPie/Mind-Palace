# 數位化 DNA/RNA/染色體/基因資料儲存與管理軟體調查

## 摘要

本報告調查目前用於儲存、管理、查詢數位化 DNA、RNA、染色體與基因資料的主要軟體系統，涵蓋六大範疇：(1) 國際公共序列資料庫、(2) 基因體資料庫管理框架、(3) 實驗室資訊管理系統 (LIMS) 與生物樣本庫管理軟體、(4) 基因體檔案格式與工具、(5) 雲端及商業分析平台、(6) 互通性標準。

---

## 1. 國際公共序列資料庫 (International Public Sequence Databases)

此為全球基因體資料的基礎儲存層，為所有定序資料的首要儲存庫。

### 1.1 International Nucleotide Sequence Database Collaboration (INSDC)

三大國際核酸序列資料庫的合作框架，成員每日交換資料，確保任一資料庫的提交可在其他兩個資料庫取得。三者共用 Feature Table 資料標準與持久性登錄號[^insdc]。

| 資料庫 | 維護機構 | 規模與特色 |
|---------|----------|------------|
| **GenBank** | NCBI（美國國家生物技術資訊中心） | 截至 2026 年 4 月第 271 版，收錄 53.9 兆個鹼基對、62.7 億筆序列記錄，涵蓋 58.1 萬個正式描述物種。提供 BLAST 比對、Entrez 檢索、E-utilities API。[^genbank] |
| **European Nucleotide Archive (ENA)** | EMBL-EBI（歐洲生物資訊研究所） | 涵蓋原始定序讀取（European Read Archive）、組裝序列及註解基因組。提供 REST API，與 UniProt、Ensembl 等 EBI 資源整合。[^ena] |
| **DNA Data Bank of Japan (DDBJ)** | NIG（日本國立遺傳學研究所） | 亞洲 INSDC 節點，提供 DRA (DDBJ Read Archive) 儲存原始定序資料。[^ddbj] |

### 1.2 NCBI RefSeq

NCBI 維護的策展非冗餘參考序列集（基因組、轉錄本、蛋白質），每個自然生物分子僅保留一條記錄，提供比對研究的穩定基準線[^refseq]。

### 1.3 NCBI Sequence Read Archive (SRA)

2009 年上線的公開高通量定序原始資料儲存庫，收錄 Illumina、PacBio 等平台的短序列讀取 (<1,000 bp)，目前超過 900 萬筆記錄、12+ PB 資料[^sra]。

### 1.4 Ensembl

EMBL-EBI 與 Wellcome Sanger Institute 共同維護的基因體資料庫與瀏覽器，提供脊椎動物基因體的證據本位註釋，整合序列、變異、基因調控與比較基因體學。支援 50,000+ 基因體（含 Ensembl Genomes 的植物、真菌、細菌、無脊椎動物）[^ensembl]。

### 1.5 UCSC Genome Browser

UC Santa Cruz 基因體學研究所維護的互動式基因體瀏覽器，提供 108+ 脊椎動物/無脊椎動物物種的基因體序列圖形存取，整合大量比對註釋軌跡。以 MySQL 為後端儲存。支援 180+ 基因體組裝及數千個 Assembly Hubs[^ucsc]。

### 1.6 GISAID

全球最大的流感與 SARS-CoV-2 基因體序列儲存庫，採用註冊制，強調資料提交者的歸屬與認可。截至 2023 年初收錄超過 1,440 萬條 SARS-CoV-2 序列[^gisaid]。

---

## 2. 基因體資料庫管理框架 (Genomic Database Management Frameworks)

此類軟體將 VCF 等變異資料與外部註釋整合進結構化資料庫，提供 SQL 或程式化查詢能力。

| 軟體 | 儲存方式 | 規模 | 查詢能力 |
|------|----------|------|----------|
| **GEMINI** | SQLite 資料庫，變異 + 基因型 + 註釋單一檔案 | 數千樣本 | 完整 SQL 查詢；內建孟德爾遺傳分析、負載測試、同型合子片段分析[^gemini] |
| **FAVORannotator** | aGDS 單一檔案格式；可選 PostgreSQL/CSV 後端 | 60,000+ WGS 樣本已測試 | 160 個註釋分數；三種部署模式（SQL/CSV/Cloud）；與 STAARpipeline 整合[^favor] |
| **GATK (Genome Analysis Toolkit)** | VCF/BCF 檔案；支援 GVCF 隊列儲存 | 數千樣本 | 業界標準胚系/體細胞變異發現；430+ 工具；支援 Spark 並行化[^gatk] |
| **Galaxy 平台** | 多種後端（資料庫 + 檔案儲存） | 雲端規模 | 9,000+ 工具的網頁 UI；可重現工作流程；400+ 輸入資料類型[^galaxy] |
| **Bioconductor** | R 套件生態系 | 變異規模 | GenomicRanges、VariantAnnotation、VcfR、GDS/SeqArray[^bioconductor] |

---

## 3. 實驗室資訊管理系統 (LIMS) 與生物樣本庫管理軟體

此類系統管理生物樣本、實驗室工作流程、定序運行及相關後設資料的完整生命週期。

### 3.1 商業 LIMS

| 軟體 | 開發者 | 主要特色 |
|------|--------|----------|
| **Clarity LIMS (LabVantage)** | LabVantage / Illumina | NGS 設施廣泛使用；樣本萃取至定序完整追蹤；儀器整合[^clarity] |
| **STARLIMS** | STARLIMS Corp. | 試管譜系追蹤；冷凍庫庫存管理；21 CFR Part 11 / GxP 法規遵循 |
| **LabVantage LIMS** | LabVantage Solutions | 分裝流程與譜系查詢；多專案/多冷凍庫管理 |
| **Freezerworks** | ATGC Labs | 冷凍庫地圖視覺化；入庫與庫存追蹤；審計軌跡 |
| **Thermo Scientific SampleManager** | Thermo Fisher Scientific | 樣本登記與搬遷流程；儀器整合 |
| **BaseSpace Sequence Hub** | Illumina, Inc. | 雲端平台；直接整合 Illumina 定序器；即時資料串流分析；比對、變異呼叫、分類[^basespace] |

### 3.2 開放原始碼 / 開源 LIMS

| 軟體 | 開發者 | 主要特色 |
|------|--------|----------|
| **OpenSpecimen** | Krishagni Solutions | 100+ 生物樣本庫使用；完整樣本生命週期；智慧冷凍庫管理；REDCap/EPIC 整合[^openspecimen] |
| **MISO** | Wellcome Sanger Institute 社群 | 專為 NGS 定序中心設計；模組化架構；支援多種 NGS 平台[^miso] |
| **Sequencescape** | Wellcome Sanger Institute | 管理 500+ 萬樣本；工單追蹤；冷凍庫管理；API 整合[^sequencescape] |
| **iSkyLIMS** | BU-ISCIII（西班牙） | 基因體學 LIMS；文庫製備至資料產出；連接濕實驗室與生物資訊分析[^iskylims] |
| **SENAITE** | SENAITE 社群 | 企業級 LIMS；品質合規管理；自動化分析證明書；完整審計軌跡[^senaite] |
| **LImBuS** | Aberystwyth University（英國） | 樣本分裝與衍生物建立；捐贈者與知情同意管理；條碼產生[^limbus] |
| **eLabFTW** | Nicolas Carpi 社群 | 電子實驗筆記本 (ELN) 兼實驗室資源管理器；區塊鏈確保資料完整性[^elabftw] |

### 3.3 LabKey Server

LabKey Software 開發的開源核心平台，具備生物樣本庫管理模組，涵蓋從收集到廢棄的完整生命週期追蹤、可自訂冷凍庫地圖、審計就緒的監管鏈、REDCap/EPIC 整合[^labkey]。

---

## 4. 基因體檔案格式與工具

| 格式 | 儲存內容 | 管理工具 |
|------|----------|----------|
| **FASTA** | 核酸 (DNA/RNA) 或蛋白質序列（純文字，無品質分數） | SeqKit、EMBOSS、samtools faidx |
| **FASTQ** | 原始定序讀取（DNA/RNA）附加 Phred 品質字串 | seqtk、fastp、Trimmomatic、Cutadapt |
| **SAM/BAM** | 已比對的定序讀取（對照參考基因體）。BAM 為二進位壓縮版，可索引。 | SAMtools、BCFtools、Picard[^samtools] |
| **CRAM** | 已比對定序讀取的高度壓縮格式（參考基因體本位壓縮，較 BAM 小 50-70%）。GA4GH 標準。 | SAMtools（v1.0+ 支援）、CRAM toolkit（EBI）[^cram] |
| **VCF/BCF** | 遺傳變異（SNP、Indel、SV、CNV）。BCF 為二進位壓縮版。 | BCFtools、VCFtools、GATK |
| **GFF/GTF** | 基因與基因體註釋（基因、轉錄本、外顯子、CDS） | gffread、AGAT、BEDTools |
| **BED** | 基因體區間（峰域、感興趣區域、引子目標） | BEDTools、BEDOPS、UCSC Table Browser |
| **BigWig/BigBed** | 基因體連續訊號 / 大型區間集（多解析度視覺化） | UCSC tools（wigToBigWig、bedToBigBed） |
| **HDF5/Zarr** | 大型多維基因體資料（Hi-C 接觸圖、單細胞基因體學） | h5py、Cooler、AnnData、PyRanges |

---

## 5. 雲端及商業分析平台

| 平台 | 維護者 | 規模 | 主要特色 |
|------|--------|------|----------|
| **Terra** | Broad Institute / Verily | 80+ PB、65K 使用者、30M+ 工作流程 | 開放原始碼（BSD-3）；聯邦資料存取；15K+ 演算法；安全資料分享[^terra] |
| **DNAnexus** | DNAnexus, Inc. | 135+ PB、64K+ 使用者 | 多體學資料攝取；隊列瀏覽；JupyterLab；AI/ML 部署；GxP 法規遵循；FedRAMP[^dnanexus] |
| **Seven Bridges (Velsera)** | Velsera | 800+ 工作流程應用 | 多雲端支援；受規範環境；癌症基因體學與族群基因體學[^sevenbridges] |
| **Golden Helix VarSeq Suite** | Golden Helix | 企業級 | ISO 13485 認證、CE IVDR；本地儲存註釋（ClinVar、gnomAD、OMIM 等）；每月更新；VSWarehouse 長期存檔[^varseq] |
| **Provectus Genomics Data Platform** | Provectus (AWS) | PB 級 | 使用 AWS HealthOmics 與 QuickSight；自稱 300 倍加速 |
| **Elucidata Polly** | Elucidata | 企業級 | Variant Store + Annotation Store 分層架構；聚合 ClinVar、gnomAD、COSMIC[^elucidata] |
| **SOPHiA GENOMICS** | SOPHiA Genetics | 診斷級 | 臨床基因體學資料管理；遺傳性及體細胞變異解讀；用於診斷實驗室 |

---

## 6. 互通性標準

**全球基因體學與健康聯盟 (GA4GH)** 制定的關鍵標準[^ga4gh]：

| 標準 | 用途 |
|------|------|
| **VCF 4.5** | 變異格式規格（GA4GH Genomic Data Toolkit 維護） |
| **CRAM** | 參考基因體本位壓縮格式（比 BAM 小 50-70%） |
| **htsget** | 定序讀取的 REST API 安全串流 |
| **Beacon API v2** | 聯邦式查詢協定（某變異是否存在於特定位置？）；支援表型/臨床查詢 |
| **RefGet** | 參考基因體發現與存取 |
| **DRS (Data Repository Service)** | 雲端基因體資料存取 |
| **Dockstore** | GA4GH 相容工作流程分享平台；可在 Terra、DNAstack、DNAnexus、Seven Bridges 間互通 |
| **WDL / CWL** | 可移植工作流程語言 |

---

## 7. 其他值得注意的專門資料庫

| 資料庫 | 管理內容 | 維護者 |
|--------|----------|--------|
| **ClinVar** | 人類遺傳變異與其臨床意義 | NCBI[^clinvar] |
| **dbSNP / dbVar** | 人類 SNP 與結構變異 | NCBI |
| **COSMIC** | 癌症體細胞突變 | Wellcome Sanger Institute[^cosmic] |
| **gnomAD** | 人類遺傳變異聚合資料庫（125K+ 外顯子、15K+ 基因體） | Broad Institute[^gnomad] |
| **CIViC** | 癌症變異的臨床解讀知識庫 | Washington University in St. Louis[^civic] |
| **UniProt / Swiss-Prot** | 蛋白質序列與註釋（源自 DNA/RNA 翻譯） | UniProt Consortium[^uniprot] |

---

## 8. 生態系總結

基因體資料管理軟體生態系可劃分為以下層次：

1. **國際公共儲存庫**（GenBank、ENA、DDBJ、SRA）— 全球定序資料的首要儲存與同步基礎設施
2. **參考基因體與註釋**（RefSeq、Ensembl、UCSC、Gencode）— 提供穩定的基因體參照與基因註釋
3. **檔案格式層**（FASTA/FASTQ/BAM/CRAM/VCF）— 標準化資料交換格式，伴隨 SAMtools、BCFtools 等核心工具
4. **資料庫框架**（GEMINI 的 SQLite、FAVOR 的 aGDS/PostgreSQL）— 將變異與註釋整合為可查詢結構
5. **實驗室管理系統 (LIMS)**（OpenSpecimen、MISO、Sequencescape、Clarity）— 管理樣本生命週期與定序工作流程
6. **雲端分析平台**（Terra、DNAnexus、Galaxy、BaseSpace）— 提供大規模運算、協作、法規合規的整合環境
7. **商業臨床平台**（Golden Helix VarSeq、SOPHiA）— 提供法規認證與臨床報告支援
8. **標準與互通性**（GA4GH 標準群）— 確保跨平台、跨機構的資料交換與協作

整體趨勢為從靜態檔案儲存轉向結構化、可查詢、註釋豐富的資料庫，並整合 AI/ML 能力以支援即時臨床決策。

---

## References

[^insdc]: International Nucleotide Sequence Database Collaboration. (n.d.). *About INSDC*. Retrieved 2026-10-01, from https://www.insdc.org/
[^genbank]: National Center for Biotechnology Information. (n.d.). *GenBank Overview*. Retrieved 2026-10-01, from https://www.ncbi.nlm.nih.gov/genbank/
[^ena]: European Bioinformatics Institute. (n.d.). *European Nucleotide Archive*. Retrieved 2026-10-01, from https://www.ebi.ac.uk/ena/browser/home
[^ddbj]: National Institute of Genetics. (n.d.). *DNA Data Bank of Japan*. Retrieved 2026-10-01, from https://www.ddbj.nig.ac.jp/
[^refseq]: National Center for Biotechnology Information. (n.d.). *RefSeq: NCBI Reference Sequence Database*. Retrieved 2026-10-01, from https://www.ncbi.nlm.nih.gov/refseq/
[^sra]: National Center for Biotechnology Information. (n.d.). *Sequence Read Archive (SRA)*. Retrieved 2026-10-01, from https://www.ncbi.nlm.nih.gov/sra
[^ensembl]: European Bioinformatics Institute & Wellcome Sanger Institute. (n.d.). *Ensembl Genome Browser*. Retrieved 2026-10-01, from https://www.ensembl.org/
[^ucsc]: UC Santa Cruz Genomics Institute. (n.d.). *UCSC Genome Browser*. Retrieved 2026-10-01, from https://genome.ucsc.edu/
[^gisaid]: Freunde von GISAID e.V. (n.d.). *GISAID: Global Initiative on Sharing All Influenza Data*. Retrieved 2026-10-01, from https://www.gisaid.org/
[^gemini]: Paila, U., Chapman, B. A., Kirchner, R., Quinlan, A. R. (2013). GEMINI: Integrative Exploration of Genetic Variation and Genome Annotations. *PLoS Computational Biology*. Retrieved 2026-10-01, from https://github.com/arq5x/gemini
[^favor]: FAVORannotator. (n.d.). *FAVORannotator: Functional Annotation of Variants*. Retrieved 2026-10-01, from https://github.com/zhouhufeng/FAVORannotator
[^gatk]: Broad Institute. (n.d.). *GATK | Genome Analysis Toolkit*. Retrieved 2026-10-01, from https://gatk.broadinstitute.org/hc/en-us
[^galaxy]: Galaxy Project. (n.d.). *Galaxy Platform*. Retrieved 2026-10-01, from https://galaxyproject.org/
[^bioconductor]: Bioconductor. (n.d.). *Bioconductor: Open-source software for bioinformatics*. Retrieved 2026-10-01, from https://www.bioconductor.org/
[^clarity]: Illumina, Inc. (n.d.). *BaseSpace Sequence Hub*. Retrieved 2026-10-01, from https://www.illumina.com/products/by-type/informatics-products/basespace-sequence-hub.html
[^openspecimen]: Krishagni Solutions. (n.d.). *OpenSpecimen - Biobank Management Software*. Retrieved 2026-10-01, from https://www.openspecimen.org/
[^miso]: MISO LIMS Project. (n.d.). *MISO: Modular and Integrated Sequencing Operations*. Retrieved 2026-10-01, from https://github.com/miso-lims/miso-lims
[^sequencescape]: Wellcome Sanger Institute. (n.d.). *Sequencescape*. Retrieved 2026-10-01, from https://github.com/sanger/sequencescape
[^iskylims]: BU-ISCIII. (n.d.). *iSkyLIMS*. Retrieved 2026-10-01, from https://github.com/BU-ISCIII/iskylims
[^senaite]: SENAITE Community. (n.d.). *SENAITE LIMS*. Retrieved 2026-10-01, from https://github.com/senaite/senaite.core
[^limbus]: Aberystwyth Systems Biology. (n.d.). *LImBuS: Laboratory Information Management System for Biobanks*. Retrieved 2026-10-01, from https://github.com/AberystwythSystemsBiology/limbus
[^elabftw]: Carpi, N. (n.d.). *eLabFTW: Electronic Lab Notebook*. Retrieved 2026-10-01, from https://github.com/elabftw/elabftw
[^labkey]: LabKey Software. (n.d.). *LabKey Server Biobank Software*. Retrieved 2026-10-01, from https://www.labkey.com/solutions/biobank-software/
[^samtools]: Li, H. et al. (2009). The Sequence Alignment/Map format and SAMtools. *Bioinformatics*, 25(16), 2078-2079. Retrieved 2026-10-01, from https://samtools.github.io/
[^cram]: Global Alliance for Genomics and Health. (n.d.). *CRAM File Format Specification*. Retrieved 2026-10-01, from https://ga4gh.org/product/cram/
[^terra]: Verily / Broad Institute. (n.d.). *Terra Platform*. Retrieved 2026-10-01, from https://terra.bio/
[^dnanexus]: DNAnexus, Inc. (n.d.). *DNAnexus Platform*. Retrieved 2026-10-01, from https://www.dnanexus.com/
[^sevenbridges]: Velsera. (n.d.). *Seven Bridges Platform*. Retrieved 2026-10-01, from https://www.sevenbridges.com/platform/
[^varseq]: Golden Helix. (n.d.). *VarSeq Suite for Clinical Variant Analysis*. Retrieved 2026-10-01, from https://www.goldenhelix.com/platform/varseq/variant-analysis
[^elucidata]: Elucidata. (n.d.). *Polly Platform*. Retrieved 2026-10-01, from https://www.elucidata.io/
[^ga4gh]: Global Alliance for Genomics and Health. (n.d.). *GA4GH Standards for Genomic Data Interoperability*. Retrieved 2026-10-01, from https://www.ga4gh.org/
[^clinvar]: National Center for Biotechnology Information. (n.d.). *ClinVar*. Retrieved 2026-10-01, from https://www.ncbi.nlm.nih.gov/clinvar/
[^cosmic]: Wellcome Sanger Institute. (n.d.). *COSMIC: Catalogue of Somatic Mutations in Cancer*. Retrieved 2026-10-01, from https://cancer.sanger.ac.uk/cosmic
[^gnomad]: Broad Institute. (n.d.). *gnomAD: Genome Aggregation Database*. Retrieved 2026-10-01, from https://gnomad.broadinstitute.org/
[^civic]: Washington University in St. Louis. (n.d.). *CIViC: Clinical Interpretation of Variants in Cancer*. Retrieved 2026-10-01, from https://civicdb.org/
[^uniprot]: UniProt Consortium. (n.d.). *UniProt: Universal Protein Resource*. Retrieved 2026-10-01, from https://www.uniprot.org/