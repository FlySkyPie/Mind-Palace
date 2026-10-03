# Payload CMS 介紹

## 概述

Payload CMS 是一個開源（MIT 授權）、全端 **Next.js 原生**的 Headless CMS 兼應用框架[^gh-repo]。它直接安裝在現有的 Next.js `/app` 目錄中，提供即用的 TypeScript 後端、自動產生的管理後臺、REST/GraphQL API、認證、存取控制、檔案儲存等功能，開發者擁有完整的資料與程式碼主權。

## 解決的問題

| 問題 | Payload 的解法 |
|---|---|
| **Headless CMS 的複雜性** — 傳統 headless CMS 需要分別管理後端、前端與膠水程式 | 將兩者合併為一個 Next.js 應用，同一個 `/app` 目錄[^docs-concept] |
| **供應商鎖定** — SaaS 型 CMS（Contentful、Sanity）使用者不擁有資料 | 完全開源 MIT，可自架，資料與基礎設施完全自主[^license] |
| **開發者體驗與編輯者體驗的取捨** — 多數 CMS 犧牲一方 | 以 TypeScript 程式碼定義結構，自動產生管理後臺[^docs-admin] |
| **「從頭打造 vs 採購」兩難** | 提供產品級速度，同時保留客製靈活度 |
| **AI/RAG 整合門檻** | 原生支援向量嵌入與內容分塊（chunking）[^ai-rag] |

## 主要功能

### 核心 CMS 功能

- **管理後臺** — 基於 React + Next.js App Router 自動產生的型別安全管理介面，支援 30+ 語言、淺色/深色主題、完整 CSS 客製[^docs-admin]
- **集合（Collections）** — 以 TypeScript 設定檔定義內容模型（部落格文章、商品等）[^docs-collections]
- **全域（Globals）** — 網站設定、導覽列等單例內容[^docs-globals]
- **REST API** — 位於 `/api` 的即時 HTTP 端點，支援分頁、深度與排序[^docs-rest]
- **GraphQL API** — 完整 GraphQL API，附內建 playground[^docs-graphql]
- **Local API** — 直接操作資料庫的 Node.js API，無 HTTP 開銷，可在 React Server Components、排程任務、Webhook 中使用[^docs-local]
- **認證** — 內建 HTTP-only Cookie、JWT、API Key、客製策略[^docs-auth]
- **存取控制** — 函數式權限，精細到文件、欄位、操作層級（CRUD）[^docs-acl]
- **版本與草稿** — 完整版本歷史、差異比對、草稿/發佈流程、回滾[^docs-versions]
- **本地化** — 欄位級 i18n，可個別發佈單一語系，支援 RTL[^docs-localization]
- **富文本編輯器（Lexical）** — 基於 Meta Lexical 的區塊式富文本編輯器，支援自訂內聯與區塊元件[^docs-lexical]
- **即時預覽** — 編輯內容時即時視覺預覽，支援 SSR[^docs-live-preview]
- **多媒體上傳** — 圖片調整、焦點裁切、批量上傳、資料夾組織、檔案版本[^docs-uploads]
- **視覺編輯** — 直接在渲染頁面上 WYSIWYG 編輯[^docs-visual-editing]
- **發佈工作流程** — 多步驟審核流程，含內聯反饋與通知[^docs-workflows]
- **任務佇列** — 內建背景任務系統（延遲任務、排程任務、多步驟流程）[^docs-jobs]
- **掛鉤（Hooks）** — CRUD 操作生命週期掛鉤[^docs-hooks]
- **區塊式版面建構器** — 以可重複使用的區塊（英雄區塊、CTA、功能網格等）組合頁面
- **A/B 測試** — 透過 Next.js SSR 進行零延遲變體測試
- **SEO 外掛** — 內建 SEO 外掛（站地圖、Meta 分析、結構化資料）[^plugin-seo]
- **表單建構外掛** — 拖放式表單建立，儲存在 Payload Schema 中[^plugin-form]

### 外掛生態

- 官方外掛：SEO、Form Builder、Nested Docs、Multi-Tenant、Cloud Storage（S3、GCS、R2）、Sentry、Redirects、MCP 等
- 社群外掛：超過 100 個（GitHub `payload-plugin` 主題）

## 架構

### 前後端分離 vs. 混合模式

Payload 同時支援 **headless** 與 **hybrid** 模式：

- **純 Headless** — 獨立前端（React、Astro、SvelteKit、Remix 等）透過 REST/GraphQL API 取用內容
- **Hybrid Monolith** — Next.js 前端與 Payload 後端在同一個 `/app` 目錄中，透過 React Server Components 直接查詢資料庫，零 HTTP 開銷[^docs-concept]

### 套件結構（模組化）

| 套件 | 角色 |
|---|---|
| `payload` | 核心 ORM 邏輯：CRUD 操作、存取控制、掛鉤、驗證、型別。可在任何 Node 環境執行 |
| `@payloadcms/next` | HTTP 層：管理後臺 UI、REST API、GraphQL API。以 Next.js `(payload)` route group 建置 |
| `@payloadcms/ui` | React 元件函式庫（伺服器+客戶端元件） |
| `@payloadcms/db-*` | 資料庫轉接器：MongoDB（Mongoose）、Postgres（Drizzle ORM）、SQLite（Drizzle）、Vercel Postgres |
| `@payloadcms/richtext-lexical` | Lexical 富文本編輯器（Slate 亦可用） |
| `@payloadcms/graphql` | GraphQL 引擎（僅在使用時載入） |

### 資料庫支援

- **MongoDB**（Mongoose）— 適合動態 schema、在地化需求高的專案、區塊與陣列
- **Postgres**（Drizzle ORM）— 適合嚴謹 schema、關聯完整性、傳統 SQL 流程
- **SQLite**（Drizzle）— 輕量級，適合開發或單一伺服器部署
- **Vercel Postgres**— 無伺服器 Postgres 轉接器

所有轉接器支援幾乎所有 Payload 功能（在地化、陣列、區塊等）[^docs-database]。

### 管理後臺

- 以 **React** + **Next.js App Router** 建置
- 支援 **React Server Components** — 在 SSR 管理檢視中直接查詢資料庫
- 可完全客製：為任何欄位、檢視或動作置換自訂元件
- 生成的檔案位於 `app/(payload)/` route group — 管理後臺與前端程式碼之間有乾淨的邊界
- 自動適應 schema 變更[^docs-admin]

## 部署選項

### 自架

Payload 可部署在任何 Node.js 可執行的地方：

- **傳統伺服器/VPS** — Docker、PM2 或任何 Node.js 託管（DigitalOcean、Linode 等）
- **AWS / GCP / Azure** — 以容器部署於 ECS、Cloud Run 等
- **Render / Railway / Fly.io** — 多種 PaaS 選項

### 一鍵部署（Serverless）

**Vercel**（Payload 官方推薦）：
- 一鍵部署：**Next.js**（前端）+ **Neon**（Postgres 資料庫）+ **Vercel Blob**（檔案儲存）
- 免費方案額度充裕
- 完整 serverless 支援（Edge Functions、ISR 等）[^deploy-vercel]

**Cloudflare Workers**：
- 一鍵部署：**Workers**（運算）+ **D1**（全球複寫 SQLite/Postgres）+ **R2**（物件儲存）
- 需 Workers Paid 方案（因大小限制）
- Cloudflare 自身使用 Payload 驅動 **Cloudflare TV**[^deploy-cloudflare]

### Payload Cloud

Payload Cloud 曾為官方代管方案（2023 年 7 月結束 Beta），但 Figma 收購後主要專注於開源產品與自架/serverless 部署路徑。Payload Cloud 頁面目前已重新導向至登入頁。

## 版本發展歷程

### Payload 3.0（2024 年 11 月 19 日釋出）

重大版本，將 Payload 重寫為 **Next.js 原生**[^blog-30]：

| 領域 | 變更 |
|---|---|
| **架構** | 直接安裝於任何 Next.js 應用，無需獨立後端伺服器，支援 Next.js App Router |
| **資料庫** | Postgres 與 Lexical RTE 標示為穩定，新增 SQLite 與 Vercel Postgres 轉接器（Drizzle） |
| **模組化** | 最小 Node-only 模式依賴從 88 降為 27，GraphQL 僅在使用時載入，純 ESM |
| **新欄位型別** | `join` 欄位（跨集合查詢）、虛擬欄位（`virtual: true`） |
| **效能** | 使用 React Compiler 最佳化管理後臺操作 |
| **任務佇列** | 完整背景任務系統，支援排程、相依性、多步驟流程 |
| **功能** | 批量上傳、文件鎖定、單一語系發佈、使用者名稱/Email 登入、多欄位排序、自訂檔案命名 |
| **即時預覽** | 支援 SSR（React Server Components） |
| **開發體驗** | 自動產生 TypeScript 型別，簡化 import |

### Payload 3.90.0（2026 年 9 月）

- 3.x 與 4.0 canary 的重大安全性修補
- 多個漏洞修復
- 包含中斷性變更與遷移指南[^security-update]

### Payload 4.0（開發中 — Beta 預計 2026 年 Q3-Q4）

| 功能 | 狀態 |
|---|---|
| **管理後臺重新設計** | 全新設計 — 更乾淨的檢視、更好的主題、Tailwind 相容性改進、移除 Sass、語意化樣式代幣[^blog-40] |
| **TanStack（React Router）支援** | 框架轉接器模式 — Payload 將能在 Next.js 之外與 TanStack 併存，已有 Demo |
| **階層結構（核心原語）** | 資料夾、標籤、分類、樹狀導覽 — 取代 Nested Docs 外掛，支援側邊欄分頁 |
| **數位資產管理（DAM）** | PDF 預覽、影片/音訊支援、在地化檔案、檔案版本、分享連結、使用引用 |
| **MCP（Model Context Protocol）** | MCP 外掛簡化，改善本地開發設定 |
| **Payload Skills** | 針對 LLM 輔助開發的代理專用指令，可安裝於 Cursor、Claude Code 等 |

### 最新穩定版本

- **v3.90.2**（截至 2026 年 9 月）
- **v4.0.0-canary.37**（2026 年 9 月 24 日）

## 與同類工具比較

### vs. WordPress（含 ACF）

| 維度 | Payload | WordPress + ACF |
|---|---|---|
| **架構** | 現代 headless/hybrid（Next.js 原生） | 單體 PHP + REST（WP REST API） |
| **開發體驗** | TypeScript、程式碼優先 schema、Git 友善 | PHP、資料庫驅動、需 WP 管理介面 |
| **管理後臺** | 自動產生、完全可客製的 React 管理介面 | 主題式、依賴外掛 |
| **部署** | Vercel、Cloudflare、Docker、任何 Node.js 主機 | 需 PHP 主機、MySQL |
| **擴充性** | TypeScript 掛鉤、自訂 React 元件、外掛 | PHP 掛鉤、action/filter 系統、外掛 |
| **效能** | GraphQL 標竿測試比 WP REST 快 2-10 倍 | PHP 開銷影響效能 |
| **適合** | 現代 Next.js 專案、自訂建置 | 既有 WP 生態系、非技術編輯者 |

Payload 明確定位為 ACF 的替代方案，提供詳細遷移指南[^wp-migration]。

### vs. Strapi

| 維度 | Payload | Strapi |
|---|---|---|
| **框架** | Next.js 原生（亦可獨立 Node.js） | Express + Koa，框架無關 |
| **管理後臺** | React + Next.js App Router，深度可客製 | React 基礎，可透過 extensions 客製 |
| **資料庫** | MongoDB、Postgres、SQLite（轉接器） | SQLite（開發用）、Postgres、MySQL、MongoDB、MariaDB |
| **認證** | HTTP-only Cookie + JWT + API Key | JWT，支援 SSO 與 providers |
| **TypeScript** | 完整 TypeScript，自動產生型別 | TypeScript 支援但整合較淺 |
| **RAG/AI** | 原生向量嵌入與內容分塊 | 無內建支援 |
| **授權** | MIT（永久免費） | MIT（免費）+ Cloud Enterprise |
| **社群** | 45.1k stars | 46k+ stars |

### vs. Sanity

| 維度 | Payload | Sanity |
|---|---|---|
| **部署** | 自架，任何 Node.js 提供商 | SaaS 專用（Sanity 管理） |
| **資料所有權** | 完全自主 | 儲存在 Sanity 基礎設施 |
| **價格** | 免費（開源） | 免費方案 + 使用量計費 |
| **API** | REST + GraphQL + Local API | GROQ（自訂查詢語言）+ GraphQL |
| **客製化** | 程式碼優先 TypeScript，完整 React 管理後臺客製 | GROQ + schema 客製，但管理後臺為 SaaS |
| **離線/自架** | 可完全自架 | 無自架選項 |
| **適合** | 需要資料主權、自訂基礎設施的團隊 | 想要零基礎設施、API 優先的團隊 |

### vs. Contentful

| 維度 | Payload | Contentful |
|---|---|---|
| **部署** | 自架 | SaaS |
| **價格** | 免費（開源） | 免費方案 + 昂貴企業方案 |
| **資料所有權** | 完全自主 | Contentful 管理 |
| **API** | REST + GraphQL + Local API | REST + GraphQL |
| **客製化** | 無限（擁有程式碼） | 限於 Contentful 開放的範圍 |
| **遷移** | 無鎖定 | 從 Contentful 遷移至 Payload 的文件完善 |

### vs. Keystone.js

Payload 與 Keystone.js 的比較：

| 維度 | Payload | Keystone.js |
|---|---|---|
| **框架** | Next.js 原生 | Express + 自訂管理 UI |
| **管理後臺** | React + Next.js | Pug templates + React |
| **資料庫** | MongoDB、Postgres、SQLite | Prisma（Postgres、SQLite、MySQL、MongoDB） |
| **TypeScript** | 深度整合 | 良好支援 |
| **成熟度** | 3 個主要版本，快速迭代 | 較老的專案（已到 v7） |
| **生態** | 外掛生態、AI/RAG、DAM、Commerce | 較簡單的外掛系統 |

### vs. Tina CMS

| 維度 | Payload | Tina CMS |
|---|---|---|
| **Git 整合** | 資料庫驅動（Mongo/Postgres/SQLite） | Git 基礎（檔案系統即資料庫） |
| **編輯模式** | 管理後臺 + 視覺編輯 | 網站就地編輯 |
| **框架** | Next.js（其他框架未到穩定） | 任何靜態網站 + React |
| **規模** | 企業級、大量資料 | 小到中型內容網站 |
| **架構** | 完整後端框架 | Git 備份的 CMS 層 |

## 何時使用 vs. 不使用 Payload

### ✅ 適合使用 Payload 的情境

| 情境 | 原因 |
|---|---|
| **你正在使用 Next.js** | 沒有其他 CMS 能如此深度整合 — 同一個 `/app` 目錄、React Server Components、零 HTTP 資料庫查詢 |
| **你需要 CMS + 應用框架** | Payload 同時處理內容管理與內部工具、管理面板、儀表板 — 全在同一個程式碼庫 |
| **資料主權是關鍵** | 你控制資料庫、程式碼與基礎設施，SaaS 供應商無法鎖定你 |
| **你需要精細的存取控制** | 函數式權限達文件、欄位、操作層級 — 適合 multi-tenant SaaS、入口網站等 |
| **你想無痛整合 AI/RAG** | 原生向量嵌入與內容分塊 — 唯一為 RAG 設計的 CMS |
| **你正在從 WordPress/ACF 遷移** | Payload 是直接的升級路徑，提供詳細遷移指南 |
| **你需要客製管理 UI** | 整個 React 管理後臺可客製 — 置換元件、新增檢視、白標 |
| **Serverless/Hybrid 部署** | Vercel + Neon + Blob 一鍵部署，Cloudflare Workers + D1 + R2 一鍵部署 |

### ❌ 不適合使用 Payload 的情境

| 情境 | 原因 |
|---|---|
| **不需要管理後臺** | 如果完全透過程式碼/Git 管理內容，Tina 或靜態網站生成器可能更適合 |
| **你適合 Webflow/Framer** | 對標準行銷網站，no-code/SaaS 建置工具可能更快 |
| **你已有完整資料層** | 如果只是要視覺化既有資料，Payload 的 schema 優先模式會增加開銷 |
| **前端不是 React/Next.js** | 雖然 Payload 可與 Astro、Remix、SvelteKit 等協作（API 模式），其最大優勢（React Server Components、單一應用目錄）僅適用於 Next.js。TanStack 支援尚未穩定 |
| **編輯者需要零程式碼** | Payload 是程式碼優先 — schema 變更需要開發者。純 no-code CMS 可考慮 Webflow 或 Sanity |
| **團隊沒有 TypeScript/Node.js 經驗** | Payload 完全由程式碼驅動，非 JavaScript 團隊可能發現 WordPress 或 Strapi 更易使用 |
| **你需要純電子商務平台** | Payload 可整合 Stripe 進行 headless commerce，但 Shopify/Magento 等對以電子商務為主要需求的專案更成熟 |

## Figma 收購

2024 年末，Payload CMS 被 Figma 收購[^figma-acq]。收購後，Payload 承諾維持開源（MIT）授權，並繼續作為獨立產品發展。Figma 收購的主要考量被認為是 Figma 自身需要強大的內容管理架構來支撐其協作設計平台的規模化。收購後的發展重點仍集中在開源產品、自架部署、以及 serverless 部署路徑的改進。

## 結論

Payload CMS 是現代 Next.js 生態中獨特的 CMS/應用框架。其「安裝在 Next.js 中」的架構、程式碼優先的 schema 定義、自動產生的 React 管理後臺、以及對 AI/RAG 的原生支援，使其成為以下場景的強力選擇：

- 以 Next.js 為基礎的團隊，希望減少 CMS 前後端之間的膠水程式
- 需要資料主權與自訂基礎設施的組織
- 希望將 CMS 與內部工具整合在單一程式碼庫的專案
- 需要 AI 整合能力的現代 Web 應用

與傳統 CMS（WordPress）或 SaaS CMS（Contentful、Sanity）相比，Payload 在靈活性、資料主權與成本控制方面具有顯著優勢，但要求團隊具備 TypeScript/Node.js 能力。

[^gh-repo]: Payload CMS. (n.d.). payloadcms/payload. Retrieved 2026-10-03, from https://github.com/payloadcms/payload
[^docs-concept]: Payload CMS. (n.d.). Concepts — How Payload works. Retrieved 2026-10-03, from https://payloadcms.com/docs/getting-started/concepts
[^docs-admin]: Payload CMS. (n.d.). Admin Panel — Overview. Retrieved 2026-10-03, from https://payloadcms.com/docs/admin/overview
[^license]: Payload CMS. (n.d.). License. Retrieved 2026-10-03, from https://github.com/payloadcms/payload
[^docs-collections]: Payload CMS. (n.d.). Collections. Retrieved 2026-10-03, from https://payloadcms.com/docs/getting-started/concepts
[^docs-globals]: Payload CMS. (n.d.). Globals. Retrieved 2026-10-03, from https://payloadcms.com/docs/getting-started/concepts
[^docs-rest]: Payload CMS. (n.d.). REST API. Retrieved 2026-10-03, from https://payloadcms.com/docs/api/rest
[^docs-graphql]: Payload CMS. (n.d.). GraphQL API. Retrieved 2026-10-03, from https://payloadcms.com/docs/api/graphql
[^docs-local]: Payload CMS. (n.d.). Local API. Retrieved 2026-10-03, from https://payloadcms.com/docs/api/local
[^docs-auth]: Payload CMS. (n.d.). Authentication. Retrieved 2026-10-03, from https://payloadcms.com/docs/authentication/overview
[^docs-acl]: Payload CMS. (n.d.). Access Control. Retrieved 2026-10-03, from https://payloadcms.com/docs/access-control/overview
[^docs-versions]: Payload CMS. (n.d.). Versions. Retrieved 2026-10-03, from https://payloadcms.com/docs/versions/overview
[^docs-localization]: Payload CMS. (n.d.). Localization. Retrieved 2026-10-03, from https://payloadcms.com/docs/localization/overview
[^docs-lexical]: Payload CMS. (n.d.). Rich Text — Lexical. Retrieved 2026-10-03, from https://payloadcms.com/docs/rich-text/lexical
[^docs-live-preview]: Payload CMS. (n.d.). Live Preview. Retrieved 2026-10-03, from https://payloadcms.com/docs/admin/live-preview
[^docs-uploads]: Payload CMS. (n.d.). Uploads. Retrieved 2026-10-03, from https://payloadcms.com/docs/upload/overview
[^docs-visual-editing]: Payload CMS. (n.d.). Visual Editing. Retrieved 2026-10-03, from https://payloadcms.com/docs/admin/visual-editing
[^docs-workflows]: Payload CMS. (n.d.). Workflows. Retrieved 2026-10-03, from https://payloadcms.com/docs/versions/workflows
[^docs-jobs]: Payload CMS. (n.d.). Jobs Queue. Retrieved 2026-10-03, from https://payloadcms.com/docs/jobs
[^docs-hooks]: Payload CMS. (n.d.). Hooks. Retrieved 2026-10-03, from https://payloadcms.com/docs/hooks/overview
[^plugin-seo]: Payload CMS. (n.d.). SEO Plugin. Retrieved 2026-10-03, from https://payloadcms.com/docs/plugins/seo
[^plugin-form]: Payload CMS. (n.d.). Form Builder Plugin. Retrieved 2026-10-03, from https://payloadcms.com/docs/plugins/form-builder
[^docs-database]: Payload CMS. (n.d.). Database — Overview. Retrieved 2026-10-03, from https://payloadcms.com/docs/database/overview
[^deploy-vercel]: Payload CMS. (2024, November 19). Payload 3.0 — The First CMS That Installs Directly Into Any Next.js App. Retrieved 2026-10-03, from https://payloadcms.com/posts/blog/payload-30-the-first-cms-that-installs-directly-into-any-nextjs-app
[^deploy-cloudflare]: Payload CMS. (2025). Deploy Payload Onto Cloudflare in a Single Click. Retrieved 2026-10-03, from https://payloadcms.com/posts/blog/deploy-payload-onto-cloudflare-in-a-single-click
[^blog-30]: Payload CMS. (2024, November 19). Payload 3.0 — The First CMS That Installs Directly Into Any Next.js App. Retrieved 2026-10-03, from https://payloadcms.com/posts/blog/payload-30-the-first-cms-that-installs-directly-into-any-nextjs-app
[^blog-40]: Payload CMS. (2026). Payload 4.0 — Admin UI Redesign, TanStack, MCP, and More. Retrieved 2026-10-03, from https://payloadcms.com/posts/blog/payload-40-admin-ui-redesign-tanstack-mcp-and-more
[^security-update]: Payload CMS. (2026, September). Payload Security Update — Available for 3.x and 4.0. Retrieved 2026-10-03, from https://payloadcms.com/posts/blog/payload-security-update-available-for-3x-and-40
[^wp-migration]: Payload CMS. (n.d.). Headless WordPress Alternative. Retrieved 2026-10-03, from https://payloadcms.com/posts/blog/headless-wordpress-alternative
[^ai-rag]: Payload CMS. (n.d.). AI Framework. Retrieved 2026-10-03, from https://payloadcms.com/enterprise/ai-framework
[^figma-acq]: Payload CMS. (2024). Payload Is Joining Figma. Retrieved 2026-10-03, from https://payloadcms.com/posts/blog/payload-is-joining-figma