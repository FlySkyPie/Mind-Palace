# Ceph 替代方案研究：熱插拔與自動磁碟資源分配

## 概述

本報告研究 Ceph 的開源替代方案，重點評估兩項關鍵功能：**熱插拔（hot-plugging）**——能在系統運行中動態新增/移除儲存裝置而無需重啟；以及**自動磁碟資源分配（auto disk allocation）**——系統能自動發現、排程並平衡新磁碟的儲存資源。

## 評估方案總覽

### 1. Longhorn

Longhorn 是 CNCF 畢業的開源（Apache 2.0）輕量級分散式區塊儲存系統，專為 Kubernetes 設計。使用微服務架構，支援 V1（extent-based filesystem）和 V2（raw block/NVMe）兩種資料引擎。[^longhorn]

**熱插拔支援：✅ 完整**
- 可透過 Longhorn UI 或 `kubectl` 在運行中動態新增/移除節點磁碟
- Longhorn 自動偵測新磁碟的儲存細節（最大/可用容量），若磁碟適合存放資料，立即開始排程 volume 至此磁碟
- 支援 filesystem 類型（ext4/xfs）和 block 類型（NVMe/VirtIO/AIO）磁碟
- 支援線上磁碟逐出（eviction）與移除

**自動分配支援：✅ 完整**
- `StorageOverProvisioningPercentage` 和 `StorageMinimalAvailablePercentage` 全域設定控制自動排程
- `Replica Auto Balance` 提供三種模式：Disabled、Least-effort、Best-effort
- `Replica Auto Balance Disk Pressure Percentage`：當磁碟壓力達到門檻時自動在同節點的其他磁碟重建 replica
- V2 引擎支援精簡配置（thin provisioning）

**與 Ceph 比較：** 輕量許多（僅區塊儲存，無原生檔案/物件），安裝簡單（單一 Helm chart），非常適合中小型 Kubernetes 叢集。Ceph 功能更全面但複雜度遠高於 Longhorn。[^longhorn_docs]

---

### 2. Vitastor

Vitastor 是高效能開源分散式 SDS（VNPL 1.1 授權，約 60K 行 C++），架構類似 Ceph 但最佳化延遲。提供區塊（QEMU/UBLK/NBD）、檔案（VitastorFS via NFS）和物件（S3 via Zenko CloudServer）儲存。[^vitastor]

**熱插拔支援：✅ 完整**
- 使用 `vitastor-disk prepare /dev/sdX` 命令線上新增 OSD
- 支援 HDD+SSD 混合設定（journal 放 SSD，資料放 HDD）
- OSD 啟動後自動被 etcd 監控發現
- 健康磁碟移除流程：設定 `--reweight 0` → 等待 rebalance 完成 → `vitastor-disk purge`
- 已故障磁碟：直接 `vitastor-cli rm-osd` 移除

**自動分配支援：✅ 完整**
- 自動在任何數量的磁碟上分配資料，支援複寫與 erasure code（N+K）
- PG 自動平衡
- 可動態調整 recovery 參數：`no_rebalance`、`no_recovery`、`recovery_queue_depth`、`recovery_sleep_us`
- 自動調速（recovery_tune_interval）：OSD 根據客戶端負載自動調整 recovery 速度

**與 Ceph 比較：** 宣稱延遲約 0.1ms（Ceph 約 1ms+），程式碼僅 Ceph 的 6%。VNPL 授權較嚴格，社群較小。適合追求低延遲的效能敏感環境。[^vitastor_docs][^palark]

---

### 3. LINSTOR（LINBIT/DRBD）

LINSTOR 是 LINBIT 開發的開源（GPLv3）軟體定義區塊儲存管理器。使用 Controller-Satellite 模型，透過 DRBD 提供同步區塊層級複寫。支援 Kubernetes、OpenStack、Proxmox VE 等多種平台。[^linstor]

**熱插拔支援：✅ 完整**
- 支援線上後端儲存即時遷移（online live migration of back-end storage）
- 可透過 REST API 或 CLI 動態新增 storage pool（支援 LVM、LVMThin、ZFS、NVMe-oF 後端）
- DRBD version 9 支援動態資源管理

**自動分配支援：⚠️ 部份**
- 支援多層儲存池（tiered storage），可自動選擇 volume 放置位置
- 但不支援完全自動發現新插磁碟——需手動設定 LVM/ZFS 後再註冊至 LINSTOR controller
- 支援精簡配置（thin provisioning）

**與 Ceph 比較：** 延遲低、效能佳（Palark 壓力測試中表現最佳之一）。DRBD 提供極低開銷的同步複寫。整合面廣泛（VMware 到 KVM 皆可）。但僅區塊儲存，無原生物件/檔案介面。[^palark]

---

### 4. OpenEBS / Mayastor

OpenEBS 是 CNCF 專案（Apache 2.0），將 Kubernetes 節點儲存轉換為本地或複寫 persistent volume。提供多種資料引擎：Mayastor（NVMe-oF 複寫）、LVM-local、ZFS-local。[^openebs]

**熱插拔支援：⚠️ 部份**
- Mayastor 支援透過 DiskPool CR 宣告式新增區塊裝置（by-id/by-path 路徑穩定識別）
- 新增 DiskPool 需建立新的 `DiskPool` Custom Resource
- 現有 pool 無法直接熱擴容——需建立新 pool

**自動分配支援：⚠️ 部份**
- 可自動發現節點上的可用區塊裝置（`kubectl mayastor get block-devices`）
- DiskPool 需明確建立，無完全自動的 OSD 式佈建
- 一旦 pool 存在，volume 排程和 replica 放置是自動的

**與 Ceph 比較：** Kubernetes 原生程度最高（全部以 Pod 運行），Mayastor 使用 NVMe-oF + SPDK 提供極低延遲。功能面較 Ceph 少（歷史版本無 COW/snapshot）。適合高效能 Kubernetes 原生部署。[^openebs_docs]

---

### 5. GlusterFS

GlusterFS 是開源（GPLv2）分散式檔案系統，提供 POSIX 相容分散式檔案儲存。[^gluster]

**熱插拔支援：✅ 完整**
- 透過 `gluster volume add-brick` 在運行中擴充 volume
- 移除 brick 支援資料遷移（`gluster volume remove-brick`）
- 更換故障 brick 亦支援線上操作

**自動分配支援：⚠️ 部份**
- 新增 brick 後需手動觸發 rebalance（`gluster volume rebalance`）
- GlusterFS 3.6+ 支援基於權重的 rebalance（考慮 brick 大小）
- NUFA 可優先使用本地 brick，但非完全自動

**與 Ceph 比較：** 僅檔案儲存（無區塊/物件）。架構較簡單，但並發效能通常低於 Ceph。設定比 Ceph 容易，但 IOPS 密集型場景擴充性較差。[^gluster_docs]

---

### 6. MinIO / AIStor

MinIO 是高效能物件儲存系統（AGPLv3），相容 Amazon S3 API。[^minio]

**熱插拔支援：⚠️ 受限**
- 支援新增 server pool 擴充叢集，但單一磁碟熱插拔並非原生功能
- 新增容量通常透過新增節點（scale-out），非熱插拔單一磁碟

**自動分配支援：⚠️ 受限**
- 基於 erasure coding 自動分配物件，新 server pool 自動參與資料放置
- 無區塊儲存系統的「磁碟資源自動分配」概念

**與 Ceph 比較：** 僅物件儲存（無區塊/檔案原生支援）。S3 相容性優於 Ceph RGW。適合純 S3 工作負載。[^minio_docs]

---

### 7. Portworx

Portworx Enterprise 是 Pure Storage 的商業軟體定義儲存覆蓋層，提供容器化有狀態應用程式的持久儲存。[^portworx]

**熱插拔支援：✅ 完整**
- 支援 `resize-drive`（垂直擴容）和 `add-drive`（水平新增），皆為線上操作
- `add-drive` 涉及資料 restriping 但不影響服務

**自動分配支援：✅ 完整**
- AutoPilot 提供基於策略的動態儲存管理，可自動根據使用率調整
- 支援自動擴展儲存節點

**與 Ceph 比較：** 商業產品，非完全開源。Kubernetes 整合更深。部署管理比 Ceph 簡單。適用於已有 Pure Storage 生態系的企業。[^portworx_docs]

---

## 比較總表

| 方案 | 授權 | 熱插拔磁碟 | 自動分配 | 區塊 | 檔案 | 物件 | K8s 原生 |
|---|---|---|---|---|---|---|---|
| **Longhorn** | Apache 2.0 | ✅ 完整 | ✅ 完整 | ✅ | ❌ | ❌ | ✅ |
| **Vitastor** | VNPL 1.1 | ✅ 完整 | ✅ 完整 | ✅ | ✅ | ✅ | 需 CSI |
| **LINSTOR** | GPLv3 | ✅ 完整 | ⚠️ 儲存池層級 | ✅ | ❌ | ❌ | 需 CSI |
| **OpenEBS** | Apache 2.0 | ⚠️ 宣告式 CR | ⚠️ 需明確建立 pool | ✅ | ❌ | ❌ | ✅ |
| **GlusterFS** | GPLv2 | ✅ 完整 | ⚠️ 需手動 rebalance | ❌ | ✅ | ❌ | 需 CSI |
| **MinIO** | AGPLv3 | ⚠️ Server-pool 層級 | ⚠️ 物件層級 | ❌ | ❌ | ✅ | 需 Operator |
| **Portworx** | 商業 | ✅ 完整 | ✅ 完整 | ✅ | ❌ | ❌ | ✅ |

---

## 結論與建議

1. **若主要需求是熱插拔 + 自動分配，最推薦 Longhorn 或 Vitastor**
   - Longhorn：Kubernetes 環境中最無縫的體驗——新增磁碟後系統立即發現並自動排程。CNCF 畢業專案，社群活躍。
   - Vitastor：裸機 + K8s 皆可，延遲極低（約 0.1ms），提供區塊/檔案/物件三種儲存。但 VNPL 授權較嚴格。

2. **若需要混合工作負載（區塊+檔案+物件）**
   - 僅 Ceph 和 Vitastor 提供三合一解決方案。Vitastor 更快但成熟度較低。

3. **若需要企業級支援與跨平台整合**
   - LINSTOR 支援最多平台（VMware 到 KVM 到雲端），商業支援由 LINBIT 提供。

4. **若環境已純 Kubernetes**
   - Longhorn 或 OpenEBS/Mayastor（高效能 NVMe-oF 需求）是最簡單的選擇。

5. **Ceph 仍是最成熟的全功能選擇**
   - 透過 Rook 在 K8s 上部署可大幅降低操作複雜度，`useAllDevices: true` 可實現自動磁碟發現與 OSD 佈建。

---

[^longhorn]: Longhorn. (n.d.). *Longhorn Documentation - Multi-Disk Support*. Retrieved 2026-09-27, from https://longhorn.io/docs/1.12.1/nodes-and-volumes/nodes/multidisk/
[^longhorn_docs]: Longhorn. (n.d.). *Longhorn Documentation*. Retrieved 2026-09-27, from https://longhorn.io/docs/
[^vitastor]: Vitastor. (n.d.). *Vitastor - High Performance Distributed Block Storage*. Retrieved 2026-09-27, from https://vitastor.io
[^vitastor_docs]: Vitastor. (n.d.). *Vitastor Administration Guide*. Retrieved 2026-09-27, from https://vitastor.io/en/docs/usage/admin.html
[^linstor]: LINBIT. (n.d.). *LINSTOR - Software Defined Storage*. Retrieved 2026-09-27, from https://linbit.com/linstor/
[^openebs]: OpenEBS. (n.d.). *OpenEBS Documentation*. Retrieved 2026-09-27, from https://openebs.io/docs/
[^openebs_docs]: OpenEBS. (n.d.). *OpenEBS Mayastor Replicated Storage*. Retrieved 2026-09-27, from https://openebs.io/docs/concepts/data-engines/replicated-storage
[^gluster]: GlusterFS. (n.d.). *GlusterFS Administration Guide - Managing Volumes*. Retrieved 2026-09-27, from https://docs.gluster.org/en/latest/Administrator-Guide/Managing-Volumes/
[^gluster_docs]: GlusterFS. (n.d.). *GlusterFS Documentation*. Retrieved 2026-09-27, from https://docs.gluster.org/
[^minio]: MinIO. (n.d.). *MinIO Object Storage Documentation*. Retrieved 2026-09-27, from https://min.io/docs/minio/linux/index.html
[^minio_docs]: MinIO. (n.d.). *MinIO Documentation*. Retrieved 2026-09-27, from https://min.io/docs/
[^portworx]: Pure Storage. (n.d.). *Portworx Enterprise Documentation*. Retrieved 2026-09-27, from https://docs.portworx.com/portworx-enterprise/
[^portworx_docs]: Pure Storage. (n.d.). *Scale a Portworx Cluster*. Retrieved 2026-09-27, from https://docs.portworx.com/portworx-enterprise/operations/scale-portworx-cluster/expand-storage-pool
[^palark]: Palark. (2022). *Kubernetes Storage Performance: LINSTOR vs Ceph vs Mayastor vs Vitastor*. Retrieved 2026-09-27, from https://palark.com/blog/kubernetes-storage-performance-linstor-ceph-mayastor-vitastor/