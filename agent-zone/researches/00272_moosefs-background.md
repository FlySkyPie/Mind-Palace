# MooseFS 專案背景調查

## 概述

MooseFS（Moose File System）是一套基於 POSIX 相容的分散式檔案系統，採用軟體定義儲存（Software-defined Storage）架構。本報告調查其背後的組織、團隊、商業模式、資金來源與社群發展等背景資訊。

## 公司/組織架構

MooseFS 的開發與維護目前由 **Saglabs SA** 負責[^about]。這是一家根據波蘭法律註冊的股份公司（joint-stock company），總部位於波蘭華沙[^contact][^privacy]。該公司於 2001 年 6 月 20 日成立[^emis]，KRS 商業註冊編號為 0000020672，VAT 稅號為 PL 6571011425[^about]。

在 Saglabs SA 之前，MooseFS 的商業營運曾經歷多次組織變更：

- **Core Technology** — 早期版權資料中的開發者之一[^wiki]。
- **Tappest sp. z o.o.** — 一家波蘭有限責任公司（KRS: 0000431953），曾負責 MooseFS 與 MooseFS Pro 的開發與支援[^merger]。
- 2023 年 1 月 1 日，**Saglabs SA 收購 Tappest sp. z o.o.**，兩家公司進行法人合併（incorporation merger）。官方部落格說明此舉是為「簡化由同一股東持有的公司所有權結構」。合併後 Tappest 團隊不變，繼續在 Saglabs SA 下工作[^merger]。

## 創辦人與核心團隊

- **Jakub Kruszona-Zawadzki** — MooseFS 的原創作者與主要開發者，GitHub 版權宣告標示為 `Copyright © 2008-2026 Jakub Kruszona-Zawadzki, Saglabs SA`[^github]。
- **Jakub Ratajczak** — 曾任 MooseFS 業務開發主管（Head of Business Development），於 2018 年 Tuxera 合作公告中出現[^tuxera]。
- **Agata Kruszona-Zawadzka** — 文件貢獻者[^docs]。
- 其他文件貢獻者包括：Małgorzata Antosik、Piotr Konopelko、Krzysztof Krygiel、Aleksander Wieliczko[^docs]。

團隊規模根據 EMIS 資料在 2015 年約有 101–250 名員工，但該數字可能涵蓋 Saglabs SA 整體而非僅 MooseFS 團隊[^emis]。

## 創投與資金來源

**未發現任何外部創投（Venture Capital）或機構投資者。** MooseFS 背後的公司為私有企業，由同一批股東持有，且未出現在 Crunchbase 或其他創投資料庫中[^searches]。商業模式仰賴 MooseFS Pro 授權銷售收入，屬於自力更生（bootstrapped）營運。

根據 EMIS 付費資料庫，Saglabs SA 在 2024 年的淨銷售收入下降 11.51%，營運利潤下降 76.45%，總資產下降 51.78%[^emis]。

## 商業模式

MooseFS 採用 **Open Core（開源核心 + 商業付費）** 模式，提供三個版本[^provscom]：

| 版本 | 授權方式 | 價格 |
|------|---------|------|
| **Community**（社群版） | GPLv2 開源 | **免費** |
| **PRO Personal**（個人版） | 商業授權（非商業用途） | 前 **20 TiB** 免費，超過部分 **9 €/TiB**（上限 200 TiB） |
| **PRO Enterprise**（企業版） | 商業授權 | **按需報價**（含 24/7 開發團隊支援） |

Pro 版本相較 Community 版新增的功能包括：自動容錯轉移（HA High Availability）、多站點部署（Geo-aware Multilocations）、進階糾刪碼（最多 9 個冗餘校驗）、原生 Windows 用戶端等[^provscom]。

支援方案方面，Pro 客戶享有 24/7 商業支援，含專屬顧問、遠端/到場協助、主動監控與專屬版本更新；社群版則透過 Stack Overflow 和 GitHub Issues 獲得社群協助[^support]。

官方網站特別強調「無訂閱費用，無意外。您的資料安全且永遠屬於您」（No subscription fees, no surprises. Your data is safe and yours forever.）[^moosefshome]。

## 合作夥伴

2018 年 6 月，MooseFS 與芬蘭儲存軟體公司 **Tuxera** 宣布合作，提供結合 Tuxera SMB 實作與安全雲端儲存閘道技術的全棧企業級儲存解決方案[^tuxera]。兩家公司曾共同以「MooseFS by Tuxera」名義參加 SC18 展會[^sc18]。此為商業技術合作，非收購關係。

## 社群活躍度

MooseFS 的 GitHub 專案（https://github.com/moosefs/moosefs）指標如下[^github]：

| 指標 | 數值 |
|------|------|
| **Stars** | 約 2,000 |
| **Watchers** | 106 |
| **Forks** | 240 |
| **Commits**（master 分支） | 758 |
| **授權條款** | GPL-2.0 |
| **Topics** | 22 個（如 distributed-file-system, high-availability, posix, snapshots 等） |

此外，專案亦活躍於 SourceForge（https://sourceforge.net/projects/moosefs）[^wiki]。

MooseFS 團隊定期參與 HPC 領域展會，包括 Supercomputing（SC16、SC18、SC24、SC25、SC26）、ISC High Performance、TechCrunch Disrupt、Cloud Expo Europe 等[^blog][^sc26]。

## 專案歷史與發展里程碑

| 時間 | 事件 |
|------|------|
| **2005 年** | MooseFS 首次投入生產環境使用[^moosefshome] |
| **2008-05-30** | 首次公開釋出（版本 1.5.x）[^github] |
| **2009-12** | MooseFS 1.6.x 釋出[^versions] |
| **2014-07** | MooseFS 2.x 釋出[^versions] |
| **2016-02-08** | GitHub 專案正式發布（從 SourceForge 遷移至 GitHub）[^githubpost] |
| **2016-06** | MooseFS 3.x / MooseFS Pro 3.x 釋出[^versions] |
| **2018-06** | 與 Tuxera 合作宣布[^tuxera] |
| **2019-05** | MooseFS Pro 4.x 釋出[^versions] |
| **2020-03/04** | 新冠疫情期間提供免費 Pro 授權與支援給抗疫組織[^blog] |
| **2023-01-01** | Saglabs SA 與 Tappest 公司合併[^merger] |
| **2024-09-25** | MooseFS 4 Community Edition 正式釋出（社群版與 Pro 版功能正式分線）[^blog4] |
| **2026-04** | MooseFS 5 Pro 釋出——新增 Geo-aware 多站點儲存與強化 I/O 安全認證[^blog5] |
| **2026-06-08** | MooseFS 5 Pro Launch 正式公告（Multilocations 功能）[^blog5] |
| **2026-08-26** | MooseFS PRO 5.1.0——新增 Per-Mount 和 Per-User I/O 限制[^blog] |
| **2026-09** | 目前最新版本：Community 4.59.2 / Pro 5.1.0[^versions] |

已終止支援的版本：1.5.x（2010 年終止）、1.6.x（2015 年終止）、2.x（2017 年終止）、3.x（2025 年 3 月終止）[^versions]。

## 分析與觀察

MooseFS 是一個典型的開源核心（Open Core）商業化案例：其開發歷史長達 20 年以上，團隊規模不大，沒有外部創投注資，依靠商業授權與技術支援服務維持營運。2024 年的財務衰退值得關注，但專案本身仍持續釋出重大版本更新（2026 年的 MooseFS 5 系列為近年最大功能更新）。社群層面，GitHub 上的 Star 數（約 2,000）與 Fork 數（240）屬於中小型開源專案規模，但在分散式儲存領域有一定知名度。

## 參考資料

[^about]: Saglabs SA. (n.d.). About us — moosefs.com. Retrieved 2026-09-28, from https://moosefs.com/about/
[^contact]: Saglabs SA. (n.d.). Contact — moosefs.com. Retrieved 2026-09-28, from https://moosefs.com/contact.html
[^privacy]: Saglabs SA. (n.d.). Privacy policy — moosefs.com. Retrieved 2026-09-28, from https://moosefs.com/privacy-policy.html
[^github]: MooseFS. (n.d.). GitHub repository — moosefs/moosefs. Retrieved 2026-09-28, from https://github.com/moosefs/moosefs
[^wiki]: Wikimedia Foundation. (2026). Moose File System — Wikipedia. Retrieved 2026-09-28, from https://en.wikipedia.org/wiki/Moose_File_System
[^merger]: Saglabs SA. (2023-01). Saglabs SA & Tappest – Companies Merger. Retrieved 2026-09-28, from https://moosefs.com/blog/saglabs-and-tappest-companies-merger.html
[^tuxera]: Saglabs SA. (2018-06-11). Tuxera partners with MooseFS. Retrieved 2026-09-28, from https://moosefs.com/blog/tuxera-partners-with-moosefs-to-provide-the-next-generation-full-stack-enterprise-storage-software-solution.html
[^sc18]: Saglabs SA. (2018). MooseFS by Tuxera exhibits at SC18. Retrieved 2026-09-28, from https://moosefs.com/blog/moosefs-by-tuxera-exhibits-at-sc18.html
[^moosefshome]: Saglabs SA. (n.d.). MooseFS — Homepage. Retrieved 2026-09-28, from https://moosefs.com/
[^provscom]: Saglabs SA. (n.d.). MooseFS PRO vs Community. Retrieved 2026-09-28, from https://moosefs.com/pro-vs-community.html
[^support]: Saglabs SA. (n.d.). MooseFS Support. Retrieved 2026-09-28, from https://moosefs.com/support.html
[^versions]: Saglabs SA. (n.d.). MooseFS versions and EOL status. Retrieved 2026-09-28, from https://moosefs.com/versions.html
[^blog]: Saglabs SA. (n.d.). MooseFS Blog. Retrieved 2026-09-28, from https://moosefs.com/blog/index.html
[^githubpost]: Saglabs SA. (2016-02-08). MooseFS GitHub Project. Retrieved 2026-09-28, from https://moosefs.com/blog/moosefs-github-project.html
[^blog4]: Saglabs SA. (2024-09-25). MooseFS 4 — What changed from version 3. Retrieved 2026-09-28, from https://moosefs.com/blog/moosefs-4-what-changed-from-version-3.html
[^blog5]: Saglabs SA. (2026-06-08). MooseFS 5 PRO Multilocations. Retrieved 2026-09-28, from https://moosefs.com/blog/moosefs-5-pro-multilocations.html
[^emis]: EMIS. (2024). Saglabs SA — Company Profile. Retrieved 2026-09-28, from https://www.emis.com/php/company-profile/PL/Saglabs_SA_en_2014441.html
[^searches]: Brave Search, Crunchbase, Dun & Bradstreet — Saglabs SA investors search. Retrieved 2026-09-28.
[^sc26]: Saglabs SA. (2026). MooseFS team is heading to SC26. Retrieved 2026-09-28, from https://moosefs.com/blog/moosefs-team-is-heading-to-sc26.html
[^docs]: MooseFS. (n.d.). MooseFS Documentation. Retrieved 2026-09-28, from https://docs.moosefs.com/