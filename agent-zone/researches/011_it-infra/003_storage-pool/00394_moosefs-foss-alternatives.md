# MooseFS FOSS 替代方案研究報告

## 研究目的

尋找 Moose File System (MooseFS) 的自由開源替代方案，並評估各方案之社群版/開源版是否具備完整功能，或功能受限制而需要付費的「企業版」才能使用。

## 概述

MooseFS 是一套成熟的分散式檔案系統，但其**社群版 (Community Edition) 功能有限** — 僅提供單一 active master server（透過 metalogger 手動容錯）、僅支援單同位元的 erasure coding（XOR），而自動容錯 master、多站點叢集、進階 EC（最多 9 同位元）、原生 Windows 用戶端等功能均在 **PRO Enterprise 付費版本**中[^moosefs-pro-vs-community]。

本報告尋找功能完整且無功能限制之 FOSS 替代方案，涵蓋 Ceph、GlusterFS、Lustre、OrangeFS、BeeGFS、OpenIO、XtreemFS 等系統。

---

## 各方案詳細評估

### 1. Ceph

**授權／版本架構：** LGPL 2.1/3.0，**100% FOSS，無社群版／企業版之分**。依據官方 FAQ：「Ceph delivers enterprise storage capabilities as free and open source software on hardware you choose, with no license fees and no lock-in。」[^ceph-faq] Red Hat、IBM、SUSE 等提供付費技術支援，但軟體本身所有功能一致。

**功能要點：**
- **儲存類型：** 物件 (RGW, S3 相容)、區塊 (RBD, 可掛載為原生區塊裝置)、檔案 (CephFS) — 三合一統一平台
- **複製 (Replication):** 預設 3 份，可依 storage pool 自訂
- **Erasure coding:** 支援 4+2、6+3、8+3 等多種 profile
- **POSIX 相容性：** 完整 — CephFS 透過 FUSE 或核心驅動提供完整 POSIX 語意
- **單點故障：** 無 — CRUSH 演算法全分散式中繼資料，無集中式 master
- **快照：** 支援 (檔案系統層級)
- **自我修復：** 自動偵測並恢復節點／磁碟故障
- **加密：** 內建靜態加密
- **監控：** 內建儀表板 + Prometheus 整合

**成熟度/穩定度：** ⭐⭐⭐⭐⭐ 極度成熟。全球生產環境廣泛使用，從小規模叢集到 PB/PB 級部署。由 Ceph Foundation（Linux Foundation 旗下）維護，多個企業贊助（Red Hat/IBM、Clyso、Croit、SUSE），活躍開發中[^ceph-wiki]。

**注意事項：** 學習曲線陡峭；建議 10 GbE+ 網路；最小 3 節點生產部署，5+ 為建議。

---

### 2. GlusterFS

**授權／版本架構：** GPLv3，**FOSS 功能完整**，無付費功能鎖定。Red Hat Gluster Storage (RHGS) 為付費支援方案，但軟體本身與社群版一致。然而 Red Hat 已於 **2024 年 12 月 31 日終止 RHGS 支援**，上游開發趨於停滯[^glusterfs-dead]。

**功能要點：**
- **儲存類型：** 僅檔案系統 — 無區塊或物件儲存
- **複製：** 支援 (replicated volumes, 非同步 geo-replication)
- **Erasure coding:** ❌ 無原生支援（dispersed volumes 為實驗性質）
- **POSIX 相容性：** 良好 — 透過 FUSE、NFS v3 或原生協定
- **單點故障：** 無 — 一致性雜湊 (elastic hash) 架構，無獨立中繼資料伺服器
- **快照：** 支援 (v3.6+ 使用者可操作)
- **Quotas：** 支援
- **隨插即用翻譯層：** 可組合 replicate/distribute/encrypt/compress 等 translator

**成熟度/穩定度：** ⭐⭐⭐ 程式碼成熟但有**終止風險**。Big Iron 2026 指南明確警告：「Don't start new GlusterFS deployments. Full stop。」[^glusterfs-2026] 安全更新緩慢，發行版套件可能消失。

**注意事項：** 小檔案效能差；設定靜態（難以 rebalance）；Red Hat 已將儲存策略轉移至 Ceph。

---

### 3. Lustre

**授權／版本架構：** GPLv2，**完全 FOSS，無社群版／企業版之分**。社群版由 Whamcloud/DDN、HPE、OpenSFS 維護，包含所有功能。DDN EXAScaler、HPE ClusterStor 等為付費支援及認證硬體方案，但軟體功能一致[^lustre-wiki]。

**功能要點：**
- **儲存類型：** 僅平行檔案系統（但支援 HSM 分層存儲、Hadoop 介面）
- **複製：** 支援 — File Level Redundancy (FLR) 提供 mirroring/RAID 0+1
- **Erasure coding:** 開發中，尚未達正式版本
- **POSIX 相容性：** 完整 — 透過核心 VFS 驅動
- **單點故障：** 無 — DNE (Distributed Namespace Environment) 允許多個 active MDS 自動目錄平衡，HA standby 機制
- **GPU Direct Storage:** 支援 — 零拷貝 RDMA 直接到 GPU 記憶體 (NVIDIA GDS)
- **HSM:** 支援 — 可歸檔至磁帶/S3/Google Drive
- **快照：** 支援（需 ZFS 後端）
- **加密：** fscrypt 用戶端加密
- **擴展性：** 數萬用戶端，數百 PB，>1 TB/s 吞吐量

**成熟度/穩定度：** ⭐⭐⭐⭐⭐ 始於 1999 年。全球前 10 大超級電腦中有 6 台使用 Lustre（包括 Frontier #1, 700 PB, 13 TB/s 的 Orion 檔案系統）。多個企業贊助（DDN/Whamcloud、HPE、AWS、Azure），活躍開發中（v2.17 2025 年 12 月）[^lustre-org]。

**注意事項：** HPC 取向 — 部署及調校需專業知識；常用 InfiniBand/OmniPath 網路；非小型叢集適用；核心驅動不在主流核心內。

---

### 4. OrangeFS

**授權／版本架構：** LGPL / Apache 2，**「one single version that is open source」** — 沒有社群版／企業版之分。Omnibond 提供付費支援，但軟體全部功能一致[^orangefs-products]。

**功能要點：**
- **儲存類型：** 平行檔案系統（多種存取方式：核心、FUSE、Direct Interface、Java/HCFS）
- **複製：** ⚠️ 僅支援**不可變更檔案 (immutable files)**，不適用於正在寫入或修改的檔案
- **Erasure coding:** ❌ 無原生支援 — 主要為 striping (RAID0-like)
- **POSIX 相容性：** 支援 — 可透過核心用戶端（Linux 核心 4.6+）或 FUSE
- **單點故障：** 無 — 分散式中繼資料，無狀態伺服器
- **分散式目錄中繼資料：** 支援（v2.9+）
- **用戶端：** 原生 Windows、Hadoop/Spark (透過 JNI)、WebDAV、S3 (透過 Apache module)
- **MPI-IO 支援：** 優化

**成熟度/穩定度：** ⭐⭐⭐⭐ 從 PVFS（1993）演化而來，OrangeFS 分支始於 2007。美國國家實驗室及 HPC 中心使用。Linux 核心 4.6+ 原生包含。v2.10.1 2025 年 9 月釋出，仍活躍維護中[^orangefs-wiki]。

**注意事項：** 可變更檔案無複製 — 與 MooseFS 相比對一般用途為明顯差距。HPC 取向設計。社群相對較小。

---

### 5. BeeGFS

**授權／版本架構：** GPL，**完全 FOSS，無功能等級之分**。ThinkParallel 提供付費顧問及支援，但軟體版本一致[^beegfs-wiki]。

**功能要點：**
- **儲存類型：** 平行檔案系統
- **複製：** 支援（用戶端端複製 + 內建複製）
- **Erasure coding:** ❌ 無原生支援
- **POSIX 相容性：** 支援（FUSE + 核心驅動，Linux 核心 5.x+）
- **單點故障：** 無 — 獨立中繼資料伺服器（自動複製實現 HA）
- **擴展性：** 10,000+ 用戶端（Fraunhofer 測試）
- **關鍵優勢：** 部署比 Lustre 及 Ceph 簡單許多 — 專為較簡單管理而設計
- **功能：** 快照、ACL、配額、分層儲存（budget-based storage pools）

**成熟度/穩定度：** ⭐⭐⭐⭐ 始於 2005 年。HPC 及企業環境使用中，正在被納入 Linux 核心。活躍社群[^beegfs]。

**注意事項：** 無 erasure coding；HPC 取向。

---

### 6-8. 已終止或不相關之方案

以下方案**不建議用於新部署**：

**OpenIO SDS**[^openio-wiki]：
- AGPL3/LGPL3，完全 FOSS，無功能限制
- 但已於 2020 年被 OVHcloud 收購後停止作為獨立產品開發
- 為物件儲存（非 POSIX），不能作為 MooseFS 的 POSIX 檔案系統替代

**XtreemFS**[^xtreemfs-wiki]：
- New BSD License，完全 FOSS，無功能限制
- 最後一個版本為 **v1.5.1（2015 年 3 月 12 日）** — 超過 11 年無更新
- 事實上已棄置

**GlusterFS**（如上所述）：
- Red Hat 已終止支援，不建議新部署

---

## 綜合比較表

| 功能 | Ceph | GlusterFS | MooseFS (社群) | Lustre | OrangeFS | BeeGFS |
|------|------|-----------|----------------|--------|-----------|--------|
| **FOSS 功能完整？** | ✅ | ✅ (但已死) | ❌ (PRO 版有 HA/多站點/進階 EC/Windows 用戶端) | ✅ | ✅ | ✅ |
| **區塊儲存** | ✅ RBD | ❌ | ❌ | ❌ | ❌ | ❌ |
| **物件儲存 (S3)** | ✅ RGW | ❌ | ❌ | ❌ | ❌ | ❌ |
| **POSIX 檔案** | ✅ CephFS | ✅ | ✅ | ✅ | ✅ | ✅ |
| **複製 (Replication)** | ✅ 3x 預設 | ✅ | ✅ 每目錄設定 | ✅ FLR 鏡像 | ⚠️ 唯不可變更檔案 | ✅ |
| **Erasure Coding** | ✅ 4+2, 6+3 等 | ❌ | ⚠️ 單一 Parity | ⚠️ 開發中 | ❌ | ❌ |
| **無單點故障** | ✅ ✅ CRUSH | ✅ 一致性雜湊 | ❌ 單一 master | ✅ MDS HA | ✅ 分散式 MDS | ✅ MDS HA |
| **2026 活躍開發？** | ✅ 非常活躍 | ⚠️ 維護模式 | ✅ 活躍 (PRO) | ✅ 非常活躍 | ✅ 中度 | ✅ 活躍 |
| **小檔案效能** | 良好 | 差 | 🏆 優異 | 差 | 中度 | 中度 |
| **學習曲線** | 陡峭 | 中度 | 低 | 陡峭 | 中度 | 中度 |
| **最小叢集** | 3+ 節點 | 2+ 節點 | 2+ 節點 | 5+ 節點 | 3+ 節點 | 3+ 節點 |

---

## 結論與建議

### 最推薦：Ceph

**Ceph** 是唯一在開源版中提供**全部功能**（複製 + erasure coding + POSIX + 區塊/物件/檔案三合一 + 無單點故障）的系統，沒有付費功能鎖定。其功能完整超越 MooseFS Community 版，且開發最活躍、企業支援最廣泛。缺點是學習曲線陡峭。

### 若簡易性優先

若可接受 MooseFS Community 版之限制（手動容錯、單一 parity EC），**繼續使用 MooseFS Community** 是最簡單的選擇。

### 若需要 HPC 級平行檔案系統

**Lustre**（功能完整，無功能鎖定，最成熟）或 **BeeGFS**（部署更簡單）為適合選項。

### 不建議新部署

GlusterFS（已死亡）、XtreemFS（2015 年停滯）、OpenIO（2020 年收購停滯，且物件儲存不具 POSIX）。

---

## Reference

[^moosefs-pro-vs-community]: MooseFS. (n.d.). PRO vs Community Edition. Retrieved 2026-10-03, from https://moosefs.com/pro-vs-community.html
[^ceph-faq]: Ceph. (n.d.). Frequently Asked Questions. Retrieved 2026-10-03, from https://ceph.io/en/users/faq/
[^ceph-wiki]: Wikipedia. (2026). Ceph (software). Retrieved 2026-10-03, from https://en.wikipedia.org/wiki/Ceph_(software)
[^glusterfs-dead]: Big Iron. (2026). The State of GlusterFS After RedHat Killed It. Retrieved 2026-10-03, from https://www.bigiron.cc/guides/the-state-of-glusterfs-after-redhat-killed-it-2026
[^glusterfs-2026]: Pi Stack. (2026). Ceph vs GlusterFS vs MooseFS — Distributed File Storage Comparison. Retrieved 2026-10-03, from https://www.pistack.xyz/posts/ceph-vs-glusterfs-vs-moosefs-distributed-file-storage-2026/
[^lustre-wiki]: Wikipedia. (2026). Lustre (file system). Retrieved 2026-10-03, from https://en.wikipedia.org/wiki/Lustre_(file_system)
[^lustre-org]: OpenSFS. (n.d.). Lustre Home. Retrieved 2026-10-03, from https://www.lustre.org/
[^orangefs-wiki]: Wikipedia. (2026). OrangeFS. Retrieved 2026-10-03, from https://en.wikipedia.org/wiki/OrangeFS
[^orangefs-products]: OrangeFS. (n.d.). Products & Support. Retrieved 2026-10-03, from http://orangefs.com/orangefs-products/
[^beegfs-wiki]: Wikipedia. (2026). BeeGFS. Retrieved 2026-10-03, from https://en.wikipedia.org/wiki/BeeGFS
[^beegfs]: BeeGFS. (n.d.). Official Site. Retrieved 2026-10-03, from https://www.beegfs.io/
[^openio-wiki]: Wikipedia. (2026). OpenIO. Retrieved 2026-10-03, from https://en.wikipedia.org/wiki/OpenIO
[^xtreemfs-wiki]: Wikipedia. (2026). XtreemFS. Retrieved 2026-10-03, from https://en.wikipedia.org/wiki/XtreemFS
[^orangefs-faq]: OrangeFS. (n.d.). FAQ. Retrieved 2026-10-03, from http://www.orangefs.org/faq/