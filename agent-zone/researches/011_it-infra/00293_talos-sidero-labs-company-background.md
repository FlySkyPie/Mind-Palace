# Talos Linux / Sidero Labs 公司背景調查

## 概述

**Sidero Labs, Inc.** 是 Talos Linux 的開發公司，成立於 2019 年，總部位於美國加州聖塔芭芭拉（Santa Barbara），為遠端優先（remote-first）的私人企業，員工人數約 11–50 人（LinkedIn 顯示約 33 人），屬雲端原生基礎設施（Cloud Native Infrastructure）領域。[^1][^2]

## 創辦人與團隊

| 姓名 | 角色 | 備註 |
|---|---|---|
| **Andrey Smirnov**（`smira`）| 共同創辦人 / 首席工程師 | Talos Linux 主要架構師，曾開發 Debian 套件管理工具 `aptly`（GitHub 2.9k stars），精通 Go 語言 |
| **Tim Jones**（`TimJones`）| 共同創辦人 | 常駐西班牙馬德里 |
| **Steve Francis** | CEO | 主導重大策略公告（如被 Yardi 收購、Enterprise Linux 發表） |
| **Justin Garrison** | 工程師 / 技術部落客 | 負責社群內容與技術文章 |
| **Sterling Koch** | 工程師 | 處理 Cluster API 與基礎設施供應商整合 |

創辦故事據 CEO 所述：「我們創立 Sidero Labs 的信念很簡單：Kubernetes 能透過一個安全、精簡、開源且不犧牲任何一項的底層作業系統而變得簡單。」[^3] 專案最初在 `talos-systems` GitHub 組織下開發，後遷移至 `siderolabs`。[^4]

## 資金與創投

**Sidero Labs 未接受任何外部創投投資，為完全自力經營（bootstrapped）的公司。** 此點在 Yardi 收購公告間接獲得證實：CEO Steve Francis 強調 Yardi 同樣是「無外部投資人、無退出時鐘的 bootsrapped 獲利私人企業」——他明確指出這項特質正是 Sidero Labs 所認同的價值。市場上查無 Crunchbase 記錄、TechCrunch 募資報導或任何 A/B 輪募資公告。[^5]

## 收購：被 Yardi 收購（2026 年 9 月 14 日）

這是該公司歷史上最重大的事件，發生於約兩週前：[^5]

- **收購方：** **Yardi** —— 超過 40 年歷史、bootstrapped、獲利、私人持有的企業軟體公司（員工 10,000+），由 Anant Yardi 創立。
- **收購原因：** Yardi 已在生產環境運行 Talos，認可其架構與潛力。兩間公司同屬聖塔芭芭拉地區。
- **收購後條件：** Sidero Labs 以 **獨立公司**（standalone company）形式運營，原團隊不變，產品決策自主。Talos Linux 持續完全開源（MPL-2.0）。
- **立即成果：** 獲得資源後隨即推出 **Talos Hypervisor**（原生 VM 支援，不需 KubeVirt）、免除 TalosCon 參與費用、擴大招聘。
- **CEO 評論：「現在我們能回應企業客戶『你們太小、風險太高，就算技術最佳我們也不採用』的質疑了。」**

## 社群指標

| 指標 | 數值 |
|---|---|
| GitHub Stars（talos）| **11,248** |
| GitHub Forks | **904** |
| 貢獻者 | **330+** |
| 年下載量 | **約 100 萬次** |
| 活躍節點數 | **110,000+** |
| CNCF 狀態 | **Silver Member** |
| GitHub 組織總 Repos | **147** |
| LinkedIn 追蹤者 | **5,223** |

[^4][^2][^6]

## 商業模式與獲利

Sidero 採三層制，核心為開源：[^7][^8]

| 層級 | 授權 | 價格 | 內容 |
|---|---|---|---|
| **Talos Linux** | MPL-2.0（開源）| **永久免費** | 不可變、API 驅動的 K8s 專用作業系統。無 SSH、無 shell。社群支援 |
| **Talos Enterprise Linux** | MPL-2.0 + 商業訂閱 | **$1,000/節點/年**（至少 10 節點）| 相同 OS + FIPS 140-3 建構、SBOM & VEX、簽署憑證、CVE SLA、24/7/365 支援、**IP indemnity**、含 Omni Enterprise。符合 NIS2/CRA 法規 |
| **Talos Omni** | BSL 1.1（source-available）| SaaS 或自託管訂閱 | 叢集管理平台：佈建、升級、配置、備份、叢集模板、SideroLink（WireGuard 通道）、Workload Proxy、OIDC/SAML 認證 |

**核心原則（官方明確聲明）：**「Talos Enterprise Linux 不會擁有 Talos Linux 所沒有的功能。兩者程式碼相同，Enterprise 僅在其上增加合規保證與支援。」——所有功能優先登陸開源版，付費版僅增加合規產物與服務等級，不鎖定功能。[^8]

## 知名採用者

| 類別 | 組織 |
|---|---|
| 企業 / 受監管行業 | **Nokia**、**Roche**、**Proton**、**SGX Group**（新加坡交易所）、**DSV** |
| 科技 / 遊戲 | **Ubisoft**、**Nexxen**、**PowerFlex**、**Berkshire Grey**、**Tremor Video** |
| 基礎設施 | **Equinix**（用於其受管 Kubernetes 服務） |
| 醫療 / 資料 | **Promptly Health**（歐洲健康資料主權）、**Sense Labs** |
| 顧問 | **TrueFullstaq**（荷蘭 K8s 顧問）、**Neosoft**（法國） |
| 其他 | **Mynewsdesk**、**Vandebron**（荷蘭能源電網）、**SCHULZ Systemtechnik**（邊緣/IoT）、**Redpill Linpro**（挪威）、**Nedap Security Atlas**、**DreeBot**（遊戲伺服器）、**Ænix/Cozystack**、**Oceanbox.io**、**Lofty** |

[^9][^10]

## 合作夥伴

- **Neosoft（法國）** —— 約 2026 年 6 月宣布正式合作，提供法國市場 Talos & Omni 顧問服務。
- **TrueFullstaq** —— 雲端原生顧問與受管 K8s 提供商，Talos 為其核心基礎元件。Edgecase 研討會共同贊助商。
- **CNCF** —— Silver Member。
- **Equinix** —— 在其受管 Kubernetes 服務（EQAP）中使用 Talos。
- **Yardi** —— 收購後提供企業級資源與後盾。

## 重大里程碑時間線

| 時間 | 事件 |
|---|---|
| **2019** | Sidero Labs 於加州聖塔芭芭拉創立 |
| **2020** | Talos Linux 達到生產就緒 |
| **~2021–2023** | 社群成長期，GitHub 10k+ stars，廣泛採用 |
| **2024** | Omni（叢集管理平台）發表 |
| **2025** | 首屆 TalosCon 社群大會 |
| **2026 年 8 月** | Talos Linux Cluster API providers 移交社群維護 |
| **2026 年 9 月 14 日** | **被 Yardi 收購** |
| **2026 年 9 月 15 日** | **Talos Enterprise Linux** 發表（$1k/節點/年）；**Talos Hypervisor** 宣布（預計 2026 年 12 月 GA） |
| **2026 年 10 月 15–16 日** | TalosCon 2026，荷蘭阿姆斯特丹（社群成員分享即可免費參加） |

## 分析與觀察

Sidero Labs 在 Kubernetes 生態系中佔有獨特定位：它沒有選擇常見的創投募資 → 快速成長 → 尋求退出的路徑，而是持續 bootsrapped 經營多年，最終被同為私人、bootstrapped 的 Yardi 收購。此路徑使其能夠在產品設計上堅持「開源優先、安全最小化」的核心理念，而不受 VC 成長壓力影響。

Talos Linux 作為「專為 Kubernetes 設計的作業系統」的定位，解決了企業在 Kubernetes 環境中對於安全性和管理簡化的核心痛點。被 Yardi 收購後，其企業市場拓展能力與資源顯著提升。

---

[^1]: Sidero Labs. (n.d.). Sidero Labs Homepage. Retrieved 2026-09-27, from https://siderolabs.com/
[^2]: Sidero Labs. (n.d.). LinkedIn. Retrieved 2026-09-27, from https://www.linkedin.com/company/sidero-labs
[^3]: Sidero Labs. (n.d.). Careers. Retrieved 2026-09-27, from https://siderolabs.com/careers/
[^4]: Sidero Labs. (n.d.). talos. Retrieved 2026-09-27, from https://github.com/siderolabs/talos
[^5]: Francis, S. (2026, September 14). Sidero Labs joins Yardi. Retrieved 2026-09-27, from https://siderolabs.com/blog/sidero-labs-joins-yardi/
[^6]: Sidero Labs. (n.d.). Talos Linux Product Page. Retrieved 2026-09-27, from https://siderolabs.com/talos-linux/
[^7]: Sidero Labs. (n.d.). Omni Product Page. Retrieved 2026-09-27, from https://siderolabs.com/omni/
[^8]: Francis, S. (2026, September 15). Introducing Talos Enterprise Linux. Retrieved 2026-09-27, from https://siderolabs.com/blog/introducing-talos-enterprise-linux/
[^9]: Sidero Labs. (n.d.). ADOPTERS.md. Retrieved 2026-09-27, from https://github.com/siderolabs/talos/blob/main/ADOPTERS.md
[^10]: Sidero Labs. (n.d.). Talos Linux Adopters. Retrieved 2026-09-27, from https://siderolabs.com/talos-linux/ (adopters logos section)