# FOSS RSS 網路爬蟲/探索專案調查

## 概述

本報告調查了專門用於**從網路中自動發掘未知 RSS/Atom 來源**的自由開源軟體（FOSS）專案。不同於一般的 RSS 訂閱器（僅抓取已知 feed），此類工具會主動爬取網站、掃描 HTML、解析 sitemap/robots.txt，以發現你原本不知道的 feed URL。

---

## 一、專用 Feed 探索爬蟲

### feedsearch-crawler（Python）— **最直接相關**

- **GitHub**：https://github.com/DBeath/feedsearch-crawler — 99 ★，MIT License
- **PyPI**：`feedsearch-crawler`

Python 非同步網頁爬蟲，專門用於探索網站上的 RSS、Atom 和 JSON Feed。

**Feed 發現策略**：
- 從種子 URL 開始爬取，掃描頁面中的 `<link>` 標籤、`<a>` 標籤，以及常見 CMS feed 路徑（如 `/rss`、`/feed`、`/atom.xml`）
- 檢查 `robots.txt`
- **平行解析 sitemap** — 甚至可以發現網頁中根本沒有連結到的 feed
- 對結果進行相關性評分
- 擷取豐富元資料（title、description、favicon 等）
- 非同步並發請求，速度快

```python
from feedsearch_crawler import search
feeds = search("xkcd.com")
# 回傳 [FeedInfo('https://xkcd.com/rss.xml'), FeedInfo('https://xkcd.com/atom.xml')]
```

**系譜**：feedfinder（Mark Pilgrim / Aaron Swartz）→ feedfinder2（Dan Foreman-Mackey）→ feedsearch → feedsearch-crawler（當前版本）。

---

### feedsearch.dev — **公開 Feed 搜尋 API**

- **URL**：https://feedsearch.dev

feedsearch-crawler 包裝成的公開 API 服務。任何人都可以查詢任意 URL 以發現關聯的 feed。

```
GET /api/v1/search?url=arstechnica.com
```

- 回傳 JSON 格式的 feed 元資料
- 快取結果，後續查詢直接回傳已儲存的 feed 而不用重新爬取
- 長期目標：建立一個全面、公開可存取的 feed 資訊儲存庫
- 商業使用需標註出處

---

### feedfinder2（Python）

- **GitHub**：https://github.com/dfm/feedfinder2 — 54 ★，MIT License

feedsearch-crawler 的前身。較簡單的同步 Python 函式庫。

```python
from feedfinder2 import find_feeds
feeds = find_feeds("xkcd.com")
```

- 基於 Mark Pilgrim 和 Aaron Swartz 原創的 [feedfinder](http://www.aaronsw.com/2002/feedfinder/)
- 功能較少（無 async、無 sitemap 解析），但對簡單場景仍夠用且依賴更輕

---

### feedsearch（Python）

- **GitHub**：https://github.com/DBeath/feedsearch — 24 ★，MIT License

feedsearch-crawler 之前的舊版本。已停止開發，但仍在 PyPI 上可取得。

---

## 二、多策略 Feed 發現工具

### hera-rss-crawler（PHP）

- **GitHub**：https://github.com/Kaishiyoku/hera-rss-crawler — 2 ★，MIT License

PHP 專用 RSS feed 發現函式庫。使用可串聯的 **Feed 發現器鏈**：

1. **FeedDiscovererByContentType** — 檢查 URL 本身是否即為 feed
2. **FeedDiscovererByHtmlHeadElements** — 解析 HTML 中的 `<link>` 標籤
3. **FeedDiscovererByHtmlAnchorElements** — 尋找含 "rss" 的 anchor 連結
4. **FeedDiscovererByFeedly** — 降級至 Feedly API

可自訂發現器順序和撰寫自訂發現器。

```php
$heraRssCrawler = new HeraRssCrawler();
$feedUrls = $heraRssCrawler->discoverFeedUrls('https://laravel-news.com/');
```

---

## 三、含 Feed 探索模組的通用工具

### riko（Python）

- **GitHub**：https://github.com/nerevu/riko — 1.6k ★，MIT License

資料串流處理引擎（類似 Yahoo Pipes / IFTTT 的開源實現），內建 `feedautodiscovery` 模組。該模組「掃描網頁或文件以發現潛在的 RSS/Atom feed URL」。

- 同步和非同步 API 皆可用
- 除了 feed 探索外，具有完整的資料處理管線能力

---

## 四、Feed 探索策略總結

| 策略 | 說明 | 支援專案 |
|------|------|----------|
| HTML `<link>` 標籤 | 掃描 `<link rel="alternate" type="application/rss+xml">` | 幾乎全部支援 |
| HTML `<a>` 錨點 | 尋找含 "rss"/"feed"/"atom" 的連結 | feedsearch-crawler、hera-rss-crawler |
| 常見路徑猜測 | 嘗試 `/rss`、`/feed`、`/atom.xml` | feedsearch-crawler |
| robots.txt | 解析 Crawl-Disallow 規則 | feedsearch-crawler |
| Sitemap 解析 | 平行抓取 sitemap 找出隱藏 feed | feedsearch-crawler |
| Feedly API 降級 | 查詢第三方索引 | hera-rss-crawler |

---

## 五、FOSS 生態缺口

**目前沒有**開源的「web-scale feed 搜尋引擎」——即類似 Google Reader 當年那樣的主動全網路爬蟲來建立 feed 索引。feedsearch.dev 是按需（on-demand）查詢而非主動掃描。這在 FOSS 生態中仍是一個未被滿足的需求。

---

## 結論

若要在未知 RSS 來源的情況下從網路發掘 feed：

- **最推薦**：`feedsearch-crawler`（Python async，策略最全面，支援 sitemap/robots.txt）
- **簡單場景**：`feedfinder2`（同步，無外部依賴）
- **API 服務**：`feedsearch.dev`（不需寫程式，直接 HTTP 查詢）
- **PHP 生態**：`hera-rss-crawler`（多策略鏈 + 自訂發現器）

---

[^fsc]: DBeath. (n.d.). feedsearch-crawler. Retrieved 2026-10-01, from https://github.com/DBeath/feedsearch-crawler
[^fsapi]: feedsearch.dev. (n.d.). Retrieved 2026-10-01, from https://feedsearch.dev
[^ff2]: Dan Foreman-Mackey. (n.d.). feedfinder2. Retrieved 2026-10-01, from https://github.com/dfm/feedfinder2
[^fs]: DBeath. (n.d.). feedsearch. Retrieved 2026-10-01, from https://github.com/DBeath/feedsearch
[^hera]: Kaishiyoku. (n.d.). hera-rss-crawler. Retrieved 2026-10-01, from https://github.com/Kaishiyoku/hera-rss-crawler
[^riko]: nerevu. (n.d.). riko. Retrieved 2026-10-01, from https://github.com/nerevu/riko