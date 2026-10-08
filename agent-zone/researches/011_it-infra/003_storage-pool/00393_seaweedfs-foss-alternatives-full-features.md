# SeaweedFS FOSS 替代方案：著重功能完整性分析

## 概述

本報告旨在尋找 SeaweedFS 的自由及開源軟體（FOSS）替代方案，並特別聚焦於一個關鍵問題：**該方案的 FOSS 版本是否擁有完整功能，未被付費「Enterprise 版本」刻意限制或閹割**。

隨著 2026 年 MinIO 轉變為商業產品 AIStor，社群對於「真正的 FOSS 分散式物件儲存」的需求更為迫切。本報告逐一檢視主要替代方案，並標示哪些專案真正值得信賴。

---

## 一、已被商業化限制的專案（不建議作為 FOSS 替代方案）

### MinIO / AIStor

- MinIO 原 AGPLv3 開源儲存庫已於 2026 年 4 月 25 日**封存（archived）**，進入唯讀狀態，不再接受新功能、錯誤修正或安全性更新。[^minio-oss-archived]
- MinIO 已更名為 **AIStor**，採用專屬授權。[^aistor-pricing]
- AIStor Free 層級雖包含完整功能，但**僅限單一節點（single-node）**，無法橫向擴展。若要使用多節點分散式部署，必須付費訂閱 AIStor Enterprise Lite 或 Enterprise。[^aistor-pricing]
- **結論：FOSS 版本已死亡。不推薦。**

---

## 二、FOSS 版本功能完整的替代方案

### 1. Ceph RADOS Gateway (RGW)

| 項目 | 內容 |
|---|---|
| **授權** | LGPL-2.1 |
| **語言** | C++ |
| **狀態** | 超過 15 年生產驗證，最成熟的選項 |
| **S3 相容性** | ✅ 極佳，透過 RGW 提供完整 S3 API |

**功能完整性：✅ FOSS 版本即為完整產品，無 Enterprise Edition。**

- Ceph 完全開放原始碼，不存在社群版／企業版的功能差異。[^ceph-about]
- 同一個叢集同時提供物件（S3/RGW）、區塊（RBD）、檔案（CephFS）三種儲存介面。
- 功能涵蓋：糾刪碼（Erasure Coding）、多站點複寫（Multi-site Replication）、快取分層（Cache Tiering）、加密、S3 API、Swift API、豐富 metadata。[^ceph-docs]
- 治理由社群基金會主導，IBM、Red Hat 等多家公司參與開發。
- 商業**支援**（非功能鎖定）可從 Red Hat、Canonical 等公司取得。
- **代價**：至少需要 3 台 monitor + 3 台 OSD 共 6 台機器，營運複雜度與學習曲線較高。

---

### 2. OpenStack Swift

| 項目 | 內容 |
|---|---|
| **授權** | Apache 2.0 |
| **語言** | Python |
| **狀態** | 成熟，OpenStack 生態系一員 |
| **S3 相容性** | ✅ 透過 Swift3 middleware 提供 |

**功能完整性：✅ FOSS 版本即為完整產品，無 Enterprise Edition。**

- Swift 是 OpenStack 的物件儲存服務，由 OpenStack 基金會支援。[^openstack-swift]
- 設計從單一機器到數千台伺服器皆可橫向擴展。[^openstack-swift-docs]
- 支援多租戶（Multi-tenant）與獨立存取控制、多區域複寫（Multi-region Replication）。
- 環形一致 hash（Ring-based Consistent Hashing）架構。
- Red Hat 等公司提供商業**支援**，但所有功能均可在 FOSS 版本中獲得。[^openstack-swift]
- **代價**：需要 OpenStack 生態系相關知識，Python 為基礎技術棧。

---

### 3. Apache Ozone

| 項目 | 內容 |
|---|---|
| **授權** | Apache 2.0 |
| **語言** | Java |
| **狀態** | Apache 軟體基金會正式專案 |
| **S3 相容性** | ✅ 原生 S3 API |

**功能完整性：✅ FOSS 版本即為完整產品。** - Apache 基金會的治理模式保證了這一點。[^ozone-about]

- 由 Apache 軟體基金會治理——這從根本上排除了「企業付費功能」的可能性。[^ozone-about]
- 功能涵蓋：複寫（Replication）、糾刪碼（Erasure Coding）、Kerberos 認證、傳輸加密（TDE）、分層儲存（Tiered Storage）、Prometheus/Grafana 觀測性。
- 深度整合 Hadoop/Spark 生態系，是大數據資料湖（Data Lake）場景的頂尖選擇。
- **代價**：一般用途物件儲存非其核心場景；生態系偏向 Java/Hadoop。

---

### 4. CubeFS

| 項目 | 內容 |
|---|---|
| **授權** | Apache 2.0 |
| **語言** | C++ / Java |
| **狀態** | CNCF 畢業專案（Graduated） |
| **S3 相容性** | ✅ 支援 S3、POSIX、HDFS 三種協定 |

**功能完整性：✅ FOSS 版本即為完整產品，由 CNCF 基金會治理。**[^cubefs-gh]

- 原由騰訊（Tencent）開發，現由 CNCF（Linux 基金會旗下）治理，確保長期開放。[^cubefs-gh]
- 同一系統同時提供物件儲存（S3）、檔案儲存（POSIX）與 HDFS 三種介面。
- 支援複寫與糾刪碼、多站點部署、快取加速。
- **代價**：社群規模小於 Ceph，生產環境部署案例較少。

---

### 5. RustFS

| 項目 | 內容 |
|---|---|
| **授權** | Apache 2.0 |
| **語言** | Rust |
| **狀態** | v1.0.0 GA 已於 2026 年 9 月正式釋出 |
| **S3 相容性** | ✅ 完整 S3 API 相容性矩陣 |

**功能完整性：✅ FOSS 版本即為完整產品，無社群／企業版之分。**[^rustfs-ga]

- 100% 功能在開放原始碼版本中，Apache 2.0 授權杜絕了「毒藥條款」。[^rustfs-about]
- 功能涵蓋：糾刪碼（Reed-Solomon）、完整 S3 API（IAM、versioning、lifecycle management、multipart upload）、網頁管理介面、OIDC/SSO/LDAP/Keystone 認證、伺服器端加密（SSE）含 KMS 整合、位元腐爛防護（Bitrot Protection）、站點複寫與儲存池伸縮。
- 可直接做為 MinIO 的 drop-in 替代方案（互換二進位）。[^rustfs-blog]
- 已有 270 萬以上部署實例。[^rustfs-blog]
- **代價**：專案相對較新（1.0 GA 剛釋出），長期生態系仍在建立中。

---

### 6. libreFS（MinIO 功能保存分支）

| 項目 | 內容 |
|---|---|
| **授權** | AGPL-3.0 |
| **語言** | Go |
| **狀態** | v1 於 2026 年 4 月釋出 |
| **S3 相容性** | ✅ 100% MinIO S3 API 相容（drop-in 替代） |

**功能完整性：✅ FOSS 版本即為完整產品，且正是為了此目的而存在。**[^librefs-about]

- libreFS 是 MinIO 最後一個完全開放原始碼版本（RELEASE.2025-04-22）的**社群分叉**。[^librefs-about]
- 其核心目標正是**保留 MinIO 移入 AIStor 付費版本的所有功能**：嵌入式 WebUI、LDAP/OIDC 登入、分散式糾刪碼、預編譯二進位檔。[^librefs-about]
- 宣言：「永久採用 AGPL-3.0 授權。」
- 100% 與 MinIO 環境變數、API、管理 API 相容。
- 可做為現有 MinIO 使用者的直接替代方案。
- **代價**：AGPL-3.0 為強式 Copyleft，若散布含有 libreFS 的軟體需分享原始碼。專案歷史較短，長期維護能量待觀察。

---

### 7. Garage

| 項目 | 內容 |
|---|---|
| **授權** | AGPL-3.0 |
| **語言** | Rust |
| **狀態** | 生產可用，持續活躍開發中 |
| **S3 相容性** | ⚠️ 核心 S3 操作支援，進階功能尚未完全實作 |

**功能完整性：✅ FOSS 版本即為完整產品（無付費版本存在）。**[^garage-about]

- 一個 EU 資助（NGI、NLnet）的專案，從設計之初即無付費版本的打算。[^garage-about]
- 核心特色為**跨地理分散式**（Geo-distributed）——專門設計用於在多個高延遲的資料中心或家庭之間運作。
- 使用 CRDT-based 一致 hash、糾刪碼內建、單一二進位無依賴、SQLite 中繼資料、極低資源使用（約 200 MB RAM，可在 Raspberry Pi 上執行）。[^garage-about]
- 容忍不穩定的網路連線與異質硬體。
- **代價**：功能面刻意保持精簡——不支援 lifecycle policies、bucket versioning、無內建 WebUI。小型到中型多站點部署的最佳選擇，但功能完整性不如 Ceph 或 RustFS。

---

### 8. Zenko（by Scality）

| 項目 | 內容 |
|---|---|
| **授權** | Apache 2.0 |
| **語言** | TypeScript / Node.js |
| **狀態** | 成熟，Scality 維護 |
| **S3 相容性** | ✅ S3 API，可作為多雲後端的統一閘道 |

**功能完整性：✅ FOSS 版本即為完整產品。**[^zenko-gh]

- 多雲資料控制器（Multi-cloud Data Controller）——統一內部部署與公有雲端儲存。[^zenko-gh]
- 支援 AWS S3、Azure Blob、Google Cloud Storage 作為後端。
- Scality 有獨立的商業產品，但 Zenko 本身為完全 FOSS。
- **代價**：非獨立分散式儲存系統，而是「儲存後端調度層」。

---

## 三、不建議的專案

| 專案 | 原因 |
|---|---|
| **MinIO / AIStor** | OSS 儲存庫封存、免費版單節點限制、多節點需付費 |
| **OpenIO** | 2020 年被 OVHcloud 收購後無活躍社群開發，實質凍結[^openio-wiki] |
| **Wasabi / Backblaze B2 / Cloudian** | 私有付費服務，非 FOSS |
| **GlusterFS** | 無原生 S3 API 支援，需第三方閘道，非直接替代方案 |

---

## 四、總結對照表

| 方案 | FOSS 功能完整？ | 授權 | S3 相容性 | 最佳適用場景 |
|---|---|---|---|---|
| **Ceph RGW** | ✅ 完整 | LGPL-2.1 | ✅ 極佳 | 企業級統一儲存（物件＋區塊＋檔案），有專屬維運團隊 |
| **OpenStack Swift** | ✅ 完整 | Apache 2.0 | ✅ 良好 | 已在或熟悉 OpenStack 生態系的組織 |
| **Apache Ozone** | ✅ 完整 | Apache 2.0 | ✅ 完整 | Hadoop/Spark 大數據資料湖場景 |
| **CubeFS** | ✅ 完整 | Apache 2.0 | ✅ 完整 | 需要多協定（S3＋POSIX＋HDFS）的 CNCF 治理方案 |
| **RustFS** | ✅ 完整 | Apache 2.0 | ✅ 完整 | 追求現代 Rust 技術棧、想要 S3 API 完整相容性的新部署 |
| **libreFS** | ✅ 完整 | AGPL-3.0 | ✅ 完整（MinIO 相容） | 現有 MinIO 使用者尋找無痛替代方案 |
| **Garage** | ✅ 完整（功能精簡但非人工限制） | AGPL-3.0 | ⚠️ 核心操作 | 跨地理低資源環境、邊際運算場景 |
| **Zenko** | ✅ 完整 | Apache 2.0 | ✅ S3 閘道 | 多雲後端統一管理 |

---

## 五、結論與建議

若您的核心需求是「**FOSS 版本擁有完整功能，不被 Enterprise 版本限制**」：

1. **若可接受較高維運複雜度** → **Ceph RGW** 是最成熟、功能最完整、社群最大的選擇，無任何付費功能鎖定。
2. **若想要現代架構（Rust）與完整 S3 相容** → **RustFS** 是強力競爭者，Apache 2.0 授權，v1.0 GA 剛正式釋出。
3. **若目前正在使用 MinIO 且需要即時替代** → **libreFS** 是功能保留最完整的分叉，但請注意 AGPL 授權的影響。
4. **若僅需輕量跨地理部署** → **Garage** 是最低資源、最簡單的選項。

---

## 參考資料

[^minio-oss-archived]: MinIO. (n.d.). MinIO OSS Repository. Retrieved 2026-10-03, from https://github.com/minio/minio — 儲存庫封存公告

[^aistor-pricing]: MinIO. (n.d.). AIStor Pricing. Retrieved 2026-10-03, from https://min.io/pricing — 定價方案與功能限制說明

[^ceph-about]: Ceph. (n.d.). Ceph Storage. Retrieved 2026-10-03, from https://ceph.io/en/

[^ceph-docs]: Ceph. (n.d.). Object Storage — RADOS Gateway (S3). Retrieved 2026-10-03, from https://ceph.com/ceph-storage/object-storage/

[^openstack-swift]: OpenStack. (n.d.). Swift — Object Store. Retrieved 2026-10-03, from https://docs.openstack.org/swift/latest/

[^openstack-swift-docs]: OpenStack Foundation. (n.d.). Swift Documentation. Retrieved 2026-10-03, from https://wiki.openstack.org/wiki/Swift

[^ozone-about]: Apache Software Foundation. (n.d.). Apache Ozone. Retrieved 2026-10-03, from https://ozone.apache.org/

[^cubefs-gh]: CubeFS. (n.d.). CubeFS GitHub Repository. Retrieved 2026-10-03, from https://github.com/cubefs/cubefs

[^rustfs-about]: RustFS. (n.d.). RustFS. Retrieved 2026-10-03, from https://rustfs.com/

[^rustfs-ga]: RustFS. (2026-09). Announcing RustFS 1.0.0 GA. Retrieved 2026-10-03, from https://rustfs.com/blog/announcing-rustfs-1-0-0-ga/

[^rustfs-blog]: Elest. (n.d.). RustFS vs SeaweedFS vs Garage: Which MinIO Alternative Should You Pick? Retrieved 2026-10-03, from https://blog.elest.io/rustfs-vs-seaweedfs-vs-garage-which-minio-alternative-should-you-pick/

[^librefs-about]: libreFS. (n.d.). libreFS. Retrieved 2026-10-03, from https://librefs.org/

[^garage-about]: Garage. (n.d.). Garage — S3-Compatible Object Store. Retrieved 2026-10-03, from https://garagehq.deuxfleurs.fr/

[^openio-wiki]: Wikipedia. (n.d.). OpenIO. Retrieved 2026-10-03, from https://en.wikipedia.org/wiki/OpenIO

[^zenko-gh]: Scality. (n.d.). Zenko GitHub Repository. Retrieved 2026-10-03, from https://github.com/scality/Zenko