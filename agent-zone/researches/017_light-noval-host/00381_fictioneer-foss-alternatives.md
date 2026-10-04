# Fictioneer 替代方案：非 WordPress 依賴之 FOSS 專案調查

## 概述

Fictioneer 是一套基於 WordPress 的主題，專供網路小說（web fiction）發布與閱讀。然而 WordPress 架構帶來了維護與安全上的額外負擔。本報告調查不依賴 WordPress 的 FOSS 替代方案，聚焦於能夠自我託管（self-hosted）且仍在維護的專案。

## 調查結果

### 1. Dragonfish（Offprint）

最成熟的即時可用方案。由 Offprint Studios 以 Rust 與 Leptos 框架開發，為完整的小說發布平台，目前已於 offprint.cafe 實際運作[^df-1]。

- **開發語言**：Rust（Leptos 框架）、Bun（JavaScript 工具鏈）
- **資料庫**：PostgreSQL 18、Redis
- **授權條款**：Apache-2.0
- **功能亮點**：使用者註冊、故事發布章節管理、Docker 部署、全文搜尋、RSS
- **GitHub**：⭐ 13 顆星 | 7 個 fork | 3,915 次提交 | 活躍開發
- **系統需求較高**：需要 Rust nightly、PostgreSQL 18、Redis，基礎設施負擔較重

### 2. Novel Platform（墨閱書齋）

以 Node.js 打造的自託管中文網路小說平台，設計上注重安全與輕量[^np-1]。

- **開發語言**：JavaScript（Node.js ≥22.13、Express、vanilla JS 前端）
- **資料庫**：SQLite（無需外部資料庫程序）
- **授權條款**：MIT
- **功能亮點**：PWA 行動應用、全文搜尋（含中文分詞）、多作者管理與審核、書籤與閱讀進度、章節評論與評分、外部來源整合（Wikisource、Project Gutenberg）、Markdown 清理渲染、單容器 Docker 部署、SSRF 防護
- **GitHub**：⭐ 1 顆星 | 0 個 fork | 活躍開發（2025-07-31 更新）
- **注意**：主要面向中文讀者，但有英文 README

### 3. Storyfoundry

以 Astro 為基礎的靜態小說發布方案，將 Markdown 章節轉換為完整網站，適合偏好靜態網站的作者[^sf-1]。

- **開發語言**：TypeScript（Astro）、Node.js
- **授權條款**：未明確標示
- **功能亮點**：Markdown 章節管理、全文搜尋、RSS feed、EPUB 與 PDF 生成、GitHub Pages 部署、Pages CMS 整合瀏覽器編輯、Pandoc + Typst 排版、GitHub Actions CI/CD
- **GitHub**：⭐ 0 顆星 | 0 個 fork | 極早期（3 次提交）

### 4. Fictionhub

以 Python Django 打造的舊型小說發布社群平台，目前已歸檔[^fh-1]。

- **開發語言**：Python（Django）、JavaScript（Node.js）
- **授權條款**：AGPL-3.0
- **最後更新**：2018-05-28
- **GitHub**：⭐ 28 顆星 | 2 個 fork | 歸檔，不再維護

### 5. publicat

法語 Wattpad/AO3 複製品，屬於學校專案，已宣告停滯[^pb-1]。

- **開發語言**：PHP、vanilla JavaScript
- **資料庫**：MySQL
- **授權條款**：MPL-2.0
- **僅 1 次提交**，法語限定

### 6. serialized-api

序列化小說網路平台 API 實驗，Node.js/Express/React/MongoDB 架構，最後更新 2023-01-03[^sa-1]。

### 7. InkWeaver-Front

Angular/TypeScript 網頁小說寫作平台前端，最後更新 2021-12-01，已停滯[^iw-1]。

### 8. Fictionalicious

PHP 簡易故事發布平台，最後更新 2013-01-14，極度老舊[^fc-1]。

## 比較摘要

| 專案 | 語言 | ⭐ 星星 | 授權 | 活躍？ | 最佳適用場景 |
|---|---|---|---|---|---|
| **Dragonfish** | Rust (Leptos) | ⭐13 | Apache-2.0 | ✅ 是 | 生產級小說發布站 |
| **Novel Platform** | Node.js | ⭐1 | MIT | ✅ 是 | 自託管輕量中文小說平台 |
| **Storyfoundry** | Astro/TS | ⭐0 | 無 | ⚠️ 極初期 | 靜態小說網站 + EPUB/PDF |
| Fictionhub | Python/Django | ⭐28 | AGPL-3.0 | ❌ 否 | 遺留社群平台 |
| serialized-api | Node.js/React | ⭐1 | 無 | ❌ 否 | API 實驗 |
| InkWeaver-Front | TypeScript | ⭐0 | 無 | ❌ 否 | 寫作介面 |
| Fictionalicious | PHP | ⭐1 | 無 | ❌ 否 | 極簡小說發布 |
| publicat | PHP | ⭐5 | MPL-2.0 | ❌ 否 | 法語 Wattpad 複製 |

## 推薦

- **若需要即時可用的生產級平台**：Dragonfish 是最完整的選擇，但需承擔 Rust + PostgreSQL + Redis 的基礎設施負擔。
- **若偏好輕量、低維護**：Novel Platform 以 SQLite 單容器運作，安全設計完善，但主要面向中文內容。
- **若偏好靜態網站**：Storyfoundry 以 Markdown 為源、生成 EPUB/PDF，工作流程簡潔。

## 免責聲明

以上資訊基於公開 GitHub 儲存庫與文件，實際功能可能因版本而異。建議在部署前自行測試。

[^df-1]: OffprintStudios. (n.d.). Dragonfish — Offprint fiction publishing platform. Retrieved 2026-10-03, from https://github.com/OffprintStudios/dragonfish
[^np-1]: epiphany131. (n.d.). Novel Platform — 墨閱書齋. Retrieved 2026-10-03, from https://github.com/epiphany131/novel-platform
[^sf-1]: yessur3808. (n.d.). Storyfoundry. Retrieved 2026-10-03, from https://github.com/yessur3808/storyfoundry
[^fh-1]: lumenwrites. (n.d.). Fictionhub. Retrieved 2026-10-03, from https://github.com/lumenwrites/fictionhub
[^pb-1]: nyukaproject. (n.d.). publicat. Retrieved 2026-10-03, from https://github.com/nyukaproject/publicat
[^sa-1]: paradoxinversion. (n.d.). serialized-api. Retrieved 2026-10-03, from https://github.com/paradoxinversion/serialized-api
[^iw-1]: Plotypus. (n.d.). InkWeaver-Front. Retrieved 2026-10-03, from https://github.com/Plotypus/InkWeaver-Front
[^fc-1]: sunnefa. (n.d.). Fictionalicious. Retrieved 2026-10-03, from https://github.com/sunnefa/Fictionalicious