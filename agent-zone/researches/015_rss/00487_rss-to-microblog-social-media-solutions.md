# RSS 轉換為微部落格／社群媒體瀏覽體驗之解決方案調查

## 摘要

本報告調查能將 RSS 訂閱內容轉換為類似 Twitter、Mastodon、Bluesky 等社群媒體時間軸瀏覽體驗的工具、應用程式與服務。調查範圍涵蓋自架開源方案與商業應用，重點關注「社群式時間軸 UI」而非傳統條列式 RSS 閱讀器。

---

## 自架開源方案

### 1. Feediverse

- **描述**：專為模擬社群媒體時間軸而設計的 PWA RSS 閱讀器。逆向時間排序的貼文流、貼文發現功能（貼上任何 URL 自動尋找其 RSS）、追蹤/取消追蹤、富媒體貼文檢視、離線閱讀。受到 iOS 專用 app HyperTexting 啟發。[^feediverse]
- **技術棧**：FastAPI + feedparser + SQLite + Alpine.js（無建置步驟）
- **授權**：MIT
- **自架**：✅ 是（可在 Raspberry Pi 或任何 Linux 主機上運作）
- **GitHub 星星**：新專案（無星數）
- **GitHub**：https://github.com/tpaixao/feediverse

### 2. FeedDeck

- **描述**：受 TweetDeck 啟發的多欄式 RSS 與社群媒體閱讀器。可在同一介面中追蹤 RSS 與 Medium、Reddit、Tumblr、GitHub 活動、Podcast、YouTube。多欄佈局具有強烈的社群媒體原生感。[^feeddeck]
- **技術棧**：Flutter + Supabase + Deno
- **授權**：MIT
- **自架**：✅ 是
- **跨平台**：Android、iOS、macOS、Windows、Linux、Web
- **GitHub 星星**：316
- **GitHub**：https://github.com/feeddeck/feeddeck
- **網站**：https://feeddeck.app

### 3. selfoss

- **描述**：多功能 RSS 閱讀器、即時串流、聚合應用程式。提供雜誌風格 / 時間軸式佈局，支援 RSS、社群媒體及其他平台整合於同一內容流中。支援 OPML 匯入及第三方行動端 app。[^selfoss]
- **技術棧**：PHP + SQLite/MySQL/PostgreSQL
- **授權**：GPL-3.0
- **自架**：✅ 是（簡易 PHP 部署）
- **GitHub 星星**：約 2,500
- **GitHub**：https://github.com/fossar/selfoss
- **網站**：https://selfoss.aditu.de

### 4. NewsBlur

- **描述**：個人新聞閱讀器結合社群新聞網路。在 RSS 訂閱基礎上加入分享、追蹤朋友閱讀內容、AI 訓練自動凸顯/隱藏文章。支援 Magazine 等多種佈局。整合 RSS、YouTube、電子報、Podcast。[^newsblur]
- **技術棧**：Django (Python) + MongoDB + Redis + PostgreSQL
- **授權**：MIT
- **自架**：✅ 是（Docker）
- **平台**：Web、iOS、Android、macOS
- **GitHub 星星**：約 7,700
- **GitHub**：https://github.com/samuelclay/NewsBlur
- **網站**：https://newsblur.com

### 5. FeedFlow

- **描述**：極簡 RSS 閱讀器，以乾淨的逆向時間排序時間軸呈現內容。支援離線閱讀、書籤、過濾、關鍵字封鎖、OPML 匯入/匯出。可與 FreshRSS、Miniflux、Feedbin 等同步。[^feedflow]
- **技術棧**：Kotlin Multiplatform + Compose Multiplatform + SwiftUI
- **授權**：Apache 2.0
- **自架**：否（僅客戶端，但可與自架服務同步）
- **平台**：Android、iOS、macOS、Windows、Linux
- **GitHub 星星**：約 1,300
- **GitHub**：https://github.com/prof18/feed-flow
- **網站**：https://www.feedflow.dev

### 6. FreshRSS

- **描述**：雖是傳統 RSS 閱讀器，但提供透過 HTML、RSS、OPML 重新分享精選文章的功能，具備社群分享面向。支援多使用者及匿名閱讀模式。可作為 FeedFlow 等社交風格閱讀器的後端。[^freshrss]
- **授權**：AGPL-3.0
- **自架**：✅ 是
- **GitHub 星星**：約 9,800+
- **GitHub**：https://github.com/FreshRSS/FreshRSS

### 7. Winds（已封存）

- **描述**：美觀的開源 RSS 與 Podcast 閱讀器。使用 Stream API 建立活動饋送，營造社群式時間軸感。**注意：儲存庫已於 2021 年 10 月封存，不再維護。**[^winds]
- **技術棧**：React + Redux + Express.js + MongoDB + Stream API
- **授權**：BSD-3-Clause
- **GitHub 星星**：約 9,100
- **GitHub**：https://github.com/GetStream/Winds

### 8. Feeds by Felfele

- **描述**：免費獨立的行動端 RSS 閱讀器，逆向時間排序時間軸。無需註冊、無追蹤、資料保留在手機本地。支援新聞網站、部落格、Reddit 子版、YouTube、Tumblr、Twitter、Mastodon。[^felfele]
- **技術棧**：React Native + TypeScript
- **授權**：AGPL-3.0
- **自架**：❌ 否（行動端 app）
- **平台**：Android、iOS
- **GitHub 星星**：約 20
- **GitHub**：https://github.com/felfele/feeds

---

## 商業／原生應用（非自架）

### 9. HyperTexting

- **描述**：專為 iPhone 設計，將 RSS 轉換為社群式時間軸的 app。逆向時間排序、個人頁面、熱門連結、討論串、內建媒體播放器。口號為「你的時間軸，沒有廣告、演算法和 AI 垃圾資訊」。還可**發文**到自有網站。[^hypertexting]
- **平台**：iOS 18.0+（僅 iPhone/iPad）
- **定價**：免費含付費功能
- **網站**：https://hypertexting.com

### 10. Tapestry（by Iconfactory）

- **描述**：將 Bluesky、Mastodon、Tumblr、RSS、Podcast、YouTube 合併為單一逆向時間排序時間軸。可直接在 Bluesky 和 Mastodon 上發文。支援靜音規則、iCloud 同步、完全私密（無伺服器追蹤）。來自 Twitterrific 開發團隊。[^tapestry]
- **平台**：iOS、iPadOS、macOS
- **定價**：免費含進階方案（$2/月 或 $20/年）
- **網站**：https://usetapestry.com

### 11. Reeder

- **描述**：統一新聞閱讀器，支援 RSS、社群媒體、Podcast、影片。自定義過濾器建立時間軸、標籤可作為公開 JSON 饋送分享。iCloud 同步，不收集資料。[^reeder]
- **平台**：iOS 17+、macOS 14+
- **定價**：付費 app
- **網站**：https://reederapp.com

### 12. Feeeed

- **描述**：自訂卡片式捲動時間軸的 iOS 閱讀器。支援 RSS、YouTube、Reddit、電子報。將任何網站轉為時間軸中的「動態卡片」。完全逆向時間排序、無帳號或伺服器，隱私至上。[^feeeed]
- **平台**：iOS
- **定價**：免費
- **網站**：https://feeeed.nateparrott.com

### 13. Inks

- **描述**：統一社群媒體時間軸（Instagram、X/Twitter、Threads、Bluesky、Facebook、LinkedIn、Substack），可**匯出為 RSS**。支援透過 Telegram、WhatsApp、Email 摘要傳送。[^inks]
- **平台**：iOS、Android、Web、Desktop
- **定價**：免費（15 個饋送）、Basic $5/月、Pro $20/月
- **網站**：https://inks.page

### 14. Surf（by Flipboard）

- **描述**：開放社群網路瀏覽器，結合 RSS、ActivityPub、AT Protocol。可建立混合來源的自訂饋送。同時扮演 fediverse 客戶端、閱讀器、策展工具。[^surf]
- **平台**：iOS、Android、Web
- **定價**：免費
- **網站**：https://about.surf.social

---

## 比較總表

| 工具 | 自架 | 開源 | 社群式時間軸 | 星星數 | 核心特色 |
|------|:----:|:----:|:------------:|:------:|----------|
| **Feediverse**     | ✅ | ✅ | ✅ | 新專案 | PWA，刻意設計為社媒風格 |
| **FeedDeck**       | ✅ | ✅ | ✅（多欄） | 316 | TweetDeck 風格多欄 |
| **selfoss**        | ✅ | ✅ | ⚠️ 部份 | 2.5k | 雜誌/時間軸佈局 |
| **NewsBlur**       | ✅ | ✅ | ✅ | 7.7k | 社群分享 + AI 訓練 |
| **FeedFlow**       | ❌ | ✅ | ✅（純時間軸） | 1.3k | 極簡乾淨時間軸 |
| **HyperTexting**   | ❌ | ❌ | ✅ | — | 最刻意模仿社媒介面 |
| **Tapestry**       | ❌ | ❌ | ✅ | — | 最佳統合社媒+RSS 體驗 |
| **Reeder**         | ❌ | ❌ | ✅ | — | 老牌成熟 app |
| **Feeeed**         | ❌ | ❌ | ✅（卡片） | — | 創意卡片式時間軸 |
| **Inks**           | ❌ | ❌ | ✅ | — | 社媒→RSS 匯出 |
| **Surf**           | ❌ | ❌ | ✅ | — | 開放社群網路瀏覽器 |
| **Felfele Feeds**  | ❌ | ✅ | ✅ | 20 | 隱私優先行動閱讀器 |

---

## 推薦

根據不同使用情境的建議：

- **最佳自架社群風格 RSS 閱讀器**：**Feediverse** — 刻意設計為社群媒體時間軸外觀與體驗，PWA 可安裝，可在 Raspberry Pi 上運行
- **最佳自架含社群功能**：**NewsBlur** — 社群分享、blurblog、活動饋送
- **最佳免麻煩托管服務（行動端）**：**HyperTexting**（iOS）或 **Tapestry**（iOS/macOS）
- **最佳多欄/桌面體驗**：**FeedDeck** — TweetDeck 風格，RSS + 社群媒體並列
- **最佳極簡逆向時間軸**：**FeedFlow** — 簡潔、快速、跨平台
- **最佳統合社群媒體+RSS 匯出**：**Inks** — 將統合時間軸轉回 RSS

---

## 參考資料

[^feediverse]: tpaixao. (n.d.). Feediverse. GitHub. Retrieved 2026-10-09, from https://github.com/tpaixao/feediverse

[^feeddeck]: FeedDeck. (n.d.). FeedDeck — RSS and social media feed reader. Retrieved 2026-10-09, from https://github.com/feeddeck/feeddeck

[^selfoss]: fossar. (n.d.). selfoss — The multipurpose RSS reader. GitHub. Retrieved 2026-10-09, from https://github.com/fossar/selfoss

[^newsblur]: Clay, S. (n.d.). NewsBlur — A personal news reader. GitHub. Retrieved 2026-10-09, from https://github.com/samuelclay/NewsBlur

[^feedflow]: prof18. (n.d.). FeedFlow — Minimal RSS reader. GitHub. Retrieved 2026-10-09, from https://github.com/prof18/feed-flow

[^freshrss]: FreshRSS. (n.d.). FreshRSS — A free, self-hostable news aggregator. GitHub. Retrieved 2026-10-09, from https://github.com/FreshRSS/FreshRSS

[^winds]: GetStream. (2021). Winds — Open source RSS & Podcast app (Archived). GitHub. Retrieved 2026-10-09, from https://github.com/GetStream/Winds

[^felfele]: felfele. (n.d.). Feeds — Free and independent mobile RSS reader. GitHub. Retrieved 2026-10-09, from https://github.com/felfele/feeds

[^hypertexting]: HyperTexting. (n.d.). RSS-powered timeline app. Retrieved 2026-10-09, from https://hypertexting.com

[^tapestry]: Iconfactory. (n.d.). Tapestry — Unified timeline app. Retrieved 2026-10-09, from https://usetapestry.com

[^reeder]: Reeder. (n.d.). Unified news reader. Retrieved 2026-10-09, from https://reederapp.com

[^feeeed]: Parrott, N. (n.d.). Feeeed — Scrollable card-based timeline reader. Retrieved 2026-10-09, from https://feeeed.nateparrott.com

[^inks]: Inks. (n.d.). Unified social media timeline with RSS export. Retrieved 2026-10-09, from https://inks.page

[^surf]: Flipboard. (n.d.). Surf — Browser for the open social web. Retrieved 2026-10-09, from https://about.surf.social