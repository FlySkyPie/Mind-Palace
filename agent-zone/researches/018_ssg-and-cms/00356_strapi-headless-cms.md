# Strapi：領先的開源 Headless CMS

## 概述

Strapi 是一個以 JavaScript/TypeScript 打造、MIT 授權的開源 headless CMS（無頭內容管理系統），由法國公司 Strapi Solutions SAS 開發[^gh-repo]。它的核心定位是讓開發者快速建立內容 API，同時提供友善的管理後臺讓非技術人員管理內容。名稱源自 "Bootstrapping your API"。

## 解決的問題

Strapi 解決了內容驅動應用開發中的核心矛盾：

- **重複的 CRUD API 開發**：定義內容類型後，自動產生 REST 與 GraphQL API，無需手動撰寫重複的增刪改查程式碼[^rest-api][^graphql-api]。
- **開發者與編輯者的需求衝突**：管理後臺供內容編輯者使用，API 供開發者使用，同一套系統同時滿足兩方需求[^features]。
- **供應商鎖定 (Vendor Lock-in)**：開放原始碼、自架 (self-hosted)，使用者完全掌控數據與程式碼，不依賴特定 SaaS 平臺[^gh-repo]。
- **JAMstack/前後端分離架構需求**：為任何前端框架 (Next.js、React、Astro、Gatsby) 提供即用型內容 API[^cms-intro]。

## 主要功能

### 內容類型建構器 (Content-Type Builder)

可視化拖曳式建構資料模型，支援：

- **集合類型 (Collection Types)**：多筆記錄的內容類型（如文章、產品）
- **單一類型 (Single Types)**：僅一筆記錄的類型（如公司簡介、首頁配置）
- **元件 (Components)**：可重複使用的欄位組合
- **動態區域 (Dynamic Zones)**：可自由排列組合的版面區塊

欄位類型包括：文字、RTF 編輯器 (Blocks & Markdown)、數字、日期、密碼、媒體、關聯（6 種）、布林值、JSON、Email、列舉、UID，以及自訂欄位。Strapi 5 新增了內容類型資料夾歸類、條件式欄位顯示，以及 AI 輔助自然語言建模[^content-type-builder]。

### 自動產生的 API

- **REST API**：完整 CRUD 端點，支援過濾 (`$eq`、`$contains`、`$gt`、`$lt` 等)、排序、分頁、欄位選擇、關聯資料嵌入 (populate)、多語系 (locale)、狀態，以及發佈過濾[^rest-api]。
- **GraphQL API**：需安裝 `@strapi/plugin-graphql` 套件，自動產生查詢與變更 (mutations)，支援 Relay 風格分頁[^graphql-api]。
- **文件服務 API (Document Service API)**：Server 端高階 API (`strapi.documents`)，v5 中取代了 v4 的 Entity Service API[^cms-intro]。
- **查詢引擎 API (Query Engine API)**：最低層級資料庫存取 (`strapi.db.query`)。

### 管理後臺

React 打造的管理儀表板，包含內容管理、媒體庫、角色權限、多語系、排程發佈、操作歷程、稽核日誌等功能。可經由前端擴展進行客製化[^features]。

### 套件系統與 Marketplace

App 內 Marketplace 與 [Strapi Community Hub](https://community.strapi.io/marketplace) 提供社群套件。內建套件：GraphQL、文件、Sentry、使用者與權限、上傳、多語系 (i18n) 等。支援自訂套件開發[^marketplace]。

### 資料庫支援

**SQLite**（開發環境）、**PostgreSQL**（建議生產用）、**MySQL**、**MariaDB**，不支援 MongoDB 或其他 NoSQL 資料庫[^cms-intro]。

### 認證與權限

- **使用者與權限套件**：JWT 認證、角色基礎存取控制（Authenticated、Public 及自訂角色）
- **JWT 管理**：傳統長效 token 或 Refresh 模式（短效 access token + 可更新的 refresh token）
- **SSO**：Google、GitHub、Facebook 等；企業版支援 Okta、Active Directory、SAML 2.0/OAuth2
- **API Token**：獨立於使用者認證的 API 存取機制
- **速率限制**：可設定，預設每分鐘 10 次[^users-permissions]

### 內容管理功能

- **草稿與發佈 (Draft & Publish)**：每種內容類型可獨立設定草稿/發佈狀態，支援批次操作[^draft-publish]。
- **多語系 (i18n)**：超過 500 個預設語系，Growth 方案提供 AI 自動翻譯[^i18n]。
- **媒體庫**：上傳、裁切、最佳化圖片/影片/音訊/文件。
- **排程發佈 (Releases)**：將多個內容變更分組並安排統一發佈時間。
- **操作歷程**：Growth 方案保留 14 天，企業方案可自訂。
- **審查流程 (Review Workflows)**：企業方案支援無限審查流程。
- **稽核日誌**：企業方案提供詳細操作紀錄。
- **Webhooks**：內容事件觸發回呼。
- **條件式欄位**：根據條件顯示或隱藏欄位。

## 架構概覽

**後端**：建構在 Koa.js (Node.js HTTP 框架) 之上。請求流程如下[^backend-customization]：

```
請求 → 全域中介層 → 路由 → 路由原則/中介層
  → 控制器 → 服務 → 模型 → 文件服務 → 查詢引擎 → 資料庫
```

**管理後臺**：React 為基礎，可透過 `src/admin/extensions/` 客製化。

**專案結構**（Strapi 5 預設 TypeScript）[^project-structure]：

```
./
├── config/          # 伺服器、資料庫、管理後臺、套件、排程、中介層設定
├── src/
│   ├── api/         # 內容類型、控制器、服務、路由、原則、中介層
│   ├── components/  # 可重複使用的元件結構定義
│   ├── plugins/     # 本地/自訂套件
│   ├── extensions/  # 套件擴展
│   ├── middlewares/ # 自訂全域中介層
│   └── index.ts     # bootstrap() 與 destroy()
├── types/generated/ # 自動產生的 TypeScript 型別
├── dist/            # 編譯後的後端程式
└── public/uploads/  # 上傳媒體
```

**Strapi 5 主要架構變革**[^v4-to-v5]：
- **文件 (Documents)** 取代 v4 的 entry 概念，使用 `documentId`（字串）取代數值 `id`
- **扁平化回應格式**：不再有 `data.attributes` 巢狀結構
- **文件服務 API** 取代 Entity Service API
- **TypeScript 優先**：TypeScript 為預設專案類型

## 優點與缺點

### 優點

| 優點 | 說明 |
|---|---|
| **開源 + 自架** | 完整資料所有權，無供應商鎖定，MIT 授權 |
| **快速 API 產生** | 定義資料模型後即時取得 REST + GraphQL API |
| **開發者優先** | 100% JavaScript/TypeScript，Koa.js 底層，完全可客製化 |
| **可擴展** | 套件系統、自訂控制器/服務/中介層、Marketplace |
| **豐富功能** | i18n、草稿/發佈、媒體庫、角色權限、Webhooks、排程發佈 |
| **龐大社群** | 73,300+ GitHub 星星，9,900+ forks，活躍 Discord 與 GitHub Discussions |
| **TypeScript 原生** | 自動產生型別，v5 以後一等公民支援 |
| **彈性部署** | 可自架、Strapi Cloud、Docker、或任何 PaaS |
| **企業功能** | SSO、稽核日誌、審查流程、SOC 2、GDPR |
| **成長中的生態系** | AI 應用 MCP server、AI 翻譯/內容建模 |

### 缺點

| 缺點 | 說明 |
|---|---|
| **僅 SQL 資料庫** | 不支援 MongoDB、NoSQL 或 Cloud Native 資料庫 |
| **v4→v5 遷移複雜** | 破壞性變更（documentId、扁平化 API、Entity Service 淘汰），v4 已於 2026 年 4 月終止支援 |
| **大規模效能挑戰** | 若 `populate` 管理不當，可能出現 N+1 查詢問題 |
| **官方託管選項有限** | Strapi Cloud 為唯一官方 PaaS，自架需要 DevOps 知識 |
| **v4/v5 套件不相容** | 套件無法跨版本共用 |
| **生產前需編譯管理後臺** | 部署前需執行 `NODE_ENV=production yarn build` |
| **GraphQL 無法上傳** | 媒體上傳僅限 REST API |
| **非開發者學習曲線陡峭** | 較 WordPress 或 Contentful 對編輯者更技術導向 |
| **無官方 Docker images** | 社群提供 `strapi-tool-dockerize` 工具 |
| **內容類型受環境限制** | 無法在生產環境建立/修改內容類型 |

## 與替代方案的比較

| 功能 | Strapi | WordPress | Contentful | Sanity | Ghost |
|---|---|---|---|---|---|
| **類型** | Headless CMS | 傳統 CMS（+ Headless WP REST API） | SaaS Headless CMS | SaaS Headless CMS | Headless CMS（部落格導向） |
| **開源** | ✅ MIT | ✅ GPLv2 | ❌ 專有 | ✅ MIT（自架：AGPLv3） | ✅ MIT |
| **自架** | ✅ | ✅ | ❌ | ✅（自架版） | ✅ |
| **語言** | JS/TS (Node.js) | PHP | N/A（僅 API） | JS（自訂查詢語言） | JS (Node.js) |
| **API 類型** | REST + GraphQL | REST (WP REST API) | REST + GraphQL | REST + GraphQL (GROQ) | REST 僅 |
| **管理後臺** | React | PHP | Web app | React (Portable Text) | 自訂 |
| **內容建模** | 可視化 + 程式碼 | 自訂文章類型 | JSON 模型 | 程式碼定義結構 | 文章 + 頁面 |
| **資料庫** | SQLite、PG、MySQL、MariaDB | MySQL/MariaDB | 專有 | PostgreSQL | SQLite/MySQL |
| **套件/外掛** | Marketplace（成長中） | 50,000+ 外掛 | 擴展（有限） | 社群整合 | 整合僅 |
| **多語系** | ✅ 內建 | ❌ 需外掛 | ✅ 內建 | ✅ 內建 | ❌ 需外掛 |
| **社群規模** | 73k⭐ GitHub | 極大 | N/A（封閉） | 6k⭐ GitHub | 47k⭐ GitHub |
| **價格（自架）** | 免費（社群版） | 免費 | $300+/月（最低） | 免費（自架版） | 免費 |
| **最適合** | **客製 Web App、JAMstack、SaaS 平臺、多渠道內容** | **部落格、行銷網站、標準網站** | **企業 SaaS、需管理基礎設施的大型團隊** | **客製編輯體驗、結構化內容** | **部落格、電子報、出版** |

## 使用案例

1. **JAMstack 網站**：Strapi + Next.js/React/Astro/Gatsby（官方提供 Next.js LaunchPad 模板）
2. **行動應用後端**：為 iOS/Android 應用自動產生 REST/GraphQL API
3. **多渠道內容**：同一內容同時服務網頁、行動應用、IoT、Kiosk、智慧顯示器
4. **SaaS 平臺**：將內容管理嵌入 SaaS 產品
5. **企業入口網站**：具 RBAC 的多語系內網與外網
6. **電子商務內容**：產品目錄、CMS 驅動的店面
7. **事件驅動內容**：行銷活動的排程發佈

知名使用者：IBM、Walmart、NASA、Airbus、Tesco、Toyota、Societe Generale、Rakuten、Delivery Hero[^about-us]。

## 社群健康度

| 指標 | 數值 |
|---|---|
| **GitHub Stars** | **73,300+** ⭐ |
| **GitHub Forks** | **9,900+** |
| **Watchers** | **653** |
| **Commits** | **37,976** |
| **貢獻者** | **~900+** |
| **NPM 下載量** | 預估每月 1M+ |
| **贊助/投資者** | Index Ventures、Accel、Stride.VC |
| **客戶數** | **3,000+**（根據 strapi.io） |
| **團隊規模** | ~30–40+ 人 |

專案極度活躍，73k+ GitHub 星星使其成為 CMS 領域最受歡迎的開源專案之一，幾乎每週都有新的版本釋出[^gh-repo]。

## 近期主要版本

### Strapi 5（當前主要版本）

- **初始釋出**：2024 年底
- **最新版本**：**v5.56.0**（2026 年 9 月 30 日）
- **v4 支援終止**：**2026 年 4 月**（v4 已停止維護）

### Strapi 5 主要變革

- `documentId` 取代數值 `id` 作為主要識別碼
- 扁平化 API 回應格式（不再有 `data.attributes`）
- TypeScript 為預設專案類型
- 文件服務 API 取代 Entity Service API
- 管理後臺 UI 翻新，Tailwind 編譯
- 內容類型資料夾歸類
- 條件式欄位顯示
- Strapi AI 整合（內容建模、自動翻譯）
- Refresh JWT 管理模式（短效 token + refresh 輪換）
- GraphQL v4 相容模式以利逐步遷移[^v4-to-v5]

### 近期版本演進

- v5.56.0（2026-09-30）：Tailwind 管理編譯、管理帳戶/Webhooks 稽核日誌
- v5.55.1（2026-09-24）：文件套件效能回歸的 Hotfix
- v5.55.0（2026-09）：功能增強
- v5.52.0–v5.54.0：持續改善、錯誤修復、依賴更新[^releases]

## 總結

Strapi 是目前開源 headless CMS 領域的主導者，以「開發者優先」為核心理念，在 API 自動化產生的便利性與功能豐富度之間取得了良好平衡。v5 的 TypeScript 原生支援、扁平化 API 設計、AI 整合等方向反映了現代 Web 開發的趨勢。然而 v4 到 v5 的遷移複雜性與生產環境的效能調校仍是潛在挑戰。對於需要快速建立內容 API、前後端分離架構、並要求資料自主權的專案，Strapi 是非常值得考慮的選擇。

---

[^gh-repo]: Strapi Solutions SAS. (n.d.). Strapi — Open-source headless CMS. Retrieved 2026-10-03, from https://github.com/strapi/strapi
[^cms-intro]: Strapi Solutions SAS. (n.d.). CMS Documentation Introduction. Retrieved 2026-10-03, from https://docs.strapi.io/cms/intro
[^features]: Strapi Solutions SAS. (n.d.). Features — Strapi. Retrieved 2026-10-03, from https://strapi.io/features
[^content-type-builder]: Strapi Solutions SAS. (n.d.). Content-Type Builder. Retrieved 2026-10-03, from https://docs.strapi.io/cms/features/content-type-builder
[^rest-api]: Strapi Solutions SAS. (n.d.). REST API. Retrieved 2026-10-03, from https://docs.strapi.io/cms/api/rest
[^graphql-api]: Strapi Solutions SAS. (n.d.). GraphQL API. Retrieved 2026-10-03, from https://docs.strapi.io/cms/api/graphql
[^users-permissions]: Strapi Solutions SAS. (n.d.). Users & Permissions. Retrieved 2026-10-03, from https://docs.strapi.io/cms/features/users-permissions
[^draft-publish]: Strapi Solutions SAS. (n.d.). Draft and Publish. Retrieved 2026-10-03, from https://docs.strapi.io/cms/features/draft-and-publish
[^i18n]: Strapi Solutions SAS. (n.d.). Internationalization. Retrieved 2026-10-03, from https://docs.strapi.io/cms/features/internationalization
[^marketplace]: Strapi Solutions SAS. (n.d.). Installing Plugins via Marketplace. Retrieved 2026-10-03, from https://docs.strapi.io/cms/plugins/installing-plugins-via-marketplace
[^backend-customization]: Strapi Solutions SAS. (n.d.). Backend Customization. Retrieved 2026-10-03, from https://docs.strapi.io/cms/backend-customization
[^project-structure]: Strapi Solutions SAS. (n.d.). Project Structure. Retrieved 2026-10-03, from https://docs.strapi.io/cms/project-structure
[^v4-to-v5]: Strapi Solutions SAS. (n.d.). v4 to v5 Migration Introduction and FAQ. Retrieved 2026-10-03, from https://docs.strapi.io/cms/migration/v4-to-v5/introduction-and-faq
[^releases]: Strapi Solutions SAS. (n.d.). Strapi Releases. Retrieved 2026-10-03, from https://github.com/strapi/strapi/releases
[^about-us]: Strapi Solutions SAS. (n.d.). About Us — Strapi. Retrieved 2026-10-03, from https://strapi.io/about-us