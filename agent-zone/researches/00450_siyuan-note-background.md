# SiYuan 思源筆記 — 專案背景調查報告

## 概述

SiYuan（思源筆記）是一套本地優先的開源筆記應用程式，核心採用 Go 語言開發，前端使用 TypeScript，支援區塊編輯器、雙向連結、PDF 註釋等功能。截至 2026 年 10 月，該專案在 GitHub 上累積約 46,700 顆星[^github]。

## 開發團隊

SiYuan 由一對夫妻檔開發者維護：

- **88250（暱稱「D」）**：主後端開發者，位於中國雲南昆明。2009 年起投入開源軟體開發，曾建立 Solo 部落格系統和 Symphony 社群平台。GitHub 自我介紹為「Talk less, code more」[^88250]。
- **Vanessa219（Vanessa）**：主前端開發者，88250 的妻子，同樣位於昆明。她是瀏覽器端 Markdown 編輯器 **Vditor**（11,400 顆星）的原創作者，該編輯器構成 SiYuan 的核心編輯引擎[^vanessa]。

兩人於 **2018 年初**辭去公司職位，正式轉為全職開源創業者，並註冊成立了一家公司[^88250]。

## 公司與法律實體

- **品牌名稱**：B3log（亦見 B3log Open Source）
- **GitHub 組織**：[`siyuan-note`](https://github.com/siyuan-note)（主專案）、[`b3log`](https://github.com/b3log)（母組織）
- **官方網站**：<https://b3log.org/siyuan/en/>
- **Docker 映像**：`b3log/siyuan`
- **社群論壇**：[鏈滴 (liuyun.io)](https://liuyun.io) — 約 50,000 名使用者
- **公司登記詳細資訊**（註冊名稱、統一編號等）未公開。

## 創投與資金

**SiYuan 完全沒有任何外部創投資金。**

團隊從未接受創投、天使投資人或機構資金。所有開發經費來自[^pricing]：

1. **PRO 功能買斷**（64 美元終身）— 第三方 S3/WebDAV 同步與備份
2. **訂閱方案**（148 美元終身）— 包含 8GB 官方雲端儲存與官方同步
3. **贊助**（GitHub Sponsors）
4. **其他產品的收入**（Sym 社群平台等）

官方定價頁面說明：「SiYuan 的官方同步服務流量成本高於雲端筆記服務。為每位使用者提供 8GB 雲端儲存是目前我們能負擔的上限」[^pricing]。

## 商業模式：半社群半商業

SiYuan 核心功能完全免費且開源（AGPL-3.0），雲端同步與進階整合功能則需付費：

| 方案 | 價格 | 內容 |
|------|------|------|
| **免費版** | 終身免費 | 所有本機功能，包含商業使用 |
| **PRO 功能** | 64 美元（終身） | 第三方 S3 與 WebDAV 同步/備份 |
| **訂閱版** | 148 美元（終身） | PRO 功能 + 8GB 官方雲端儲存 + 官方資料同步 |

## 社群規模

| 指標 | 數據 |
|------|------|
| GitHub 星數 | **46,700** |
| GitHub Fork 數 | 3,050 |
| 提交次數 | 28,932 |
| 貢獻者總數 | 99+ |
| 組織倉庫數 | 38 |
| 論壇會員數 | 8,346+ |
| 外掛/主題市集開發者 | 238 人[^github] |
| Vditor（相關專案） | 11,400 顆星 |

## 發展歷史時間線

| 年份 | 事件 |
|------|------|
| **2009** | 88250 與 Vanessa 開始共同開發開源軟體，啟動 Solo 部落格系統 |
| **2015** | 爭議事件：大量推廣郵件騷擾 GitHub 使用者 |
| **2017** | Pipe 開源專案內含瀏覽器端加密貨幣挖礦腳本（已於 README 揭露），引發社群反彈 |
| **2018** | 夫妻辭去工作，全職投入開源創業，註冊公司 |
| **2019** | GitHub OAuth 自動加星/追蹤爭議 → b3log 組織遭 GitHub 封鎖，星數歸零 |
| **~2020** | 思源筆記（SiYuan）啟動開發 |
| **2020-08-31** | 首個公開版本釋出 |
| **2021-02-19** | v1.0.1 首個穩定版 |
| **2022** | 團隊公開發表道歉文，承認過往爭議[^apology] |
| **2023** | 全面開源 — 前端與核心均以 AGPL-3.0 釋出 |
| **2024–2026** | 持續快速迭代，加入 AI 功能（OpenAI 整合、MCP）、CLI 工具、OIDC 支援 |

## 歷史爭議

團隊曾公開承認三項重大爭議，並已道歉且改變作法[^apology]：

1. **2015 年**：透過 GitHub 大量寄送推廣郵件為專案招募貢獻者
2. **2017 年**：在 Pipe 開源專案中加入瀏覽器端門羅幣挖礦腳本（雖有在 README 揭露，仍引發爭議）
3. **2019 年**：其社群網站（黑客派/liuyun.io）的 GitHub OAuth 登入機制被設為自動為 b3log/symphony 倉庫加星並追蹤 88250 帳號，未經使用者明確同意，導致 b3log 組織遭 GitHub 封鎖

團隊強調思源筆記**從未**包含任何挖礦程式碼，且因全面開源而完全可受審查[^apology]。

## 關鍵結論

- **微型家庭經營**：核心僅夫妻兩人
- **完全自力更生**：無任何創投或外部資金，依靠付費方案與贊助維生
- **全球受歡迎**：46,700+ GitHub 星數，是同類開源筆記應用中星數最高的專案之一
- **中國團隊**：位於雲南昆明，主要文件以簡體中文撰寫
- **持續進步**：2023 年全面開源後快速成長，AI 整合與生態系統持續擴充

## 參考來源

[^github]: siyuan-note. (n.d.). *SiYuan*. GitHub. Retrieved 2026-10-09, from https://github.com/siyuan-note/siyuan
[^88250]: 88250. (n.d.). *88250*. GitHub. Retrieved 2026-10-09, from https://github.com/88250
[^vanessa]: Vanessa219. (n.d.). *Vanessa219*. GitHub. Retrieved 2026-10-09, from https://github.com/Vanessa219
[^pricing]: B3log. (n.d.). *Pricing*. B3log SiYuan. Retrieved 2026-10-09, from https://b3log.org/siyuan/en/pricing.html
[^apology]: D & V. (2022). *SiYuan Note Developers' Black History*. Liuyun.io. Retrieved 2026-10-09, from https://www.achuan-2.top/post/about-the-black-history-of-siyuan-notes-developers-1pyd1o.html