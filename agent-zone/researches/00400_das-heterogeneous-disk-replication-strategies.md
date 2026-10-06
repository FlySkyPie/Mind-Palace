# 8Bay USB 3.0 DAS 異質硬碟副本式儲存池方案比較

## 問題概述

一台 8Bay SATA3-to-USB 3.0 DAS（Direct Attached Storage），內裝八顆容量不一的硬碟（異質硬碟），目標是將這些硬碟組合成一個統一的儲存池，具備**冗餘機制**，且**偏好副本（replication/copies）而非奇偶校驗（parity/erasure coding）**，並考量最低的運算效能消耗與最簡單的復原流程。

**已排除方案：** Ceph, GlusterFS, Lustre, ZFS, LVM。

## 篩選原則

| 原則 | 說明 |
|------|------|
| ✅ 副本式冗餘 | 儲存多份完整拷貝，避免奇偶校驗的運算開銷與重建風險 |
| ✅ 低運算開銷 | USB 3.0 頻寬（理論 5 Gbps）才是瓶頸，CPU 不應成為瓶頸 |
| ✅ 簡易復原流程 | 硬碟故障時，替換與資料重建越簡單越好 |
| ✅ 異質硬碟相容 | 各容量硬碟的空間需能被有效利用，不浪費大量容量 |
| ✅ 單機 DAS 適用 | 方案應設計給或可部署於單一主機的 JBOD 環境 |

## 方案比較總表

| 方案 | 冗餘方式 | 空間效率（以 2+3+4+6 TB 為例） | 運算開銷 | 復原簡易度 | 異質硬碟 | 成熟度 |
|------|----------|:---------------------------:|:--------:|:---------:|:--------:|:------:|
| **Btrfs RAID1** | 區塊層 2 副本 | ~7 TB / 15 TB (47%) | ★★★★★ 極低（核心原生） | ★★★★☆ `btrfs device replace` | ★★★★☆ 區塊級配對演算法 | ★★★★★ 穩定 |
| **MooseFS (goal=2)** | 檔案區塊 2 副本 | 以最小硬碟為瓶頸，無固定公式 | ★★★★☆ 低（C 語言用戶態） | ★★★★★ 自動重新複製 | ★★☆☆☆ 容量極度不均時效益差 | ★★★★☆ 活躍開發 |
| **MergerFS + 外部複製** | 無內建 → 需搭配 rsync/borg | 100%（底層各 FS 完整可用） | ★★★★★ 趨近零（FUSE 穿透） | ★★☆☆☆ 視備份工具而定 | ★★★★★ 最佳 | ★★★★★ 穩定 |
| **Greyhole** | 檔案層 N 副本 | 視複本數而定（1/N） | ★★★☆☆ 中（Samba VFS） | ★★★★☆ 資料為一般檔案 | ★★★★☆ 好 | ★★★☆☆ 低維護 |
| **SeaweedFS** | 複本（熱資料）+ EC（冷資料） | 視設定（2x~1.4x） | ★★★★★ 極低（Go） | ★★☆☆☆ 需手動修復 | ★★★☆☆ 需手動設定 -max | ★★★★☆ 活躍開發 |

## 推薦方案一：Btrfs RAID1（最推薦）

### 原理

Btrfs 在區塊層（chunk-level，預設 1 GiB）實作 RAID1，每個 chunk 在兩顆不同硬碟上各存一份拷貝。不同於傳統 RAID1 要求成對等容量的硬碟，Btrfs 的 chunk 配對演算法能將任意兩顆硬碟配對，因此支援混搭容量。[^btrfs-archwiki]

### 異質硬碟容量利用率

Btrfs RAID1 的有效容量由**區塊配對演算法**決定：

> 若最大硬碟容量 ≤ 其餘硬碟容量總和，則可用空間 ≈ 原始總容量 ÷ 2。
> 若最大硬碟容量 > 其餘硬碟容量總和，則可用空間 = 其餘硬碟容量總和。[^btrfs-faq]

舉例：4 顆硬碟（2 TB + 3 TB + 4 TB + 6 TB = 15 TB 原始）：
- 6 TB + 4 TB → 配對貢獻 4 TB，6 TB 剩 2 TB
- 3 TB + 2 TB（剩餘）→ 配對貢獻 2 TB，3 TB 剩 1 TB
- 2 TB（原始） + 1 TB（剩餘）→ 配對貢獻 1 TB
- **有效容量 ≈ 7 TB**（47%）

對於 8 顆硬碟，比例會因硬碟數量增加而趨近 50%。[^btrfs-profiles]

### 多副本模式

Btrfs 支援三種副本模式：

| 模式 | 副本數 | 最低硬碟數 | 可承受故障數 | 空間效率 |
|:----:|:------:|:---------:|:----------:|:--------:|
| RAID1 | 2 | 2 | 1 顆 | ~50% |
| RAID1c3 | 3 | 3 | 2 顆 | ~33% |
| RAID1c4 | 4 | 4 | 3 顆 | ~25% |

三者皆為生產環境穩定（OK）等級。[^btrfs-status]

### 復原流程

硬碟故障時：

```bash
# 1. 確認故障裝置
btrfs device usage /mnt/storage

# 2. 若硬碟尚可讀取，直接替換
btrfs device replace /dev/sdb /dev/sdh /mnt/storage

# 3. 若硬碟已完全損壞，強制移除
btrfs device remove missing /mnt/storage

# 4. 加入新硬碟
btrfs device add /dev/sdh /mnt/storage

# 5. 重新平衡（讓新硬碟參與副本配置）
btrfs balance /mnt/storage
```

整個流程可在線進行，不需卸載檔案系統。[^btrfs-volume]

### 其他優點

- **核心原生**：無需 FUSE、無需額外背景服務，運算開銷趨近零
- **checksum 保護**：所有資料與元資料都有 CRC32C 檢查，可自動修復位元腐壞[^btrfs-archwiki]
- **快照**：寫入時複製（CoW）快照，低開銷
- **透明壓縮**：可啟用 lz4/zstd 壓縮，節省空間

### 限制

- 硬碟無法獨立掛載——Btrfs RAID1 是單一檔案系統，無法抽出一顆硬碟到另一台機器直接讀取
- 極度不均的硬碟容量組合會降低空間效率（例如 1 TB + 10 TB 配對時浪費較多）
- `btrfs balance` 需要預留約 10–20% 空餘空間作為工作空間[^btrfs-balance]

---

## 推薦方案二：MooseFS（進階選擇）

### 原理

MooseFS 是分散式檔案系統，透過 `moosefs-chunkserver` 將每顆硬碟掛載點註冊給 `moosefs-master`，用戶端透過 FUSE 掛載單一 POSIX 命名空間。其冗餘機制為**副本目標（goal）**——設定 `goal=2` 即每份資料保留兩份拷貝，分別放置在不同硬碟上。[^moosefs-arch]

### 單機 DAS 部署

每顆 USB 硬碟各自掛載至 `/mnt/usb1`–`/mnt/usb8`，在 `/etc/mfs/mfshdd.cfg` 中列出所有掛載點。實務上已有使用者以 14 顆硬碟在同一台主機上運行 14 個 chunkserver 的案例。[^moosefs-disc510]

### 異質硬碟的挑戰

MooseFS 的空間平衡策略是基於**已使用空間百分比**，而非絕對容量。若硬碟容量差異過大，較小的硬碟會先被填滿，成為容量瓶頸。MooseFS 維護者指出：

> 「唯一的平衡方法是讓各 chunkserver 容量大致相等，尤其是在 chunkserver 數量不多的情況下。12+ 顆以上時，MooseFS 對不同容量才有較好的平衡能力。」[^moosefs-disc600]

對於 8 顆異質硬碟，嚴重不對稱的組合（如 1 TB + 10 TB）將導致大量容量無法有效利用。

### 復原流程

若 `goal≥2`，硬碟故障時 MooseFS Master 會自動偵測遺失的 chunk，並在其他硬碟上重新建立副本，不需人工介入。若硬碟本身尚可讀取，移至新主機掛載後，Master 也能辨識既有 chunk 資料。[^moosefs-issue382]

### 限制

- 雖然可在單機使用，但其架構本質上為多節點叢集設計
- Master 將所有元資料保持在記憶體中，大量小檔案時消耗 RAM
- 異質硬碟容量不對稱時，儲存效率顯著下降

---

## 比較場景：8 顆硬碟模擬

假設 8 顆 USB 硬碟容量為：1、1、2、2、3、4、4、6 TB（原始總計 23 TB）：

| 方案 | 有效容量估算 | 副本數 | 說明 |
|------|:----------:|:------:|------|
| Btrfs RAID1 | ~11.5 TB（50%） | 2 | chunk 配對演算法充分利用多硬碟優勢 |
| Btrfs RAID1c3 | ~7.7 TB（33%） | 3 | 三副本安全邊際高，但容量代價大 |
| MooseFS (goal=2) | 受限最小硬碟，估計 ~6–9 TB | 2 | 1 TB 硬碟很快填滿，限制整體上限 |
| MergerFS + 手動複製 | 23 TB（但複本佔額外空間） | 視設定 | 彈性最高，但需自行管理冗餘 |

---

## 結論

### 首要推薦：Btrfs RAID1

在給定的限制條件（無 Ceph/GlusterFS/Lustre/ZFS/LVM、偏好副本、低運算開銷、簡易復原、單機 DAS）之下，**Btrfs RAID1 是最符合需求的方案**：

- ✅ 副本（非奇偶）：RAID1/1c3/1c4 均為完全複製
- ✅ 運算開銷極低：核心原生，無 FUSE/用戶態開銷
- ✅ 復原極簡：`btrfs device replace` 單指令在線完成
- ✅ 異質硬碟相容：chunk 級配對比傳統 RAID 靈活得多
- ✅ 原生 checksum + 快照 + 壓縮

### 何時選擇 MooseFS

若需要硬碟可獨立抽出的靈活性（Btrfs RAID1 無法辦到），且硬碟容量大致均等（或可接受容量縮水），MooseFS 是優秀的替代方案。

### 何時選擇 MergerFS + 外部複製

若異質硬碟容量極度不對稱（如 1 TB 搭配 16 TB），且能接受自行設計與維護備份/複製排程（rsync、borg、restic），MergerFS 提供 100% 容量利用率，但冗餘機制需外部實作。

### 不推薦方案簡述

- **SeaweedFS**：功能強大但為叢集架構，單機 DAS 使用時復原需手動操作，且異質硬碟管理為手動設定，不符「簡易」原則。
- **Greyhole**：以 Samba VFS 為中心，不適合非 Samba 使用場景，且維護不活躍。
- **SnapRAID / NonRAID**：皆為奇偶校驗方案，與「偏好副本」的前提衝突。

---

## 參考文獻

[^btrfs-archwiki]: ArchWiki. (n.d.). Btrfs. Retrieved 2026-10-01, from https://wiki.archlinux.org/title/Btrfs
[^btrfs-faq]: Btrfs Wiki. (n.d.). FAQ — 4.11: Can I mix different size drives in RAID-1 mode?. Retrieved 2026-10-01, from https://archive.kernel.org/oldwiki/btrfs.wiki.kernel.org/index.php/FAQ.html
[^btrfs-profiles]: tnonline. (n.d.). Btrfs Profiles. Retrieved 2026-10-01, from https://wiki.tnonline.net/w/Btrfs/Profiles
[^btrfs-status]: Btrfs Documentation. (n.d.). Status. Retrieved 2026-10-01, from https://btrfs.readthedocs.io/en/latest/Status.html
[^btrfs-volume]: Btrfs Documentation. (n.d.). Volume Management. Retrieved 2026-10-01, from https://btrfs.readthedocs.io/en/latest/Volume-management.html
[^btrfs-balance]: Btrfs Documentation. (n.d.). btrfs-balance. Retrieved 2026-10-01, from https://btrfs.readthedocs.io/en/latest/btrfs-balance.html
[^moosefs-arch]: MooseFS. (n.d.). MooseFS — What is MooseFS. Retrieved 2026-10-01, from https://moosefs.com/what-is-moosefs
[^moosefs-disc510]: GitHub. (2024). MooseFS Discussion #510 — Multiple HDDs on one server. Retrieved 2026-10-01, from https://github.com/moosefs/moosefs/discussions/510
[^moosefs-disc600]: GitHub. (2024). MooseFS Discussion #600 — Disk space balancing with different capacities. Retrieved 2026-10-01, from https://github.com/moosefs/moosefs/discussions/600
[^moosefs-issue382]: GitHub. (2024). MooseFS Issue #382 — Recover data after disk failure. Retrieved 2026-10-01, from https://github.com/moosefs/moosefs/issues/382