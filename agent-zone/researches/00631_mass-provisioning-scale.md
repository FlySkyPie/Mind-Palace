# 大量安裝工具所指的「大量」是哪種量級？

探討 FAI、Cobbler、Foreman、MAAS、Clonezilla 等「大量佈署（mass deployment）」工具時，常在文件裡看到「大量」「大規模」「可管理數千至數萬台」這類描述。本文整理這些工具官方文件、專案 README 與使用者案例中實際出現的數字，釐清「大量」到底對應哪一種量級，並區分兩種截然不同的計量方式。

## 摘要

「大量」在這類工具中並不是單一數字，而是取決於工具所解決的問題屬於下列哪一類：

| 類別 | 代表工具 | 「大量」計量的是什麼 | 文件常見量級 |
|:---|:---|:---|:---|
| 映像佈署（imaging / cloning） | Clonezilla（SE、lite server）、DRBL | **同時（併發）**被複製的機器數 | 數十台／批次，單次 40 台以上[^cz-home] |
| 安裝／佈署伺服器（provisioning） | FAI、Cobbler、MAAS、Foreman | **被納管、被安裝的機群總數**（fleet size） | 數百至數萬台[^fai-feat][^cobbler-home][^foreman-readme][^maas-perf] |

換句話說，Clonezilla 的「大量」指「一次同時對多少台機器做映像」，而 FAI、Cobbler、Foreman、MAAS 的「大量」指「長期要管多少台機器」。兩者的天花板、瓶頸與擴充方式完全不同，混用會導致錯誤的容量規劃。

## 一、映像佈署類：以「同時併發數」計

Clonezilla 官方首頁描述其伺服器版本時寫道：

> Clonezilla lite server 或 SE 適合大量佈署，可**同時複製許多（40 台以上！）電腦**。[^cz-home]

同頁也給出一個具體效能基準：在一個 **42 節點**的叢集、使用高階硬體的情況下，multicast 還原速率達 **8 GB/min**。[^cz-home]

官方 step-by-step 文件本身即以「一間 **40 台電腦**的教室」為範例：從一台機器存出映像，再還原到其餘 **39 台**。[^cz-se]

使用者實測案例（官方 testimonials 頁）：

- **1,084 台**：以 DRBL 搭配 multicast，並將機器分成 **每批 80–100 台**，每批耗時約 16–38 分鐘。[^cz-testi]
- **64 台**：Cisco Systems 以 multicast 同時對 64 台機器佈署 5 GB 映像，5 分鐘內完成。[^cz-testi]
- **20 台**：曼徹斯特某單位以 16 GB 映像對 20 台 PC 做 multicast，耗時 55 分鐘。[^cz-testi]
- **173 台筆電**：以 Clonezilla 伺服器在 3 天內完成，且可跨 25 個子網、20 個辦公室外遠端佈署。[^cz-testi]

DRBL 首頁則以「40、30 甚至 10 台客戶端」作為典型描述框架。[^drbl-home]

**重點**：映像類工具的瓶頸是**網路頻寬與交換器**（單一 Gigabit 埠、multicast / IGMP snooping 設定），而不是資料庫或管理邏輯。因此「大量」在此通常指**單一批次的數十台**，超大規模則靠**分批（每批 80–100 台）**或使用 Clonezilla lite server 的 **BitTorrent** 模式分散負載。[^cz-home][^cz-testi]

## 二、安裝／佈署伺服器類：以「機群總數」計

這一類工具要管理的是機器清單、設定資料、開機設定與授權狀態，因此「大量」指**長期納管的機群規模**。文件敘述如下：

### FAI

官方 Features 頁明言：

> **可擴充（Scalable）**。FAI 使用者管理的電腦基礎設施從**數台到數千台**機器不等。[^fai-feat]

並強調 FAI 輕量：「沒有特殊 daemon 常駐，也不需要資料庫」。[^fai-feat]

官方使用者案例頁（350 份以上回報）出現的規模包括：

- **Memorysolution GmbH**：佈署安裝 **> 10,000** 台裸機伺服器。[^fai-report]
- **Sinh Lam**：目前 **10k+**；其前東家在全球佈署「數萬台（10s of thousands）」伺服器。[^fai-report]
- **Ocado Technology**：**數千台（many thousands）**，用於資料中心與邊緣運算。[^fai-report]
- **Max Planck Institute for Gravitational Physics (AEI)**：目前 **3,000+** 台主機，12 年來累計安裝逾 10,000 次。[^fai-report]
- **Sagemcom**：自 2015 年起生產環境 **> 3,000** 台。[^fai-report]
- **Internet Archive**：**1,197** 台裸機，另有約 800 台 KVM 虛擬機。[^fai-report]
- 大量教育與研究單位落於 **數十至數百台**（如 Sorbonne Université 50+、University of Angers 80、McGill University 275、TU Braunschweig 約 180）。[^fai-report]

### Cobbler

官方首頁的敘述為：

> 自動化是速度、一致性與可重複性的關鍵。無論基礎設施由**少數幾台伺服器**或**數千台伺服器**組成，這些特性都至關重要。[^cobbler-home]

Cobbler 4.0.0 的發行準備 issue 中，主要開發者 SchoolGuy 明確寫下效能目標：

> 「將效能改善到可接受的水準。**目標最大項目數（Item Count）約 500,000**。」[^cobbler-3999]

此目標對應 SUSE Bugzilla 的企業需求。對應的效能 PR #4035 指出「Cobbler 確實隨項目數呈線性（linear）成長」，並警告在**一百萬（1 million）**項目規模下「許多操作會耗時過久」，因此不建議。[^cobbler-4035] 該 PR 的基準測試以約 1,000 項目為 CI 規模、並實測到 100,000 項目。[^cobbler-4035]

（注意：此處「項目（item）」是 Cobbler 內部物件，包含系統、組態檔、發行版、網路介面等，並非等同「機器台數」。因此 500,000 這個數字不宜直接理解為 50 萬台機器。）[^cobbler-3999]

### Foreman

Foreman 官方 GitHub README 寫道：

> Foreman 是一個成熟專案，已在許多組織佈署，**管理從數十台到數萬台（10s to 10000s）的伺服器**。[^foreman-readme]

官方手冊則刻意**不給硬性上限**，只說明硬體需求「主要取決於它會收到多少請求，取決於組態管理客戶端數量、Web UI 活動量及其他使用 API 的系統」。預設安裝（含 Puppet server）僅需 **4 GB 記憶體、2 GB 磁碟**；最小安裝為 **2 GB 記憶體、1 GB 磁碟**。[^foreman-manual]

其下游企業產品 Red Hat Satellite 的效能調校指南則給出**依納管主機數分級的硬體設定檔**，是本主題中最具體的數字來源之一：

| 分級（profile） | 納管主機數 | 建議記憶體 | 建議核心數 |
|:---|:---|:---|:---|
| default | 0–5,000 | 20 GiB | 4 |
| medium | 5,000–10,000 | 32 GiB | 8 |
| large | 10,000–20,000 | 64 GiB | 16 |
| extra-large | 20,000–60,000 | 128 GiB | 32 |
| extra-extra-large | 60,000+ | 256 GiB+ | 48+ |

[^satellite-tuning]

### MAAS

Canonical 官方文件中**唯一明列的硬性上限**是：

> **每台 rack controller 在單一子網路上有 1,000 台機器的上限（1000-machine cap per subnet）。**[^maas-req]

官方效能頁說明其持續測試環境為 **5 台 rack controller、每 fabric 48 台機器**，並**每日在 10、100、1000 台機器**三個規模執行模擬測試。[^maas-perf]

Canonical 官網上的聯絡表單亦以規模分級：Small（1–50 nodes）、Medium（51–250）、Large（251–1000）、Enterprise（1000+），並以 Roblox 為案例，稱其「管理、重建**數千台伺服器**」。[^maas-home]

## 三、維度對照：五個工具的「大量」量級

| 工具 | 文件中最明確的數字 | 「大量」對應的維度 | 主要瓶頸 |
|:---|:---|:---|:---|
| Clonezilla SE / lite server | 同時 **40+** 台；實例分批 80–100 台、單案 1,084 台[^cz-home][^cz-testi] | 同時併發映像數 | 網路頻寬、multicast 設定 |
| DRBL | 典型 10–40 台客戶端[^drbl-home] | 同時併發映像數 | 同上 |
| FAI | 「數台到數千台」；案例出現 >10,000 台[^fai-feat][^fai-report] | 納管機群總數 | 幾乎僅受網路與 NFS 吞吐限制 |
| Cobbler | 「少數幾台到數千台伺服器」；內部目標 ~500,000 項目[^cobbler-home][^cobbler-3999] | 納管機群總數／資料庫項目數 | 序列化後端（JSON/SQLite/MongoDB）、DHCP |
| MAAS | 每 rack controller 每子網 **1,000 台**上限；測試至 1,000 台；實例「數千台」[^maas-req][^maas-perf][^maas-home] | 納管機群總數 | 每子網的 rack controller 容量、PostgreSQL 連線數 |
| Foreman | 「數十台到數萬台」；Satellite 分級到 60,000+ 台[^foreman-readme][^satellite-tuning] | 納管機群總數 | Puma worker 數、Dynflow/Sidekiq、資料庫成長 |

一個實務上的粗略分界可歸納如下：

- **數十台／批次**：映像工具（Clonezilla、DRBL）的「大量」。
- **數百台**：安裝／佈署工具的常見起點，也是教育與中小企業最典型的規模（如 Sorbonne 50+、McGill 275、TU Braunschweig 約 180）。[^fai-report]
- **數千台**：FAI、Cobbler、Foreman、MAAS 文件敘述與多數企業案例的主要落點。[^fai-feat][^cobbler-home][^maas-home]
- **數萬台**：Foreman 宣稱可達的規模，以及 Red Hat Satellite「large」以上的分級；FAI 亦有 >10,000 台的實例。[^foreman-readme][^satellite-tuning][^fai-report]

## 四、對來源的批判性反思

1. **官方行銷數字 vs. 實測數字**：Cobbler 的「~500,000」與 Foreman 的「10000s」屬於**目標或宣稱值**，未必有等比例的可重現基準；Cobbler 的 ~500,000 甚至以「項目（item）」而非「機器」計，且其開發者同時指出 100 萬項目不可行。[^cobbler-3999][^cobbler-4035][^foreman-readme] 因此這些數字應視為數量級（order of magnitude）指引，而非容量保證。

2. **測試規模通常遠小於宣稱上限**：MAAS 每日自動測試只到 **1,000** 台；Cobbler 的 CI 基準為約 **1,000** 項目、實測到 100,000。[^maas-perf][^cobbler-4035] 官方在遠超測試規模處的表現，多半缺乏公開基準。

3. **使用者案例多為自我回報且可能過時**：FAI 與 Clonezilla 的案例來自使用者自填表單或 testimonials，時間橫跨 2007 至 2024 年，工具版本與硬體世代差異極大，數字僅供量級參考。[^fai-report][^cz-testi]

4. **瓶頸往往不在工具本身**：Cobbler 的 issue 提到 dnsmasq 在企業級網路「擴充性不佳」，需改用 ISC DHCP/Kea；MAAS 的硬性上限實際是**每個子網一台 rack controller 1,000 台**，而非 MAAS 整體上限。[^cobbler-4035][^maas-req] 這說明「大量」的真正限制常落於網路與 DHCP 架構，而非工具程式本身。

5. **「同時併發」與「機群總數」不可互換**：Clonezilla 的 40+ 是**同時**數量，Foreman 的 10,000s 是**累計納管**數量。將前者當成後者（或反之）會嚴重誤判所需資源。

## 結論

- 對 **Clonezilla、DRBL** 這類映像工具，「大量」指的是**單一批次同時佈署的機器數**，文件中以 **40 台以上**為常見門檻，實務單案可達約 **1,084 台**（分批 80–100 台執行）。[^cz-home][^cz-testi]
- 對 **FAI、Cobbler、Foreman、MAAS** 這類安裝／佈署伺服器，「大量」指的是**長期納管的機群總數**，文件敘述普遍為「**數台到數千台**」，企業實例與宣稱值可達**數千至數萬台**。[^fai-feat][^cobbler-home][^foreman-readme][^maas-home]
- 最有數字感的兩個官方來源是 Red Hat Satellite 的**依主機數分級硬體設定檔（0–5,000 一路到 60,000+）**與 MAAS 的**每子網 rack controller 1,000 台上限**。[^satellite-tuning][^maas-req]
- 選擇工具與規劃容量時，應先確認自己在意的是「一次要同時打幾台」還是「總共要管幾台」，再對照該工具的瓶頸（網路頻寬 vs. 資料庫／後端），而非直接套用行銷文案中的最大數字。

## 參考文獻

[^cz-home]: Clonezilla. (n.d.). *Clonezilla - About*. Retrieved 2026-10-24, from https://clonezilla.org/

[^cz-se]: Clonezilla. (n.d.). *Clonezilla SE (Server Edition)*. Retrieved 2026-10-24, from https://clonezilla.org/clonezilla-SE/

[^cz-testi]: Clonezilla. (n.d.). *Clonezilla - Testimonials*. Retrieved 2026-10-24, from https://clonezilla.org/testimonials/

[^drbl-home]: DRBL. (n.d.). *DRBL - Diskless Remote Boot in Linux*. Retrieved 2026-10-24, from https://drbl.org/

[^fai-feat]: FAI - Fully Automatic Installation. (n.d.). *Features*. Retrieved 2026-10-24, from https://fai-project.org/features/

[^fai-report]: FAI - Fully Automatic Installation. (n.d.). *Case studies / User reports*. Retrieved 2026-10-24, from https://fai-project.org/reports/

[^cobbler-home]: Cobbler. (n.d.). *Welcome to Cobbler!*. Retrieved 2026-10-24, from https://cobbler.github.io/

[^cobbler-3999]: cobbler/cobbler. (n.d.). *Cobbler 4.0.0 Release preparation (Issue #3999)*. Retrieved 2026-10-24, from https://github.com/cobbler/cobbler/issues/3999

[^cobbler-4035]: cobbler/cobbler. (n.d.). *Performance improvements for the development branch (Pull Request #4035)*. Retrieved 2026-10-24, from https://github.com/cobbler/cobbler/pull/4035

[^foreman-readme]: theforeman/foreman. (n.d.). *README*. Retrieved 2026-10-24, from https://github.com/theforeman/foreman

[^foreman-manual]: The Foreman. (n.d.). *Foreman 5.0 Manual — System Requirements & Scaling notes*. Retrieved 2026-10-24, from https://theforeman.org/manuals/latest/index.html

[^satellite-tuning]: Red Hat. (n.d.). *Tuning Performance of Red Hat Satellite 6.12*. Retrieved 2026-10-24, from https://access.redhat.com/documentation/en-us/red_hat_satellite/6.12/html/tuning_performance_of_red_hat_satellite/index

[^maas-perf]: Canonical. (n.d.). *Performance — MAAS documentation*. Retrieved 2026-10-24, from https://canonical.com/maas/docs

[^maas-req]: Canonical. (n.d.). *Installation requirements — MAAS documentation*. Retrieved 2026-10-24, from https://canonical.com/maas/docs

[^maas-home]: Canonical. (n.d.). *MAAS (Metal as a Service)*. Retrieved 2026-10-24, from https://canonical.com/maas
