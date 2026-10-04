# Fictioneer／なろう 替代方案調查：非 WordPress・FOSS 自架可行之小說發佈平台

## 概述

本報告調查 [Fictioneer](https://github.com/Tetrakern/fictioneer)（WordPress 主題）以及 [小説家になろう（ncode.syosetu.com）](https://ncode.syosetu.com) 的替代方案，尋找**不依賴 WordPress** 的 FOSS（自由開放原始碼軟體）且**可自架（self-hosted）** 的小說發佈平台。以 GitHub Star 數量為重要參考指標，並以日語關鍵字進行搜尋。

**結論：** 目前**不存在**不依賴 WordPress、同時具備「なろう」完整功能（作者註冊、章節連載、讀者留言、探索發現）的成熟 FOSS 自架平台。惟存在數個近似替代方案。

## 調查方法

以日語（「小説投稿」「WEB小説」「セルフホスト」「なろう系」）及英語關鍵字對 GitHub 及網路進行廣泛搜尋，確認各專案的 repository、Star 數、程式語言、授權條款及維護狀態。

## 候選列表

### 1. WriteFreely ⭐ 5,262

通用型寫作暨發佈平台。相當於 Medium / Write.as 的自架版本。

- **Repository**：<https://github.com/writefreely/writefreely>
- **程式語言**：Go（單一二進位檔、輕量、可執行於 Raspberry Pi）
- **授權條款**：AGPL-3.0
- **功能**：Markdown 編輯、ActivityPub 聯邦（可與 Mastodon 等互通）、多部落格、草稿、匿名支援、20+ 語言、OAuth 2.0
- **小說特化程度**：低（通用部落格引擎）
- **實績**：Write.as 上承載超過 55 萬個部落格[^writefreely]

### 2. OTW Archive（Archive of Our Own）⭐ 2,300

驅動 [AO3](https://archiveofourown.org) 的正式級軟體，專注於二次創作（fanfiction）。

- **Repository**：<https://github.com/otwcode/otwarchive>
- **程式語言**：Ruby on Rails
- **授權條款**：GPL-2.0
- **功能**：作品與章節管理、標籤、留言、書籤、收藏、使用者管理，16,000+ 次提交
- **小說特化程度**：高（惟設計上以二次創作為導向）
- **注意事項**：面向多人使用之檔案庫，營運需具備 Rails 知識[^otwarchive]

### 3. Plume ⭐ 2,200

以 Rust 撰寫的聯邦式部落格引擎，支援 ActivityPub。

- **Repository**：<https://github.com/Plume-org/Plume>
- **程式語言**：Rust（Rocket、Diesel ORM）
- **授權條款**：AGPL-3.0
- **資料庫**：PostgreSQL
- **功能**：多部落格、聯邦、多媒體管理、Docker 支援、20+ 語言
- **小說特化程度**：低（部落格引擎）[^plume]

### 4. show-me-the-story ⭐ 609

AI 小說生成平台。Go 單一二進位檔搭配 Svelte UI。

- **Repository**：<https://github.com/Nigh/show-me-the-story>
- **程式語言**：Go + Svelte
- **功能**：大綱 → 逐章撰寫、伏筆檢查、校對、全文潤飾
- **小說特化程度**：中（惟以生成為導向，發佈功能有限）
- **注意事項**：偏向寫作輔助工具，非發佈平台[^showme]

### 5. Nonograph ⭐ 318

極簡匿名發佈工具。無需帳號、無追蹤。

- **Repository**：<https://github.com/du82/nonograph>
- **程式語言**：Rust（Rocket）
- **授權條款**：Unlicense（公有領域）
- **功能**：Markdown、Tor 支援、即時發佈、Docker
- **小說特化程度**：極低（僅單篇發佈）[^nonograph]

### 6. novel-builder.js ⭐ 33

CLI 工具，支援なろう、カクヨム等投稿格式。

- **Repository**：<https://github.com/8amjp/novel-builder>
- **程式語言**：JavaScript（Node.js）
- **授權條款**：MIT
- **功能**：Markdown 稿件轉換為各平台格式、EPUB 產生、校對
- **小說特化程度**：中（惟為 CLI 工具，非平台）
- **日語支援**：支援なろう、カクヨム、ハーメルン、アルファポリス[^novelbuilder]

### 7. Shelf ⭐ 2

針對連載小說及輕小說的自架閱讀器。

- **Repository**：<https://github.com/ShogyX/Shelf>
- **程式語言**：Python（FastAPI）+ React（TypeScript）
- **授權條款**：MIT
- **資料庫**：SQLite
- **功能**：排版控制、多主題、閱讀進度、有聲書支援
- **小說特化程度**：中（主要為個人閱讀藏書管理）[^shelf]

### 8. Cedium ⭐ 2

現代化創作寫作平台。

- **Repository**：<https://github.com/Rinisnotarobot/Cedium>
- **程式語言**：TanStack Start + React 19 + Prisma + PostgreSQL
- **授權條款**：MIT
- **功能**：富文字編輯、草稿/已發佈/已封存狀態、標籤、追蹤、巢狀留言、按讚、書籤、OTP 認證
- **小說特化程度**：中（文章型平台）[^cedium]

### 9. FicNest ⭐ 1

現代化 fanfiction / 網路小說發佈平台。

- **Repository**：<https://github.com/FicNest/ficnest-platform>
- **程式語言**：TypeScript（React + Express + Supabase / PostgreSQL）
- **功能**：作者儀表板、章節閱讀器、留言、評論、深色模式
- **小說特化程度**：高（專為發佈平台設計）[^ficnest]

### 10. novel-platform（墨閱書齋）⭐ 1

中文網路小說自架平台，設計上最接近「なろう」。

- **Repository**：<https://github.com/epiphany131/novel-platform>
- **程式語言**：JavaScript（Node.js + Express、vanilla JS 前端）
- **授權條款**：MIT
- **資料庫**：SQLite（無需外部資料庫程序）
- **功能**：PWA、全文搜尋、多作者及審查流程、管理後臺、留言及評分、外部來源整合、SSRF 防護、單一 Docker 容器
- **小說特化程度**：極高（惟 UI 為中文）[^novelplatform]

### 11. Zax Kodelex ⭐ 1

支援漫畫、Webtoon 及小說的綜合性自架平台。

- **Repository**：<https://github.com/ZaxMil/zax-kodelex>
- **程式語言**：TypeScript（React + Vite + Supabase）
- **授權條款**：MIT
- **功能**：現代化閱讀器、發佈工具、主題、會員功能、收益化（PayPal/Ko-fi）、PWA
- **小說特化程度**：中至高（偏重漫畫，惟亦支援小說）[^zaxkodelex]

### 12. Clara ⭐ 1

日語直排小說寫作暨投稿 Web 服務。

- **Repository**：<https://github.com/m19e/clara>
- **程式語言**：TypeScript（Next.js + Tailwind + Firebase）
- **授權條款**：MIT
- **功能**：直排顯示
- **注意事項**：依賴 Firebase，難以完全自架[^clara]

### 13. inovel ⭐ 0

現代化網路小說平台（開發初期）。

- **Repository**：<https://github.com/vvvv31/inovel>
- **程式語言**：Python（Flask/Django/React）
- **授權條款**：MIT
- **狀態**：7 次提交，極初期[^inovel]

## 比較表（依 Star 數排序）

| 專案 | ⭐ Star 數 | 程式語言 | 授權條款 | 小說特化程度 | 維護狀態 |
|---|---|---|---|---|---|
| **WriteFreely** | 5,262 | Go | AGPL-3.0 | 低（通用） | ✅ 活躍 |
| **OTW Archive** | 2,300 | Ruby on Rails | GPL-2.0 | 高（二次創作） | ✅ 活躍 |
| **Plume** | 2,200 | Rust | AGPL-3.0 | 低（部落格） | ✅ 活躍 |
| **show-me-the-story** | 609 | Go+Svelte | — | 中（生成特化） | ✅ 活躍 |
| **Nonograph** | 318 | Rust | Unlicense | 極低 | ✅ 活躍 |
| **novel-builder.js** | 33 | Node.js | MIT | 中（CLI 工具） | ✅ 活躍 |
| **Shelf** | 2 | Python+React | MIT | 中（閱讀管理） | ✅ 活躍 |
| **Cedium** | 2 | React+Prisma | MIT | 中（創作文章） | ✅ 活躍 |
| **FicNest** | 1 | TS+Supabase | — | 高 | ⚠️ 略為停滯 |
| **novel-platform** | 1 | Node.js+SQLite | MIT | 極高 | ✅ 活躍 |
| **Zax Kodelex** | 1 | TS+Supabase | MIT | 中至高（偏漫畫） | ✅ 活躍 |
| **Clara** | 1 | Next.js+Firebase | MIT | 中（直排） | ⚠️ 略為停滯 |
| **inovel** | 0 | Python | MIT | 中（開發初期） | ⚠️ 極初期 |

## 分析

### 不依賴 WordPress 且實用性較高的候選

**WriteFreely**（⭐5,262）與 **OTW Archive**（⭐2,300）在 Star 數及成熟度上最為突出。惟兩者皆非「なろう」的完全替代品：

- **WriteFreely**：ActivityPub 聯邦為其優勢。輕量、營運成本低。缺乏小說專用功能（章節管理、排名、分類等），但作為連載部落格已足夠。
- **OTW Archive**：專注於 fanfiction 的正式級軟體。功能豐富，惟設定及營運門檻較高。設計上足以承載 AO3 本身超過 200 萬部作品的規模。

### 設計上最接近「なろう」的專案

**novel-platform**（⭐1）在功能設計上最接近「なろう」（作者註冊、審查、管理、留言、評分、作品管理），惟 Star 數僅 1、UI 為中文、個人開發。

**FicNest**（⭐1）設計理念亦接近，惟依賴 Supabase 且略為停滯。

### 日語生態系統的獨特性

以日語關鍵字搜尋 GitHub 後確認：**目前不存在專為なろう、カクヨム設計的 FOSS 自架替代方案**。日語小說投稿平台在以下方面具獨特複雜性：

- なろう記法（特殊文字格式）
- 直排支援
- 排名機制（日間/週間/月間 PV 統計）
- 評論、書籤、しおり（閱讀進度）功能
- 作者與讀者之間的緊密互動

## 建議

| 需求 | 最佳方案 |
|---|---|
| **著重 Star 數・輕量營運** | WriteFreely（⭐5,262） |
| **專注 fanfiction・正式級** | OTW Archive / AO3（⭐2,300） |
| **最接近「なろう」的功能設計** | novel-platform（Fork 後進行日語/正體中文本地化為現實路徑） |
| **重視聯邦（Fediverse）互通** | WriteFreely 或 Plume |
| **自行開發為前提** | 以 FicNest 或 Cedium 為基礎進行改造 |

## 免責聲明

本報告基於 2026-10-03 當下之網路搜尋及 GitHub 資訊。各專案之授權條款、功能及維護狀態可能隨時間變化。導入前請務必確認各 Repository 之最新狀態。

[^writefreely]: WriteFreely. (n.d.). WriteFreely — open-source publishing platform. Retrieved 2026-10-03, from https://github.com/writefreely/writefreely
[^otwarchive]: Organization for Transformative Works. (n.d.). otwarchive — AO3 software. Retrieved 2026-10-03, from https://github.com/otwcode/otwarchive
[^plume]: Plume. (n.d.). Plume — federated blogging engine. Retrieved 2026-10-03, from https://github.com/Plume-org/Plume
[^showme]: Nigh. (n.d.). show-me-the-story. Retrieved 2026-10-03, from https://github.com/Nigh/show-me-the-story
[^nonograph]: du82. (n.d.). Nonograph. Retrieved 2026-10-03, from https://github.com/du82/nonograph
[^novelbuilder]: 8amjp. (n.d.). novel-builder.js. Retrieved 2026-10-03, from https://github.com/8amjp/novel-builder
[^shelf]: ShogyX. (n.d.). Shelf. Retrieved 2026-10-03, from https://github.com/ShogyX/Shelf
[^cedium]: Rinisnotarobot. (n.d.). Cedium. Retrieved 2026-10-03, from https://github.com/Rinisnotarobot/Cedium
[^ficnest]: FicNest. (n.d.). ficnest-platform. Retrieved 2026-10-03, from https://github.com/FicNest/ficnest-platform
[^novelplatform]: epiphany131. (n.d.). novel-platform（墨閱書齋）. Retrieved 2026-10-03, from https://github.com/epiphany131/novel-platform
[^zaxkodelex]: ZaxMil. (n.d.). zax-kodelex. Retrieved 2026-10-03, from https://github.com/ZaxMil/zax-kodelex
[^clara]: m19e. (n.d.). Clara. Retrieved 2026-10-03, from https://github.com/m19e/clara
[^inovel]: vvvv31. (n.d.). inovel. Retrieved 2026-10-03, from https://github.com/vvvv31/inovel