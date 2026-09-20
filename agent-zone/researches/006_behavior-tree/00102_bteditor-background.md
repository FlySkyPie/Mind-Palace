# bteditor.dev 背景調查報告

## 概述

**bteditor.dev** 是一個免費、開源的行為樹視覺化編輯器（Behavior Tree Editor），為 100% 客戶端運行的網頁應用程式，無需後端伺服器、無需安裝、無需註冊帳號。目標使用者群為機器人工程師、遊戲開發者、自動化工程師及教育/學術領域使用者。[^homepage]

該專案採用 **MIT License** 釋出，並標榜「無追蹤、無資料收集、無後端連線」。[^about]

---

## 公司 / 專案主體

**沒有任何可辨識的正式公司或法律實體**存在於 bteditor.dev 背後。網站上未揭露任何公司名稱、組織名稱、或法律登記資訊。[^about][^contact]

該專案應被視為**個人或極小團隊的獨立開源專案**，而非註冊的商業機構。

---

## 創辦人與團隊

**未公開任何創辦人或團隊成員的姓名。** 網站完全沒有列出任何個人名稱、簡歷、團隊照片、或特定開發者的歸屬資訊。[^about][^contact]

聯絡頁面使用角色式的電子郵件別名：

| 用途 | 電子郵件 |
|---|---|
| 安全性通報 | `security@bteditor.dev` |
| 合作與整合 | `partnerships@bteditor.dev` |
| 新聞與學術使用 | `press@bteditor.dev` |
| 隱私相關 | `privacy@bteditor.dev` |

這些郵件皆使用 `@bteditor.dev` 網域，但沒有任何個人姓名與之關聯。

網站文案採用「我們」（we）的口吻（如「Why we built it」、「We'd love to hear from you」），可能暗示有團隊存在，但也可能是單人開發者的編輯式「we」用法。

---

## 資金與投資人

**無任何已知的資金來源或投資人。**

- 無創投（VC）投資
- 無天使投資人
- 無機構補助或贊助
- 無付費方案、無企業版、無 premium 分層

網站上唯一與營利相關的是 **Google AdSense**（僅出現於資訊頁面）。[^privacy] 工具本身完全免費使用，沒有商業化跡象。[^about]

---

## 技術棧

| 層級 | 技術 |
|---|---|
| 前端框架 | Vanilla JavaScript（無框架、無建置步驟） |
| 節點編輯器核心 | [Drawflow](https://github.com/jerosoler/Drawflow) — 開源純 JS 節點編輯器，由 **jerosoler / Josep Jaume Rey** 開發 |
| 樣式 | HTML5 + CSS3（深色/淺色主題） |
| 資料格式支援 | JSON、XML、BehaviorTree.CPP XML（BTCPP） |
| 回放功能 | NDJSON 格式執行紀錄，支援逐 tick 步進（0.5x–10x 速度） |
| 即時監控 | WebSocket 遠端連線推送即時節點狀態 |
| 分析工具 | Cloudflare Insights、Cloudflare Web Analytics、Microsoft Clarity |
| 廣告 | Google AdSense（資訊頁面） |
| 主機/CDN | Cloudflare（CDN + DNS + SSL） |
| 儲存 | 僅 `localStorage`（主題、語系、側欄狀態），無 cookies |

---

## GitHub 儲存庫與開源狀況

網站多處引用 GitHub 儲存庫：[^about][^contact]

- GitHub Organization: `github.com/bteditor`
- Repository: `github.com/bteditor/bteditor`
- Issues: `github.com/bteditor/bteditor/issues`

然而，**以上所有連結在調查期間（2026-09-20）皆回傳 404 錯誤**。GitHub API 也確認 `github.com/bteditor` organization 不存在於公開記錄中。[^github404]

這可能表示：
1. 儲存庫設為 **private**（非公開）
2. 儲存庫在網站最新更新（2026-07-09）後被**刪除或移至他處**
3. 網站連結有誤

此差異是值得注意的疑點。

---

## 社群與網路足跡

**幾乎沒有發現任何社群存在：**

| 平台 | 狀態 |
|---|---|
| GitHub | 404（不存在或未公開） |
| LinkedIn | 無頁面 |
| Twitter / X | 無帳號 |
| Discord | 無伺服器 |
| Reddit | 無討論串 |
| YouTube | 無頻道 |
| ProductHunt | 無清單 |
| Hacker News | 無貼文 |
| npm | 無套件 |

專案在開源社群中幾乎沒有能見度。

---

## 網域註冊資訊

- **Domain**: `bteditor.dev`（`.dev` TLD）
- **註冊商**: Google Registry（`.dev` TLD 由 Google 管理）
- **WHOIS**: `.dev` TLD 不公開 WHOIS 記錄，無法查詢註冊人或註冊日期。[^whois]
- **SSL**: 強制 HTTPS（`.dev` TLD 強制要求）

---

## 時間線

| 事件 | 日期 |
|---|---|
| 網站最後更新（About / Privacy / Contact） | 2026-07-09 |
| 文件最後更新 | 2026-07-16 |

該專案很可能在 **2025 年至 2026 年間** 啟動，但無法從公開資訊確認精確日期。

---

## 結論

bteditor.dev 是一個**匿名獨立開發者（或極小團隊）的開源專案**，具備良好的技術文件與使用者體驗，但在公司登記、團隊公開資訊、資金來源、社群經營與開源程式碼的可驗證性上**幾乎完全空白**。其 GitHub 儲存庫無法存取是最大的疑點。目前沒有任何證據顯示該專案有商業化意圖或機構支援。

---

## 參考來源

[^homepage]: bteditor.dev. (n.d.). *Behavior Tree Editor*. Retrieved 2026-09-20, from https://bteditor.dev/
[^about]: bteditor.dev. (n.d.). *About*. Retrieved 2026-09-20, from https://bteditor.dev/about
[^contact]: bteditor.dev. (n.d.). *Contact*. Retrieved 2026-09-20, from https://bteditor.dev/contact.html
[^privacy]: bteditor.dev. (n.d.). *Privacy Policy*. Retrieved 2026-09-20, from https://bteditor.dev/privacy.html
[^docs]: bteditor.dev. (n.d.). *Documentation*. Retrieved 2026-09-20, from https://bteditor.dev/docs/
[^drawflow]: Rey, J. J. (jerosoler). (n.d.). *Drawflow*. GitHub. Retrieved 2026-09-20, from https://github.com/jerosoler/Drawflow
[^github404]: GitHub. (n.d.). *github.com/bteditor*. Retrieved 2026-09-20, from https://api.github.com/orgs/bteditor (HTTP 404)
[^whois]: WHO.is. (n.d.). *bteditor.dev WHOIS lookup*. Retrieved 2026-09-20, from https://who.is/whois/bteditor.dev
[^builtwith]: BuiltWith. (n.d.). *bteditor.dev Technology Profile*. Retrieved 2026-09-20, from https://builtwith.com/bteditor.dev