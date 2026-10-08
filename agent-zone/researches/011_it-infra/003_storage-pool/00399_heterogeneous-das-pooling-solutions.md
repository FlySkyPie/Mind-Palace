# 8Bay SATA3→USB 3.0 DAS 異質硬碟儲存池方案比較

排除 Ceph、GlusterFS、Lustre、ZFS 及 LVM 後，針對 USB 介面、異質硬碟（不同容量/品牌）、具備冗餘的儲存池需求，以下分析現有主流方案。

## 核心候選方案

### 1. SnapRAID + MergerFS（推薦）

**架構**：兩套獨立工具協作。MergerFS 是 FUSE 聯合檔案系統，將多棵掛載樹（如 `/mnt/disk1`、`/mnt/disk2`）合併為單一掛載點（`/mnt/storage`）[^mergerfs_gh]。SnapRAID 是快照式同位元檢查工具，以排程（如每日凌晨）計算檔案層級的同位元資料，存放在專屬同位元硬碟上[^snapraid_gh]。

**異質硬碟支援**：
- 完全混合任何容量組合。唯一限制：每顆同位元硬碟須 ≥ 最大資料硬碟[^datahoarder]
- 每個資料碟格式化為獨立的 ext4/XFS，可個別掛載在任何 Linux 主機上讀取
- 可加入已裝有資料的硬碟，無需抹除或重建

**USB 適用性**：
- 每顆硬碟獨立運作，USB 斷線只影響該碟檔案——不會破壞整個陣列
- 僅在排程執行時才寫入同位元資料，USB 介面偶發降速/中斷不會造成 write-hole
- 可透過 `nofail`、UUID 掛載、UAS quirks 等方式強化 USB 穩定性[^bigiron_usb]

**缺點**：
- 同位元非即時——兩次 sync 之間寫入的資料無保護
- 不適合頻繁更新的小檔（資料庫、VM、應用程式目錄）
- 純 CLI 管理（可搭配 OpenMediaVault 的 SnapRAID/MergerFS 套件獲得網頁設定介面）

### 2. Unraid（可考慮但非首選）

**架構**：付費 NAS 作業系統，提供即時區塊層級同位元保護。每個硬碟格式化為 XFS/Btrfs，透過 User Shares 呈現[^unraid_forum_usb]。

**異質硬碟支援**：良好，同 SnapRAID——僅同位元硬碟須為最大。

**USB 適用性**——社群共識為「不建議用於主要陣列」[^unraid_usb_yesno]：
- USB 瞬間斷線被 Unraid 視為硬碟故障，可能將硬碟踢出陣列
- 部分 USB 橋接器（bridge）不傳遞硬碟序號，橋接器故障後需同型號才能讀取資料
- 同位元檢查/重建期間 USB 斷線極危險
- 建議用途：僅用「Unassigned Devices」外掛做備份目標

**缺點**：付費（起 ~$60）、新品須先抹除（無法直接加入既有資料碟）、最多 2 顆同位元硬碟。

### 3. Btrfs RAID（不推薦）

**RAID1 混用容量**：浪費空間——16TB + 8TB → 僅 8TB 可用，16TB 碟剩餘空間無法使用[^archwiki_btrfs]。
**RAID5/6**：核心實作有致命缺陷，生產環境禁用[^archwiki_btrfs]。
**單一模式（-d single）**：可用全部空間但無冗餘，等同 JBOD。
**USB 適用性**：RAID1 模式下 USB 斷線可能導致 split-brain 或需要重建。

### 4. mdadm RAID（不推薦）

**混用容量**：RAID5/6 受限於最小硬碟，大碟空間浪費。
**USB 適用性**：斷線被視為成員失敗，回連後需完整 resync，極不適合 USB 介面[^datahoarder_compare]。

## 比較總表

| 面向 | SnapRAID+MergerFS | Unraid (+USB) | Btrfs RAID | mdadm RAID |
|---|---|---|---|---|
| 異質容量 | ✅ 完整 | ✅ 完整 | ❌ RAID1 浪費空間 | ❌ 受限最小碟 |
| USB 斷線容忍 | ✅ 優雅降級 | ⚠️ 高風險 | ⚠️ RAID1 有風險 | ❌ 危險 |
| 即時同位元 | ❌ 排程 | ✅ 即時 | ❌ (RAID5/6 壞掉) | ✅ 即時 |
| 位元腐蝕偵測 | ✅ (scrub) | ⚠️ (Btrfs 選項) | ✅ | ❌ 無 |
| 花費 | 免費 | 付費 | 免費 | 免費 |
| 個別硬碟可讀 | ✅ (任何 Linux) | ✅ (XFS) | ⚠️ (需 btrfs tools) | ❌ (需 mdadm) |
| 最大同位元數 | 6 | 2 | N/A | 2 (RAID6) |
| 加入既有資料碟 | ✅ 可直接加入 | ❌ 須先抹除 | ✅ 可直接加入 | ❌ 需重建 |
| 網頁管理 | ❌ CLI (可搭 OMV) | ✅ 內建 | ❌ CLI | ❌ CLI |
| 單碟獨立休眠 | ✅ 可 | ✅ 可 | ❌ 需全碟上線 | ❌ 需全碟上線 |

## 推薦架構：SnapRAID + MergerFS

```
USB DAS (8Bay SATA3→USB 3.0)
    ↓  每碟獨立 ext4（standalone，可個別讀取）
    ↓
MergerFS → 合併為 /mnt/storage（統一命名空間）
    ↓
SnapRAID → 排程產生同位元（保護最多 6 碟同時故障）
    ↓
Samba/NFS 分享 或 Docker 容器使用
```

**設定摘要**：
- `/etc/fstab` 以 UUID 掛載各碟，加入 `nofail`[^bigiron_debian]
- MergerFS create policy 建議 `pfrd`（percentage free random distribution），避免 `epmfs` 在目錄填滿後出現「No space left on device」卻仍有大量可用空間的矛盾[^easyhtpc]
- `snapraid.conf` 指向各資料碟掛載點，非 MergerFS 掛載點
- 系統計時器每日定時執行 `snapraid sync`，並定期 `snapraid scrub`

**同位元硬碟選擇**：
- CMR 碟（如同級 IronWolf）放同位元位置，承受較多寫入[^easyhtpc]
- SMR 碟可放資料位置（SnapRAID 對資料碟僅讀取）

**補充：資料碟加密**
若加密資料碟，務必同時加密同位元碟，否則同位元檔案會洩漏真實資料[^easyhtpc]。

## 結論

**SnapRAID + MergerFS** 是將 8Bay USB DAS 中的異質硬碟組成具冗餘儲存池的最佳方案。相較其他方案，它同時滿足三個關鍵需求：
1. 完全支援混合容量硬碟——不浪費空間
2. 容忍 USB 介面的不穩定——每碟獨立，斷線不波及整池
3. 即使主機或橋接器故障，每顆硬碟可獨立掛載讀取

若需要網頁管理介面，可考慮底層同樣採用 SnapRAID+MergerFS 的 **OpenMediaVault** 發行版。

---

[^mergerfs_gh]: mergerfs (trapexit). (n.d.). mergerfs - a featureful FUSE union filesystem. GitHub. Retrieved 2026-10-03, from https://github.com/trapexit/mergerfs
[^snapraid_gh]: SnapRAID (amadvance). (n.d.). SnapRAID - a filesystem based snapshot parity tool. GitHub. Retrieved 2026-10-03, from https://github.com/amadvance/snapraid
[^datahoarder]: DataHoarder.io. (n.d.). The DIY NAS: MergerFS + SnapRAID. Retrieved 2026-10-03, from https://datahoarder.io/snapraid-mergerfs/
[^bigiron_usb]: Big Iron. (n.d.). A USB Drive on a Home Server — Stable Mounts, UAS Quirks and Spin-down. Retrieved 2026-10-03, from https://www.bigiron.cc/guides/a-usb-drive-on-a-home-server-stable-mounts-uas-quirks-and-spin-down
[^bigiron_debian]: Big Iron. (n.d.). SnapRAID and MergerFS on Debian — A Media Array That Survives Drive Failures. Retrieved 2026-10-03, from https://www.bigiron.cc/guides/snapraid-and-mergerfs-on-debian-a-media-array-that-survives-drive-failures
[^easyhtpc]: EasyHTPC. (n.d.). MergerFS + SnapRAID Guide — Real Configs from a Live Encrypted 18TB Server. Retrieved 2026-10-03, from https://easyhtpc.com/media-servers/mergerfs-snapraid-guide/
[^unraid_forum_usb]: Unraid Community Forum. (n.d.). External USB Hard Drives with Unraid. Retrieved 2026-10-03, from https://forums.unraid.net/topic/116526-external-usb-hard-drives-with-unraid/
[^unraid_usb_yesno]: Unraid Community Forum. (n.d.). USB Drives — Yes or No?. Retrieved 2026-10-03, from https://forums.unraid.net/topic/134711-usb-drives-yes-or-no/
[^archwiki_btrfs]: ArchWiki. (n.d.). Btrfs — RAID, multi-device configurations, and RAID5/6 status. Retrieved 2026-10-03, from https://wiki.archlinux.org/title/Btrfs
[^datahoarder_compare]: DataHoarder.io. (n.d.). SnapRAID vs Unraid — Detailed Comparison Table. Retrieved 2026-10-03, from https://datahoarder.io/snapraid-vs-unraid/