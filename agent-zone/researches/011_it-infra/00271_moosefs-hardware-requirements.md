# MooseFS 硬體需求研究報告

## 概述

本文探討 MooseFS 分散式檔案系統在不同角色（Master Server、Follower、Metalogger、Chunkserver、Client）下的硬體需求，包含最低配置、建議規格、以及不同規模部署的參考指南。

---

## 一、最低叢集規模

| 版本 | 最低機器組成 |
|------|------------|
| **Community Edition (CE)** | 1 台 Master Server + 2 台 Chunkserver + 1 台 Client |
| **PRO 版 (HA 高可用)** | 至少 2 台 Master Server（Leader + Follower）+ 至少 3 台 Chunkserver + 1 台 Client |

生產環境強烈建議使用上述最低配置，單機測試不建議用於正式環境。另可選配 Metalogger 來備份中繼資料[^moosefs-req]。

[^moosefs-req]: MooseFS. (n.d.). *Requirements.* Retrieved 2026-09-26, from https://docs.moosefs.com/requirements/

---

## 二、各角色硬體需求詳解

### 2.1 Master Server (Leader / 主控伺服器)

Master Server 是整個 MooseFS 叢集的核心，負責儲存所有檔案系統中繼資料（檔案名稱、目錄結構、屬性等）於記憶體中，為**單執行緒（single-threaded）**程序。

#### CPU

- 單執行緒密集運算，**高時脈、少核心** 的處理器最適合。
- 官方建議關閉 Hyper-Threading，並在 BIOS 中設定為「最大效能」模式。
- 參考 [CPU Benchmark 單執行緒評分](https://www.cpubenchmark.net/singleThread.html) 挑選[^master-req]。

#### RAM

- 中繼資料資料庫完全存在記憶體中。
- 每個檔案/目錄（inode）約佔 **300～350 bytes**。
- 記憶體需求與**檔案數量成正比**，與叢集總容量無關。
- 範例：1.5 億個檔案/目錄 → 約需 **42 GiB RAM**（僅中繼資料，不含作業系統開銷）[^master-req]。

#### 磁碟

- 建議 **RAID 1 或 RAID 1+0** 的冗餘本地儲存，掛載於 `/var/lib/mfs`。
- 磁碟空間計算公式：
  ```
  DISK_SPACE = RAM × (BACK_META_KEEP_PREVIOUS + 2) + (BACK_LOGS + 1) [GiB]
  ```
  其中 `BACK_LOGS` 預設 50、`BACK_META_KEEP_PREVIOUS` 預設 1。
- 預設值：`RAM × 3 + 51 GiB`。例如 128 GiB RAM → 至少 **435 GiB**[^master-req]。

#### 網路

- 不傳輸實際檔案資料，只處理中繼資料（小量 I/O）。
- 但每個 Client 與 Chunkserver 都會與 Master 通訊，有大量小型網路 I/O。
- 建議使用**兩張獨立網路介面卡**：一張服務 Client，另一張伺服器間通訊（不建議使用 LACP 綁定）[^network-req]。

[^master-req]: MooseFS. (2020-07-15). *Master Server Requirements.* Retrieved 2026-09-26, from https://moosefs.com/blog/master-servers-requirements.html
[^network-req]: MooseFS. (2020-07-15). *What are the MooseFS Network Requirements?* Retrieved 2026-09-26, from https://moosefs.com/blog/what-are-the-moosefs-network-requirements.html

---

### 2.2 Follower Master Server (PRO HA 備援主控)

- 在 MooseFS PRO 的高可用（HA）配置中，Follower 是非同步維護中繼資料副本的備援 Master。
- Leader 故障時可由多數 Chunkserver 投票選舉為新的 Leader。
- **硬體需求與 Leader Master Server 完全相同**[^moosefs-req]。

---

### 2.3 Metalogger（中繼資料記錄器）

- 從 Leader Master Server 非同步收集中繼資料變更日誌，儲存到本地磁碟。
- **磁碟空間需求與 Master Server 相同**（至少同等空間）。
- **若要作為 Master Server 的故障接管**：RAM 與 CPU 時脈應至少與 Master Server 相同或接近。
- Community Edition（無 HA）中**強烈建議至少設置一台 Metalogger**；PRO 版已有 Follower Master，Metalogger 為選配[^moosefs-req]。

---

### 2.4 Chunkserver（資料儲存伺服器）

Chunkserver 主要工作是傳輸資料（磁碟 ↔ 網路），不進行大量運算。

#### CPU

- **多執行緒程序**，建議多核心處理器。
- 一般情況下約使用 CPU **1 個核心**。
- 若使用 Erasure Coding（糾刪碼），寫入與修復時會增加 CPU 使用率，但一般硬體可達 5 GB/s 運算速度[^chunk-req]。

#### RAM

- 每個 chunk（資料區塊）在記憶體中約佔 **150～200 bytes**。
- 基礎程序約需 **350 MiB（Virtual）/ 200 MiB（Resident）**。
- 實務建議：典型配置 **8～12 GiB**（其餘可用於資料快取）[^chunk-req]。

#### 磁碟

| 項目 | 建議 |
|------|------|
| **配置方式** | **JBOD**（不建議使用 RAID 控制器） |
| **檔案系統** | **XFS**（建議格式） |
| **掛載方式** | 每顆磁碟獨立掛載（如 `/mnt/chunk01`、`/mnt/chunk02`） |
| **磁碟數量** | 建議較多的 Chunkserver × 較少磁碟以提升平行效能 |

RAID 不建議的原因：
1. MooseFS 無法偵測 RAID 陣列狀態，可能誤報已損壞 RAID 為健康。
2. 單顆 2 TiB 磁碟故障 → 複製修復約 20～60 分鐘；36 TiB RAID 失敗 → 修復需 12～18 小時，資料風險大增。

SSD 與 HDD 可混合建置（透過 Storage Classes），但不建議同一 Chunkserver 程序中混合兩者[^chunk-req] [^best-practices]。

#### 網路

- 建議至少 **1 Gbps**，**10 Gbps** 更佳。
- 可使用兩張獨立網路介面卡（與 Master Server 相同配置）[^network-req]。

[^chunk-req]: MooseFS. (2020-07-15). *Chunkserver Hardware Requirements.* Retrieved 2026-09-26, from https://moosefs.com/blog/chunkserver-hardware-requirements.html
[^best-practices]: MooseFS. (2020-07-15). *10 MooseFS Best Practices to Maximize Performance.* Retrieved 2026-09-26, from https://moosefs.com/blog/10-moosefs-best-practices-to-maximize-performance.html

---

### 2.5 Client（客戶端）

- 需要 **FUSE**（建議 ≥ 2.7.2，最低 2.6）。
- **io_uring 支援**（效能優化）：需 Linux 核心 ≥ 6.14 + FUSE ≥ 3.19。
- 支援平台：Linux、FreeBSD、macOS、Windows（Server 2008 R2 SP1+、Windows 7 SP1+）[^os-req]。

[^os-req]: MooseFS. (2020-07-15). *What are the Operating Systems and Networking Requirements?* Retrieved 2026-09-26, from https://moosefs.com/blog/what-are-the-operating-systems-and-networking-requirements.html

---

## 三、網路需求總覽

| 項目 | 建議 |
|------|------|
| **通訊協定** | TCP/IP（已測試 Ethernet、InfiniBand IP-over-IB） |
| **最低頻寬** | 1 Gbps |
| **建議頻寬** | 10 Gbps 以上（8 台中階 Chunkserver 即可飽和 40 Gbps） |
| **Jumbo Frames** | 建議設定 MTU = 9000 |
| **交換器** | 兩台交換器以 LACP 連接，無單點故障 |
| **updatedb** | Linux 上建議在 `/etc/updatedb.conf` 的 PRUNEFS 中加入 `fuse.mfs` |

資料來源：[^network-req]。

---

## 四、不同規模部署建議

| 規模 | 檔案/目錄數 | Master RAM 建議 | Master 磁碟建議 | Chunkserver 配置 |
|------|------------|----------------|----------------|-----------------|
| **小型**（測試/個人） | 數百萬 | 4～8 GiB | 100～200 GiB RAID1 | 2～3 台，各 2～4 顆 HDD |
| **中型**（企業） | 數千萬 | 16～32 GiB | 200～400 GiB RAID1 | 4～10 台，各 4～8 顆 HDD |
| **大型**（1.5 億檔案） | 1.5 億 | 42 GiB | 435 GiB RAID1/10 | 10+ 台，依容量擴充 |
| **超大規模**（PB 級） | 數億+ | 64～128+ GiB | RAM×3+51 GiB | 建議 goal=3（RL=2），EC 配置至少 8+n 台 |

---

## 五、結論與關鍵要點

1. **Master Server 是瓶頸**：由於單執行緒特性，時脈比核心數重要。
2. **記憶體決定檔案數量上限**：每 100 萬檔案約需 300～350 MB RAM。
3. **Chunkserver 使用 JBOD 而非 RAID**：善用 MooseFS 自身的資料備援機制。
4. **網路是吞吐量關鍵**：10 Gbps 為大型部署建議起點。
5. **Community Edition 建議搭配 Metalogger**：作為中繼資料備援。

---

## 參考資料

[^moosefs-req]: MooseFS. (n.d.). *Requirements.* Retrieved 2026-09-26, from https://docs.moosefs.com/requirements/

[^hardware-req]: MooseFS. (n.d.). *Hardware Requirements.* Retrieved 2026-09-26, from https://docs.moosefs.com/requirements/hardware/

[^master-req]: MooseFS. (2020-07-15). *Master Server Requirements.* Retrieved 2026-09-26, from https://moosefs.com/blog/master-servers-requirements.html

[^chunk-req]: MooseFS. (2020-07-15). *Chunkserver Hardware Requirements.* Retrieved 2026-09-26, from https://moosefs.com/blog/chunkserver-hardware-requirements.html

[^network-req]: MooseFS. (2020-07-15). *What are the MooseFS Network Requirements?* Retrieved 2026-09-26, from https://moosefs.com/blog/what-are-the-moosefs-network-requirements.html

[^os-req]: MooseFS. (2020-07-15). *What are the Operating Systems and Networking Requirements?* Retrieved 2026-09-26, from https://moosefs.com/blog/what-are-the-operating-systems-and-networking-requirements.html

[^best-practices]: MooseFS. (2020-07-15). *10 MooseFS Best Practices to Maximize Performance.* Retrieved 2026-09-26, from https://moosefs.com/blog/10-moosefs-best-practices-to-maximize-performance.html

[^install-guide]: MooseFS. (2020-07-15). *How to Install MooseFS.* Retrieved 2026-09-26, from https://moosefs.com/blog/how-install-moosefs.html

[^iouring]: MooseFS. (2022). *MooseFS and io_uring — Getting Ahead of the Curve on FUSE Performance.* Retrieved 2026-09-26, from https://moosefs.com/blog/moosefs-and-io-uring-getting-ahead-of-the-curve-on-fuse-performance.html