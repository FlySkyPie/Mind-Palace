# Directus — 協作式後端平台介紹

## 概述

Directus 是一個**資料庫優先（database-first）、原始碼可用（source-available）** 的協作式後端平台，能將任何 SQL 資料庫即時包裝為 REST API、GraphQL API、視覺化管理介面（The Studio），以及供 AI 代理使用的原生 MCP（Model Context Protocol）伺服器。其核心理念為：*"Turn your DB into a headless CMS, admin panels, or apps with a custom UI, instant APIs, auth & more."*[^directus-gh]

它在開發者、內容編輯者、營運人員與 AI 代理之間提供**共享的資料層**，讓所有人基於同一套權限系統同時工作，無需來回排隊等待。

---

## 解決的問題

傳統開發流程中，團隊需要自行建置與維護 API 層、管理後台、認證系統與權限邏輯——這些重複性工作耗費大量時間。Directus 提供了開箱即用的方案，讓團隊專注於應用邏輯而非後端樣板。

---

## 核心功能

| 功能 | 說明 |
|---|---|
| **即時 REST & GraphQL API** | 自動根據現有資料庫 schema 產生，零設定 |
| **The Studio（Vue 3）** | 非技術使用者可直接操作的視覺化管理介面 |
| **AI Assistant** | Studio 內建的對話式 AI，可建立、翻譯、操作內容 |
| **原生 MCP 伺服器** | 讓 Claude Desktop、Cursor、ChatGPT 等 MCP 相容 AI 直接連接資料 |
| **基於政策的存取控制** | 精細到欄位層級的權限，同時適用於人類與 AI |
| **WebSocket / GraphQL 訂閱** | 即時資料變更推送，原生支援 | [^directus-docs]
| **Flows（視覺化自動化）** | 拖拉式自動化流程：觸發、條件、運算、自訂 JS 步驟 |
| **Insights（原生儀表板）** | 直接在即時資料上建立圖表、指標與報表 |
| **數位資產管理** | 上傳、組織、即時轉換檔案，支援 CDN 與簽章 URL |
| **Many-to-Any（M2A）關係** | 多型關聯，單一欄位可指向任何集合的項目 |
| **內建多語系** | 任何內容類型皆可設定翻譯欄位 |
| **擴充系統** | 自訂端點、鉤子（hooks）、介面、模組 |

---

## 架構哲學

Directus 是 **database-first**：它直接讀取現有 SQL schema，資料表成為 collection，欄位成為 field。**資料庫本身就是 schema**——在 Directus 外部變更資料庫結構，下一次重整就會在 Studio 中反映。這與 code-first 的 CMS（如 Strapi）有根本性差異[^directus-vs-others]。

三大核心概念：

1. **Data Model** — collection 與 field 直接對應資料庫 table 與 column
2. **Permissions** — 精細到 collection、field、item 層級的存取控制
3. **Flows** — 在事件、排程或 webhook 上觸發的自動化邏輯

---

## 支援的資料庫

- PostgreSQL
- MySQL
- MariaDB
- Microsoft SQL Server（MSSQL）
- SQLite
- OracleDB
- CockroachDB

這是 headless CMS 中最廣泛的資料庫支援之一，尤其涵蓋了企業常見的 MSSQL 與 Oracle[^directus-docs]。

---

## 適用對象與使用案例

| 角色 | 使用方式 |
|---|---|
| **開發者** | 即產生的 REST/GraphQL API、SDK、擴充、自架設、schema 控制 |
| **內容團隊** | Studio 的無程式碼介面——編輯、翻譯、發布內容 |
| **營運/分析人員** | Flows（視覺化自動化）、Insights（儀表板）、資產管理 |
| **AI 代理** | 透過 MCP 連接——Claude Desktop、Cursor、ChatGPT，受相同權限約束 |

### 具體使用案例

- **Headless CMS** — 搭配 Next.js、Nuxt、Astro 等前端框架的內容網站
- **管理後台** — 內部工具的 back-office
- **主資料管理** — Ryanair 用於 90+ 個主資料集[^directus-about]
- **行動 app 後端** — Weber 智慧燒烤 app（600 萬+ 會話）
- **AI 內容操作** — 利用 AI Assistant 進行翻譯、標記、摘要
- **即時資料 / IoT** — 原生 WebSocket 訂閱
- **取代試算表** — 以結構化資料 + 治理化介面取代 Excel/Google Sheets

### 知名使用者

Tripadvisor、Ryanair、Weber Grills、Club Med、Prusa、Copa Airlines、Rescue.org、Ripley's[^directus-about]

---

## 技術棧

| 層級 | 技術 |
|---|---|
| **後端** | Node.js (^22)、TypeScript |
| **前端（Studio）** | Vue 3 |
| **套件管理** | pnpm (^10) — 單一儲存庫（monorepo） |
| **API 輸出** | REST、GraphQL、WebSocket、GraphQL 訂閱 |
| **AI 層** | 原生 AI Assistant、原生 MCP 伺服器 |
| **執行環境** | Node.js 22+、Docker |

### 主要套件

`@directus/api`、`@directus/app`、`@directus/sdk`、`@directus/cli`、`@directus/extensions-sdk`、多種儲存驅動（S3、GCS、Azure、Cloudinary、Supabase、本地）[^directus-gh-package]

---

## 社群與生態系統

| 指標 | 數值 |
|---|---|
| **GitHub Stars** | 38,000+ |
| **Forks** | 5,000+ |
| **總下載數** | 4,500 萬+ |
| **部署專案數** | 50 萬+ |
| **G2 評分** | 4.9 / 5 |
| **認證** | SOC 2 Type II、GDPR 合規 |

### 社群管道

- **Discord**（最活躍的即時聊天）
- **社群論壇**（community.directus.io）
- **GitHub Issues**（錯誤回報與功能請求）
- **擴充市集**（社群與官方擴充）
- **公開藍圖**（roadmap.directus.com）
- **夥伴目錄**（整合合作夥伴）

### 公司背景

- 創辦人：**Benjamin Haynes**（CEO）與 **Rijk van Zanten**（CTO）
- 公司名稱：**Monospace Inc.**
- 100% 遠端、非同步優先的國際團隊
- 投資者：**Handshake Ventures**、**True Ventures**、**F-Prime**、**Eight Roads**、**Preston-Warner Ventures**
- 董事顧問包含 **Tom Preston-Werner**（GitHub 共同創辦人）[^directus-about]

---

## 近期版本發展

### 目前版本：v12.4.1（2026 年 9 月下旬）

| 版本 | 日期 | 重點 |
|---|---|---|
| **v12.4.1** | 2026-09-23 | 錯誤修正（folder endpoint 權限）[^directus-release-1241] |
| **v12.4.0** | 2026-09-22 | Flows 移入獨立模組，支援資料夾/搜尋/過濾/匯入匯出；Mailtrap 郵件傳輸支援；MapLibre GL 升級（1.15→6.9）；權限強化[^directus-release-1240] |
| **v12.3.1** | 2026-08-25 | WebSocket 心跳修正；GraphQL fragment null 欄位修正；SDK 取消訂閱修正[^directus-release-1231] |
| **v12.3.0** | 2026-08-18 | 儲存連線洩漏修正；Flow 操作安全改善（空 key/query 不再影響所有項目） |

### 發展方向

- **AI 與 MCP** — AI Assistant 與原生 MCP 伺服器為近期重大新增功能
- **Flows** — 視覺化自動化持續獲得重大投資
- **授權轉換** — 從 Business Source License（BSL）遷移至自訂的 **Monospace Sustainable Core License（MSCL）1.0**
- **Rust 探索** — 職缺顯示正在招募 Rust 後端工程師，暗示可能引入 Rust 技術棧

---

## 與競爭方案的比較

### Directus vs. Strapi

| 維度 | Directus | Strapi |
|---|---|---|
| **Schema 哲學** | **Database-first**——讀取現有 SQL schema | Code-first——定義 content type 後 Strapi 建立/遷移 DB |
| **授權** | 原始碼可用（MSCL） | **MIT**（OSI 核准開放原始碼） |
| **支援資料庫** | Postgres、MySQL、MariaDB、**MSSQL、Oracle、CockroachDB** | Postgres、MySQL、MariaDB、SQLite |
| **API 輸出** | REST、GraphQL、**WebSocket、GraphQL 訂閱** | REST、GraphQL |
| **管理 UI** | The Studio（Vue 3） | Admin Panel（React） |
| **原生 AI 助手** | **有** | 無（第三方套件） |
| **原生 MCP** | **有** | 無 |
| **視覺化自動化** | **Flows**（拖拉式） | 無（僅程式碼 lifecycle hooks） |
| **原生儀表板** | **Insights** | 無 |
| **欄位層級權限** | 有 | 有（Enterprise 版） |
| **最佳使用場景** | 既有資料庫、需要儀表板/自動化/AI/即時功能、MSSQL/Oracle 使用者 | 需要 OSI 開放原始碼、全新專案、較大社群生態 |

### Directus vs. Contentful

| 維度 | Directus | Contentful |
|---|---|---|
| **託管模式** | **自架設**（Docker/Node）或 Directus Cloud | SaaS-only（專有雲端） |
| **資料歸屬** | **你的 SQL 資料庫**（你擁有資料） | Contentful 雲端空間（供應商鎖定） |
| **授權** | 原始碼可用（MSCL） | 專有、封閉原始碼 |
| **原生即時功能** | **WebSocket、GraphQL 訂閱** | 無（僅 webhook） |
| **原生 AI / MCP** | **有** | AI Actions（有限）、無 MCP |
| **資産 CDN** | BYO（S3+CDN、Cloudinary） | **內建全球 CDN（Fastly）** |
| **費用模式** | 可預測——自架設免費或按環境計費 | 按使用者/API 呼叫配額（可能意外增加） |
| **最佳使用場景** | 自架設、資料主權、即時功能、多服務共用資料 | 零運維 SaaS、大型編輯團隊搭配 Compose/Lunch 流程 |

### 其他方案的關鍵差異

| 方案 | 關鍵差異 |
|---|---|
| **Sanity** | 專有雲端、即時協作編輯、自訂查詢語言（GROQ） |
| **Payload** | MIT 授權、code-first、Node.js/TypeScript、自架設 |
| **Airtable** | 試算表-資料庫混合、SaaS-only、較少開發者導向 |
| **WordPress** | 單體 CMS（非 headless-first）、PHP、龐大套件生態 |
| **Supabase** | 資料庫即服務（Firebase 替代方案），從後端角度重疊 |

---

## 總結

Directus 是一個**資料庫優先、原始碼可用**的協作式後端平台，能將任何 SQL 資料庫即時包裝為 REST/GraphQL API、Vue 3 視覺化 Studio，以及原生 MCP 伺服器。其關鍵差異點為：

1. **Database-first 架構** — 直接讀取現有 schema，而非擁有 schema
2. **最廣泛的資料庫支援** — 包含 MSSQL、Oracle、CockroachDB
3. **原生 AI 與 MCP** — 深度整合，非事後附加
4. **即時 API** — 內建 WebSocket 與 GraphQL 訂閱
5. **視覺化自動化與儀表板** — Flows 與 Insights 內建於核心
6. **可自架設** — 可在自建基礎設施或 Directus Cloud 上運作

最適合希望**將資料保留在自己 SQL 資料庫中**、需要開發者/內容編輯者/營運人員/AI 代理共用同一個後端，且偏好 database-first 思維模式的團隊。主要取捨在於它不是 OSI 核准的開放原始碼（仍有 MIT 替代方案如 Strapi 與 Payload），且自架設有一定的學習曲線。

---

## 參考文獻

[^directus-gh]: Directus. (n.d.). Directus — The collaborative backend for builders & AI. GitHub repository. Retrieved 2026-10-01, from https://github.com/directus/directus

[^directus-docs]: Directus. (n.d.). Directus Documentation. Retrieved 2026-10-01, from https://directus.com/docs

[^directus-about]: Directus. (n.d.). About Directus. Retrieved 2026-10-01, from https://directus.com/about

[^directus-vs-others]: Directus. (n.d.). Directus vs. Strapi / Directus vs. Contentful. Retrieved 2026-10-01, from https://directus.com/strapi and https://directus.com/contentful

[^directus-gh-package]: Directus. (n.d.). Directus package.json. GitHub. Retrieved 2026-10-01, from https://github.com/directus/directus/blob/main/package.json

[^directus-release-1241]: Directus. (2026-09-23). Release v12.4.1. GitHub Releases. Retrieved 2026-10-01, from https://github.com/directus/directus/releases

[^directus-release-1240]: Directus. (2026-09-22). Release v12.4.0. GitHub Releases. Retrieved 2026-10-01, from https://github.com/directus/directus/releases

[^directus-release-1231]: Directus. (2026-08-25). Release v12.3.1. GitHub Releases. Retrieved 2026-10-01, from https://github.com/directus/directus/releases