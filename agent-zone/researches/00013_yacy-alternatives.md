# YaCy 替代搜尋引擎調查報告

本報告整理所有具備自有爬蟲 (Crawler) 與自有索引 (Index) 的自由開源 (FOSS) 搜尋引擎專案，排除純粹的中繼搜尋 (Metasearch) 與僅提供 API 但不維護自有索引的專案。

## 活躍維護中

### 1. Hister

個人導向的隱私搜尋引擎，透過瀏覽器擴充功能自動索引你造訪過的網頁。擁有自己的爬蟲與全文檢索引擎，並支援語意搜尋及 MCP 伺服器。

- **GitHub**: <https://github.com/asciimoo/hister>
- **Stars**: ~3,700 ⭐
- **授權**: AGPL-3.0
- **語言**: Go
- **狀態**: ✅ 非常活躍，持續更新[^hister]

### 2. Mwmbl

非營利、社群驅動的開源搜尋引擎，擁有自己的爬蟲與集中化索引。採用獨特的 hash-map 索引設計取代傳統倒排索引。

- **GitHub**: <https://github.com/mwmbl/mwmbl>
- **Stars**: ~1,900 ⭐
- **授權**: AGPL-3.0
- **語言**: Python
- **狀態**: ✅ 活躍，有活躍社群[^mwmbl]

### 3. Alexandria

一個 C++ 寫成的搜尋引擎，可索引檔案與網站。2024 年中重新復活發布 v2.0.1。

- **GitHub**: <https://github.com/alexandria-org/alexandria>
- **Stars**: ~200 ⭐
- **授權**: GPL-2.0
- **語言**: C++
- **狀態**: ✅ 2026 年 6 月剛發布 v2.0.1，已復活[^alexandria]

### 4. SOSSE (Selenium Open Source Search Engine)

以 Selenium + Chromium/Firefox 渲染 JavaScript 重頁面的搜尋引擎與網頁歸檔工具，支援排程爬蟲、全頁快照與全文搜尋。

- **GitHub**: <https://github.com/biolds/sosse> (GitLab: <https://gitlab.com/biolds1/sosse>)
- **Stars**: ~411 ⭐
- **授權**: AGPL-3.0
- **語言**: Python
- **狀態**: ✅ 活躍[^sosse]

### 5. Wiby

專注於較小、較單純的「非商業」網站的手工篩選搜尋引擎。

- **GitHub**: <https://github.com/wibyweb/wiby/>
- **Stars**: ~429 ⭐
- **授權**: —（原始碼可用）
- **語言**: —
- **狀態**: ✅ 活躍[^wiby]

### 6. Sphider & Sphider-plus

輕量級 PHP 爬蟲與搜尋引擎，適合為單一網站提供站內搜尋。Sphider-plus 為社群加強版，新增超過 400 項功能。

- **官方網站**: <https://www.sphider.eu/>、<https://sphider-plus.eu/>
- **SourceForge**: <https://sourceforge.net/projects/sphider-plus/>
- **授權**: GPL
- **語言**: PHP + MySQL
- **狀態**: ✅ 經典專案仍可使用[^sphider]

## 值得關注（較新或較小）

### 7. Marginalia Search

專注於文字內容豐富、非商業的「小型網路」，用自己的爬蟲刻意排除大部分商業內容。可自架 barebones 模式。

- **GitHub**: <https://github.com/MarginaliaSearch/MarginaliaSearch>
- **Stars**: ~2,000 ⭐
- **授權**: AGPL-3.0
- **語言**: Java
- **狀態**: ⚠️ 活躍但硬體需求較高（建議 32GB RAM）[^marginalia]

### 8. Gigablast

歷史悠久的獨立搜尋引擎，完整 C/C++ 分散式爬蟲與索引系統。原始搜尋引擎已於 2023 年下線，但原始碼仍可取得。

- **GitHub**: <https://github.com/gigablast/open-source-search-engine>
- **Stars**: ~1,600 ⭐
- **授權**: Apache-2.0
- **語言**: C/C++
- **狀態**: ❌ 不再維護（2023 年停止）[^gigablast]

### 9. Stract

用 Rust 撰寫的獨立搜尋引擎，擁有自訂爬蟲與索引，支援可自訂排名「Optics」。託管服務已關閉。

- **GitHub**: <https://github.com/StractOrg/stract>
- **Stars**: ~2,400 ⭐
- **授權**: AGPL-3.0
- **語言**: Rust
- **狀態**: ❌ 已封存（2026 年 4 月唯讀）[^stract]

### 10. Librengine

以 C++ 寫成的隱私搜尋引擎，自有 BFS 爬蟲（支援 robots.txt、Proxy、追蹤器偵測），後端使用 Typesense。

- **GitHub**: <https://github.com/liameno/librengine>
- **Stars**: ~75 ⭐
- **授權**: AGPL-3.0
- **語言**: C++
- **狀態**: ⚠️ 開發者宣稱不再維護，建議轉向 Sightnet 但 Sightnet 倉庫已消失[^librengine]

### 11. Froxy

模組化網頁索引引擎，Go 爬蟲搭配 Qdrant 向量資料庫與 FastEmbed 嵌入，支援語意搜尋。

- **GitHub**: <https://github.com/MultiX0/froxy>
- **Stars**: ~22 ⭐
- **授權**: MIT
- **語言**: Go, TypeScript
- **狀態**: ⚠️ 早期但具功能性的專案[^froxy]

### 12. NeoSearch

AI 驅動搜尋引擎，使用 ASP.NET Core 9.0 與 SQL Server。注意依賴 Google Custom Search API。

- **GitHub**: <https://github.com/bartjellema/NeoSearch>
- **Stars**: ~32 ⭐
- **授權**: —
- **語言**: C#
- **狀態**: ⚠️ 非常早期，幾乎只有框架代碼。需注意該專案使用 Google Custom Search API 爬取資料，非自有爬蟲[^neosearch]

### 13. DuskRail

從零打造的 PHP 搜尋引擎，使用真實 Chrome Headless 進行爬取（非 HTTP 客戶端），後端使用 Manticore Search。

- **GitHub**: <https://github.com/FedgeNo/DuskRail>
- **Stars**: ~0 ⭐
- **授權**: MIT
- **語言**: PHP
- **狀態**: ⚠️ 早期但確實有實質代碼（119 commits），非空殼專案[^duskrail]

## 不建議使用

### 14. Inquire

TypeScript 全端搜尋引擎，Redis 佇列 + Playwright 混合爬蟲 + Elasticsearch。僅 14 commits 的個人實驗。

- **GitHub**: <https://github.com/ahnaf-zamil/inquire>
- **Stars**: ~1 ⭐
- **狀態**: 極早期，不建議用於生產[^inquire]

### 15. Deepsearch

號稱「深網」發現與搜尋引擎，目標開放目錄、FTP、憑證透明日誌。但僅 3 commits，幾乎是空殼。

- **GitHub**: <https://github.com/Indiblog/deepsearch>
- **Stars**: ~0 ⭐
- **狀態**: 無實質代碼[^deepsearch]

### 16. Sodeom

Python Flask 寫成的隱私中繼搜尋引擎（聚合 DuckDuckGo、Bing、Brave 結果），非自有索引。

- **GitHub**: <https://github.com/sodeom/sodeom>
- **Stars**: ~2 ⭐
- **注意**: ❌ 屬中繼搜尋（Metasearch），不符合需求[^sodeom]

## 比較表

| 專案 | 自有爬蟲 | 自有索引 | ⭐ | 語言 | 狀態 |
|------|---------|---------|---|------|------|
| Hister | ✅ | ✅ | 3.7k | Go | 活躍 |
| Mwmbl | ✅（分散式） | ✅（集中） | 1.9k | Python | 活躍 |
| SOSSE | ✅ | ✅ | 411 | Python | 活躍 |
| Wiby | ✅ | ✅ | 429 | — | 活躍 |
| Marginalia | ✅ | ✅ | 2.0k | Java | 活躍 |
| Alexandria | ✅ | ✅ | 200 | C++ | 復活 |
| Sphider-plus | ✅ | ✅ (MySQL) | — | PHP | 穩定 |
| Gigablast | ✅ | ✅ | 1.6k | C/C++ | 停止維護 |
| Stract | ✅ | ✅ | 2.4k | Rust | 已封存 |
| Librengine | ✅ | ✅ (Typesense) | 75 | C++ | 不再維護 |
| Froxy | ✅ | ✅ (Qdrant) | 22 | Go/TS | 早期 |
| DuskRail | ✅ (Chrome) | ✅ (Manticore) | 0 | PHP | 早期 |

## 排除項目

以下專案不符合需求（無自有爬蟲與索引）：

- **SearXNG** — 中繼搜尋引擎
- **Websurfx** — 中繼搜尋引擎
- **Sodeom** — 中繼搜尋引擎
- **Whoogle** — Google 代理前端
- **Elasticsearch / Meilisearch / Typesense / Manticore / OpenSearch / Apache Solr** — 通用搜尋函式庫，非網頁搜尋引擎，沒有自己的網路爬蟲
- **Aleph** — 文件索引工具非網頁爬蟲
- **Fess** — 企業搜尋伺服器，無爬蟲
- **Terrier** — IR 研究平台，無爬蟲
- **Namazu** — 全文檢索引擎，非自主爬蟲
- **Brave Search** — 非自架且非完全開源
- **Mojeek** — 非開源
- **bitmagnet** — BitTorrent DHT 爬蟲，非網頁搜尋
- **Amgix / Meme Search** — 非通用網頁搜尋引擎

[^hister]: asciimoo. (n.d.). *Hister — Private web search engine*. Retrieved 2026-09-12, from <https://github.com/asciimoo/hister>
[^mwmbl]: Mwmbl. (n.d.). *Mwmbl — Non-profit open source search engine*. Retrieved 2026-09-12, from <https://github.com/mwmbl/mwmbl>
[^alexandria]: alexandria-org. (2026). *Alexandria — Open source search engine*. Retrieved 2026-09-12, from <https://github.com/alexandria-org/alexandria>
[^sosse]: biolds. (n.d.). *SOSSE — Selenium Open Source Search Engine*. Retrieved 2026-09-12, from <https://github.com/biolds/sosse>
[^wiby]: wibyweb. (n.d.). *Wiby — Search engine for the non-commercial web*. Retrieved 2026-09-12, from <https://github.com/wibyweb/wiby/>
[^sphider]: Saabas, A. (2005). *Sphider — A PHP search engine*. Retrieved 2026-09-12, from <https://www.sphider.eu/>
[^marginalia]: Marginalia Search. (n.d.). *Marginalia Search — Open source search engine*. Retrieved 2026-09-12, from <https://github.com/MarginaliaSearch/MarginaliaSearch>
[^gigablast]: Gigablast. (2017). *Open source search engine — Gigablast*. Retrieved 2026-09-12, from <https://github.com/gigablast/open-source-search-engine>
[^stract]: StractOrg. (2026). *Stract — Web search done right*. Retrieved 2026-09-12, from <https://github.com/StractOrg/stract>
[^librengine]: liameno. (n.d.). *Librengine — Privacy web search engine (not meta, own crawler)*. Retrieved 2026-09-12, from <https://github.com/liameno/librengine>
[^froxy]: MultiX0. (n.d.). *Froxy — Modular web indexing engine*. Retrieved 2026-09-12, from <https://github.com/MultiX0/froxy>
[^neosearch]: bartjellema. (n.d.). *NeoSearch — AI-powered search engine*. Retrieved 2026-09-12, from <https://github.com/bartjellema/NeoSearch>
[^duskrail]: FedgeNo. (n.d.). *DuskRail — Search engine built from the ground up in PHP*. Retrieved 2026-09-12, from <https://github.com/FedgeNo/DuskRail>
[^inquire]: ahnaf-zamil. (n.d.). *Inquire — Self-hosted search engine*. Retrieved 2026-09-12, from <https://github.com/ahnaf-zamil/inquire>
[^deepsearch]: Indiblog. (n.d.). *DeepSearch — Self-hosted deep web discovery and search engine*. Retrieved 2026-09-12, from <https://github.com/Indiblog/deepsearch>
[^sodeom]: sodeom. (n.d.). *Sodeom — Privacy-first meta-search engine*. Retrieved 2026-09-12, from <https://github.com/sodeom/sodeom>