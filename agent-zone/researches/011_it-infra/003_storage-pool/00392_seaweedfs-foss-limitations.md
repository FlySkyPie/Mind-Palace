# SeaweedFS FOSS 版本的制限

## 概述

SeaweedFS 採用 Open-Core 商業模式，其 Apache 2.0 授權的開源版本（FOSS）雖然功能完整可用，但在資料保護、儲存效率、運維管理與進階容錯等面向存在顯著制限。本文從 **FOSS 版本缺乏什麼** 的角度切入，逐一檢視社群版本與 Enterprise 版之間的功能鴻溝。

---

## 一、無法使用的功能一覽

根據官方比較頁面，以下功能為 **Enterprise 獨佔**，FOSS 版本完全無法使用[^comparison]：

### 資料保護與救援

| 功能 | 作用 | FOSS |
|------|------|:----:|
| **Data Recovery（反刪除）** | 從 Admin UI 還原被刪除的檔案/物件 | ❌ |
| **Point-in-Time Recovery** | 將資料夾、Bucket 或物件倒回至任意過去秒數 | ❌ |
| **Self-Healing Storage Format** | 崩潰後自動偵測並移除損毀條目，無需人工干預 | ❌ |

### 抹除碼（Erasure Coding）進階功能

| 功能 | 作用 | FOSS |
|------|------|:----:|
| **自訂 EC 比率** | 每 Volume 可調整比率（如 20+4=1.2× 備援） | ❌ — FOSS 固定 **10+4** |
| **自動 EC Shard 修復** | 自動偵測並重建缺失/損毀的 EC 分片 | ❌ |
| **自動 EC Volume Vacuum** | 自動回收 EC Volume 中被刪除資料佔用的空間 | ❌ |
| **EC Bitrot Scrub** | 以 checksum 定期巡檢 EC 分片，捕捉靜態損毀 | ❌ |

### 儲存效率

| 功能 | 作用 | FOSS |
|------|------|:----:|
| **zstd 壓縮** | 新資料以 zstd 壓縮（比 gzip 小 10–30%，解碼更快） | ❌ — FOSS 僅有 **gzip** |
| **Sealed Directories** | 冷目錄條目壓縮打包成 Volume 塊，**減少 18–58 倍元數據** | ❌ |
| **Remote Volume Vacuum** | 在雲端分層 Volume 中遠端回收被刪除位元組 | ❌ |

### 運維管理

| 功能 | 作用 | FOSS |
|------|------|:----:|
| **Admin UI OIDC 登入** | 透過 SSO（Keycloak、Okta、Azure AD 等）登入 | ❌ |
| **Admin Audit Log** | 記錄所有管理階段（誰做了什麼、結果） | ❌ |
| **Central Metrics Dashboard** | 全節點即時 Prometheus 儀表板（健康、吞吐、延遲、錯誤） | ❌ — FOSS 僅提供單節點指標 |
| **Multi-Tenancy & S3 QoS** | 租戶隔離與 S3 請求速率限制 | ❌ |

---

## 二、EC 的關鍵制限

FOSS 雖然支援抹除碼（Erasure Coding），但制限顯著[^ec][^zaira]：

| 面向 | FOSS | Enterprise |
|------|:----:|:----------:|
| **EC 比率** | **固定 10+4** Reed-Solomon | 可自訂（如 20+4、8+4、6+3） |
| **儲存備援開銷** | ~1.4×（10 資料 + 4 同位 = 14 單元） | 可低至 ~1.2×（20+4 = 24 單元） |
| **容錯能力** | 承受 4 顆同時磁碟故障（相同） | 相同或可調低/調高 |
| **修復** | 人工手動 | **自動偵測並重建** |
| **空間回收** | 人工手動 | **自動壓縮**刪除資料後的 EC Volume |
| **Bitrot 偵測** | 無 | **EC Bitrot Scrub** 定期檢查 |

**影響**：若需以 20+4 等低備援比部署，FOSS 無法實現。若 EC 分片損毀，FOSS 需運維人員手動介入修復。

---

## 三、容量限制 — 常見誤解澄清

FOSS 版本**無技術性的容量上限**，儲存多少資料僅受限於硬體。Chris Lu 在 GitHub 討論中明確表示：*「正確。[開源版本沒有技術性儲存容量限制。]」*[^discuss8162]

然而官方提供的免費 **Dev & Test** 方案（25 TB 以下）與 FOSS 版本是不同概念：

| 版本 | 容量限制 | 適用場景 |
|------|---------|---------|
| **FOSS（Apache 2.0 編譯）** | **無限制** | 任何規模，但遺失 Enterprise 功能 |
| **Enterprise Dev & Test** | 25 TB 以下免費 | 開發測試用，具完整 Enterprise 功能但無商業使用權 |
| **Enterprise 生產** | 依授權 TB 數 | 生產環境必購授權 |

**總結**：FOSS 無容量上限，但取得 Enterprise 功能必需要授權金鑰。

---

## 四、FOSS 的二進位制與 Enterprise 不相容

這是一個技術層面的關鍵制限[^discuss8162][^discuss8213]：

- **二進位制不同** — FOSS 與 Enterprise 的可執行檔不同，同一叢集中不可混合兩者節點。
- **儲存格式不同** — Enterprise 使用不同的儲存格式，FOSS Volume Server 無法直接載入 Enterprise 寫入的資料。
- **降級困難** — 有使用者回報：Enterprise 轉回 FOSS 並非順暢，FOSS Volume Server 會拒絕啟動已有 Enterprise 資料的 Volume。
- **官方降級保障**：Enterprise 授權到期後會**自動退回開源模式**，資料仍可存取，但 Self-Healing 等功能停用。

**實質影響**：一旦使用 Enterprise，無法輕鬆回到 FOSS；評估時需審慎決定。

---

## 五、FOSS 仍然包含的功能

為求平衡，FOSS 版本的基礎能力仍相當完整[^comparison]：

- ✅ 完整 S3 API（73 項 Bucket/Object 操作 + 36 S3 Tables + 39 IAM + 5 STS）
- ✅ POSIX FUSE 掛載 + Kernel 掛載
- ✅ WebDAV、SFTP、HDFS 相容
- ✅ Apache Iceberg REST Catalog 與 Table Maintenance
- ✅ 抹除碼（固定 10+4 Reed-Solomon）
- ✅ 複製（Rack/DataCenter 感知放置碼）
- ✅ 分層儲存與雲端分層（Cloud Tiering）
- ✅ Active-Active 跨叢集複製
- ✅ S3 Object Versioning、Lifecycle Rules、CORS、SSE 加密
- ✅ IAM、Bucket Policy、速率限制（基本）
- ✅ Admin UI（不含 OIDC SSO 或中央儀表板）
- ✅ Prometheus 指標（單節點，非聚合儀表板）
- ✅ Filer 元資料儲存（PostgreSQL、Redis、LevelDB 等）
- ✅ Kubernetes Helm Chart、CSI Driver、Operator

---

## 六、其他社群觀察

- **Derails 部落格（2025-12）**：*「Apache 2.0 核心，你想做什麼都行。但 Enterprise 功能（EC、跨機房複製、self-healing）需要他們的公司授權。首 25 TB 免費。之後 $1/TB/月（當時價格，現為 $2）。分割很聰明：愛好者得到自由，企業為可靠付費。」*[^derails]
- **OffshoreServerHosting（2026）**：*「企業功能較少（無原生 Multi-Tenancy、加密儲存選項有限）。部署比 MinIO 複雜，需協調多個 Process。」*[^osh]
- **Zaira Labs（2026）**：*「開源版支援固定 10+4 Reed-Solomon EC；可自訂比率需 Enterprise 版。」*[^zaira]
- **IMTI（2026）**：一份 MinIO 遷移至 SeaweedFS 的實際案例，指出乾淨的 Apache 2.0 授權是關鍵考量，同時確認 self-healing、data recovery、PITR 為 Enterprise only。[^imti]

---

## 七、總結對照

| 面向 | FOSS（Apache 2.0） | Enterprise |
|------|:-----------------:|:----------:|
| **授權** | Apache 2.0，永久免費 | 專屬授權，按 TB 計費 |
| **容量上限** | 無（僅受硬體限制） | Dev & Test 25 TB 以下免費；生產付費 |
| **S3 / FUSE / HDFS / Iceberg** | ✅ 完整 | ✅ 相同 |
| **抹除碼** | ✅ 固定 10+4 RS 僅 | ✅ 自訂比率 + 自動修復/Vacuum/Bitrot |
| **壓縮** | ✅ gzip | ✅ zstd（小 10–30%，更快） |
| **Data Recovery（反刪除）** | ❌ | ✅ |
| **Point-in-Time Recovery** | ❌ | ✅ |
| **Self-Healing Storage** | ❌ | ✅ |
| **Sealed Directories** | ❌ | ✅（減少 18–58× 元數據） |
| **Admin UI OIDC / Audit Log** | ❌ | ✅ |
| **中央指標儀表板** | ❌ | ✅ |
| **Multi-Tenancy / S3 QoS** | ❌ | ✅ |
| **升級相容性** | → Enterprise 100% 相容 | 套入授權金鑰即可 |
| **降級相容性** | N/A | ⚠️ 儲存格式不同，降級困難 |
| **支援** | 社群（GitHub、Slack、Telegram） | + 私管、承諾回應時間 |
| **授權到期** | N/A | ✅ 自動退回 FOSS 模式 |

---

## 資料來源

[^comparison]: SeaweedFS. (n.d.). *Open Source vs Enterprise Feature Comparison*. Retrieved 2026-10-03, from https://seaweedfs.com/docs/comparison/
[^pricing]: SeaweedFS. (n.d.). *Enterprise Pricing*. Retrieved 2026-10-03, from https://seaweedfs.com/docs/pricing/
[^discuss8162]: SeaweedFS GitHub Discussion #8162. *Chris Lu confirms no OSS capacity limit, 100% compatible migration, binary difference*. Retrieved 2026-10-03, from https://github.com/seaweedfs/seaweedfs/discussions/8162
[^discuss8213]: SeaweedFS GitHub Discussion #8213. *Chris Lu confirms different storage format; user reports OSS refuses enterprise data*. Retrieved 2026-10-03, from https://github.com/seaweedfs/seaweedfs/discussions/8213
[^ec]: SeaweedFS Source Code. (n.d.). *Erasure Coding — Open-source SeaweedFS always uses fixed 10+4 layout*. Retrieved 2026-10-03, from https://pkg.go.dev/github.com/seaweedfs/seaweedfs/weed/storage/erasure_coding
[^zaira]: Zaira Labs. (2026). *SeaweedFS Guide — Open-source edition supports fixed 10+4 RS EC; customizable ratios are enterprise-only*. Retrieved 2026-10-03, from https://zairalabs.ai/guide/tools/seaweedfs/
[^derails]: Derails Blog. (2025-12). *MinIO vs RustFS vs SeaweedFS — Storage Wars*. Retrieved 2026-10-03, from https://www.derails.dev/blog/minio-rustfs-seaweedfs-storage-wars/
[^osh]: OffshoreServerHosting. (2026). *Self-Hosted Object Storage 2026 — SeaweedFS, Garage, MinIO Replacement*. Retrieved 2026-10-03, from https://www.offshoreserverhosting.com/blog/self-hosted-object-storage-2026-seaweedfs-garage-minio-replacement/
[^imti]: IMTI. (2026). *SeaweedFS Object Storage — Real-world deployment experience*. Retrieved 2026-10-03, from https://imti.co/seaweedfs-object-storage/