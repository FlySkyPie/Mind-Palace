# 異質硬碟 USB DAS 儲存池方案比較：Ceph、GlusterFS、Lustre 與替代方案

## 問題背景

一臺「8Bay SATA3 to USB 3.0 DAS」（Direct Attached Storage）搭載多顆不同容量、轉速、品牌的硬碟（異質硬碟），連接到單一主機。目標是將這些硬碟組合成一個更大的、具備冗餘（Redundancy）的統一儲存池。

不考慮的方案：ZFS、LVM。

本報告評估候選方案：Ceph、GlusterFS、Lustre，以及一個更務實的替代方案 mergerfs + SnapRAID。

---

## 架構關鍵區別：分散式 vs 單機

Ceph、GlusterFS、Lustre 均是**分散式叢集檔案系統**，設計目標是多節點、高速網路互連、不同主機各自提供儲存。而 8Bay USB DAS 是**單一主機、同一條 USB 匯流排**的情境。

| 方案 | 架構類型 | 最小節點數 | 適合單機 USB DAS？ |
|---|---|---|---|
| Ceph | 分散式物件/區塊/檔案儲存 | 3 節點（官方最小建議） | ❌ |
| GlusterFS | 分散式叢集檔案系統 | 2+ 節點（複製需多節點） | ❌ |
| Lustre | HPC 平行分散式檔案系統 | 3+ 節點（MGS/MDS/OSS） | ❌ |
| mergerfs + SnapRAID | 聯合檔案系統 + 快照奇偶校驗 | 1 節點 | ✅ |

---

## 各方案評估

### Ceph

Ceph 是一個分散式物件儲存系統（RADOS），設計以多個獨立伺服器透過網路協同工作[^ceph-doc]。

**硬碟異質性支援：** Ceph 透過 CRUSH 權重機制將不同容量的 OSD 納入同一個 pool。每個 OSD 可以分配不同的權重，CRUSH 演算法按比例分配資料[^crush-map]。但官方文件提醒：「Ceph 在同一個 pool 中使用統一硬體時效果最佳。」[^ceph-osd-doc]

**冗餘方式：**
- **複製（Replication）**：預設 size=3（3 份副本，200% 開銷），可設為 size=2（鏡像，100% 開銷）。
- **抹除碼（Erasure Coding）**：支援 k+m 配置（如 k=4+m=2 開銷 1.5×），但需要至少 k+m 個故障域（通常為主機節點）[^ceph-ec]。

**USB DAS 相容性問題：**
1. `ceph-volume` 會主動封鎖卸除式裝置（removable USB devices），需手動修補 Python 原始碼才能繞過[^ceph-bug-38833]。
2. 實際部署測試顯示：單一 USB OSD 加入 NVMe 叢集後，整體 IOPS 從 ~3000 暴跌至 ~659（-78%），因為一個慢速 OSD 拖垮所有 PG[^proxmox-usb]。
3. USB 匯流排共享造成還原（backfill）速度僅 ~6–13 MiB/s，2.2TiB OSD 還原需 **4–5 天**[^mcgarrah]。
4. 所有 OSD 在同一主機、同一 USB 控制器下，Ceph 的冗餘優勢（跨越獨立故障域）完全失去意義。

**結論：** Ceph 原則上能用，但效能極差、單點故障問題嚴重、需要修補原始碼。**不推薦。**

---

### GlusterFS

GlusterFS 是無後設資料伺服器的分散式檔案系統，使用基於雜湊的資料配置（hash-based data location）[^gluster-doc]。

**硬碟異質性支援：** 分散式磁碟區（distributed volume）可用不同大小的 brick。但無自動權重機制，配置不均勻時需人工調整。

**冗餘方式：**
- **複製（Replication）**：replica 2 或 3 模式，但會產生 split-brain 問題且需手動處理[^gluster-splitbrain]。
- **分散式複製（Distributed-Replicated）**：結合 distribution 與 replication。
- **抹除碼（EC）**：後期版本支援，但成熟度低於 Ceph。

**USB DAS 相容性問題：**
1. GlusterFS 假設穩定、低延遲的網路互連。USB 3.0 的傳輸不穩定（瞬間斷線、控制器重設）會破壞 GlusterFS 的內部複製狀態。
2. 單一節點執行 GlusterFS 理論可行，但既無分散好處也無冗餘實際效益（所有 brick 在同一主機上），只增加管理複雜度。
3. 在 USB 斷線時進行寫入可能導致檔案系統損毀。

**結論：** GlusterFS 的設計出發點與 USB DAS 場景完全不符。**不推薦。**

---

### Lustre

Lustre 是高效能運算（HPC）領域的平行分散式檔案系統，典型部署在超級電腦上[^lustre-wiki]。

**硬碟異質性支援：** 支援 OST Pool 機制，可將不同效能的 OST 分組。但效能最佳化要求 OST 容量一致，否則最小 OST 成為瓶頸。

**冗餘方式：** Lustre **不提供檔案系統層級的冗餘**，官方文件強烈建議使用底層硬體 RAID（RAID-6）[^lustre-manual]。FLR（File-Level Replication）存在但無人維護。

**USB DAS 相容性問題：**
1. Lustre 需要專屬 MGS/MDS 與 OSS 節點，即使全部在同一臺機器，仍需設定完整 Lustre Networking（LNet）堆疊。
2. 官方需求：MDS 至少 2GB RAM（測試），生產建議 64–128GB+。對 USB 儲存場景極度浪費。
3. 無內建冗餘 + 硬碟直接掛載 = 任何一顆硬碟故障就造成資料完全遺失。
4. USB 控制器問題會導致 OST 被強制踢出（約 100 秒逾時）。
5. TRIM/SMART 穿透性差，不利健康監控。

**結論：** Lustre 是為超級電腦設計的工具，不適合單機 USB DAS。**極度不推薦。**

---

## 務實方案：mergerfs + SnapRAID

這是針對「單一主機、多顆異質 USB 外接硬碟、需要冗餘」場景唯一合理的選擇。

### mergerfs

mergerfs 是一個 FUSE 聯合檔案系統（union filesystem），將多個掛載點（路徑）組合成一個統一目錄[^mergerfs]。

- **支援任意混搭硬碟**：不同容量、品牌、檔案系統（ext4、XFS、NTFS 皆可）[^mergerfs-faq]。
- **無容量浪費**：每顆硬碟的完整容量都貢獻給 pool。
- **獨立掛載**：拔掉任何一顆硬碟，插到其他 Linux 機器可直接讀取該硬碟的資料。此為 Ceph/GlusterFS/Lustre 無法做到的關鍵優勢。
- **故障隔離**：一顆硬碟故障只影響該硬碟上的檔案，pool 其餘部分正常運作。
- **掛載政策**：支援 mfs（最多剩餘空間）、epmfs（既有路徑最多空間）、pfrd（比例隨機）等[^mergerfs-policies]。

### SnapRAID

SnapRAID 提供基於快照（snapshot）的奇偶校驗與位元腐蝕（bitrot）保護[^snapraid-how]。

- **奇偶校驗計算**：使用 XOR + Reed-Solomon，支援 1–6 顆奇偶校驗硬碟。
- **完整性檢查**：每個檔案儲存 128-bit SpookyHash 檢查碼，能精確定位哪顆硬碟的資料已損毀（而非只知道「有錯誤」）。
- **位元腐蝕修復**：scrub 指令重新計算檢查碼，不一致時用奇偶校驗資料重建正確內容。
- **硬碟異質性**：唯一的限制是奇偶校驗硬碟 ≥ 最大資料硬碟容量。資料硬碟之間可任意組合。

### 關鍵取捨

| 面向 | 優勢 | 劣勢 |
|---|---|---|
| 即時性 | 硬碟獨立可讀、故障隔離佳 | 奇偶校驗非即時（排程執行），兩次 sync 間的資料無保護 |
| 容量 | 零浪費、任意混搭 | 奇偶校驗硬碟必須 ≥ 最大資料硬碟 |
| 資源 | 極低（512MB RAM 即可運行） | FUSE 開銷對小隨機寫入有影響 |
| 維護 | 擴容簡單（掛載 → mergerfs 重新掛載 → snapraid sync） | 需自行設定 systemd 排程、snapraid.conf、fstab |
| 適用場景 | 媒體收藏、備份、歸檔類寫一次讀多次（WORM） | 不適合資料庫、VM 映像、即時編輯 |

典型配置流程[^diy-mediaserver]：

1. 每顆硬碟格式化為 ext4
2. `/etc/fstab` 加入掛載條目（含 `nofail`）
3. 用 mergerfs 將全部掛載點合併為單一目錄
4. 設定 `snapraid.conf` 指定資料硬碟與奇偶校驗硬碟
5. 執行 `snapraid sync` 建立奇偶校驗
6. 設定 systemd timer 每日自動 sync + scrub

---

## 結論

| 方案 | 異質硬碟支援 | 冗餘 | USB 友善度 | 複雜度 | 推薦 |
|---|---|---|---|---|---|
| Ceph | ✅（CRUSH 權重） | ✅ 複製/EC | ❌（需修補、USB 匯流排問題） | 極高 | ❌ |
| GlusterFS | ✅（不同 size brick） | ✅ 複製/EC | ❌（斷線破壞複製狀態） | 高 | ❌ |
| Lustre | ⚠️（OST Pool） | ⚠️（僅底層 RAID） | ❌（無內建冗餘、硬體要求高） | 極高 | ❌ |
| **mergerfs + SnapRAID** | **✅ 最佳** | **✅ 快照奇偶校驗 1–6 硬碟** | **✅ 設計即為此場景** | **低** | **✅** |

對於「8Bay SATA3 to USB 3.0 DAS」配異質硬碟、不考慮 ZFS/LVM 的情境，**mergerfs + SnapRAID 是唯一合理選擇**。Ceph、GlusterFS、Lustre 是分散式叢集方案，其架構前提（多節點、網路互連、獨立故障域）與單機 USB DAS 完全牴觸。

---

## 參考資料

[^ceph-doc]: Ceph. (n.d.). Ceph Documentation — Adding/Removing OSDs. Retrieved 2026-10-03, from https://docs.ceph.com/en/latest/rados/operations/add-or-rm-osds/
[^crush-map]: Ceph. (n.d.). CRUSH Map Documentation. Retrieved 2026-10-03, from https://docs.ceph.com/en/latest/rados/operations/crush-map/
[^ceph-osd-doc]: Ceph. (n.d.). Hardware Recommendations. Retrieved 2026-10-03, from https://docs.ceph.com/en/latest/start/hardware-recommendations/
[^ceph-ec]: Ceph. (n.d.). Erasure Code Documentation. Retrieved 2026-10-03, from https://docs.ceph.com/en/latest/rados/operations/erasure-code/
[^ceph-bug-38833]: Ceph Bug Tracker. (2020). Issue #38833: ceph-volume doesn't support removable USB devices. Retrieved 2026-10-03, from https://tracker.ceph.com/issues/38833
[^proxmox-usb]: Proxmox Forum. (2023). FYI do not extend Ceph with OSDs connected via USB. Retrieved 2026-10-03, from https://forum.proxmox.com/threads/fyi-do-not-extend-ceph-with-osds-connected-via-usb.140975/
[^mcgarrah]: McGarrah. (n.d.). WAL vs DB Performance on USB Ceph OSDs. Retrieved 2026-10-03, from https://mcgarrah.org/ceph-wal-vs-db-performance-test/
[^gluster-doc]: Gluster. (n.d.). Gluster Documentation — Quick Start Guide. Retrieved 2026-10-03, from https://docs.gluster.org/en/latest/Quick-Start-Guide/
[^gluster-splitbrain]: Gluster. (n.d.). Administrator Guide — Split-brain and Ways to Deal with It. Retrieved 2026-10-03, from https://docs.gluster.org/en/latest/Administrator-Guide/Split-brain-and-ways-to-deal-with-it/
[^lustre-wiki]: Lustre. (n.d.). Lustre Quick Start Guide. Retrieved 2026-10-03, from https://wiki.lustre.org/Lustre_Quick_Start_Guide
[^lustre-manual]: VatslavDS. (n.d.). Lustre Manual — Configuring Storage on a Lustre File System. Retrieved 2026-10-03, from https://github.com/VatslavDS/lustre-manual/blob/master/02.03-Configuring%20Storage%20on%20a%20Lustre%20File%20System.md
[^mergerfs]: trapexit. (n.d.). mergerfs — GitHub README. Retrieved 2026-10-03, from https://github.com/trapexit/mergerfs
[^mergerfs-faq]: mergerfs. (n.d.). mergerfs.com — FAQ. Retrieved 2026-10-03, from https://mergerfs.com/
[^mergerfs-policies]: DIY Media Server. (2026). mergerfs Media Servers. Retrieved 2026-10-03, from https://diymediaserver.com/post/2026/mergerfs-media-servers-2026/
[^snapraid-how]: SnapRAID. (n.d.). How It Works. Retrieved 2026-10-03, from https://www.snapraid.it/howitworks
[^diy-mediaserver]: DIY Media Server. (2026). mergerfs + SnapRAID on Linux. Retrieved 2026-10-03, from https://diymediaserver.com/post/2026/mergerfs-media-servers-2026/