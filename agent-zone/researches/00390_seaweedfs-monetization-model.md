# SeaweedFS 的營利機制

## 概述

SeaweedFS 是一套以 **Apache 2.0 開源核心**為基礎的分散式檔案系統，透過 **Open-Core（開放核心）商業模式**獲利。核心功能免費，進階功能則以 **SeaweedFS Enterprise** 付費授權形式提供。專案由創辦人 **Chris Lu**（LinkedIn 頭銜為 "Seaweed Data"）自 2015 年起獨立維護，未公開揭露任何創投（VC）募資紀錄。

---

## 營利管道一覽

| 管道 | 說明 | 預估收入規模 |
|------|------|-------------|
| **Enterprise 容量授權** | 按 TB 計費的軟體授權（主要收入） | 主力收入來源 |
| **付費支援合約** | 年約制技術支援（$2,000–$85,000+/年） | 次要收入 |
| **Patreon 群眾贊助** | 每月 $10–$2,500 的會員制 | ~$463/月[^patreon] |
| **GitHub Sponsors** | 一次性或定期捐贈 | 小量，已知贊助者含 PostHog |
| **第三方代管服務** | 生態系間接收入（如 Elestio 提供代管） | 非直接收入 |

---

## 1. Enterprise 容量授權（主要收入）

這是 SeaweedFS 最主要的獲利引擎。不同於 AWS S3 等按 API 請求次數 + 流量 + 儲存三維計費的模式，SeaweedFS Enterprise 採取**純容量定價**，無 API 請求費、無出口流量費。[^pricing]

| 方案 | 價格 | 說明 |
|------|------|------|
| **Dev & Test（免費）** | $0 | 25 TB 以下無需授權，功能無閹割 |
| **Enterprise 月訂** | **$2/TB/月** | 隨用隨付，可隨時取消 |
| **Enterprise 年訂** | **$20/TB/年**（≈ $1.67/TB/月） | 比月訂節省 ~17% |
| **Enterprise 5 PB+** | 客製報價 | 大量折扣、自訂合約與發票 |

**計費細則**：授權以**使用量（含複製/抹除碼（Erasure Coding）備援開銷）** 計算。例如 10 TB 邏輯資料搭配 2 倍複製（Replication），則以 20 TB 計價。

**購買流程**：自行部署 SeaweedFS → 從 Admin UI 取得 Cluster UUID → 透過 Stripe 購入授權 → 取得授權金鑰檔。

**與公有雲儲存比較**（官方宣稱）：

| 服務 | 價格/TB/月 |
|------|-----------|
| SeaweedFS Enterprise | **$2** |
| AWS S3 | ~$23 |
| Azure Blob | ~$18 |
| Google Cloud Storage | ~$20 |

官方定位為公有雲方案約 **10 倍便宜**。[^pricing]

---

## 2. 付費支援合約

支援方案**獨立於容量授權**，需另購。[^support]

| 方案 | 年費 | 適用規模 | 回應目標 | 諮詢時數 |
|------|------|----------|---------|---------|
| **社群** | $0 | 不限 | 無保證（僅 GitHub） | 0 |
| **Essential Production** | **$2,000/年** | 100 TB 以下 | 5 個工作日 | 1 小時/月 |
| **Small Production** | **$5,000/年** | 250 TB 以下 | 3 個工作日 | 2 小時/月 |
| **Growth Production** | **$15,000/年** | 1 PB 以下 | 2–3 工作日 | 4 小時/月 |
| **Standard Production** | **$40,000/年** | 多 PB | 1 工作日 | 8 小時/月 |
| **Critical Production** | **$85,000/年** | 承載營收 | P1 1 小時內 | 12 小時/月 |
| **Mission Critical** | 客製報價 | 24/7 生產 | 24/7 P1 升級 | 32 小時/月 |
| **OEM / MSP / Reseller** | 客製 | 轉售方案 | 客製 | 客製 |

支援內容包含：私人溝通管道（Email / Slack / Zoom，視方案而定）、案件分類分流、以及**諮詢時數**（以 15 分鐘為單位，當月有效，用於架構審查、升級規劃、效能調校）。

---

## 3. Patreon 群眾贊助

Chris Lu 經營的 Patreon 頁面擁有約 **93 名會員**，月收入約 **$463 USD**。[^patreon][^patreonstats]

| 方案 | 月費 | 福利 |
|------|------|------|
| **Backer** | **$10** | 姓名列於 repo 中的 `backers.md` |
| **Generous Backer** | **$50** | 姓名置頂於 `backers.md` |
| **Gold Sponsor** | **$500** | 名稱/標誌在 README + 部署計畫審查 |
| **Platinum Sponsor** | **$2,500** | 名稱/標誌在 README 頂端 + 優先功能請求 |

---

## 4. GitHub Sponsors

SeaweedFS 在 GitHub Sponsors 上約有 **8 位贊助者**，其中公開可追蹤的企業贊助者為 **PostHog**。此項收入規模較小。[^sponsors]

---

## 5. 開原始碼與 Enterprise 的功能界線

**開源核心**（Apache 2.0）包含：

- S3 相容 API、POSIX FUSE 掛載、WebDAV、HDFS 支援
- Apache Iceberg 表格儲存
- 固定比率的抹除碼（Erasure Coding）
- gzip 壓縮
- 基本 Admin UI

**Enterprise 附加功能**（驅動授權購買的核心價值）：

- ✅ **Point-in-Time Recovery**（任一時間點還原）
- ✅ **Data Recovery**（保留期限內反刪除）
- ✅ **Self-Healing Storage**（自動修復損毀）
- ✅ **自訂抹除碼比率**（例如 20+4 = 僅 1.2× 備援開銷）
- ✅ **自動 EC Shard 修復**
- ✅ **EC Bitrot Scrub**（靜態損毀偵測）
- ✅ **zstd 壓縮**（較 gzip 好 10–30%）
- ✅ **Sealed Directories**（減少 18–58 倍元數據）
- ✅ **Admin UI OIDC 登入**（SSO）
- ✅ **Multi-Tenancy & S3 QoS**（速率限制）
- ✅ **中央指標監控**
- ✅ **優先支援**

**重要設計**：若授權到期，叢集會退回開源模式，**資料仍可存取**，不產生廠商鎖定（Vendor Lock-in）風險。[^comparison]

---

## 第三方代管生態系

雖非 SeaweedFS 的直接收入，但有第三方提供付費代管服務，擴大了專案的影響力：

- **Elestio** — 全管理 SeaweedFS as a Service，起價約 $16/月，支援自動備份、SSL、更新、監控。可部署於 Hetzner、DigitalOcean、Vultr、AWS、Linode、Scaleway 等。
- **WZ-IT** — 另一家代管服務提供者。

---

## 商業模式總結

| 面向 | 細節 |
|------|------|
| **模式** | Open-Core（Apache 2.0 開源核心 + 付費 Enterprise） |
| **創辦人** | Chris Lu（約 2015 年起單人維護） |
| **公司實體** | 推測以獨立身份營運（"Seaweed Data"） |
| **VC 資金** | 無公開募資紀錄 |
| **主要收入** | Enterprise 容量授權（$2/TB/月） |
| **次要收入** | 支援合約（$2K–$85K+/年） |
| **群眾贊助** | Patreon（~$463/月）+ GitHub Sponsors |
| **定價哲學** | 純容量制、無 API/流量費、定位為公有雲的 1/10 |
| **免費上線** | 25 TB 以下 Dev & Test 可獲得完整 Enterprise 功能（無授權金鑰） |
| **無 Lock-in** | 授權到期退回開源模式，資料不遺失 |

---

## 資料來源

[^pricing]: SeaweedFS. (n.d.). *Enterprise Pricing*. Retrieved 2026-10-03, from https://seaweedfs.com/docs/pricing/
[^support]: SeaweedFS. (n.d.). *Support Plans*. Retrieved 2026-10-03, from https://seaweedfs.com/docs/support/
[^comparison]: SeaweedFS. (n.d.). *Open Source vs Enterprise Feature Comparison*. Retrieved 2026-10-03, from https://seaweedfs.com/docs/comparison/
[^patreon]: Chris Lu. (n.d.). *SeaweedFS Patreon Page*. Retrieved 2026-10-03, from https://www.patreon.com/seaweedfs
[^patreonstats]: PatreonStats. (n.d.). *SeaweedFS Patreon Earnings Statistics*. Retrieved 2026-10-03, from https://patreonstats.com/creator/seaweedfs
[^sponsors]: ForOpenSource. (n.d.). *SeaweedFS Company Sponsors*. Retrieved 2026-10-03, from https://foropensource.com/sponsors/seaweedfs/
[^elestio]: Elestio. (n.d.). *SeaweedFS Managed Hosting Plans*. Retrieved 2026-10-03, from https://elest.io/open-source/seaweedfs/resources/plans-and-pricing