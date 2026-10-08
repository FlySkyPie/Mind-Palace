# SSG 結合資料庫之靜態網站方案調查

## 問題陳述

以 Docusaurus 為代表的檔案型靜態網站產生器（SSG, Static Site Generator），在文件規模增長時會面臨編譯時間顯著膨脹的問題。核心原因在於每次建構都需從大量純文字檔案重新讀取、解析、轉換全部內容。這引發了一個架構性問題：是否存在 SSG 方案，允許在編輯階段使用資料庫（DB）輔助內容管理，但在建構後產出純靜態檔案，部署時完全不需要資料庫？

## 術語定位

此類方案可歸類為 **DB-aided SSG**（資料庫輔助型靜態網站產生器）或 **two-phase SSG**（兩階段式靜態網站產生器）：

- **編輯階段（Authoring Phase）**：使用資料庫或 CMS 做為內容儲存後端，提供結構化編輯、關聯查詢、多媒體管理等功能
- **建構階段（Build Phase）**：從資料庫萃取內容，編譯為純靜態 HTML/CSS/JS 檔案
- **部署階段（Deployment Phase）**：僅需靜態檔案伺服器，無需資料庫連線

## 現有方案分析

### 1. Publii — 最接近「理想型」的方案

Publii 是一套桌面 CMS 應用程式（Electron 建構，跨平台），作者在本機桌面應用程式中以視覺化介面管理內容，最終產出純靜態 HTML 部署。其 README 描述：

> "Publii provides an easy-to-understand UI much like server-based CMSs such as WordPress or Joomla!... the app runs locally on your desktop rather than on the site's server." [^publii-repo]

**特點：**
- 部署時零資料庫、零 PHP、零管理後台
- 使用 Handlebars 模板系統
- 支援區塊編輯器 / WYSIWYG / Markdown
- 可上傳至 Netlify、S3、GitHub Pages、Google Cloud、SFTP
- 約 7.3k GitHub 星星，GPL-3.0 授權

**限制：** Publii 使用 Electron 內部本機儲存而非 SQLite，且其模板生態系遠小於主流 SSG。

### 2. Git-based CMS 模式

這是目前最成熟的替代路徑：在編輯階段提供 CMS 體驗，但內容最終儲存為 Git 儲存庫中的純文字檔（Markdown/YAML/JSON），再由 SSG 於建構階段產出靜態 HTML。

#### Decap CMS（前身為 Netlify CMS）

> "Decap CMS is a single-page app that you pull into the /admin part of your site. It presents a clean UI for editing content stored in a Git repository." [^decap-repo]

- 約 19.4k GitHub 星星
- 編輯透過 Web UI 提交至 Git 儲存庫
- 搭配 Hugo、Gatsby、Astro 等 SSG 使用
- 部署時僅需靜態檔案

#### TinaCMS

> "TinaCMS stores everything as Markdown in your Git repo, so your content stays clean, portable, and AI-friendly from the moment you publish." [^tina-repo]

- 約 13.8k GitHub 星星
- 提供 GraphQL API 與視覺編輯介面
- 內容以 Markdown 儲存於 GitHub 儲存庫
- 可搭配 Next.js、Hugo、Astro 等 SSG 使用

### 3. 在建構階段查詢外部資料來源的 SSG

許多主流 SSG 可在建構階段從資料庫或 API 拉取資料，產出靜態 HTML。這不是「編輯階段使用資料庫」的原生方案，但可透過自訂腳本達成：

| SSG | 建構階段資料來源 | 部署階段是否需要 DB |
|-----|------------------|-------------------|
| **11ty (Eleventy)** | 可透過 JavaScript 在建構階段查詢任何 SQL/NoSQL 資料庫 | 否 |
| **Hugo** | 支援 CSV/JSON/YAML/TOML 資料檔，可透過 shortcodes 呼叫外部 API | 否 |
| **Gatsby** | 在建構階段透過 GraphQL/REST 從 Headless CMS 擷取內容 | 否 |
| **Astro** | 可透過伺服器端程式碼在建構階段存取資料庫/API | 否 |
| **Next.js (static mode)** | 在建構階段進行 SSR，可查詢資料庫 | 否 |
| **Jekyll** | 支援 `_data/` 目錄資料檔，可透過外掛查詢 DB | 否 |

以上 SSG 皆未將 SQLite 等嵌入式資料庫內建為一級內容來源，但均可透過自訂建構腳本達成。

### 4. Headless CMS + SSG 建構階段擷取

此模式中 Headless CMS 做為編輯階段的「資料庫」，SSG 在建構階段透過 API 取回內容並轉為靜態 HTML：

- **Contentful + SSG**：編輯在 Contentful 中進行，建構時透過 Contentful API 拉取內容
- **Sanity + SSG**：提供結構化內容 API，SSG 在建構階段查詢
- **Strapi + SSG**：自託管 Headless CMS，建構階段 SSG 取用

此類方案的問題在於：建構階段仍需依賴 CMS API 的可用性（網路連線），且若無適當快取，建構可能因 API 限速而變慢。

## 關鍵發現：原生 SQLite 輔助型 SSG 的空白

**截至調查日期，尚無廣泛使用的 SSG 將 SQLite 或其他嵌入式資料庫原生整合為建構階段的內容來源。** 這是生態系中的一項空白。

最接近的方案比較：

| 方案 | 編輯階段有 DB？ | 部署階段需 DB？ | 成熟度 |
|------|---------------|---------------|-------|
| Publii | ✅ 本機桌面儲存 | ❌ | 中（生態系小） |
| Decap CMS + SSG | ✅ Git 版控（無傳統 DB） | ❌ | 高 |
| TinaCMS + SSG | ✅ GraphQL + Git | ❌ | 中高 |
| 11ty + 自訂 SQLite 腳本 | △ 需自訂實作 | ❌ | 中 |
| Gatsby + Headless CMS | △ 依賴 API 可用性 | ❌ | 高 |
| Hugo + 資料檔案 | △ 非 DB 查詢 | ❌ | 高 |

## 建議路徑

若需要在編輯階段享有資料庫的結構化管理優勢，同時在部署時保持純靜態，可考慮：

1. **立即可行（零開發）：** Decap CMS + Hugo/Astro/11ty —— Git-backed 編輯體驗，無需自建資料庫，但編輯階段不以 SQL 查詢的方式管理內容

2. **高度自訂（需要開發）：** 11ty + SQLite 自訂資料來源外掛 —— 撰寫建構腳本，從 SQLite 資料庫讀取內容並產生頁面，編輯階段可使用任何 SQLite 管理工具

3. **桌面應用體驗：** Publii —— 最接近「所見即所得資料庫編輯 → 靜態部署」的無程式碼路徑，但受限於 Publii 本身的生態系

4. **混合方法：** 使用 Notion / Airtable 等作為內容編輯後端，SSG 在建構階段透過 API 拉取內容，搭配本地快取以避免建構失敗

## 結論

目前不存在將 SQLite 原生整合為建構階段內容來源的主流 SSG。但透過 Git-based CMS（Decap CMS、TinaCMS）或自訂建構腳本（11ty + SQLite），可以有效達到「編輯階段使用資料庫或類似體驗，部署階段零資料庫」的目標。對於文件規模達到 Docusaurus 開始變慢的量級，遷移至 Hugo 或 11ty 並搭配 Git-based CMS 是目前最成熟的替代方案。

---

[^publii-repo]: Publii. (n.d.). Publii — Open-source static site CMS. Retrieved 2026-10-04, from https://github.com/getpublii/publii

[^publii-site]: Publii. (n.d.). Publii Official Website. Retrieved 2026-10-04, from https://getpublii.com

[^decap-repo]: Decap CMS. (n.d.). Decap CMS — Git-based CMS for Static Site Generators. Retrieved 2026-10-04, from https://github.com/decaporg/decap-cms

[^tina-repo]: TinaCMS. (n.d.). TinaCMS — The Git-backed headless CMS. Retrieved 2026-10-04, from https://github.com/tinacms/tinacms

[^tina-site]: TinaCMS. (n.d.). TinaCMS Official Website. Retrieved 2026-10-04, from https://www.tina.io

[^staticgen]: StaticGen. (n.d.). Static Site Generator Directory. Retrieved 2026-10-04, from https://www.staticgen.com

[^bevry]: Bevry. (n.d.). Static Site Generators Directory. Retrieved 2026-10-04, from https://staticsitegenerators.bevry.me

[^hackernoon]: Hackernoon. (n.d.). How We Built Our Software Documentation On Docusaurus. Retrieved 2026-10-04, from https://hackernoon.com/how-we-built-our-software-documentation-on-docusaurus