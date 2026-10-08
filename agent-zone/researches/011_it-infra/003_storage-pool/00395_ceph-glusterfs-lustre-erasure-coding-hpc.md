# Ceph、GlusterFS、Lustre 比較備註「無 erasure coding；HPC 取向」之意涵

## 背景

在分散式儲存系統的比較表中，經常看到 Ceph、GlusterFS、Lustre 三者並列。其中針對 Lustre 的備註欄常出現 **「無 erasure coding；HPC 取向」** 一語。本報告說明此備註的技術意涵與背景。

## 三者對 erasure coding 的支援

### Ceph — 完整支援

Ceph 原生支援 erasure code（糾刪碼）池，做為與副本複製（replication）並列的資料保護方式。Ceph 的 EC 功能豐富且生產就緒，支援多種 plugin（ISA-L、Jerasure、LRC、SHEC 等），可靈活配置 k（資料塊數）與 m（校驗塊數）參數，並自 Luminous 版起支援 EC Overwrites（部分覆寫）。[^ceph-ec]

### GlusterFS — 支援

GlusterFS 自 3.6 版起透過 **Dispersed Volume（分散式卷/糾刪碼卷）** 支援 erasure coding。資料經 GF(2⁸) 上線性編碼，產生 k 個資料片段與 m 個冗餘片段，分散至 n 個 brick。配置彈性為 `1 ≤ m < k`，空間效率約 `k/(k+m)`，優於多副本。[^gluster-ec]

### Lustre — 原生不支援

Lustre **原生不支援 erasure coding**。中科院高能物理研究所（IHEP）的 Lustre 說明文件明確指出：「由於 LUSTRE 元數據伺服器的限制，在管理海量小文件時不具有優勢，且**不支持纠删码、副本等功能**，因此 LUSTRE 文件系統被主要用於海量的實驗數據存儲。」[^ihep-lustre]

Lustre 依賴條帶化（striping）分佈於多個 OSS 來實現高頻寬並行，資料保護主要透過基於共享儲存的 HA 方案或檔案層級冗餘（File Level Redundancy, FLR）來達成，而非糾刪碼。[^clug-ec]

社群方面，Lustre 正在開發 FLR-ECRO（File Level Erasure Coding Read-Only），目標為 Lustre 2.18，但此功能仍處於高層設計階段，尚未生產可用。[^lustre-ecro]

## HPC 取向

Lustre 是**專為高效能運算（HPC）設計的並行分散式檔案系統**。Lustre 官方網站說明：「The Lustre® file system is an open-source, parallel file system that supports many requirements of leadership class HPC simulation environments。」[^lustre-about] 它驅動全球多數 TOP500 超級電腦（包括 Frontier、Perlmutter、Summit），自 2005 年以來持續被至少半數前十名超級電腦採用。[^wikipedia-lustre]

相比之下，Ceph 是通用型分散式儲存系統（統一區塊/檔案/物件），GlusterFS 是無元伺服器的橫向擴展網路檔案系統，二者均非專為 HPC 打造。

## 綜合解讀

備註 **「無 erasure coding；HPC 取向」** 在比較表中是對 **Lustre** 的定性描述，包含兩個關鍵差異點：

| 面向 | 意涵 |
|------|------|
| **無 erasure coding** | Lustre 原生不支援糾刪碼，資料保護依賴硬體 RAID 或 FLR 副本；Ceph 與 GlusterFS 皆原生支援 EC |
| **HPC 取向** | Lustre 專為 HPC/超級電腦場景設計，追求最大化順序吞吐量與並行 MPI-IO；Ceph 與 GlusterFS 為通用型儲存，非 HPC 專用 |

此備註的言下之意是：Lustre **犧牲了 erasure coding 支援**，以換取**原始的 HPC 吞吐效能**——這是一項取捨設計（trade-off），而非功能缺失。

---

[^ceph-ec]: Ceph Documentation. (n.d.). Erasure code. Retrieved 2026-10-05, from https://docs.ceph.com/en/latest/rados/operations/erasure-code/
[^gluster-ec]: Gluster Documentation. (n.d.). Setting Up Volumes — Creating Dispersed Volumes. Retrieved 2026-10-05, from https://docs.gluster.org/en/latest/Administrator-Guide/Setting-Up-Volumes/
[^ihep-lustre]: 中國科學院高能物理研究所計算中心. (n.d.). Lustre 檔案系統. Retrieved 2026-10-05, from http://afsapply.ihep.ac.cn/cchelp/zh/local-cluster/storage/Lustre/
[^clug-ec]: 許震宇. (2020). Lustre 文件系統的文件級副本和糾刪碼. CLUG 2020 大會. Retrieved 2026-10-05, from http://lustrefs.cn/wp-content/uploads/2020/10/CLUG2020_%E8%AE%B8%E9%9C%87%E5%AE%87_Lustre%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F%E7%9A%84%E6%96%87%E4%BB%B6%E7%BA%A7%E5%89%AF%E6%9C%AC%E5%92%8C%E7%BA%A0%E5%88%A0%E7%A0%81.pdf
[^lustre-ecro]: Lustre Wiki. (n.d.). Erasure Coding Read-Only High Level Design. Retrieved 2026-10-05, from https://wiki.lustre.org/Erasure_Coding_Read-Only_High_Level_Design
[^lustre-about]: Lustre. (n.d.). About the Lustre® File System. Retrieved 2026-10-05, from https://www.lustre.org/about/
[^wikipedia-lustre]: Wikipedia. (n.d.). Lustre (file system). Retrieved 2026-10-05, from https://en.wikipedia.org/wiki/Lustre_(file_system)