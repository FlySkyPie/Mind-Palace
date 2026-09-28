# FOSS 軟體定義儲存方案：JBOD 熱插拔與自動磁碟資源分配比較

## 摘要

本報告針對自由開源（FOSS）軟體定義儲存（Software Defined Storage, SDS）方案進行調查，聚焦於 **JBOD（Just a Bunch of Disks）環境下的硬碟熱插拔支援** 與 **自動磁碟資源分配** 能力。核心評估標準為：從 JBOD 新增或移除硬碟所需的手續與管理成本越低越好。

## 評估方案

以下八個主流 FOSS SDS 方案納入比較：Ceph、GlusterFS、LINSTOR、SeaweedFS、OpenEBS、Longhorn、MooseFS、MinIO。

## 比較總表

| 方案 | 授權條款 | JBOD 熱插拔支援度 | 自動磁碟分配 | 資料重新平衡 | 最小介入成本 |
|------|----------|-------------------|-------------|-------------|-------------|
| **SeaweedFS** | Apache 2.0 | 優 — 啟動新 volume server 指向新磁碟目錄即可 | ⚠️ 無自動平衡，新寫入自動使用新容量 | 手動：`volume.balance -force` | **極低** — 一條命令完成 |
| **MooseFS** | GPLv2 | 優 — 將掛載點加入 `mfshdd.cfg` 後重啟 chunkserver | ✅ 自動分配新區塊至可用磁碟 | 自動（基於冗餘目標） | 低 — 編輯設定檔 + 重啟服務 |
| **Ceph** | LGPL / Apache | 部分 — OSD 可線上增刪，但需多步驟手動操作（`ceph osd create`、`mkfs`、mount、CRUSH map） | ✅ CRUSH 演算法自動重新分布 | 自動（CRUSH） | 中 — 需手動準備每個 OSD |
| **GlusterFS** | GPLv2 + LGPLv3 | 部分 — brick 可線上增刪，但需手動指定路徑與觸發平衡 | ⚠️ 需手動執行 `gluster volume rebalance` | 手動：`rebalance` 命令 | 中 — 需手動新增 brick + 觸發平衡 |
| **LINSTOR** | GPLv2 / Apache | 中等 — 需先透過 LVM/ZFS 建立 storage pool；支援一 pool 對一磁碟的熱抽換模型 | ✅ 透過 Resource Groups 自動分配 | 自動（resource groups + DRBD） | 中 — 需先建立 LVM/ZFS pool |
| **OpenEBS** | Apache 2.0 | 部分 — K8s 原生，磁碟由 K8s node discovery 管理 | ✅ K8s scheduler 自動排程 | 自動（K8s scheduler） | 低（需 K8s 環境） |
| **Longhorn** | Apache 2.0 | 部分 — K8s 原生，透過 UI/CLI 管理磁碟 | ✅ 自動 replica 排程 | 自動 | 低（需 K8s 環境） |
| **MinIO** | AGPLv3 | 差 — 固定 erasure set 結構，容量以整個 pool 為單位成長 | ❌ 無 | 不適用 | 高 — 已停止開發（2026 年 4 月） |

## 詳細分析

### 1. SeaweedFS — 最小介入成本首選

SeaweedFS 的設計使新增磁碟極為簡單。其 volume server 可透過 `-dir` 參數指定多個目錄（每目錄對應一顆磁碟），啟動新 volume server 指向新磁碟後，master 節點立即透過 heartbeat 辨識新容量。[^seaweed-readme]

**新增磁碟流程：**

```bash
# 方案 A：在現有 volume server 追加 -dir 參數（需重啟）
weed volume -dir=/data/disk1,/mnt/new-disk -master=<master>:9333 -max=0,0

# 方案 B：為每顆磁碟啟動獨立 volume server（推薦用於熱插拔）
weed volume -dir=/mnt/new-disk -master=<master>:9333 -port=8082 -max=0
```

關鍵特性：**不自動重新平衡既有資料**，但新寫入會自動分配到新磁碟。官方文件明確指出：「Adding a server adds capacity with no data reshuffle」。[^seaweed-scale]

如需重新平衡，可透過 `weed shell` 手動執行：

```bash
echo "lock; volume.balance -force; unlock" | weed shell
```

SeaweedFS 亦提供 maintenance worker 可設定排程自動執行平衡、清理等維護任務。[^seaweed-volume]

### 2. MooseFS — JBOD 原生設計

MooseFS 是唯一 **明確以 JBOD + XFS 為設計目標** 的方案。官方最佳實踐文件指出：「Give MooseFS the disks directly — one filesystem per disk, formatted XFS, no RAID underneath. MooseFS detects and isolates a failing disk.」[^moosefs-best]

**新增磁碟流程：**

```bash
# 1. 格式化（XFS）
mkfs.xfs /dev/sdb1
mkdir /mnt/mfschunks3
mount /mnt/mfschunks3
chown mfs:mfs /mnt/mfschunks3
chmod 770 /mnt/mfschunks3

# 2. 加入設定檔 /etc/mfs/mfshdd.cfg
echo "/mnt/mfschunks3" >> /etc/mfs/mfshdd.cfg

# 3. 重啟 chunkserver
systemctl restart moosefs-chunkserver.service
```

重啟後，新容量立即被 master 辨識並用於新區塊寫入。[^moosefs-install]

MooseFS 隨發行版附贈 systemd unit 檔案，安裝套件時自動設定。[^moosefs-systemd]

### 3. Ceph — 最成熟的自動分布

Ceph 的 CRUSH 演算法提供最完善的資料自動重新平衡機制。當 OSD 加入或移除時，資料會自動遷移。[^ceph-osd]

**新增 OSD 流程：** 需手動執行 `ceph osd create`、格式化磁碟 (`mkfs.xfs`)、掛載、加入 CRUSH map 等步驟。[^ceph-add-osd] 流程較多步驟，且 CRUSH 拓樸變更可能觸發大規模資料遷移。

Ceph 官方建議使用均勻硬體配置：「Ceph works best with uniform hardware across pools。」[^ceph-hardware]

### 4. LINSTOR — 政策驅動的自動分配

LINSTOR 透過 **Resource Groups** 實現政策驅動的磁碟資源自動分配。可設定 `--place-count`、`--replicas-on-different` 等約束條件，由 Autoplacer 根據加權策略（MaxFreeSpace、MinRscCount、MaxThroughput 等）自動選擇存放位置。[^linstor-rg]

```bash
linstor resource-group create my_group --storage-pool pool_ssd --place-count 2
linstor volume-group create my_group
linstor resource-group spawn-resources my_group my_res 20G
```

LINSTOR 支援 **一 storage pool 對一實體磁碟**的熱抽換模型，旨在將故障域限制在單一磁碟。[^linstor-pool] 但 LINSTOR 不直接使用原始磁碟，需透過 LVM 或 ZFS 層。[^linstor-providers]

### 5. OpenEBS / Longhorn — Kubernetes 原生

兩者均為 Kubernetes 原生方案。磁碟透過 K8s node discovery 自動管理，K8s scheduler 負責 volume 排程。適合已有 K8s 基礎設施的環境。[^openebs-docs][^longhorn-docs]

### 6. GlusterFS — 維護模式

Brick 可線上增刪，但每增減一個 brick 都需手動指定路徑，且複本式 volume 的 brick 數量必須為複本數的倍數。GlusterFS 目前處於維護模式，開發活躍度低。[^glusterfs-status]

### 7. MinIO — 已停止開發

MinIO 的固定 erasure set 結構（每 set 2-16 顆硬碟）使其難以動態應對 JBOD 熱插拔。且已於 2026 年 4 月停止開發。[^minio-eol]

## 自動化整合方向

所有方案仍無法達到「插入硬碟即自動辨識使用」的完全零介入理想。但可透過以下方式補強：

1. **udev 規則**：偵測新磁碟插入後自動執行掛載與設定
2. **systemd mount units**：磁碟掛載穩定後自動啟動對應服務
3. **Ansible / SaltStack 等組態管理工具**：批次管理多節點磁碟設定

## 建議摘要

| 使用情境 | 推薦方案 | 理由 |
|----------|----------|------|
| **最低介入成本的 JBOD 熱插拔** | SeaweedFS | 一條命令啟動 volume server，新容量立即可用 |
| **JBOD 原生設計 + POSIX 檔案系統** | MooseFS | 明確以 JBOD+XFS 為設計目標，內建 systemd 支援 |
| **自動資料重新平衡** | Ceph | CRUSH 演算法提供最完善的自動資料分布 |
| **Kubernetes 原生環境** | Longhorn 或 OpenEBS | K8s scheduler 自動管理磁碟排程 |
| **區塊儲存 + DRBD 高可用** | LINSTOR | Resource Groups 政策驅動自動分配，內建 DRBD 複寫 |
| **小檔案高效儲存** | SeaweedFS | 每個檔案僅 ~40 bytes 中繼資料開銷 |

## 結論

若以「從 JBOD 新增/移除硬碟所需手續與管理成本越低越好」為唯一評量標準，**SeaweedFS** 以「啟動 volume server 指向新磁碟目錄即可」的極簡流程位居首位。**MooseFS** 以 JBOD+XFS 原生設計與內建 systemd 支援緊追在後。兩者均不自動重新平衡既有資料，但新容量皆能立即使用。

若要同時滿足自動資料重新平衡，**Ceph** 的 CRUSH 演算法最為成熟，但 OSD 加入流程步驟較多。**LINSTOR** 的 Resource Groups 提供政策驅動的替代方案，但需額外的 LVM/ZFS 層。

## 參考資料

[^seaweed-readme]: SeaweedFS. (n.d.). SeaweedFS README — Scale Out. Retrieved 2026-09-27, from https://github.com/seaweedfs/seaweedfs
[^seaweed-scale]: SeaweedFS. (n.d.). Production Setup — Adding Volume Servers. Retrieved 2026-09-27, from https://github.com/seaweedfs/seaweedfs/wiki/Production-Setup
[^seaweed-volume]: SeaweedFS. (n.d.). Volume Management. Retrieved 2026-09-27, from https://github.com/seaweedfs/seaweedfs/wiki/Volume-Management
[^seaweed-shell]: SeaweedFS. (n.d.). Weed Shell. Retrieved 2026-09-27, from https://github.com/seaweedfs/seaweedfs/wiki/weed-shell
[^seaweed-systemd]: SeaweedFS. (n.d.). Server Startup via Systemd. Retrieved 2026-09-27, from https://github.com/seaweedfs/seaweedfs/wiki/Server-Startup-via-Systemd
[^seaweed-vs-moosefs]: SeaweedFS. (n.d.). SeaweedFS Compared to MooseFS. Retrieved 2026-09-27, from https://github.com/seaweedfs/seaweedfs
[^moosefs-best]: MooseFS. (n.d.). Chunkserver Hardware Requirements — Best Practices. Retrieved 2026-09-27, from https://moosefs.com/blog/chunkserver-hardware-requirements.html
[^moosefs-install]: MooseFS. (n.d.). How to Install MooseFS. Retrieved 2026-09-27, from https://moosefs.com/blog/how-install-moosefs.html
[^moosefs-systemd]: MooseFS. (n.d.). systemd directory. Retrieved 2026-09-27, from https://github.com/moosefs/moosefs/tree/master/systemd
[^moosefs-readme]: MooseFS. (n.d.). GitHub README. Retrieved 2026-09-27, from https://github.com/moosefs/moosefs
[^ceph-osd]: Ceph. (n.d.). RADOS Operations — Add or Remove OSDs. Retrieved 2026-09-27, from https://docs.ceph.com/en/reef/rados/operations/add-or-rm-osds/
[^ceph-operating]: Ceph. (n.d.). RADOS Operations — Operating a Cluster. Retrieved 2026-09-27, from https://docs.ceph.com/en/reef/rados/operations/operating/
[^ceph-hardware]: Ceph. (n.d.). Ceph Documentation. Retrieved 2026-09-27, from https://docs.ceph.com/en/reef/
[^linstor-rg]: LINBIT. (n.d.). LINSTOR User Guide — Resource Groups & Auto-Placement. Retrieved 2026-09-27, from https://linbit.com/drbd-user-guide/linstor-guide-1_0-en/
[^linstor-pool]: LINBIT. (n.d.). LINSTOR User Guide — Storage Pools. Retrieved 2026-09-27, from https://linbit.com/drbd-user-guide/linstor-guide-1_0-en/
[^linstor-providers]: LINBIT. (n.d.). LINSTOR User Guide — Storage Providers. Retrieved 2026-09-27, from https://linbit.com/drbd-user-guide/linstor-guide-1_0-en/
[^linstor-readme]: LINBIT. (n.d.). LINSTOR Server GitHub README. Retrieved 2026-09-27, from https://github.com/LINBIT/linstor-server
[^linstor-features]: LINBIT. (n.d.). LINBIT SDS Features. Retrieved 2026-09-27, from https://linbit.com/linbit-sds-features/
[^openebs-docs]: OpenEBS. (n.d.). Documentation. Retrieved 2026-09-27, from https://docs.openebs.io/
[^longhorn-docs]: Longhorn. (n.d.). What is Longhorn. Retrieved 2026-09-27, from https://longhorn.io/docs/1.8.1/what-is-longhorn/
[^glusterfs-status]: GlusterFS. (n.d.). GitHub Repository. Retrieved 2026-09-27, from https://github.com/gluster/glusterfs
[^minio-eol]: MinIO. (n.d.). MinIO Container Documentation. Retrieved 2026-09-27, from https://min.io/docs/minio/container/index.html