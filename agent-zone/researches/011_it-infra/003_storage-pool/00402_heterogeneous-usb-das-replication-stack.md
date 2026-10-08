# 異質 USB DAS 硬碟複本式儲存池方案調研

> 已知一組 8-bay SATA3-to-USB 3.0 DAS 外接陣列，內插不同容量硬碟。  
> 目標：將此異質磁碟集合組合成具冗餘的儲存池。  
> 限制：排除分散式檔案系統（Ceph、GlusterFS、Lustre、MooseFS、SeaweedFS），  
> 排除 zfs / Btrfs / LVM 等軟 RAID（因 USB 磁碟於開機序列中掛載過慢）。  
> 偏好：副本式備餘（replication）而非奇偶驗算（parity），以降低運算開銷與復原風險。

## 權衡架構

將整批 USB 外接硬碟變成一個具冗餘儲存池，可拆解為兩層責任：

1. **聚合層（Pooling）** — 將多個獨立掛載點合併為單一統一視野  
2. **備餘層（Redundancy）** — 確保資料在兩顆以上實體碟片各存一份

此分離有兩大好處：聚合層可使用容許任意異質磁碟、容許延遲掛載的工具；備餘層可獨立選擇以複本（而非 parity）實作。

## 方案比較

### 方案一：mergerfs + Lsyncd（推薦主方案）

| 層級 | 工具 | 機制 |
|---|---|---|
| 聚合 | mergerfs（FUSE） | 目錄級 union mount，各硬碟保持原 filesystem（ext4 / XFS 皆可） |
| 備餘 | Lsyncd（inotify + rsync） | 監控 mergerfs mount point 的檔案事件，即時 rsync 至第二顆實體碟 |

**Why this fits the constraints**

- **聚合層對異質磁碟無任何要求**：mergerfs 可合併任意數量、任意容量、任意 filesystem 的目錄，單一檔案完整存在於某一實體碟，不切割。
- **延遲掛載友善**：mergerfs 提供 `branches-mount-timeout` 選項（預設 0，可設為 `branches-mount-timeout=30`），等候慢速 USB 磁碟在開機後才出現，不會像 mdadm / LVM / zpool 在 boot sequence 中因裝置未到而組態失敗[^mergerfs-mount-timeout]。
- **複本非 parity**：Lsyncd 使用 rsync 逐檔案即時鏡像至指定目標碟，無需 CPU 密集的 parity 運算，復原時直接拷貝回即可，無 reconstruct 風險。
- **各碟獨立可讀**：抽出一顆 USB 碟接上任何 Linux 主機，檔案直接可讀 — 無專屬中繼資料或 RAID config 相依[^mergerfs-nonraid]。

**注意點**

- Lsyncd 在極大量小檔案短時間變動時可能落後，可搭配 `--delay` 參數聚合事件。
- rsync 方向需明確（例如 A→B 單向鏡像；若要雙向則需 Unison，但 conflict 處理複雜）。
- 典型配置範例：8 碟中 4 碟為 primary pool，4 碟為 mirror targets；或以 2+2+2+2 組成兩組獨立鏡像 pairing（視可用容量與冗餘程度需求而定）。

### 方案二：mergerfs + rsync（cron 排程）

即方案一的簡化版：以 cron 定時執行 rsync（例如每小時）替代 Lsyncd 的即時觸發。

- 優點：無需額外 daemon，極度穩定。
- 缺點：排程間隔內寫入的資料無複本保護。
- 適用場合：非頻繁寫入的唯讀多媒體儲存池[^mergerfs-rsync]。

### 方案三：Greyhole（Samba 層複本池）

Greyhole 並非一般 filesystem，而是一套以 Samba 為基礎的儲存池應用程式，專為異質 USB 硬碟設計。

- 聚合任意數量、任意容量、任意連接方式（USB / eSATA / SATA）的硬碟。
- 以 **share 為單位設定副本數量**（例如每份檔案存 2 份），副本自動分散至不同實體碟[^greyhole]。
- 磁碟可隨時熱插拔，遺失一顆只損失該碟上的檔案副本，不影響整體池。
- 各檔案為原生格式，抽出個別硬碟即可直接讀取。

**限制**：依賴 Samba（SMB/CIFS），用戶端需透過網路磁碟機存取，而非本機 mount point，對本機應用程式不透明。若使用場景為區域網路 NAS 則適合，若需本機 mount 則 mergerfs 更直接。

### 方案四：mergerfs + Syncthing

以 Syncthing 節點間同步作為複本機制。

- 同一主機上的兩個 Syncthing 節點（各指向不同實體碟目錄），進行單向或雙向同步。
- 優點：版本管理、conflict 處理較 rsync 完善。
- 缺點：非熱路徑同步（非 FUSE/kernel 層級），Overhead 較大，且 Syncthing 原生設計為跨主機，單主機配置稍有迂迴。

## 排除方案理由

| 方案 | 排除原因 |
|---|---|
| ZFS / Btrfs / LVM RAID | 於 boot sequence 中等待 USB 裝置出現的時窗不足，經常導致 degraded pool 或 failed assemble |
| SnapRAID + mergerfs | SnapRAID 為 parity 式，不符複本偏好，且 parity sync 排程期間的變動具暴露窗[^snapraid] |
| mhddfs | 已於 2024-11 自 Debian testing 移除，原作者直接標示「PLEASE DON'T USE THIS — buggy, security issues」，自 2012 停止維護[^mhddfs] |
| unionfs / aufs | 兩者僅為 overlay union，不提供任何複製或鏡像機制，檔案只存於單一分支 |
| GlusterFS / Ceph / MooseFS 等 | 分散式檔案系統，對單一 DAS 主機場景屬過度設計，需額外網路 daemon、擇 leader、split-brain 等複雜度 |

## 總結建議

| 場景 | 推薦方案 |
|---|---|
| 最大相容性、最低複雜度 | mergerfs（pooling） + Lsyncd（即時複本） |
| 極簡、唯讀多媒體池 | mergerfs（pooling） + rsync cron（排程複本） |
| 需 SMB 網路存取（NAS 情境） | Greyhole |
| 重視版本化同步 | mergerfs（pooling） + Syncthing（複本同步） |

以「USB 開機延遲掛載、不同容量硬碟、複非奇偶試錯」三條核心制約下，**mergerfs + Lsyncd / rsync 為最能同時滿足所有條件的實作組合**。

---

[^mergerfs-mount-timeout]: trapexit. (2025). mergerfs documentation — mount options: `branches-mount-timeout`. Retrieved 2026-10-01, from https://trapexit.github.io/mergerfs/latest/  

[^mergerfs-nonraid]: trapexit. (2025). mergerfs documentation — Non-features: Redundancy / RAID. Retrieved 2026-10-01, from https://trapexit.github.io/mergerfs/latest/overview/non-features/  

[^mergerfs-rsync]: trapexit. (2024). mergerfs discussion #1378 — maintainer recommends rsync/unison for redundancy. Retrieved 2026-10-01, from https://github.com/trapexit/mergerfs/discussions/1378  

[^greyhole]: Boudreau, G. (2024). Greyhole — Samba-based storage pool with file replication. Retrieved 2026-10-01, from https://github.com/gboudreau/Greyhole  

[^snapraid]: SnapRAID project. (2024). snapraid.it — Snapshot RAID with parity. Retrieved 2026-10-01, from https://www.snapraid.it/  

[^mhddfs]: Debian BTS. (2024). mhddfs removed from testing. Retrieved 2026-10-01, from https://tracker.debian.org/news/1585271/mhddfs-removed-from-testing/