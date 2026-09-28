# Ceph 開源替代方案調查：熱插拔與自動磁碟資源分配

## 摘要

本報告調查 Ceph 分散式儲存系統的 FOSS（自由及開源軟體）替代方案，重點評估兩項關鍵功能：**熱插拔**（Hot plugging，即在不中斷服務的情況下動態新增/移除儲存裝置）與**自動磁碟資源分配**（Auto allocate disk resource，即資料在儲存叢集中自動分佈與平衡）。調查結果顯示，**SeaweedFS** 與 **MooseFS** 為最符合需求的替代方案，前者以物件/檔案混合儲存著稱，後者則提供成熟的 POSIX 分散式檔案系統。

---

## 1. 研究背景與評估標準

Ceph 是廣泛使用的開源分散式儲存系統，支援區塊（Block）、物件（Object）與檔案（File）三種介面，其 CRUSH 演算法實現了自動化的資料分佈與自我修復[^ceph-wiki]。然而 Ceph 的部署與維運複雜度較高，促使使用者尋找替代方案。

本報告評估的關鍵功能：

- **熱插拔**：在不重啟服務或中斷 I/O 的情況下，動態新增、移除或更換實體或邏輯儲存裝置。
- **自動磁碟資源分配**：當新儲存資源加入叢集時，系統能自動將資料均勻分佈到所有可用儲存節點，無需手動干預。

次要評估維度包括：架構類型（區塊/物件/檔案）、成熟度、Kubernetes 原生支援、專案活躍度。

---

## 2. 替代方案詳細比較

### 2.1 SeaweedFS

SeaweedFS 是一個高度可擴展的分散式**物件、檔案與表格儲存**系統，可同時提供 S3 相容物件儲存、POSIX 檔案系統（透過 FUSE）與 Iceberg 表格儲存[^seaweedfs-readme]。

- **熱插拔**：✅ **完整支援** — 官方文檔明確指出「Adding a server adds capacity with no data reshuffle」，只需啟動新的 Volume Server 並指向 Master 即可。新增容量**完全不需要重新平衡資料**，這在分散式儲存系統中極為罕見[^seaweedfs-readme]。
- **自動資源分配**：✅ **完整支援** — Master 節點會自動將寫入指派到任何可寫入的 Volume。若某個 Volume 寫入失敗，會自動選擇其他 Volume。新增 Volume 的過程極度簡化[^seaweedfs-readme]。
- **成熟度**：非常成熟 — 35k+ GitHub stars，活躍開發中（2014 年至今）。
- **K8s 支援**：可透過 Helm Chart / CSI 驅動部署。
- **限制**：無原生區塊儲存介面；Master 為集中式節點（但有備援機制）。

### 2.2 MooseFS

MooseFS 是一個容錯的分散式 POSIX 檔案系統，將資料分散在多個通用伺服器上，提供統一的命名空間[^moosefs-about]。

- **熱插拔**：✅ **完整支援** — 可動態新增 Chunkserver 與磁碟，系統在線期間可進行滾動升級與硬體更換[^moosefs-features]。
- **自動資源分配**：⚠️ **部分支援** — 資料會自動冗餘分佈到儲存節點，但複製因子（Goal）是以檔案/目錄為單位手動設定，而非 CRUSH 式全域自動雜湊分佈[^moosefs-community-pro]。
- **成熟度**：非常成熟 — 2005 年起生產使用，最新穩定版 v4.59.2（2026 年 5 月），活躍開發中[^moosefs-releases]。
- **K8s 支援**：無原生支援，需手動部署。
- **限制**：僅 POSIX 檔案介面；Community Edition 的糾刪碼功能受限（僅 n=1）[^moosefs-community-pro]。

### 2.3 Longhorn

Longhorn 是一個輕量級的**區塊儲存**系統，專為 Kubernetes 設計，採用微服務架構——每個 Volume 對應一個儲存控制器（Engine）[^longhorn-docs]。

- **熱插拔**：✅ **支援** — 可動態新增/移除儲存節點與磁碟，Replica 可在不中斷服務的情況下新增/移除[^longhorn-docs]。
- **自動資源分配**：⚠️ **部分支援** — Replica 會根據可用磁碟空間與節點標籤進行排程，但使用者需手動設定 Replica 數量與排程規則。不像 Ceph 的 CRUSH 演算法進行全域自動資料分佈[^longhorn-docs]。
- **成熟度**：CNCF Incubating 專案，穩定可用。
- **K8s 支援**：✅ **原生支援**。
- **限制**：僅區塊儲存（iSCSI）；無原生物件或 POSIX 檔案介面；每個 Volume 有獨立控制器（故障域較小）。

### 2.4 OpenEBS (Mayastor)

OpenEBS 是 Kubernetes 原生的容器儲存解決方案，提供 Local（單節點）與 Replicated（多節點，透過 NVMe-oF）持久化 Volume[^openebs-docs]。

- **熱插拔**：✅ **支援** — Kubernetes 原生設計意味著節點與磁碟可透過標準 K8s 操作動態新增[^openebs-docs]。
- **自動資源分配**：⚠️ **部分支援** — Replicated Volume 會將資料同步複製到固定節點，但無 CRUSH 式的全域自動資料分佈[^openebs-docs]。
- **成熟度**：CNCF Sandbox 專案，Mayastor 引擎穩定。
- **K8s 支援**：✅ **原生支援**。
- **限制**：僅區塊儲存（NVMe-oF）；K8s-only；無物件/檔案介面。

### 2.5 BeeGFS

BeeGFS 是一個高效能並行叢集檔案系統，專為 HPC 與 AI 工作負載設計，Metadata 與 Storage 可獨立擴展[^beegfs-docs]。

- **熱插拔**：✅ **支援** — 官方文檔宣稱「Add capacity exactly where your workload demands it, without downtime, without rearchitecting」[^beegfs-docs]。
- **自動資源分配**：⚠️ **部分支援** — Capacity Pool 會自動平衡可用空間，但資料放置是以 Pool 為基礎，而非自動雜湊分佈[^beegfs-docs]。
- **成熟度**：非常成熟，HPCwire 獎項獲獎者，用於 TOP500/IO500 排名系統。
- **K8s 支援**：無原生支援。
- **限制**：僅 POSIX 檔案系統；**Server 端為專有軟體**（僅 Client 為 GPLv2）。

### 2.6 LINSTOR + DRBD

LINSTOR 是開源的軟體定義儲存**管理系統**，通常用於管理 DRBD 複製區塊儲存資源[^linstor-docs]。

- **熱插拔**：✅ **支援** — LINSTOR Controller 可動態管理跨節點的儲存資源生命週期[^linstor-docs]。
- **自動資源分配**：⚠️ **部分支援** — Controller 根據設定策略進行資源放置，但不會自動重新平衡資料。DRBD 負責指定節點間的同步複製[^linstor-docs]。
- **成熟度**：成熟，由 LINBIT 積極開發。
- **K8s 支援**：可透過 CSI 驅動整合。
- **限制**：僅區塊儲存；無物件/檔案介面；DRBD 為核心層級複製，無 CRUSH。

### 2.7 GlusterFS

GlusterFS 曾為 Red Hat 主推的擴展式 NAS 檔案系統，但目前**已進入維護期**，Red Hat 建議使用者遷移至 Ceph 或其他解決方案[^glusterfs-eol]。

- **熱插拔**：⚠️ 可動態新增 Brick，但新增容量後需手動執行 Rebalance，可能造成服務中斷[^glusterfs-docs]。
- **自動資源分配**：⚠️ DHT（Distributed Hash Table）使用一致性雜湊進行檔案放置，但新增/移除 Brick 會改變雜湊範圍，需重新平衡[^glusterfs-docs]。
- **成熟度**：EOL（End of Life）—— Red Hat Gluster Storage 已於 2024-12-31 終止支援[^glusterfs-eol]。
- **限制**：不建議新部署使用。

### 2.8 LizardFS

LizardFS 為 MooseFS 的 GPLv3 分支，設計上以動態磁碟/伺服器新增為核心特色[^lizardfs-wiki]。

- **熱插拔**：✅ **完整支援** — 官方文檔指出「the file system is designed so that it is possible to add more disks and servers 'on the fly,' without the need for any server reboots or shutdowns」[^lizardfs-wiki]。
- **自動資源分配**：✅ **支援** — 自動資料複製與重新平衡。
- **成熟度**：**專案停滯** — 最後穩定版為 v3.12.0（2017 年 12 月）[^lizardfs-wiki]。
- **限制**：專案目前已無活躍開發，不建議新專案採用。

---

## 3. 綜合比較表

| 系統 | 類型 | 成熟度 | 熱插拔 | 自動分配 | K8s 原生 | 專案狀態 |
|---|---|---|---|---|---|---|
| **SeaweedFS** | 物件+檔案 | 非常成熟 | ✅ 完整 | ✅ 完整 | Via Helm | ✅ 活躍 |
| **MooseFS** | 檔案(POSIX) | 非常成熟 | ✅ 完整 | ⚠️ 部分 | ❌ 無 | ✅ 活躍 |
| **Longhorn** | 區塊 | CNCF Incubating | ✅ 支援 | ⚠️ 部分 | ✅ 原生 | ✅ 活躍 |
| **OpenEBS** | 區塊(NVMe-oF) | CNCF Sandbox | ✅ 支援 | ⚠️ 部分 | ✅ 原生 | ✅ 活躍 |
| **BeeGFS** | 檔案(POSIX) | 非常成熟 | ✅ 支援 | ⚠️ 部分 | ❌ 無 | ✅ 活躍(Server 專有) |
| **LINSTOR+DRBD** | 區塊 | 成熟 | ✅ 支援 | ⚠️ 部分 | Via CSI | ✅ 活躍 |
| **GlusterFS** | 檔案(POSIX) | EOL | ⚠️ 需手動 | ⚠️ 需手動 | Via CSI | ❌ EOL |
| **LizardFS** | 檔案(POSIX) | 停滯 | ✅ 完整 | ✅ 支援 | ❌ 無 | ❌ 停滯 |

---

## 4. 建議

### 4.1 若兩項關鍵功能（熱插拔 + 自動分配）為最高優先

**SeaweedFS** 為最佳選擇。其「新增 Server 不觸發資料重新洗牌」的設計在分散式儲存中極為獨特，且同時支援 S3 物件儲存與 POSIX 檔案系統，適合需要混合介面的場景。

### 4.2 若需純 POSIX 檔案系統且專案活躍度重要

**MooseFS** 是最活躍的開選項（2026 年仍有新版發布），熱插拔支援完整，但自動資源分配需透過 Goal 機制手動設定。

### 4.3 若為 Kubernetes 原生區塊儲存場景

**Longhorn** 與 **OpenEBS** 皆為優秀選擇，熱插拔透過 K8s 原生機制實現，但自動資源分配均不如 SeaweedFS 或 Ceph 完整。

### 4.4 若可接受 Server 端專有軟體

**BeeGFS** 提供優秀的 HPC 級熱插拔能力，但 Server 非完全開源。

### 4.5 不建議採用的專案

- **GlusterFS**：已 EOL，不適合新部署。
- **LizardFS**：專案停滯（2017 年後無新版）。
- **MinIO**：原開源版本已停止開發[^seaweedfs-readme]。

---

## 5. 參考資料

[^ceph-wiki]: Wikipedia. (n.d.). *Ceph (software)*. Retrieved 2026-09-26, from https://en.wikipedia.org/wiki/Ceph_(software)
[^seaweedfs-readme]: SeaweedFS. (n.d.). *SeaweedFS README — Philosophy*. Retrieved 2026-09-26, from https://github.com/seaweedfs/seaweedfs
[^moosefs-about]: MooseFS. (n.d.). *What is MooseFS*. Retrieved 2026-09-26, from https://moosefs.com/what-is-moosefs.html
[^moosefs-features]: MooseFS. (n.d.). *Advantages and Features*. Retrieved 2026-09-26, from https://moosefs.com/advantages-and-features.html
[^moosefs-community-pro]: MooseFS. (n.d.). *Community vs Pro Edition*. Retrieved 2026-09-26, from https://moosefs.com/pro-vs-community.html
[^moosefs-releases]: MooseFS. (n.d.). *MooseFS Releases*. Retrieved 2026-09-26, from https://github.com/moosefs/moosefs/releases
[^longhorn-docs]: Longhorn. (n.d.). *What is Longhorn*. Retrieved 2026-09-26, from https://longhorn.io/docs/1.6.0/what-is-longhorn/
[^openebs-docs]: OpenEBS. (n.d.). *OpenEBS Architecture*. Retrieved 2026-09-26, from https://openebs.io/docs/concepts/architecture
[^beegfs-docs]: BeeGFS. (n.d.). *BeeGFS Architecture Overview*. Retrieved 2026-09-26, from https://doc.beegfs.io/latest/architecture/overview.html
[^linstor-docs]: LINBIT. (n.d.). *LINSTOR Overview*. Retrieved 2026-09-26, from https://linbit.com/linstor/
[^glusterfs-docs]: GlusterFS. (n.d.). *GlusterFS Architecture*. Retrieved 2026-09-26, from https://docs.gluster.org/en/latest/Administrator-Guide/overview/
[^glusterfs-eol]: Red Hat. (2022). *Red Hat Gluster Storage Life Cycle*. Retrieved 2026-09-26, from https://access.redhat.com/support/policy/updates/gluster
[^lizardfs-wiki]: Wikipedia. (n.d.). *LizardFS*. Retrieved 2026-09-26, from https://en.wikipedia.org/wiki/LizardFS