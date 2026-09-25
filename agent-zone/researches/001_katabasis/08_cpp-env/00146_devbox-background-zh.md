# Devbox (jetify-com/devbox) 背景調查報告

## 專案概述

Devbox 是一個開源的命令列工具，用於建立可攜帶、可重複、隔離的開發環境。專案使用 Go 語言撰寫，底層由 Nix 套件管理器驅動，但抽象了 Nix 語言的複雜性。採用 Apache 2.0 授權條款釋出[^devbox-repo]。

Devbox 的核心定位是 Docker 與原始 Nix 的替代品——比 Docker 更輕量、比 Nix 更易上手，提供宣告式的 `devbox.json` 配置檔即可建立一致的開發環境。

## 公司資訊：Jetify Inc.

Jetify Inc.（前身為 Jetpack Technologies Inc./jetpack.io）是 Devbox 背後的開發公司[^rebrand]。

| 項目 | 內容 |
|------|------|
| 公司全名 | Jetify Inc. |
| 成立時間 | 2020 年 |
| 總部位置 | Oakland, California 94609, USA |
| 員工人數 | 2–10 人 |
| 公司類型 | 私人控股 (Privately Held) |
| 官方網站 | https://www.jetify.com |
| LinkedIn | linkedin.com/company/jetify-com |

2024 年 4 月 9 日，公司從 jetpack.io 正式更名為 Jetify。公司願景為「幫助開發者以更快、更簡單、更可靠的方式構建雲端應用程式，同時消除樣板程式碼」[^rebrand]。

註：Jetify Inc. 與 WordPress/WooCommerce 的 Jetpack 完全無關，官網 footer 有明確備註。

## 創辦人與團隊

### Daniel Loreto — CEO 兼共同創辦人

Loreto 曾任職於 Google、Airbnb、Twitter 擔任工程師，加入 Virta Health 管理工程部門後創立 Jetpack.io。他是來自委內瑞拉的移民創辦人，重視建立多元包容的工作環境[^tc-2023]。

TechCrunch 在 2023 年 2 月的報導中均以「創辦人 Daniel Loreto」的單數形式提及，未公開確認有第二位共同創辦人[^tc-2023]。

### Mike Landau (GitHub: @mikeland73)

Devbox 專案的主要維護者，負責絕大多數的程式碼提交、版本發布與錯誤修復。畢業於史丹佛大學，所在地為加州聖地牙哥。2026 年 9 月仍有活躍提交記錄。

### 其他核心貢獻者

- **John Lago** (@Lagoja) — 核心貢獻者，功能開發與外掛改進
- **Jeff T.** (@jefft) — 服務與外掛修復（MariaDB、PHP、Nginx）
- **Savil** (@savil) — 貢獻者
- **外部貢獻者**：@timgates42、@joshgodsiff、@fvioz、@humtta 等，顯示社群貢獻活躍

團隊組成以小而精為特色。LinkedIn 上描述的創始團隊背景涵蓋 Airbnb、Google、Facebook、Stripe、Twitter、Microsoft 等頂尖公司，但目前僅確認 Loreto 的經歷符合此描述。

## 募資情況

### 種子輪：1,000 萬美元

根據 TechCrunch 2023 年 2 月的報導，Jetpack.io 完成了 1,000 萬美元的種子輪募資，由 **Coatue** 和 **GV (Google Ventures)** 共同領投。該輪次在報導前一年（約 2022 年初）關閉，此前未曾公開[^tc-2023]。

報導時公司僅有 10 名員工。Loreto 表示：「我在花錢方面非常自律，確保只有在聽到客戶正確回饋時才花錢。」

### 後續輪次

**目前未發現任何公開的 Series A 或後續募資資訊。** 公司可能仍以種子輪資金運作，或進行了未公開的募資。

### 營利模式

Jetify Cloud 提供多層定價方案，是公司主要的收入來源：

| 方案 | 價格 | 說明 |
|------|------|------|
| Solo | $5/月 | 1 人，首月免費 |
| Starter | $25/月 | 最多 10 人 |
| Scale-Up | $250/月 | 最多 50 人 |
| Enterprise | 客製化 | 無限用戶，支援 SSO/BYOC |

按量計費項目包括：Devspace ($0.39/vCPU-小時)、Deploy ($0.10/vCPU-小時)、Cache ($0.60/GB-月)。

此外，Jetify 也透過 Open Collective 回饋開源社群，每月贊助 Appium $500，每年贊助 Selenium $1,000。

## 產品生態系

| 產品 | 說明 | 定位 |
|------|------|------|
| **Devbox** | 開源 CLI 開發環境工具（12.4k stars） | 免費 / Apache 2.0 |
| **Jetify Cloud** | 雲端開發平台（Devspace、Deploy、Secrets、Cache） | 付費 SaaS |
| **TestPilot** | AI 驅動的端到端測試代理（2025 年 1 月發表） | 商業產品 |
| **TypeID** | 型別安全、K-sortable 的全域唯一 ID（3.6k stars） | 開源 / Apache 2.0 |
| **Nixhub** | 80,000+ Nix 套件版本的搜尋索引 | 免費 |
| **AI framework for Go** | Go 語言的 AI 應用框架（262 stars） | 開源 / Apache 2.0 |

Devbox 在 Jetify 的策略中扮演開源漏斗角色：免費工具吸引開發者，再引導至 Jetify Cloud 付費服務。v0.18.0 版本移除了 CLI 中的 Cloud 功能，明確劃分開源與商業的界線。

## 社群活躍度

### GitHub 統計數據（截至 2026-09-25）

| 指標 | 數值 |
|------|------|
| Stars | 12,400+ |
| Forks | 357 |
| Commits | 1,841 |
| 開放 Issues | ~80+ |
| 開放 PRs | 37 |
| 已關閉 PRs | 2,082 |
| 最新穩定版 | v0.18.3 (2026-09-16) |
| VS Code 擴充安裝數 | 17,901 |

### 成長趨勢

從 2024 年 4 月的 ~7,000 stars 成長至 2026 年 9 月的 12,400+ stars，兩年半內增加約 5,400 stars，平均約 180 stars/月，呈穩定有機成長。

### 發布頻率

版本發布極為頻繁，2024 年 9 月至 10 月即發布了 v0.13.0（Python UV 支援）與 v0.13.2（process-compose 整合）等版本，顯示專案積極維護中。

## 專案健康度分析

### 優勢

1. **快速發布節奏** — 數週內即有多個版本，維護積極
2. **穩定成長** — 無 plateau 跡象，持續吸引新用戶
3. **主流維護者活躍** — @mikeland73 負責功能、修復、CI、發布全流程
4. **社群貢獻開放** — 外部貢獻者 regularly merging PRs
5. **生態整合** — GitHub Actions、VS Code、Direnv、DevContainer 支援完整
6. **Thoughtworks Technology Radar 入選** — 2024 年 10 月入選第 50 期

### 風險

1. **高 bus factor** — 絕大多數提交來自 @mikeland73，如其缺席開發速度將大幅下降
2. **核心團隊規模小** — 公開可見的活躍維護者非常有限
3. **Nixhub 更新滯後** — 有 issue 反映套件版本索引落後於實際 nixpkgs
4. **Cloud 功能移除可能引發不滿** — v0.18.0 移除 Cloud 功能的策略性決定可能影響部分用戶

## 結論

Devbox 是一個健康且穩定成長的開源專案，擁有清晰的產品定位（Nix 的簡化替代方案）、穩定的社群參與度和背後的商業支持（Jetify）。其主要風險集中在維護者集中度過高，但鑑於穩定的成長趨勢與生態整合，預期將持續正向發展。

[^devbox-repo]: Jetify Inc. (n.d.). Devbox - GitHub Repository. Retrieved 2026-09-25, from https://github.com/jetify-com/devbox

[^rebrand]: Jetify Inc. (2024-04-09). Jetpack is now Jetify. Retrieved 2026-09-25, from https://www.jetify.com/blog/jetpack-is-now-jetify

[^tc-2023]: TechCrunch. (2023-02-01). Jetpack.io helps developers focus on applications instead of infrastructure. Retrieved 2026-09-25, from https://techcrunch.com/2023/02/01/jetpack-io-helps-developers-focus-on-applications-instead-of-infrastructure/

[^tc-2025]: TechCrunch. (2025-01-28). Jetify launches Testpilot, its AI QA engineer. Retrieved 2026-09-25, from https://techcrunch.com/2025/01/28/jetify-launches-testpilot-its-ai-qa-engineer/

[^linkedin]: Jetify Inc. LinkedIn Profile. Retrieved 2026-09-25, from https://www.linkedin.com/company/jetify-com/

[^pricing]: Jetify Inc. Jetify Cloud Pricing. Retrieved 2026-09-25, from https://www.jetify.com/cloud/pricing

[^opencollective]: Jetify Inc. Open Collective Profile. Retrieved 2026-09-25, from https://opencollective.com/jetify

[^vscode]: Jetify Inc. Devbox - Visual Studio Marketplace. Retrieved 2026-09-25, from https://marketplace.visualstudio.com/items?itemName=jetpack-io.devbox