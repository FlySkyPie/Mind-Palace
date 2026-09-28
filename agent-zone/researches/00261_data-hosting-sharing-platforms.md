# 資料託管與分享平台概覽 — GitHub 以外的資料共享選擇

## 背景

GitHub 是當今最普及的程式碼託管與協作平台，然而其設計目標是原始碼，並非資料集或大型檔案的理想載體。研究者、資料科學家與一般使用者若需要託管與分享資料（而非程式碼），有許多專門的平台可供選擇。本文將這些平台依用途分類介紹，並比較其儲存限制、版本控制支援、DOI 分配、策展（curation）機制與價格。

## 學術與通用研究資料儲存庫

### Zenodo

由 CERN 與 OpenAIRE 共同營運的開放取用儲存庫，接受任何學科、任何檔案格式，是目前最廣泛使用的通用型資料儲存庫之一[^zenodo-about]。

主要特色：
- DataCite DOI 自動分配，支援概念 DOI（concept DOI）與版本 DOI（version DOI）的連結關係
- 內建版本控制，每個版本保留獨立 DOI
- GitHub 整合：可一鍵封存 GitHub 儲存庫的 tagged release
- 可建立社群（communities）進行策展分類
- 支援開放／禁制／限制／關閉等存取權限設定
- OAI-PMH 中繼資料擷取協定

**儲存限制**：每個記錄（record）上限 50 GB，最多 100 個檔案；可申請一次性提高到 200 GB[^zenodo-limit]。記錄數量無上限（只要控制在 50 GB 以下）。

**價格**：完全免費。

**限制**：無人為策展（metadata 品質完全依賴上傳者）；超過 50 GB 的巨型資料集（如基因體學、醫學影像）不適合；截至本文撰寫時尚未取得 CoreTrustSeal 認證。

---

### Figshare

由 Digital Science（Springer Nature 旗下）營運的商業化開放資料發布平台，可上傳資料集、圖表、媒體、程式碼、簡報等各種格式[^figshare-about]。

主要特色：
- DataCite DOI 分配與版本控制
- 瀏覽器內建常見格式預覽
- REST API
- Altmetric 使用統計
- 機構版（Figshare for Institutions）可自訂品牌

**儲存限制（免費）**：帳戶總計 20 GB 私有空間，單檔最大 20 GB[^figshare-limit]。

**儲存限制（機構版）**：單檔可達 5 TB。

**價格（Figshare+ 大量資料發布）**：
- 250 GB 以下：$1,225（一次性）
- 5 TB：$24,500（一次性）
- 20 GB 以下資料集完全免費[^figshare-plus]

**限制**：免費方案相當嚴格（20 GB）；免費方案無人為策展；商業公司營運，偏好完全開放基礎設施的使用者可能有疑慮。

---

### Harvard Dataverse

由哈佛大學量化社會科學研究所（IQSS）營運，是開源 Dataverse 專案的主要實例，開放所有研究者使用，並提供豐富的學科特定中繼資料綱要（metadata schema）[^dataverse-about]。

主要特色：
- 無硬性單檔大小限制（但瀏覽器上傳限制 2.5 GB，更大檔案需特殊安排）
- 可自訂中繼資料區塊：社會科學、地理空間、生命科學、天文學等
- 瀏覽器內建表格資料探索（CSV、TSV、Stata、SPSS、R 等格式）
- 完整資料集版本控制，附變更記錄
- 強大的 REST API
- 層級化組織結構（子 Dataverse → 資料集 → 檔案）

**儲存限制**：每位研究者約 1 TB 標準配額；哈佛所屬人員可申請更多。

**價格**：所有研究者（無論是否為哈佛成員）完全免費。

**限制**：介面較為老舊；不同 Dataverse 實例的使用者經驗不一；在通用搜尋引擎中的可發現性（discoverability）不如 Zenodo 與 Figshare。

---

### Dryad

由非營利社群治理組織 Dryad 營運，專注於與同儕審查期刊論文相關聯的研究資料集，特色是每個上傳都經過專業策展[^dryad-about]。

主要特色：
- 專業人為策展：檢查可存取性、可再利用性、檔案完整性與 metadata 完整性
- DataCite DOI 分配
- ORCID 整合
- 期刊投稿流程整合
- 僅採用 CC0 授權

**儲存限制**：單次存放（deposit）無硬性上限，價格依大小調整。

**價格**：
- 每次資料存放：$150（Data Publishing Charge）
- 同儕審閱期間設為私有：$50
- 許多所屬機構有會員協議可減免費用

**限制**：每次 $150 的費用對無經費補助的研究者或需多次小量存放的團隊構成障礙；僅接受資料（不包含軟體、簡報等）；CC0 要求不適合敏感性資料。

---

### Mendeley Data

由 Elsevier 營運的免費通用資料儲存庫，同時也是資料發現層，索引來自數千個外部儲存庫超過 2,000 萬筆資料集[^mendeley-about]。

主要特色：
- DataCite DOI 分配
- 免費存放
- 支援多種 Creative Commons 授權
- 與 Elsevier 期刊投稿流程整合
- 符合 FAIR 原則

**儲存限制**：每個資料集 10 GB[^mendeley-limit]；Digital Commons Data 機構版支援每資料集最高 100 GB。

**價格**：10 GB 以下免費。機構版價格依合約而定。

**限制**：10 GB 上限是通用儲存庫中最嚴格的之一；無人為策展；Elsevier 擁有，部分社群對完全開放基礎設施有疑慮。

---

### Open Science Framework (OSF)

由開放科學中心（Center for Open Science）營運，結合資料儲存與研究專案管理，同時可當作預印本伺服器、專案工作區與資料儲存庫使用[^osf-about]。

主要特色：
- DOI 與 ARK 識別碼
- 巢狀專案元件（nested project components），各自獨立存取控制
- 研究預註冊（preregistration）支援
- 可連接 GitHub、Dropbox、Google Drive、Amazon S3、Dataverse、Figshare 等外部服務
- 凍結時間戳記的註冊（registration）

**儲存限制**：預設儲存 5 GB 每檔案；可透過外部儲存串接提高。

**價格**：完全免費。

**限制**：每檔 5 GB 限制對大型資料集較嚴格；相較於專用資料儲存庫，metadata 與策展能力較弱；透過外部搜尋引擎的發現可靠度較低。

---

## AI/機器學習專用資料平台

### Hugging Face Datasets Hub

目前最受歡迎的開放式機器學習資料集平台，隸屬於 Hugging Face 生態系（含模型、Spaces、推理 API），已託管超過 60 萬個資料集[^huggingface-about]。

主要特色：
- Git-based 版本控制（搭配 Git LFS）
- 資料集瀏覽器與預覽功能
- 與 Hugging Face `datasets` 函式庫深度整合
- Parquet 與 WebDataset 格式支援
- 社群功能（按讚、下載、討論）
- Storage Buckets（非版本化的可變儲存空間）

**儲存限制（免費）**：公開儲存採 best-effort 模式。私有儲存：免費方案 100 GB、PRO 方案 1 TB、Team/Enterprise 方案每人 1 TB[^huggingface-pricing]。

**儲存限制（付費）**：PRO 方案最多含 10 TB 公開儲存空間。Team 方案 12 TB 基礎 + 每人 1 TB。Enterprise 方案 200 TB 基礎 + 每人 1 TB。額外公開發布儲存附加空間約 $12/TB/月。

**價格**：免費方案可用；Pro $9/月；Team $20/使用者/月；Enterprise $50/使用者/月。額外私有儲存 $18/TB/月。

**限制**：專為 ML 資料集設計；免費方案為 best-effort；Git-based 架構對儲存庫大小與檔案數量有限制（建議 <100k 檔案、<200 GB 每檔案、<500 GB 單一檔案硬上限）；大規模託管需付費方案。

---

### Kaggle Datasets

Google 旗下的資料科學競賽與公開資料集託管平台，擁有龐大的使用者社群與整合的筆記本環境[^kaggle-about]。

主要特色：
- 公開資料集託管
- 與 Kaggle Notebooks 及競賽整合
- 社群投票與討論
- 資料集版本控制
- Google Cloud 整合

**儲存限制**：公開資料集無明確硬性上限，但設計上適合競賽規模的資料集。

**價格**：公開資料集完全免費。

**限制**：僅公開存取（無私有資料集）；面向 ML/資料科學領域；資料需完全開放分享；不適合非 ML 的研究資料。

---

## 資料版本控制工具（非託管平台，但可實現資料版本控制）

### DVC (Data Version Control)

開源命令列工具，將 Git 風格的版本控制帶入資料、ML 模型與管線。本身不是託管平台——而是搭配 Git + 雲端儲存來對大型檔案進行版本控制[^dvc-about]。

**價格**：免費開源。

**限制**：需自備雲端儲存空間；需 Git 知識；命令列為主；設計上偏向 ML/資料科學工作流程。

---

### lakeFS

開源（source-available）工具，將物件儲存轉變為類似 Git 的儲存庫，具備分支、提交與合併功能，專為以 S3/GCS/Azure Blob 為基礎的資料湖設計[^lakefs-about]。

**價格**：開源核心免費，企業版需付費。

**限制**：基礎設施層級工具，非面向終端使用者的資料分享平台；需大量設定；企業功能需付費授權。

---

## 通用雲端儲存（可作為資料儲存庫使用）

AWS S3、Google Cloud Storage、Azure Blob Storage 等標準雲端物件儲存，常被資料團隊用來儲存與分享資料集。儲存空間近乎無限，按用量付費，但缺乏內建資料發現、引用與 DOI 分配功能，須搭配其他工具才能實現版本控制與資料目錄化。

---

## 比較總表

| 平台 | 免費方案 | 儲存上限 | 版本控制 | DOI | 策展 | 最適合 |
|---|---|---|---|---|---|---|
| **Zenodo** | ✅ 免費 | 50 GB/record | ✅ 有 | ✅ DataCite | ❌ 無 | 通用研究資料、免費選項 |
| **Figshare** | ✅ 免費 (20 GB) | 20 GB 免費；5 TB 付費 | ✅ 有 | ✅ DataCite | ✅ (付費方案) | 有付費授權的機構使用者 |
| **Harvard Dataverse** | ✅ 免費 | ~1 TB/研究者 | ✅ 有 | ✅ DataCite | ❌ 自助 | 社會科學、結構化 metadata |
| **Dryad** | ❌ $150/存放 | 無硬性上限 | ✅ 有 | ✅ DataCite | ✅ 專家策展 | 與期刊論文相關、需策展的資料 |
| **Mendeley Data** | ✅ 免費 | 10 GB/資料集 | ✅ 有 | ✅ DataCite | ❌ 無 | 快速免費存放、Elsevier 使用者 |
| **OSF** | ✅ 免費 | 5 GB/檔案 | ✅ (Registration) | ✅ DOI & ARK | ❌ 無 | 研究專案全生命週期管理 |
| **Hugging Face Hub** | ✅ 免費 (best-effort) | ~10 TB 公開 (PRO) | ✅ Git-based | ❌ 無 | ❌ 無 | ML/AI 資料集 |
| **Kaggle** | ✅ 免費 | 僅公開 | ✅ 基本 | ❌ 無 | ❌ 無 | ML/資料科學競賽 |
| **DVC** | ✅ 免費開源 | 無限制（自備儲存） | ✅ Git-based | ❌ 無 | ❌ 無 | ML 管線資料版本控制 |
| **lakeFS** | ✅ 開源 | 無限制（自備儲存） | ✅ Git-based | ❌ 無 | ❌ 無 | 大規模資料湖版本控制 |

---

## 簡易選擇指南

- **需要免費、無負擔的資料存放？** → **Zenodo**（50 GB）或 **Mendeley Data**（10 GB）
- **需要專業策展與品質保證？** → **Dryad**（$150/次）
- **需要機構整合與大量儲存空間？** → **Figshare for Institutions**（最高 5 TB）
- **需要結構化、學科特定的中繼資料？** → **Harvard Dataverse**
- **需要 ML/AI 資料集樞紐與社群？** → **Hugging Face Datasets Hub**
- **需要研究專案管理 + 資料分享？** → **OSF**
- **需要在自己的基礎設施上實現資料版本控制？** → **DVC** 或 **lakeFS**
- **需要尋找特定學科的儲存庫？** → **re3data.org**（收錄 3,300+ 個研究資料儲存庫的目錄）

此外，獲得美國國家衛生研究院（NIH）GREI 計畫認可的通用儲存庫包括：Dataverse、Dryad、Figshare、Mendeley Data、OSF、Vivli 與 Zenodo[^nih-grei]。

---

## Reference

[^zenodo-about]: CERN & OpenAIRE. (n.d.). About Zenodo. Retrieved 2026-09-26, from https://about.zenodo.org/
[^zenodo-limit]: Zenodo. (n.d.). What are the size limitations of Zenodo? Retrieved 2026-09-26, from https://support.zenodo.org/help/en-gb/1-upload-deposit/80-what-are-the-size-limitations-of-zenodo
[^figshare-about]: Digital Science. (n.d.). About Figshare. Retrieved 2026-09-26, from https://figshare.com/about
[^figshare-limit]: Figshare. (n.d.). File size limits and storage. Retrieved 2026-09-26, from https://info.figshare.com/user-guide/file-size-limits-and-storage/
[^figshare-plus]: Figshare. (n.d.). Figshare+. Retrieved 2026-09-26, from https://info.figshare.com/figshare-plus/
[^dataverse-about]: Harvard IQSS. (n.d.). Dataverse Software Features. Retrieved 2026-09-26, from https://dataverse.org/software-features
[^dryad-about]: Dryad. (n.d.). About Dryad. Retrieved 2026-09-26, from https://datadryad.org/about
[^mendeley-about]: Elsevier. (n.d.). Mendeley Data FAQ. Retrieved 2026-09-26, from https://data.mendeley.com/faq
[^mendeley-limit]: Elsevier. (n.d.). What are the file size limits for Mendeley Data? Retrieved 2026-09-26, from https://data.mendeley.com/faq
[^osf-about]: Center for Open Science. (n.d.). OSF Features. Retrieved 2026-09-26, from https://osf.io/features
[^huggingface-about]: Hugging Face. (n.d.). Hugging Face Datasets Hub. Retrieved 2026-09-26, from https://huggingface.co/docs/hub/datasets-overview
[^huggingface-pricing]: Hugging Face. (n.d.). Hugging Face Pricing. Retrieved 2026-09-26, from https://huggingface.co/pricing
[^kaggle-about]: Google. (n.d.). About Kaggle Datasets. Retrieved 2026-09-26, from https://www.kaggle.com/datasets
[^dvc-about]: DVC. (n.d.). Data Version Control. Retrieved 2026-09-26, from https://dvc.org/
[^lakefs-about]: lakeFS. (n.d.). lakeFS Documentation. Retrieved 2026-09-26, from https://lakefs.io/
[^nih-grei]: NIH Office of Data Science Strategy. (n.d.). Generalist Repository Ecosystem Initiative (GREI). Retrieved 2026-09-26, from https://datascience.nih.gov/data-ecosystem/generalist-repository-ecosystem-initiative