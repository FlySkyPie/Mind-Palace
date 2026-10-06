# Greyhole FOSS 替代方案研究報告

## 概述

Greyhole 是一套基於 Samba 的 JBOD 儲存池整合與檔案複製守護程式，可將任意數量、任意規格的硬碟整合為單一儲存池，並提供可設定的跨磁碟檔案複製備援[^greyhole]。本報告針對 Greyhole 的 FOSS 替代方案進行調查，重點在於 JBOD（Just a Bunch of Disks）儲存池化方案，排除 Ceph、GlusterFS、Lustre、MooseFS、SeaweedFS 等分散式檔案系統。

## 替代方案一覽

### 1. mergerfs — 首要推薦

**mergerfs** 是一套基於 FUSE 的聯邦檔案系統（union filesystem），可將多個磁碟或目錄合併為單一掛載點。每個磁碟保留其獨立的檔案系統（ext4、XFS 等）。寫入時根據可設定的策略（如最多剩餘空間、既存路徑、隨機等）選擇底層磁碟。[^mergerfs]

- **JBOD 儲存池化**：✅ 核心功能，可混合任意大小的磁碟，隨意增減
- **備援/複製**：⚠️ 無內建即時複製，但附屬工具 `mergerfs.dup`（屬於 `mergerfs-tools`）可依需求跨磁碟複製檔案，可透過 cron 排程執行[^mergerfs_tools]
- **專案狀態**：活躍開發（GitHub 5,600+ stars），成熟穩定
- **適合對象**：任何需要在 Linux 上進行簡單可靠 JBOD 儲存池化的使用者

### 2. SnapRAID — 同位元檢查式備援

**SnapRAID** 是一套同位元計算程式，在既有的檔案系統之上計算同位元資料，可於磁碟故障時重建資料。支援最多 6 顆同位元磁碟。[^snapraid]

- **JBOD 儲存池化**：❌ 非主要功能（唯讀符號連結式儲存池化不適合實際使用）
- **備援/備份**：✅ 透過同位元（類似 RAID5/6 但為快照式，非即時）
- **運作方式**：透過 cron 定時執行 `snapraid sync` 更新同位元
- **適合對象**：保護大量、相對靜態的資料（如媒體庫、檔案庫）免於磁碟故障

### 3. mergerfs + SnapRAID 組合 — 最受歡迎的「窮人版 Unraid」

此組合為 homelab/自架 NAS 社群中最常見的 Greyhole 替代方案：[^union_guide]

| 工具 | 角色 |
|------|------|
| **mergerfs** | 即時 JBOD 儲存池化 — 將所有磁碟呈現為單一掛載點 |
| **SnapRAID** | 排程同位元 — 保護免受磁碟故障影響 |
| **mergerfs.dup**（選用） | 特定檔案/資料夾跨磁碟複製 |

優勢：
- ✅ 混合大小磁碟（無 RAID 的空間浪費）
- ✅ 即時讀寫儲存池化
- ✅ 同位元式備援（SnapRAID 每 6 小時透過 cron 同步）
- ✅ 可選檔案層級複製（mergerfs.dup）
- ✅ 各磁碟維持獨立可讀 — 無鎖定效應

### 4. mhddfs — 歷史前身（不建議新建置）

最早的 FUSE 聯邦檔案系統，mergerfs 的靈感來源。

- **JBOD 儲存池化**：✅
- **備援**：❌
- **狀態**：自 2015 年起未再維護，有已知的段錯誤（Segmentation fault）穩定性問題[^mhddfs]
- **僅建議**：如需在非常老舊的系統且無法安裝 mergerfs 的情況下使用

### 5. unionfs-fuse — 疊層導向（不適合 JBOD 儲存池化）

基於 FUSE 的聯邦檔案系統，專注於疊層行為（overlay）包含寫入時複製（CoW）語意。[^unionfs_fuse]

- **JBOD 儲存池化**：⚠️ 技術上可行，但非設計用途，主要功能是將可寫入層疊加在唯讀層之上
- **備援**：❌
- **適合對象**：Live CD 環境、容器式疊層，非一般 NAS 儲存池化

### 6. PolicyFS（pfs）— 新進方案，具路由規則

較新的 FUSE 儲存守護程式，將多個儲存路徑統一於單一掛載點下，具備明確的讀寫路由規則與選擇性的 SQLite 元資料索引。[^policyfs][^hieutdo_policyfs]

- **JBOD 儲存池化**：✅ 可作為 mergerfs 基本替代方案
- **備援**：❌ 目前不支援
- **特色功能**：基於模式的路由（如 `match: 'library/movies/**'`）、排程分層搬移（SSD→HDD）、選擇性元資料索引以減少 HDD 喚醒
- **取捨**：POSIX 相容性不如 mergerfs；部分操作（刪除、重新命名）可能延後至維護任務

### 7. aufs — 核心層級（大多數發行版已淘汰）

曾嘗試進入 Linux 核心但失敗的核心層級聯邦檔案系統。[^aufs]

- **JBOD 儲存池化**：✅
- **備援**：❌
- **狀態**：大多數現代 Linux 發行版不再提供，開發多年停滯。不建議用於實際部署。

### 8. NonRAID — 即時同位元（進階）

Unraid 儲存陣列核心驅動程式的分支，在區塊層級提供 RAID5 式同位元，但各裝置保持獨立。[^nonraid]

- **JBOD 儲存池化**：⚠️ 間接支援，需搭配 mergerfs 提供統一視圖
- **備援**：✅ 即時同位元（不同於 SnapRAID 的排程同位元）
- **專案狀態**：小型、特殊用途，增加核心層級複雜度

## 比較總表

| 方案 | JBOD 儲存池化 | 備援/複製 | 即時？ | 活躍維護？ | 複雜度 |
|------|:---:|:---:|:---:|:---:|:---:|
| **mergerfs** | ✅ 核心 | ⚠️ 需工具 | ✅ | ✅ 非常 | 低 |
| **SnapRAID** | ❌ 唯讀 | ✅ 同位元（最多 6 碟） | ❌ 排程 | ✅ | 中 |
| **mergerfs + SnapRAID** | ✅ | ✅ 同位元 + 選用複製 | ✅ 池 / ⚠️ 同位元 | ✅ | 中 |
| **mhddfs** | ✅ | ❌ | ✅ | ❌ 終止 | 低 |
| **unionfs-fuse** | ⚠️ 疊層導向 | ❌ | ✅ | ⚠️ 小型 | 低 |
| **PolicyFS** | ✅ | ❌ | ✅ | ⚠️ 較新 | 中 |
| **aufs** | ✅ | ❌ | ✅ | ❌ 終止 | 中 |
| **NonRAID** | ⚠️ 需 mergerfs | ✅ 即時同位元 | ✅ | ⚠️ 小眾 | 高 |

## 建議

若需要 Greyhole 的直接替代方案，用來整合 JBOD 磁碟並選擇性複製檔案：

1. **以 mergerfs 為起點**進行 JBOD 儲存池化 — 最簡單、支援最廣泛的選項
2. **附加 SnapRAID**以同位元方式保護磁碟故障（排程式，非即時）
3. **使用 `mergerfs.dup`**（來自 `mergerfs-tools`）若需要類似 Greyhole 的跨磁碟複製 — 透過 cron 排程執行
4. **考慮 PolicyFS**若需要明確的路徑路由規則或分層儲存（SSD → HDD）

**mergerfs + SnapRAID** 組合是最經過實戰驗證、社群廣泛推薦的做法，也是多數從 Greyhole 或 StableBit DrivePool 遷移至 Linux 的使用者採用的方案。

---

[^greyhole]: gboudreau. (n.d.). Greyhole. Retrieved 2026-10-03, from https://github.com/gboudreau/Greyhole
[^mergerfs]: trapexit. (n.d.). mergerfs. Retrieved 2026-10-03, from https://github.com/trapexit/mergerfs
[^mergerfs_tools]: trapexit. (n.d.). mergerfs-tools. Retrieved 2026-10-03, from https://github.com/trapexit/mergerfs-tools
[^snapraid]: snapraid.it. (n.d.). SnapRAID. Retrieved 2026-10-03, from https://www.snapraid.it/
[^union_guide]: pistack.xyz. (2026-05-20). Self-Hosted Union File Systems: SnapRAID, MergerFS, UnionFS Guide. Retrieved 2026-10-03, from https://www.pistack.xyz/posts/2026-05-20-self-hosted-union-file-systems-snapraid-mergerfs-unionfs-guide/
[^mhddfs]: Roman Mamedov. (n.d.). mhddfs. Retrieved 2026-10-03, from https://romanrm.net/mhddfs
[^unionfs_fuse]: rpodgorny. (n.d.). unionfs-fuse. Retrieved 2026-10-03, from https://github.com/rpodgorny/unionfs-fuse
[^policyfs]: PolicyFS. (n.d.). Use Cases. Retrieved 2026-10-03, from https://docs.policyfs.org/use-cases/
[^hieutdo_policyfs]: hieutdo. (n.d.). policyfs. Retrieved 2026-10-03, from https://github.com/hieutdo/policyfs
[^aufs]: aufs project. (n.d.). aufs — Advanced Union File System. Retrieved 2026-10-03, from https://github.com/sfjro/aufs
[^nonraid]: qvr. (n.d.). nonraid. Retrieved 2026-10-03, from https://github.com/qvr/nonraid