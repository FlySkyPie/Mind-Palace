# FOSS RSS 搜尋引擎調查

## 概述

本報告調查 FOSS 生態中能**主動透過 RSS/Atom feed 爬取網路內容並建立全文搜尋索引**的專案。不同於 RSS 訂閱器（reader，僅為使用者呈現 feed 條目）或 feed 發現工具（僅找出 feed URL），此類專案的核心是**搜尋引擎**——具備爬取、索引、查詢三階段的完整 pipeline。

---

## 一、專用 RSS 搜尋引擎（爬取 + 索引 + 搜尋）

### DATO.RSS（Ruby/Rails）— **最成熟、最多星星**

- **GitHub**：https://github.com/davidesantangelo/dato.rss — 618 ★，MIT License
- **作者**：David Santangelo（亦為 SearQ 作者）

以 RSS feed 為核心的完整搜尋引擎，含 RESTful API。

**架構**：
- **爬取**：背景 Sidekiq worker 持續輪詢 feed 清單，以 Feedjira 解析新條目
- **索引**：PostgreSQL 全文檢索（tsvector）
- **儲存**：PostgreSQL（條目） + Redis（快取/佇列）
- **API**：RESTful JSON API，支援搜尋與 feed 管理

**獨特功能**：
- 出廠附帶 300 萬筆預索引條目的 SQL dump
- OpenRank 排序（基於 PageRank 資料集）
- Dandelion API 整合（實體辨識、語意分析）
- 情感分析

**啟用方式**：

```bash
git clone https://github.com/davidesantangelo/dato.rss
cd dato.rss
# 需 PostgreSQL、Redis；設定 .env 中的 feed 來源
rails db:seed  # 載入 300 萬條目
rails server
```

```bash
curl "http://localhost:3000/api/v1/search?q=artificial+intelligence"
```

---

### SearQ（Ruby/Rails）

- **GitHub**：https://github.com/davidesantangelo/searq.org — 95 ★，MIT License
- **作者**：與 DATO.RSS 同作者，定位更輕量

RSS feed 驅動的搜尋 API，以 **Meilisearch** 取代 PostgreSQL FTS。

**架構**：
- 爬取：Sidekiq + Feedjira（同 DATO.RSS）
- 索引：Meilisearch（typo-tolerant、prefix search、同義詞、自訂排序）
- 儲存：PostgreSQL + Redis

**與 DATO.RSS 差異**：
| 維度 | DATO.RSS | SearQ |
|------|----------|-------|
| 搜尋引擎 | PostgreSQL FTS | Meilisearch |
| 預載資料 | 300 萬條目 | 無 |
| 定位 | 完整產品 | API 服務 |
| 複雜度 | 較高 | 較低 |

---

### hackwage（Python/Node.js）

- **GitHub**：https://github.com/santinic/hackwage — 59 ★，GPL-3.0
- **Live**：https://hackwage.com

原為 IT 職缺 RSS 搜尋引擎，但**架構通用**，可套用至任意 RSS/JSON feed 來源。

**架構**：
- **爬取**：Node.js 腳本定時抓取 `sources.json` 中定義的 RSS/JSON 來源
- **索引**：Elasticsearch（全文 + 相關性排序）
- **前端**：Python Django Web UI
- **快取**：Memcached

**適用場景**：若你已有已知的 feed 清單並需要全文搜尋，hackwage 是最快上手的方案。修改 `sources.json` 即可更換 feed 來源。

---

### Open Semantic Search（Python + Java/Solr）

- **GitHub**：https://github.com/opensemanticsearch/open-semantic-search — ~1200 ★，GPL-3.0
- **網站**：https://opensemanticsearch.org

完整開源語意搜尋引擎，內建 **RSS Connector**，可「從 RSS/Atom feed 索引網頁」。

**架構**：
- **爬取**：內建網頁爬蟲（遵循 robots.txt），**RSS Connector 訂閱 feed 並提取其中的 URL 進行爬取**
- **索引**：Apache Solr（全文 + 分面 + 錯字容錯）
- **ETL**：OCR、命名實體辨識、知識圖譜
- **UI**：內建搜尋網頁

**重要性**：這是本調查中唯一具備「從 feed 中提取 URL → 爬取實際內容 → 建立全文索引」完整鏈的專案。RSS Connector 使其從一般搜尋引擎變身為 RSS 驅動搜尋引擎。

```bash
docker run -d -p 8983:8983 opensemanticsearch/open-semantic-search
# 透過 Web UI 新增 RSS feed 來源
# Connector 會定期爬取 feed 中的文章
```

---

### BlogNerd.app（Go）

- **GitHub**：https://github.com/alastairrushworth/blognerd.app — 3 ★
- **Live**：https://blognerd.app

基於**向量嵌入**的 blog/RSS 語意搜尋引擎。

**架構**：
- RSS feed 發現 → Pinecone 向量資料庫（Voyage AI embeddings）→ 語意搜尋
- OPML/CSV 匯出
- 內容類型過濾（部落格、學術、新聞）
- Docker 支援

**限制**：依賴 Pinecone（專有服務）和 Voyage AI，非完全自給自足。

---

### EchoNews（TypeScript + Python）

- **GitHub**：https://github.com/RJohnPaul/EchoNews — 2 ★
- **Live**：https://newsaihyd.vercel.app

AI 驅動的新聞搜尋平台，聚合 50+ RSS feed。

**架構**：
- FastAPI（Python）後端
- MiniLM-L12-v2 嵌入 + LLaMA 摘要
- Next.js 前端

---

### OfflineWebSearch（Kotlin/Android）

- **GitHub**：https://github.com/rumca-js/OfflineWebSearch — 37 ★，GPL-3.0
- **F-Droid**：可安裝

**隱私優先**的離線 RSS 書籤與搜尋引擎，SQLite 全文檢索。

**特色**：
- 完全不發送網路請求
- 預先建置的 SQLite 資料庫（含 Feeds、Top、YouTube 等）
- RSS 來源探索、標籤、投票、正規表示式過濾
- OPML 匯入

---

### page-finder-service（Kotlin/Spring Boot）

- **GitHub**：https://github.com/loxal/page-finder-service — 1 ★

多租戶網站搜尋即服務，支援 RSS/Atom feed 匯入至 Elasticsearch。

**架構元素**：
- crawler4j（爬蟲，遵守 robots.txt）
- Rome（feed 解析）
- Apache Tika（PDF 文字提取）
- Swagger UI

---

## 二、Feed 發現工具（非搜尋引擎，但可作為上游資料來源）

| 專案 | 語言 | 策略 | 搜尋引擎整合 |
|------|------|------|-------------|
| **feedsearch-crawler**[^fsc] | Python | HTML `<link>`/`<a>`、sitemap、robots.txt、常見路徑猜測 | 可作為 crawler，餵入 Elasticsearch/Solr |
| **feedfinder2**[^ff2] | Python | 同上（較輕量） | 同步 API，適合簡單場景 |
| **hera-rss-crawler**[^hera] | PHP | Content-Type、HTML head/anchor、Feedly 降級 | 可自訂發現器鏈 |

---

## 三、推薦方案對照

| 使用情境 | 推薦方案 |
|----------|---------|
| 需要完整開箱即用的 RSS 搜尋引擎 | **DATO.RSS**（最成熟，附 300 萬條目） |
| 需要輕量 RSS 搜尋 API | **SearQ**（Meilisearch，速度快） |
| 已有 feed 清單，需要全文搜尋 | **hackwage**（Elasticsearch，易設定） |
| 需要從 feed 中爬取文章全文再索引 | **Open Semantic Search**（唯一完整連結 RSS→爬取→索引） |
| 想要語意/向量搜尋 | **BlogNerd**（Pinecone + Voyage AI） |
| 離線/隱私優先 | **OfflineWebSearch**（SQLite，完全不聯網） |

---

## 四、生態缺口

目前 FOSS 生態中**沒有**一個專案能做到「Google Reader 級別的主動全網路 RSS 爬取 + 全文索引 + 公開搜尋」。多數專案（DATO.RSS、SearQ、hackwage、EchoNews）需要使用者**自行提供 feed 來源清單**，而非自行從網路上發掘 feed。唯一涵蓋 RSS→爬取→索引完整鏈的是 Open Semantic Search，但它並非專為 RSS 設計。

若要填補此缺口，理想組合為：
1. **feedsearch-crawler** 或 **feedsearch.dev** API 進行 feed 發現
2. **上述 RSS 搜尋引擎之一**（DATO.RSS / SearQ / hackwage）進行索引與搜尋
3. 定時排程（cron / Sidekiq）進行週期性重新爬取

---

[^dato]: David Santangelo. (n.d.). DATO.RSS. Retrieved 2026-10-01, from https://github.com/davidesantangelo/dato.rss
[^searq]: David Santangelo. (n.d.). SearQ. Retrieved 2026-10-01, from https://github.com/davidesantangelo/searq.org
[^hackwage]: santinic. (n.d.). hackwage. Retrieved 2026-10-01, from https://github.com/santinic/hackwage
[^oss]: Open Semantic Search. (n.d.). open-semantic-search. Retrieved 2026-10-01, from https://github.com/opensemanticsearch/open-semantic-search
[^blognerd]: Alastair Rushworth. (n.d.). blognerd.app. Retrieved 2026-10-01, from https://github.com/alastairrushworth/blognerd.app
[^echo]: RJohnPaul. (n.d.). EchoNews. Retrieved 2026-10-01, from https://github.com/RJohnPaul/EchoNews
[^ows]: rumca-js. (n.d.). OfflineWebSearch. Retrieved 2026-10-01, from https://github.com/rumca-js/OfflineWebSearch
[^pfs]: loxal. (n.d.). page-finder-service. Retrieved 2026-10-01, from https://github.com/loxal/page-finder-service
[^fsc]: DBeath. (n.d.). feedsearch-crawler. Retrieved 2026-10-01, from https://github.com/DBeath/feedsearch-crawler
[^ff2]: Dan Foreman-Mackey. (n.d.). feedfinder2. Retrieved 2026-10-01, from https://github.com/dfm/feedfinder2
[^hera]: Kaishiyoku. (n.d.). hera-rss-crawler. Retrieved 2026-10-01, from https://github.com/Kaishiyoku/hera-rss-crawler