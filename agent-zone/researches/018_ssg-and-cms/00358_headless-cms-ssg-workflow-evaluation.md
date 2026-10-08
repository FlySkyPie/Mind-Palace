# Headless CMS 作為 SSG 建構階段內容來源之方案評估

## 研究背景

靜態網站產生器（SSG, Static Site Generator）如 Docusaurus 採用純檔案式內容管理，在文件規模增長時可能面臨建構時間膨脹的問題。一個替代思路是：在編輯階段使用資料庫（DB）輔助內容管理，享受結構化查詢與關聯管理等優勢，但在建構階段將內容編譯為純靜態 HTML，部署時完全不需要資料庫。

本報告針對三個知名 Headless CMS 專案，評估其在此架構下的適用性：

- **Payload CMS** — Next.js 原生 Headless CMS
- **Directus** — 開放式資料庫包裝頭無頭 CMS
- **Strapi** — 開源 Headless CMS

評估核心問題：這三個工具能否支援「編輯階段使用資料庫 → 建構階段產出靜態 HTML → 部署時零資料庫」的工作流程？

---

## 方案一：Payload CMS

### 概述

Payload CMS 是一套 Next.js 原生 Headless CMS，本質上是一個需要即時資料庫連線的應用框架。其管理後台、REST/GraphQL API、Local API、hooks、權限控制——所有功能皆依賴即時資料庫。[^payload-whatis]

### 靜態匯出能力

**Payload 不具備任何靜態匯出功能。** 官方文件完全沒有提及「static export」、「static site」、「SSG」作為 Payload 功能存在。唯一的相關討論是關於 Next.js 自身的 SSG（靜態生成）功能——該頁面說明的是如何關閉 Next.js SSG 以避免建構時需要資料庫連線，而非 Payload 提供任何靜態匯出能力。[^payload-building-without-db]

> "Payload by itself does not have this requirement, But Next.js' SSG does if any of your route segments have SSG enabled... and use the Payload Local API."

### SSG 整合模式

Payload 可被其他 SSG 在建構階段當作 API 來源使用（理論上可透過 REST/GraphQL API 或 Local API 查詢內容再餵給 SSG），但 **完全沒有任何官方文件、外掛或模板** 支援此模式。文件中從未提及 11ty、Hugo、Gatsby 或 Astro。[^payload-concepts]

### 資料庫支援

Payload 支援三種資料庫配接器：[^payload-database]

| 資料庫 | 套件 | ORM |
|--------|------|-----|
| MongoDB | `@payloadcms/db-mongodb` | Mongoose |
| PostgreSQL | `@payloadcms/db-postgres` | Drizzle |
| **SQLite** | **`@payloadcms/db-sqlite`** | **Drizzle / libSQL** |

SQLite 所有功能皆支援（除 Point 欄位外），Cloudflare D1 亦支援。

### 結論：不適合

| 面向 | 結果 |
|------|------|
| 編輯階段使用資料庫 | ✅ 原生支援 |
| 靜態匯出功能 | ❌ 無內建功能 |
| 部署階段零資料庫 | ❌ 非設計目標 |
| 作為 SSG 建構階段內容來源 | ⚠️ 技術上可能，但無官方支援 |
| SQLite 支援 | ✅ 完整支援 |

Payload 不是適合此需求的工具。其設計目標是作為一個長期運行的 Next.js 應用，與資料庫緊密耦合。

---

## 方案二：Directus

### 概述

Directus 是一套開放式 Headless CMS，本質上是現有 SQL 資料庫的即時包裝層，自動提供 REST/GraphQL API 與管理後台。[^directus-docs]

### 靜態匯出能力

**Directus 不具備內建「一鍵匯出為靜態網站」功能。** 但官方文件明確提供了 **針對 SSG 的整合指南**，展示如何在建構階段使用 Directus 作為內容來源：[^directus-eleventy]

> "Eleventy（有時稱為 11ty）是一個輕量且無傾向性的靜態網站生成器。在本指南中，你將學習如何使用 Directus 作為無頭 CMS 來構建網站。"

### SSG 整合模式（官方支援）

Directus 提供以下 SSG 框架的官方指南：

- **Eleventy (11ty)**：專屬的文件指引，使用 `@directus/sdk` 在 11ty 建構階段透過 API 拉取內容（頁面、部落格文章、全域元數據），生成靜態 HTML。[^directus-eleventy-data]
- **Astro**：透過 `getStaticPaths()` 在建構階段取得內容。[^directus-astro-data]
- 文件首頁亦列出 Next.js、Nuxt、SvelteKit 等框架，但不限於靜態模式。

### 資料庫支援

Directus 支援多種 SQL 資料庫，包括 **SQLite**：[^directus-database]

> `DB_CLIENT` 支援 `pg` / `postgres`、`mysql`、`oracledb`、`mssql`、`sqlite3`、`cockroachdb`。

使用 SQLite 時設定 `DB_FILENAME` 指定資料庫檔案路徑。

### 結論：適合

| 面向 | 結果 |
|------|------|
| 編輯階段使用資料庫 | ✅ 原生支援 |
| 靜態匯出功能 | ❌ 無一鍵匯出，但有官方 SSG 指南 |
| 部署階段零資料庫 | ✅ SSG 建構後無需 Directus |
| 作為 SSG 建構階段內容來源 | ✅ 官方支援（11ty、Astro 均有指南） |
| SQLite 支援 | ✅ 完整支援 |

Directus 是三個方案中最符合需求的。官方文件直接支援「在 Directus 管理後台創作內容 → SSG 於建構階段取用 → 部署靜態檔案」的工作流程。

---

## 方案三：Strapi

### 概述

Strapi 是一套開源 Headless CMS，本質上是即時 API 伺服器，管理後台與資料庫緊密綁定。[^strapi-intro]

### 靜態匯出能力

**Strapi 不具備內建靜態匯出功能。** `strapi export` CLI 指令存在，但其輸出是加密的 `.tar.gz.enc` 壓縮檔，專為 Strapi 實例之間傳輸用，非供 SSG 使用。[^strapi-export]

> "The `strapi export` command exports data from a local Strapi instance. By default, the `strapi export` command exports data as an encrypted and compressed `tar.gz.enc` file."

### SSG 整合模式（社群驅動）

Strapi 官方有整合頁面列出 Astro、Next.js、Nuxt、Vue、React 等，但這些整合主要針對即時 API 而非靜態建構。[^strapi-integrations]

- **Strapi + Astro**：社群開發了專屬 loader 套件，在建構階段透過 Astro Content Layer API 取用 Strapi 內容。[^strapi-astro-loader]
- **Strapi + Next.js**：可在建構階段（Server Components）或執行階段取用 API。[^strapi-nextjs]
- **Strapi + Gatsby**：曾存在整合頁面，目前已移除。

此模式在社群中廣泛使用但 **非 Strapi 官方文件涵蓋**。

### 資料庫支援

Strapi 支援多種 SQL 資料庫，**SQLite 是預設選項**：[^strapi-database]

| 資料庫 | 最低版本 | 建議版本 |
|--------|---------|---------|
| MySQL | 8.0 | 8.4 |
| MariaDB | 10.3 | 11.4 |
| PostgreSQL | 14.0 | 17.0 |
| **SQLite** | **3** | **3** |

### 結論：有條件適合

| 面向 | 結果 |
|------|------|
| 編輯階段使用資料庫 | ✅ 原生支援 |
| 靜態匯出功能 | ❌ 無內建，`strapi export` 非為 SSG 設計 |
| 部署階段零資料庫 | ✅ SSG 建構後無需 Strapi |
| 作為 SSG 建構階段內容來源 | ✅ 社群驅動模式，搭配 Astro/Next.js 成熟 |
| SQLite 支援 | ✅ 完整支援（預設資料庫） |

Strapi 可以支援「編輯階段使用資料庫，部署階段零資料庫」的工作流程，但需依賴社群工具自行搭建 CI pipeline，缺乏官方文件指引。

---

## 橫向比較

| 面向 | Payload | Directus | Strapi |
|------|---------|----------|--------|
| 內建靜態匯出 | ❌ | ❌ 但有官方 SSG 指南 | ❌ 僅社群模式 |
| SSG 官方整合文件 | ❌ | ✅ 11ty、Astro | ❌ |
| 作為 SSG 建構階段內容來源 | ⚠️ 理論可行 | ✅ 官方推薦模式 | ✅ 社群成熟模式 |
| SQLite 支援 | ✅ | ✅ | ✅（預設） |
| 部署後無需 DB | ❌ 需即時 DB | ✅ SSG 模式下可 | ✅ SSG 模式下可 |
| 文件規模限制 | ❌ Next.js 架構限制 | ✅ 無 SSG 耦合 | ✅ 無 SSG 耦合 |
| 社群成熟度 | 中（發展中） | 高 | 高 |

---

## 綜合結論

對於「編輯階段使用資料庫，SSG 建構後靜態部署」的工作流程：

1. **Directus** 是三方案中最理想的選擇。其官方文件直接提供 Eleventy 和 Astro 作為 SSG 前端的指南，明確支援「管理後台創作 → SSG 建構階段取用 → 部署靜態檔案」模式，SQLite 支援完整。

2. **Strapi** 是可行的替代方案。社群已在 Astro/Next.js 生態系中建立了成熟的使用模式，且有專屬的 Astro Content Layer loader。但缺乏官方靜態匯出的文件指引，需自行搭建 CI pipeline。

3. **Payload CMS** 不適合此需求。其設計目標是作為長期運行的 Next.js 應用，無法脫離資料庫運行，且完全沒有面向靜態建構的生態系支援。

對於檔案型 SSG 在內容規模增長時建構時間可能變慢的問題，**Directus + 11ty/Astro** 是最值得採用的替代路徑：編輯階段享有結構化資料庫管理，SSG 建構階段僅處理需要的內容，部署時不需任何資料庫依賴。

---

[^payload-whatis]: Payload CMS. (n.d.). What is Payload? Retrieved 2026-10-04, from https://payloadcms.com/docs/getting-started/what-is-payload

[^payload-building-without-db]: Payload CMS. (n.d.). Building without a DB connection. Retrieved 2026-10-04, from https://payloadcms.com/docs/production/building-without-a-db-connection

[^payload-concepts]: Payload CMS. (n.d.). Concepts. Retrieved 2026-10-04, from https://payloadcms.com/docs/getting-started/concepts

[^payload-database]: Payload CMS. (n.d.). Database Overview. Retrieved 2026-10-04, from https://payloadcms.com/docs/database/overview

[^payload-database-sqlite]: Payload CMS. (n.d.). SQLite. Retrieved 2026-10-04, from https://payloadcms.com/docs/database/sqlite

[^directus-docs]: Directus. (n.d.). Directus Documentation. Retrieved 2026-10-04, from https://docs.directus.io/

[^directus-eleventy]: Directus. (n.d.). Eleventy Framework Guide. Retrieved 2026-10-04, from https://docs.directus.io/frameworks/eleventy/

[^directus-eleventy-data]: Directus. (n.d.). Eleventy Data Fetching. Retrieved 2026-10-04, from https://docs.directus.io/frameworks/eleventy/data-fetching

[^directus-astro-data]: Directus. (n.d.). Astro Data Fetching. Retrieved 2026-10-04, from https://docs.directus.io/frameworks/astro/data-fetching

[^directus-database]: Directus. (n.d.). Database Configuration. Retrieved 2026-10-04, from https://docs.directus.io/configuration/database

[^strapi-intro]: Strapi. (n.d.). Strapi Documentation — Introduction. Retrieved 2026-10-04, from https://docs.strapi.io/cms/intro

[^strapi-export]: Strapi. (n.d.). Data Management — Export. Retrieved 2026-10-04, from https://docs.strapi.io/cms/features/data-management/export

[^strapi-integrations]: Strapi. (n.d.). Integrations. Retrieved 2026-10-04, from https://strapi.io/integrations

[^strapi-astro-loader]: Strapi. (n.d.). Astro Integration. Retrieved 2026-10-04, from https://strapi.io/integrations/astro

[^strapi-nextjs]: Strapi. (n.d.). Next.js CMS Integration. Retrieved 2026-10-04, from https://strapi.io/integrations/nextjs-cms

[^strapi-database]: Strapi. (n.d.). Database Configuration. Retrieved 2026-10-04, from https://docs.strapi.io/cms/configurations/database