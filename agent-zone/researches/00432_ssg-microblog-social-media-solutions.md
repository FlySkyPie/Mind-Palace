# FOSS 靜態網站生成器（SSG）：微網誌／社群媒體方案調查

## 概述

沒有任何單一 SSG 能「直接生成 Facebook 或 Twitter」，但這個領域可拆分為**三大類別**，彼此可互相搭配：

1. **專為微網誌設計的 SSG** — 純靜態輸出
2. **自託管微網誌平台** — 動態伺服，但提供完整社群互動體驗
3. **IndieWeb / ActivityPub 層** — 在既有 SSG 上疊加按讚、留言、聯邦等功能

最有彈性的做法是：將 SSG（如 Eleventy 或 Hugo）與 IndieWeb 工具（Micropub、Webmention、ActivityPub）組合，獲得完整的社群媒體體驗[^indieweb-eleventy]。

---

## 一、專屬微網誌 SSG（純靜態產生）

### tumblelog

- **語言**：Perl 與 Python 版本（輸出相同）
- **授權**：MIT 近似
- **功能**：從**單一 Markdown 檔案**產生靜態微網誌／碎碎念部落格（tumblelog）。每筆記錄包含日期、標題與內容。輸出 HTML5 頁面、JSON Feed、RSS。
- **社群功能**：無 — 純粹的部落格產生器，無留言、無按讚、無聯邦
- **適合對象**：想要簡潔時間軸式短文清單、不想要任何互動功能的使用者
- **限制**：無發文 UI、無多使用者、無社交互動

### molly/static-timeline-generator

- **語言**：JavaScript（基於 Eleventy/11ty）
- **授權**：MIT
- **功能**：產生**時間軸風格**的靜態網頁，依時間排序。可包含日期時間、分類、圖片、顏色、圖示與連結[^static-timeline]。
- **社群功能**：無 — 純粹是時間軸**展示**引擎，非互動社群平台
- **適合對象**：製作專案歷史、個人里程碑等視覺時間軸

### hugo-microblog（mandarvaze/hugo-microblog）

- **語言**：Go（Hugo 主題）
- **授權**：MIT
- **功能**：將 Hugo 變成**混合型部落格＋微網誌**。區分 `post`（長文）與 `microposts`（短文）兩種內容類型。
- **社群功能**：內建 **Webmention** 支援（來自 IndieWeb 的留言／按讚）[^hugo-microblog]
- **適合對象**：已在使用 Hugo，並想加入微網誌功能的使用者

### microblog.py

- **語言**：Python
- **授權**：AGPL-3.0
- **功能**：約 600 行 Python 的靜態微網誌產生器，專為 Neocities 設計。支援標籤、標籤雲、Webring、分頁、TOML 設定[^microblog-py]。
- **社群功能**：無（作者明言「不適合做為對話起點的貼文」）
- **適合對象**：在 Neocities 上做回顧性／歸檔式微網誌

---

## 二、自託管微網誌平台（動態、非純靜態）

這些**不是 SSG**，而是自託管的網頁應用程式，能動態提供內容。但它們在「社群媒體體驗」這個題目上遠優於任何靜態產生器。

### Ech0

- **語言**：Go
- **授權**：AGPL-3.0-or-later
- **GitHub 星星**：2,100+
- **定位**：自託管個人微網誌，附有**可分享的時間軸** — 最接近「自己架 Twitter／Facebook」的解決方案[^ech0-github][^ech0-site]
- **主要特色**：
  - ✍️ Markdown 編輯器，即時預覽
  - 🏷️ 標籤管理與篩選
  - 🎬 多格式媒體附件（圖片、音訊、影片）
  - 💬 內建留言系統，附管理審核功能
  - 🃏 按讚與分享
  - 🔑 OAuth2/OIDC、通行金鑰登入、多使用者權限管理
  - 📦 **可攜式內容膠囊（Portable Content Capsules）** — 可將所有內容匯出為自成一體的靜態網站！
  - 🌐 多國語系、PWA、深色模式、響應式設計
  - 🤖 MCP 伺服器與 AI Copilot（基於 RAG 的內容問答）
  - 🗂️ S3 儲存支援、自動備份
  - 單一 Docker 映像檔部署（`docker run sn0wl1n/ech0:latest`）
  - **社群互動感**：非常高 — 被設計為社群媒體時間軸的體驗
  - **公開目錄**：[Ech0 Hub](https://hub.ech0.app/) 可彙整多個實例的時間軸
- **適合對象**：想在 60 秒內擁有完整微網誌體驗（留言、按讚、時間軸、多使用者）的使用者

### Known（known.fm / withknown.com）

- **語言**：PHP
- **授權**：Apache 2.0
- **定位**：社群發布平台 — 可在自己的網站上發布狀態更新、部落格文章、照片，同時自動同步至 Twitter/Facebook
- **特色**：多使用者、IndieWeb 基礎元件（Micropub、Webmention）、響應式設計、支援群組／內部網路
- **社群互動感**：高 — 被設計為取代 Facebook/Twitter 的主要發布工具
- **現況**：最後穩定版 v1.5（2023 年 6 月）。維護較不活躍[^known][^known-site]。

### microblog.pub

- **語言**：Python
- **授權**：AGPL
- **定位**：可自託管、單使用者的微網誌軟體，完整支援 IndieWeb
- **特色**：IndieAuth、microformats2、Micropub、Webmention、**完整 ActivityPub 支援**（可與 Mastodon 完全聯邦）
- **社群互動感**：非常高 — 本質上就是個人的 Mastodon 相容實例
- **⚠️ 現況**：截至 2026 年 2 月，網站與文件已無法載入。原始碼仍存放於 sourcehut。專案狀態不明[^microblog-pub]。

---

## 三、IndieWeb / ActivityPub 層（為靜態網站疊加社群功能）

這是「魚與熊掌兼得」的做法 — 使用一般 SSG，再疊加社群功能於其上[^elevindieweb][^jekyll-activitypub][^fedipage][^staticpub][^static-activitypub-1]。

### ElevIndieWeb（dumaurier/ElevindieWeb）

- **SSG**：Eleventy（11ty）
- **授權**：MIT
- **定位**：最完整的 **IndieWeb 靜態網站入門套件**。包含自託管 IndieAuth、Micropub 伺服器與客戶端、Webmention 整合、POSSE 同步發布[^elevindieweb]。
- **內容類型**：Posts（文章）、Notes（無標題短文）、Bookmarks（書籤）、Replies（回覆） — 完全比照社群媒體
- **社群功能**：
  - 📝 **Micropub 伺服器** — 可從任何 Micropub 客戶端（手機 App 等）發文
  - 📋 **內建管理頁面** (`/admin`) — 可直接在瀏覽器中發文
  - 🔄 **POSSE 同步發布** — 自動同步至 Bluesky 與 Mastodon
  - 💬 **Webmentions** — 顯示來自網路的按讚、轉發、回覆
  - 🔑 **自託管 IndieAuth** — 不需外部認證服務
  - 4 種內容類型對應 IndieWeb 貼文類型
- **部署**：Cloudflare Pages + Cloudflare Functions（動態端點）
- **適合對象**：想要純靜態網站但表現得像社群媒體個人檔案的使用者

### jekyll-activitypub-static（Social Web Foundation）

- **SSG**：Jekyll
- **授權**：LGPL-3.0-or-later
- **定位**：Jekyll 外掛，可產生**靜態 ActivityPub 訊息串**（FEP-b06c ActivityPoll）
- **特色**：產生 actor、webfinger、outbox、inbox、貼文等靜態 JSON-LD 檔案。同時支援 ActivityStreams 的 `Article`（長文）與 `Note`（短文）類型[^jekyll-activitypub]。
- **社群功能**：讓你的 Jekyll 網站可在 Mastodon/Fediverse 上被發現。他人可像追蹤使用者一樣追蹤你的網站。注意：唯獨 — 無法處理 inbox 回覆。
- **適合對象**：不需要伺服器端程式碼，就能為現有 Jekyll 網站加上聯邦功能

### Fedipage

- **SSG**：基於 Hugo
- **授權**：AGPL
- **定位**：具備**完整 ActivityPub 支援**的靜態網站產生器（需 Vercel + Firebase 以啟用完整功能）[^fedipage]
- **特色**：
  - ✅ 追蹤確認
  - ✅ 新貼文時的通知
  - ✅ 在靜態頁面上顯示 Fediverse 留言、按讚、轉發
  - ✅ ActivityPub 標籤（可見或隱藏）
  - ✅ 側邊欄顯示 Fediverse 別名內容
  - ✅ 多種部落格分類
- **限制**：無後端時可運作但無法確認追蹤
- **適合對象**：想讓 Hugo 靜態網站成為完整 Fediverse 公民的使用者

### StaticPub（lvm/staticpub）

- **語言**：Python
- **授權**：BSD-3-Clause
- **定位**：從 Markdown + 設定產生**靜態 ActivityPub 實例**的 Python 腳本[^staticpub]
- **社群功能**：產生 actor、outbox、followers、following、webfinger 等靜態 JSON-LD 端點
- **限制**：唯讀 — 無 inbox（無法接收追蹤／回覆）、無法傳遞
- **適合對象**：ActivityPub 實驗、建立可在 Fediverse 上被看見的靜態「個人檔案」

### static-website-activitypub（ned14）

- **語言**：Python（CherryPy 框架）
- **授權**：Apache-2.0
- **定位**：為任何 SSG（Hugo、Jekyll）加上 ActivityPub C2S API 的包裝層[^static-activitypub-1]
- **特色**：WebFinger、Actor 端點、POST 至 outbox（新增貼文）、觸發重新產生
- **現況**：Alpha 階段 — 僅部分實作
- **適合對象**：需要為既有靜態網站工作流程加入發文 API

### 11ty ActivityPub Plugin（lewisdale.dev）

- **SSG**：Eleventy
- **NPM 套件**：`eleventy-plugin-activity-pub`
- **功能**：讓任何 Eleventy 網站成為 Fediverse 上可被發現的 ActivityPub 使用者[^eleventy-plugin-ap]
- **適合對象**：Eleventy 使用者想要快速加入聯邦功能

---

## 四、輔助工具（非 SSG 本身）

### Granary（snarfed/granary）

- **定位**：「社交網路翻譯機」 — 在社交網路、microformats2、ActivityStreams/ActivityPub、Atom、RSS、JSON Feed 之間互轉[^granary][^granary-site]
- **授權**：Apache-2.0
- **用途**：橋接靜態網站內容與 Twitter、Mastodon、Facebook 等平台

### IndieKit

- **定位**：為靜態網站設計的 Micropub 伺服器 — 可從手機發文至 Git 版控的靜態網站[^micropub]
- **用途**：搭配 ElevIndieWeb 等方案，實現從任何 Micropub 客戶端對靜態網站發文

---

## 比較表[^ech0-github][^known][^elevindieweb][^microblog-pub][^fedipage][^hugo-microblog][^jekyll-activitypub][^tumblelog][^staticpub][^static-timeline]

| 解決方案 | 類型 | 社群時間軸感 | 發文 UI | 留言／按讚 | 聯邦（ActivityPub） | 部署複雜度 | 使用者檔案 |
|---|---|---|---|---|---|---|---|
| **Ech0**[^ech0-github][^ech0-site] | 動態（Go） | ★★★★★ | 內建 | ✅ 內建 | ✅（已規劃／RSS） | 極低（Docker） | ✅ 多使用者 |
| **Known**[^known][^known-site] | 動態（PHP） | ★★★★☆ | 內建 | ✅ Webmention | 部分 | 中等 | ✅ 多使用者 |
| **ElevIndieWeb**[^elevindieweb] | 靜態（Eleventy） | ★★★★☆ | 管理頁＋Micropub | ✅ Webmentions | ✅（透過外掛） | 中等（Cloudflare） | 單使用者 |
| **microblog.pub**[^microblog-pub] | 動態（Python） | ★★★★☆ | Micropub 客戶端 | ✅ Webmention | ✅ 完整 | 中等 | 單使用者 |
| **Fedipage**[^fedipage] | 靜態（Hugo） | ★★★☆☆ | 無（檔案編輯） | ✅ 經由 Fediverse | ✅ 完整（需後端） | 高（Vercel+Firebase） | 單使用者 |
| **hugo-microblog**[^hugo-microblog] | 靜態（Hugo） | ★★★☆☆ | 無（檔案編輯） | ✅ Webmentions | ❌（手動） | 低 | 單使用者 |
| **jekyll-ap-static**[^jekyll-activitypub] | 靜態（Jekyll） | ★★☆☆☆ | 無 | ❌ 唯讀 | ✅ 唯讀（ActivityPoll） | 低 | 單使用者 |
| **tumblelog**[^tumblelog][^tumblelog-github] | 靜態（Perl/Py） | ★★☆☆☆ | 無 | ❌ | ❌ | 極低 | 單使用者 |
| **StaticPub**[^staticpub] | 靜態產生器 | ★☆☆☆☆ | 無 | ❌ | ✅ 唯讀 | 極低 | 單使用者 |
| **static-timeline-gen**[^static-timeline] | 靜態（Eleventy） | ★★☆☆☆ | 無 | ❌ | ❌ | 極低 | 單使用者 |

---

## 實際範例

1. **Plurrrr**（[plurrrr.com](http://plurrrr.com/)） — 使用 `tumblelog` 建立。簡潔的微網誌訊息串[^tumblelog]。
2. **Ech0 Preview**（[memo.sn0w.fyi](https://memo.sn0w.fyi/)） — Ech0 官方展示站。完整的社群時間軸體驗[^ech0-site]。
3. **microblog.desipenguin.com** — 使用 `hugo-microblog`。混合微網誌與傳統部落格[^hugo-microblog]。
4. **Ashton McAllan**（[acegiak.net](https://acegiak.net/)） — 使用 `microblog.pub` 分支，與 Mastodon 完全聯邦[^microblog-pub]。
5. **bw3.dev** — 使用 `microblog.pub` 作為主站[^microblog-pub]。
6. **social.thej.in** — Thejesh GN 的 microblog.pub 實例[^microblog-pub]。
7. **Fedipage 本身**（[fedipage.com](https://fedipage.com/)） — 自家使用 Hugo+ActivityPub 方案[^fedipage]。
8. 多位 IndieWeb 社群成員（Zach Leatherman、Max Böck、Paul Robert Lloyd）執行具 Webmention 功能的 Eleventy 網站，表現得像社群媒體個人檔案[^indieweb-eleventy]。
---

## 使用場景建議

**「我想要 60 秒內擁有個人 Twitter 複製品」** → **Ech0**[^ech0-github][^ech0-site]。用 Docker 啟動，立即獲得時間軸、留言、按讚、多使用者，以及可匯出的內容膠囊。

**「我想要純靜態網站但表現得像社群個人檔案」** → **ElevIndieWeb**[^elevindieweb]（Eleventy + Cloudflare Functions）。包含自託管 IndieAuth、Micropub（可從手機發文）、Webmentions（留言／按讚）、POSSE 同步發布至 Mastodon/Bluesky。

**「我想讓現有 Jekyll/Hugo 網站出現在 Fediverse 上」** → `jekyll-activitypub-static`[^jekyll-activitypub]（Jekyll）、`eleventy-plugin-activity-pub`[^eleventy-plugin-ap]（Eleventy）、或 **Fedipage**[^fedipage]（Hugo）。

**「我只想要簡單的按時間排序微網誌訊息串」** → **tumblelog**[^tumblelog]（最簡潔）或 **hugo-microblog**[^hugo-microblog]（已在使用 Hugo 者）。

**「我想要 IndieWeb 原則下的完整控制權」** → 選擇 SSG（Eleventy 最有彈性），搭配 **Granary**[^granary] 做資料轉換、**webmention.io**[^webmention] 接收提及、**Brid.gy** 橋接社交網路、**IndieKit**[^micropub] 或 **Micropub**[^micropub] 進行發文。

---

## 參考來源

[^ech0-github]: Lin Snow. (n.d.). *Ech0 — Self-hosted microblogging platform.* Retrieved 2026-09-30, from https://github.com/lin-snow/Ech0
[^ech0-site]: Ech0. (n.d.). *Ech0 — Self-hosted microblog platform.* Retrieved 2026-09-30, from https://ech0.app/
[^elevindieweb]: dumaurier. (n.d.). *ElevIndieWeb: A Static IndieWeb Starter Kit.* Retrieved 2026-09-30, from https://github.com/dumaurier/ElevindieWeb
[^indieweb-eleventy]: IndieWeb. (n.d.). *Eleventy.* Retrieved 2026-09-30, from https://indieweb.org/Eleventy
[^jekyll-activitypub]: Social Web Foundation. (n.d.). *jekyll-activitypub-static — Static ActivityPub feed for Jekyll.* Retrieved 2026-09-30, from https://github.com/social-web-foundation/jekyll-activitypub-static
[^fedipage]: Fedipage. (n.d.). *Static site generator with ActivityPub support.* Retrieved 2026-09-30, from https://fedipage.com/
[^tumblelog]: Bokma, J. (n.d.). *tumblelog: A static microblog/tumblelog generator.* Retrieved 2026-09-30, from https://johnbokma.com/articles/tumblelog/
[^tumblelog-github]: Bokma, J. (n.d.). *tumblelog source.* Retrieved 2026-09-30, from https://github.com/john-bokma/tumblelog
[^hugo-microblog]: Mandarvaze. (n.d.). *hugo-microblog theme.* Retrieved 2026-09-30, from https://github.com/mandarvaze/hugo-microblog
[^static-timeline]: Molly. (n.d.). *static-timeline-generator.* Retrieved 2026-09-30, from https://github.com/molly/static-timeline-generator
[^microblog-pub]: IndieWeb. (n.d.). *microblog.pub.* Retrieved 2026-09-30, from https://indieweb.org/microblog.pub
[^static-activitypub-1]: ned14. (n.d.). *static-website-activitypub.* Retrieved 2026-09-30, from https://github.com/ned14/static-website-activitypub
[^staticpub]: lvm. (n.d.). *StaticPub — Static ActivityPub instance generator.* Retrieved 2026-09-30, from https://github.com/lvm/staticpub
[^granary]: Snarfed. (n.d.). *Granary — The social web translator.* Retrieved 2026-09-30, from https://github.com/snarfed/granary
[^granary-site]: Granary. (n.d.). *Granary social web translator.* Retrieved 2026-09-30, from https://granary.io/
[^known]: Wikipedia. (n.d.). *Known (software).* Retrieved 2026-09-30, from https://en.wikipedia.org/wiki/Known_(software)
[^known-site]: Known. (n.d.). *Known — Social publishing platform.* Retrieved 2026-09-30, from https://withknown.com/
[^webmention]: IndieWeb. (n.d.). *Webmention.* Retrieved 2026-09-30, from https://indieweb.org/Webmention
[^micropub]: Micropub. (n.d.). *Micropub protocol.* Retrieved 2026-09-30, from https://micropub.net/
[^static-activitypub-2]: Mahomedalid. (n.d.). *Almost static ActivityPub.* Retrieved 2026-09-30, from https://github.com/mahomedalid/almost-static-activitypub
[^swf-activitypub]: Social Web Foundation. (2026-09-04). *Static ActivityPub publishing.* Retrieved 2026-09-30, from https://socialwebfoundation.org/2026/09/04/static-activitypub-publishing/
[^eleventy-plugin-ap]: lewisdale.dev. (n.d.). *Eleventy ActivityPub plugin.* Retrieved 2026-09-30, from https://www.npmjs.com/package/eleventy-plugin-activity-pub
[^microblog-py]: 32bit.cafe Discourse. (n.d.). *microblog.py — Static microblog generator with tags and webringing.* Retrieved 2026-09-30, from https://discourse.32bit.cafe/t/microblog-py-static-microblog-generator-with-tags-and-webringing-gnu-agpl/765
[^ech0-hub]: Ech0. (n.d.). *Ech0 Hub.* Retrieved 2026-09-30, from https://hub.ech0.app/