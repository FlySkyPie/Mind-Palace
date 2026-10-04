# 自架 FOSS 網路小說平台：ncode.syosetu.com（小説家になろう）替代方案調查

## 概要

本報告調查可自架（self-hosted）的自由開源軟體（FOSS）替代方案，以取代日本網路小說平台 **ncode.syosetu.com（小説家になろう / Shousetsuka ni Narou）**。重點尋找具備作者註冊、章節連載、讀者互動（留言、書籤）、作品分類發現等功能的平台級軟體。

## 1. Fictioneer — WordPress 網路小說主題 ⭐ 188

**最接近 Narou 體驗的自架方案。**

Fictioneer 是專為網路小說（web fiction）、網路連載（web novel）、粉絲創作（fanfiction）設計的 WordPress 主題，提供完整的章節管理與讀者互動功能。

- **GitHub**: https://github.com/Tetrakern/fictioneer
- **官網**: https://fictioneer-theme.com/
- **授權**: GPL v3
- **技術棧**: WordPress（PHP）、MySQL/MariaDB

**功能特色**:
- 原生內容類型：故事、章節、選集、推薦
- 自訂網頁閱讀器（暗/亮模式切換）
- 內建 ePUB 轉換
- 書籤、閱讀進度、文字轉語音
- AJAX 留言系統（含回覆訂閱）
- 進階搜尋、標籤、類型、內容警告、作品世界設定
- OAuth 2.0 登入（Discord、Google、Twitch、Patreon）
- Patreon 付費牆、角色管理、私人留言
- RSS、SEO、GDPR 合規
- Elementor 頁面建構器相容

**適用場景**: 需要一個完整的、如同 Narou 的網路小說平台。作者註冊 → 發表章節 → 讀者留言追蹤。需自備 WordPress 主機。[^fictioneer]

## 2. WriteFreely — 極簡聯邦式出版平台

乾淨、Markdown 為基礎的寫作平台。可視為自架版的 Medium/Write.as，且支援聯邦協議（ActivityPub）。

- **GitHub**: https://github.com/writefreely/writefreely ⭐ 5,300
- **官網**: https://writefreely.org
- **授權**: AGPL-3.0
- **技術棧**: Go（單一二進位檔，極輕量，可跑在樹莓派）

**功能特色**:
- 無干擾、自動儲存的 Markdown 編輯器
- ActivityPub 聯邦（讀者可從 Mastodon 等平台追蹤）
- 單一帳號可建立多個部落格
- 草稿、靜態頁面
- 注重隱私（最小資料收集、可匿名）
- 20+ 語言在地化、RTL 支援
- 社區/實例模型 — 可託管一群作者
- OAuth 2.0
- 成熟穩定（Write.as 上支援超過 55 萬個部落格）

**適用場景**: 想建立輕量級寫作社群，並與聯邦宇宙（Fediverse）互通。缺乏小說專用功能（類型、標籤、章節管理），但極易維護。[^writefreely]

## 3. Plume — Rust 聯邦式部落格引擎

以 Rust 撰寫的聯邦式部落格引擎，支援 ActivityPub，適合想與 Fediverse 保持連結的作者。

- **GitHub**: https://github.com/Plume-org/Plume ⭐ 2,200
- **官網**: https://joinplu.me/
- **授權**: AGPL-3.0
- **技術棧**: Rust、Rocket framework、Diesel ORM、PostgreSQL

**功能特色**:
- 部落格為核心 — 單一帳號可建立多個部落格
- ActivityPub 聯邦（與 Mastodon、Pleroma、WriteFreely 互通）
- 多媒體管理（圖片、Podcast 音檔）
- 多人協作寫作（預計功能）
- Docker 部署
- 20+ 語言

**適用場景**: 與 WriteFreely 類似，但更偏重部落格形式。適合連載小說的作者，但缺乏 Narou 式的小說分類與章節管理。[^plume]

## 4. ManuHaven — 自架小說寫作工作室

一個新興但功能完整的開源寫作工作室，自稱「Scrivener + Vellum 的開源自架替代」。完全在瀏覽器中運行。

- **GitHub**: https://github.com/julianavellaneda/manuhaven ⭐ 新專案
- **授權**: AGPL-3.0
- **技術棧**: Next.js 16、React 19、PostgreSQL 18、Docker

**功能特色**:
- 完整稿編輯器（Tiptap）— 章節管理、拖曳排序、自動儲存、版本歷史
- AI 編輯輔助（自備 API 金鑰 — Anthropic、OpenAI、Google）
- 匯出 EPUB 3 及印刷級 PDF（透過 Pandoc + WeasyPrint）
- 版稅追蹤（匯入 KDP、Apple、Kobo、StreetLib CSV）
- 編輯報告、風格分析、連續性檢查
- 專案儀表板（書本封面、狀態追蹤）
- 雙語（英語 + 西班牙語）
- 匯入 .docx 及 .txt

**適用場景**: 偏向寫作與排版工具而非公開連載平台。適合需要強大編輯與匯出功能的作者，但缺乏讀者社群功能。[^manuhaven]

## 5. OpenWrite — AI 輔助小說寫作平台

以 Cloudflare 為基礎的開源寫作平台，搭配 AI 輔助，使用自備 API 金鑰。

- **GitHub**: https://github.com/ilrein/openwrite ⭐ 55
- **官網**: https://openwrite.iliareingold.com
- **授權**: AGPL-3.0
- **技術棧**: TypeScript（React 19、Hono、Cloudflare Workers、D1/SQLite）、Bun

**功能特色**:
- 故事地圖（Story map）— 基於節點的 AI 生成（前提 → 幕 → 章節 → 場景）
- 富文本編輯器，自動儲存，各專案進度追蹤
- 章節管理，安全並行編輯偵測
- AI 寫作助手（自備金鑰 — OpenRouter、OpenAI、Anthropic、Groq、Gemini、Cohere、Ollama）
- Codex 世界建構（角色、地點、傳說、劇情點）
- Markdown 匯出
- 支援小說、三部曲、系列、短篇、劇本等專案類型
- 可免費自架在 Cloudflare 免費方案上

**適用場景**: 寫作規劃工具。對小說組織能力強，但非公開平台。Cloudflare Workers 模型使其自架成本極低。[^openwrite]

## 6. narou-viewer — Narou/Kakuyoku 自架閱讀器

專為 Narou 生態系打造的自架閱讀器，用於抓取與閱讀小説家になろう及カクヨム（Kakuyomu）的作品。

- **GitHub**: https://github.com/iuill/narou-viewer ⭐ 1
- **授權**: MIT
- **技術棧**: Go、TypeScript、React、Bun、Docker

**功能特色**:
- 本地圖書館管理（抓取 Narou/Kakuyomu 作品）
- 自訂網頁閱讀器（直排/橫排、字型大小、行距、主題）
- 書籤、閱讀位置追蹤
- 文字轉語音（速度與聲音設定）
- AI 角色/術語提取及小說問答
- AI 校對（保留原文）
- 離線支援
- Docker Compose 部署

**適用場景**: 這是**消費型工具**，用於離線閱讀 Narou 作品，並非發布平台。對 Narou 生態系的讀者有用，但非替代方案。[^narouviewer]

## 7. Pressbooks — 書籍內容管理系統

基於 WordPress Multisite 的全書 CMS，用於出版書籍至網頁並匯出為 EPUB、PDF、XML。

- **GitHub**: https://github.com/pressbooks/pressbooks ⭐ 大型社群
- **官網**: https://pressbooks.org
- **授權**: GPL v3
- **技術棧**: WordPress Multisite（PHP）

**功能特色**:
- 多格式匯出（EPUB、PDF、XML、HTML）
- 網頁優先的書籍出版
- 常見格式匯入
- 使用者角色與權限
- 詞彙表、腳註、索引支援
- 主題化、無障礙支援
- 大型社群與商業支援

**適用場景**: 偏重完整書籍出版，而非逐章連載。適合出版已完成作品的作者，對 Narou 風格的連載平台來說過於厚重。[^pressbooks]

## 8. Nonograph — 匿名自架出版平台

極簡、注重隱私的匿名出版平台 — 可視為自架版的 Telegraph（最小發布工具）。

- **GitHub**: https://github.com/du82/nonograph ⭐ 318
- **官網**: https://nonogra.ph
- **授權**: Unlicense（公有領域）
- **技術棧**: Rust（Rocket framework）、Docker

**功能特色**:
- 無帳號、無追蹤、無分析
- 即時發布 — 撰寫即取得分享連結
- Markdown 編輯器
- Tor 隱藏服務支援
- 極輕量

**適用場景**: 最小匿名發布工具。適合發布單篇故事或章節，但完全缺乏 Narou 的社群功能（個人簡介、留言、連載管理）。[^nonograph]

## 比較總結

| 專案 | 最適合 | 自架難度 | 小說功能 | 社群功能 | 語言 |
|---|---|---|---|---|---|
| **Fictioneer** | 運行 Narou 式網路小說網站 | ⭐⭐ WordPress | ✅ 完整 | ✅ 完整 | 英語（可翻譯） |
| **WriteFreely** | 極簡聯邦部落格社群 | ⭐ Go 單一二進位 | ⚠️ 一般部落格 | ✅ ActivityPub | 20+ 語言 |
| **Plume** | 聯邦多部落格出版 | ⭐⭐ Rust + PostgreSQL | ⚠️ 部落格引擎 | ✅ ActivityPub | 20+ 語言 |
| **ManuHaven** | 瀏覽器內寫作排版 | ⭐ Docker | ✅ 稿編輯器 + AI + 匯出 | ⚠️ 單一使用者 | 英語 + 西班牙語 |
| **OpenWrite** | AI 輔助小說規劃/寫作 | ⭐ Cloudflare Workers | ✅ 故事地圖 + Codex | ⚠️ 單一使用者 | 英語 |
| **narou-viewer** | 閱讀/歸檔 Narou 內容 | ⭐ Docker | ⚠️ 僅閱讀 | ❌ 僅閱讀 | 日語 |
| **Pressbooks** | 完整書籍出版 | ⭐⭐⭐ WordPress Multisite | ⚠️ 書籍層級 | ⚠️ 有限社群 | 英語 |
| **Nonograph** | 匿名單篇發布 | ⭐ Docker/Rust | ❌ 極簡 | ❌ 無帳號 | 英語 |

## 推薦

若想尋找最接近 **Narou 的自架替代方案**（作者逐章連載網路小說、讀者可追蹤、留言、發現作品）：

1. **🥇 Fictioneer** — 最完整的匹配。搭配 WordPress 可獲得完整的網路小說網站：故事、章節、類型、內容警告、留言、書籤、閱讀模式、RSS、作者簡介。是唯一專為網路小說平台設計的主題。

2. **🥈 WriteFreely** — 若偏好輕量聯邦方案。適合建立寫作社群並與 ActivityPub 互通。維護成本極低。

3. **🥉 ManuHaven** — 若重點在**寫作/排版環節**（稿編輯器 → EPUB/PDF）而非社群閱讀平台。最適合需要強大編輯工具的作者。

針對日語網路小說生態系，**narou-viewer** 可作為補充工具（用於離線抓取與閱讀 Narou 內容），但它並非發布平台。

[^fictioneer]: Tetrakern. (n.d.). Fictioneer — WordPress theme for web fiction. Retrieved 2026-10-03, from https://github.com/Tetrakern/fictioneer
[^writefreely]: WriteFreely. (n.d.). WriteFreely — An open-source publishing platform. Retrieved 2026-10-03, from https://writefreely.org
[^plume]: Plume. (n.d.). Plume — A federated blogging engine. Retrieved 2026-10-03, from https://github.com/Plume-org/Plume
[^manuhaven]: Avellaneda, J. (n.d.). ManuHaven — Self-hosted novel writing studio. Retrieved 2026-10-03, from https://github.com/julianavellaneda/manuhaven
[^openwrite]: Rein, I. (n.d.). OpenWrite — AI-powered open-source writing platform. Retrieved 2026-10-03, from https://github.com/ilrein/openwrite
[^narouviewer]: iuill. (n.d.). narou-viewer — Self-hosted web novel viewer for Narou/Kakuyomu. Retrieved 2026-10-03, from https://github.com/iuill/narou-viewer
[^pressbooks]: Pressbooks. (n.d.). Pressbooks — Book content management system. Retrieved 2026-10-03, from https://pressbooks.org
[^nonograph]: du82. (n.d.). Nonograph — Anonymous self-hosted publishing platform. Retrieved 2026-10-03, from https://github.com/du82/nonograph