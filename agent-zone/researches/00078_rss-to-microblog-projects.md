# 將 RSS 包裝成微部落格形式（如 xikipedia）的開源專案調查

## 背景

[xikipedia](https://github.com/rebane2001/xikipedia) 是一個將維基百科文章包裝成社群媒體動態牆（Twitter/TikTok 式無限滾動）的實驗性專案，全跑在瀏覽器端，使用演算法推薦內容[^xikipedia]。本報告調查是否存在類似精神、但針對 RSS/Atom Feed 的開源專案——亦即將 RSS 訂閱項目以微部落格/社群媒體時間軸形式呈現。

## 結果摘要

**沒有專案明確標榜自己是「RSS 版的 xikipedia」**，但存在數個在精神上高度相似的專案：它們將 RSS Feed 的項目以 Twitter 式卡片時間軸或社群媒體儀表板呈現。

## 最接近的專案

### 1. ssddanbrown/rss（已遷移至 Codeberg）⭐ 568

GitHub 上作者已標註「PROJECT MIGRATED TO CODEBERG」，現活躍倉庫為 [codeberg.org/danb/rss](https://codeberg.org/danb/rss)[^danbrss]。

- **語言**：PHP（Laravel 10 + Inertia.js + Vue.js + Tailwind CSS）
- **授權**：MIT
- **部署方式**：提供 Docker image（`ghcr.io/ssddanbrown/rss:latest`），一鍵啟動
- **核心概念**：作者 Dan Brown（BookStack 作者）明確描述為 *"a simple twitter-feed-style RSS aggregator"*[^danbdesc]
- **Feed 管理**：透過純文字檔案 `feeds.txt` 設定，無 UI
- **三種版面**：Card（卡片式，最像 Twitter）、List、Compact
- **特色**：自訂 Feed 名稱與顏色、OG 圖片抓取、暗色模式、全文搜尋、自動裁切舊文
- **定位**：簡單、無使用者系統、唯讀、刻意窄範圍

### 2. FeedDeck ⭐ 315

GitHub: [feeddeck/feeddeck](https://github.com/feeddeck/feeddeck)[^feeddeck]

- **語言**：Flutter（Dart），後端 Supabase + Deno
- **授權**：MIT
- **核心概念**：*"Follow your RSS and Social Media Feeds. Inspired by TweetDeck."* — 多欄式 TweetDeck 風格
- **跨平台**：Android、iOS、macOS、Windows、Linux、Web
- **支援來源**：RSS、YouTube、Reddit、Medium、GitHub、Tumblr、Podcast、Google News
- **特色**：可自託管、匯入/匯出 Deck 配置、內建播客播放器

### 3. selfoss ⭐ 2,500

GitHub: [fossar/selfoss](https://github.com/fossar/selfoss)[^selfoss]

- **語言**：PHP（前端 React）
- **授權**：GPL-3.0
- **核心概念**：*"multipurpose RSS reader, live stream, mashup, and aggregation web application"* — 單一時間軸串流
- **Spout 系統**：除 RSS/Atom 外可外掛 Twitter、YouTube、Reddit 等來源
- **特色**：OPML 匯入/匯出、全文萃取、標籤、搜尋、鍵盤快捷鍵、多使用者
- **活躍維護**：2026 年 9 月仍有提交，v2.19 穩定版

### 4. Flare ⭐ 1,500

GitHub: [DimensionDev/Flare](https://github.com/DimensionDev/Flare)[^flare]

- **語言**：Kotlin Multiplatform
- **授權**：AGPL-3.0
- **核心概念**：*"Browse Mastodon, Bluesky, X, Misskey, Nostr, Pixiv, Fanbox and RSS all in one app. One timeline, all your accounts."*
- **RSS 支援**：RSS/Atom 是第一公民，可與社群平台混合同一時間軸、支援 RSSHub、OPML 匯入/匯出
- **跨平台**：Android、iOS、macOS、Windows、Linux、Web（自託管 Server 元件）
- **特色**：跨平台發文、內建 AI 摘要、匿名模式

## 傳統版面但可作為基礎的 RSS 閱讀器

| 專案 | Stars | 語言 | 說明 |
|------|-------|------|------|
| FreshRSS[^freshrss] | 16.1k | PHP | 最流行的自託管 RSS 閱讀器，有 API 可接第三方前端（如 FriRSS） |
| Miniflux[^miniflux] | 9.7k | Go | 極簡主義、單一二進位檔、PostgreSQL |
| Yarr[^yarr] | 4k | Go | 單一二進位檔、零依賴、SQLite/Postgres |
| Stringer[^stringer] | 4.1k | Ruby on Rails | *"anti-social RSS reader"*、簡潔乾淨 |
| FriRSS[^frijss] | 77 | TypeScript/React | FreshRSS 的現代化前端、無限滾動三欄版面 |

## CLI 工具（非 Web UI）

- **rss-timeline**[^rsstimeline]：Go 寫的 CLI 工具，將多個 RSS/Atom Feed 合併為按日期排序的時間軸輸出至 stdout。

## 結論

**ssddanbrown/rss**（現 Codeberg 上的 danb/rss）是與 xikipedia 精神最接近的專案：它明確設計為 Twitter 風格的 Feed 聚合器，將 RSS 項目以卡片式時間軸呈現。差異在於 xikipedia 使用演算法推薦，而 danb/rss 使用時間順序排列。

若需要更完整的功能（多使用者、社群平台整合），**selfoss** 與 **Flare** 是更成熟的選擇。

---

[^xikipedia]: rebane2001. (n.d.). *xikipedia — Wikipedia as a social media feed*. GitHub. Retrieved 2026-09-20, from https://github.com/rebane2001/xikipedia

[^danbrss]: Dan Brown. (n.d.). *danb/rss — A simple, opinionated, RSS feed aggregator*. Codeberg. Retrieved 2026-09-20, from https://codeberg.org/danb/rss；原 GitHub 倉庫（唯讀歸檔）：https://github.com/ssddanbrown/rss

[^danbdesc]: Dan Brown. (2022). *Built a simple twitter-feed-style RSS reader*. Reddit r/selfhosted. Retrieved 2026-09-20, from https://www.reddit.com/r/selfhosted/comments/vqldys/built_a_simple_twitterfeedstyle_rss_reader/

[^feeddeck]: Rico Berger. (n.d.). *FeedDeck — Follow your RSS and Social Media Feeds. Inspired by TweetDeck*. GitHub. Retrieved 2026-09-20, from https://github.com/feeddeck/feeddeck

[^selfoss]: fossar. (n.d.). *selfoss — multipurpose RSS reader, live stream, mashup, and aggregation web application*. GitHub. Retrieved 2026-09-20, from https://github.com/fossar/selfoss

[^flare]: DimensionDev. (n.d.). *Flare — Browse Mastodon, Bluesky, X, Misskey, Nostr, Pixiv, Fanbox and RSS all in one app*. GitHub. Retrieved 2026-09-20, from https://github.com/DimensionDev/Flare

[^freshrss]: FreshRSS. (n.d.). *FreshRSS — A free, self-hostable news aggregator*. GitHub. Retrieved 2026-09-20, from https://github.com/FreshRSS/FreshRSS

[^miniflux]: Miniflux. (n.d.). *Miniflux — Minimalist and opinionated feed reader*. GitHub. Retrieved 2026-09-20, from https://github.com/miniflux/v2

[^yarr]: nkanaev. (n.d.). *Yarr — yet another rss reader*. GitHub. Retrieved 2026-09-20, from https://github.com/nkanaev/yarr

[^stringer]: stringer-rss. (n.d.). *Stringer — A self-hosted, anti-social RSS reader*. GitHub. Retrieved 2026-09-20, from https://github.com/stringer-rss/stringer

[^frijss]: Fripix. (n.d.). *FriRSS — A modern, self-hosted, customizable web frontend for FreshRSS*. GitHub. Retrieved 2026-09-20, from https://github.com/Fripix/Frirss

[^rsstimeline]: bridget-otter. (n.d.). *rss-timeline — CLI that merges RSS/Atom feeds into one timeline sorted by date*. GitHub. Retrieved 2026-09-20, from https://github.com/bridget-otter/rss-timeline