# LLM 驅動的 RSS 改善：開源專案調查

## 概述

RSS 是一項經典但日漸老化的技術。本報告調查以 LLM（大型語言模型）改善 RSS 體驗的開放原始碼專案，特別聚焦於**自動 RSS 探索（auto discover）**與**自動爬取（auto crawl）**方向──人類太懶得自己找 RSS 了。

## 分類架構

可將 LLM + RSS 的開源專案分為三大類：

### A. RSS 自動探索（Auto Discovery）

給一個網址或主題，自動找出相關的 RSS feed。

### B. RSS 內容增強（Content Enhancement）

已訂閱 feed 的內容，用 LLM 進行摘要、翻譯、重寫、分類。

### C. RSS feed 生成（Feed Generation）

從沒有 RSS 的網頁，用 LLM 自動產生 RSS feed。

---

## A. 自動探索（Auto Discovery）

### 1. SkyHustle/rss_feed_discovery_tool

- **GitHub**: <https://github.com/SkyHustle/rss_feed_discovery_tool>
- **Stars**: 0（極早期）
- **語言**: Python
- **做法**: 接收一個「主題提示詞」（topic prompt），先問 LLM（透過 OpenRouter/Gemini）推薦相關新聞網站，再檢查那些網站是否有 RSS feed，最後回傳最多 10 個有效 feed。
- **LLM 角色**: 用 LLM 來**猜測哪些網站可能與主題相關**──這是目前最接近「自動探索」的作品。
- **侷限**: 非常早期（9 個 commits），僅支援 OpenRouter API，無 UI。

### 2. saptarshisama/rss-feed-discovery

- **GitHub**: <https://github.com/saptarshisama/rss-feed-discovery>
- **Stars**: 1
- **語言**: Python
- **做法**: 給定網址清單，透過 HTML 解析、`<link>` 標籤、常見 URL 模式（`/feed`、`/rss`、`/atom.xml`）來發現 RSS/Atom feed。使用 Type A/B 發現策略──Type A 從提供的路徑提取，Type B 先找出最佳內容子頁（blog、news 等）再找 feed。
- **LLM 角色**: 無（純啟發式/模式匹配）。
- **價值**: 可作為 LLM 管線的前置過濾層。

### 3. DBeath/feedsearch-crawler

- **GitHub**: <https://github.com/DBeath/feedsearch-crawler>
- **PyPI**: `feedsearch-crawler`
- **語言**: Python（asyncio/aiohttp）
- **做法**: 給一個網站 URL，非同步爬取網頁，檢查 `<link>` 標籤，測試常見 feed URL 路徑，透過 `feedparser` 驗證 feed，回傳結構化的 `FeedInfo` 物件（依相關性分數排序）。
- **LLM 角色**: 無，但它是理想的 LLM 管線底層元件。

### 4. michaelrhodes/feed-discover

- **GitHub**: <https://github.com/michaelrhodes/feed-discover>
- **語言**: Node.js
- **做法**: 提供一個 transform stream，讀入 HTML 後寫出 feed URL。
- **LLM 角色**: 無。

---

## B. RSS 內容增強（Content Enhancement）

### 5. Seanium/FeedMe

- **GitHub**: <https://github.com/Seanium/FeedMe>
- **Stars**: 749
- **語言**: 靜態網站（GitHub Pages / Vercel / Docker）
- **做法**: 自託 RSS reader，設定 YAML 中的 feed URL 後，透過 GitHub Actions cron 定時抓取，用 LLM（OpenAI-compatible API）自動為每篇文章產生摘要，輸出為靜態網站。
- **LLM 角色**: 文章摘要生成。
- **注意**: 不包含 feed 自動探索──需手動設定來源。

### 6. fabriziosalmi/UglyFeed

- **GitHub**: <https://github.com/fabriziosalmi/UglyFeed>
- **Stars**: 320
- **語言**: Python
- **做法**: 完整的 LLM 重寫管線──擷取 RSS feed → 相似度聚合 → 用 LLM（OpenAI、Ollama、Groq、Anthropic、Gemini）重寫內容 → 儲存為 JSON → 轉換為有效 RSS XML → HTTP 提供 → GitHub/GitLab CDN 部署。支援翻譯、審核、內容評價。
- **LLM 角色**: 用自訂提示重寫/再生 RSS 內容（翻譯、摘要、角色扮演等）。
- **注意**: 不包含 feed 自動探索。

### 7. yinan-c/RSSBrew

- **GitHub**: <https://github.com/yinan-c/RSSbrew>
- **Stars**: 296
- **做法**: Django Web GUI，聚合多個 feed 到一個，自訂過濾（包含/排除、正規表達式、AND/OR/NOT 群組），產生 AI 摘要，支援每日/每週 AI 摘要。
- **LLM 角色**: 任何 OpenAI-compatible 模型，文章摘要 + 定期摘要。

### 8. DanielZhangyc/RLLM

- **GitHub**: <https://github.com/DanielZhangyc/RLLM>
- **Stars**: 100
- **平台**: iOS（Swift/SwiftUI）
- **做法**: iOS RSS reader 整合 AI：文章摘要生成、洞見分析、每日閱讀 AI 摘要。
- **LLM 角色**: Anthropic、Deepseek、OpenAI API 摘要生成。

### 9. leozqin/Precis

- **GitHub**: <https://github.com/leozqin/precis>
- **Stars**: 94
- **做法**: FastAPI 為基礎的自託 RSS reader，支援 Ollama/OpenAI 摘要、Matrix/Slack/Jira/ntfy 通知、OPML 匯入/匯出、多主題。
- **LLM 角色**: 文章摘要（可設定提示詞）。

### 10. jaypetez/glean

- **GitHub**: <https://github.com/jaypetez/glean>
- **Stars**: 7
- **做法**: Python daemon，從 RSS、爬取、Reddit、HN、搜尋 API 拉取內容，經 LLM（Ollama、Anthropic、OpenAI）處理，定時推送到 Telegram、Discord、Slack、Email、ntfy。支援每來源不同 LLM 調度。
- **LLM 角色**: 多來源內容摘要/消化。

### 11. maxwelljensen/llm_aggregator

- **GitHub**: <https://github.com/maxwelljensen/llm_aggregator>
- **語言**: Go（單一二進制，零執行期依賴）
- **做法**: CLI 工具，從多個 RSS/Atom/JSON feed 並行擷取（附速率限制），關鍵字/日期過濾，排序，送給 LLM 摘要。支援 TUI 即時進度。
- **LLM 角色**: 使用者自訂提示（如「今天頂尖科技故事是什麼？」）。

### 12. laplacef/digest-generator

- **GitHub**: <https://github.com/laplacef/digest-generator>
- **PyPI**: `digest-generator`
- **做法**: Python 管線，從設定的 feed 擷取文章，用 Ollama 生成摘要，Zero-shot NLI（HuggingFace）分類，產出 Markdown digest。完全本地執行。
- **LLM 角色**: Ollama 摘要 + HuggingFace 分類。

### 13. cvlc/freshrss-ai-assistant

- **GitHub**: <https://github.com/cvlc/freshrss-ai-assistant>
- **Stars**: 8
- **做法**: FreshRSS 擴展，使用 OpenAI-compatible LLM 進行智慧重新標題、執行摘要、自動標籤、分類摘要。

### 14. tav607/rss-digest

- **GitHub**: <https://github.com/tav607/rss-digest>
- **做法**: 兩階段管線 → (1) 從 FreshRSS 資料庫讀文章，平行生成每篇文章摘要 → (2) 產生分類的全球 digest → 推送到 Telegram。支援 cron 排程。

### 15. LeslieLeung/glean

- **GitHub**: <https://github.com/LeslieLeung/glean>
- **Stars**: 870
- **做法**: 自託 RSS reader + 知識管理器。AI 功能（智慧推薦、摘要、自動標籤、關鍵字提取）仍在路線圖上。

### 16. rss-ai-middleware（PyPI）

- **做法**: MCP（Model Context Protocol）中介軟體，橋接 Miniflux/FreshRSS 與 LLM，將 feed 作為「工具」暴露給 LLM agent。

---

## C. RSS Feed 生成（Feed Generation）

### 17. leontloveless/ai-rss-feeds

- **GitHub**: <https://github.com/leontloveless/ai-rss-feeds>
- **Stars**: 46
- **理念**: 「LLM 是編譯器，不是直譯器」──用 LLM 分析部落格 HTML 結構一次，產出 CSS selector 設定，之後用低成本 parser 定時執行。
- **做法**: 7 種 parser 模式（CSS、JSON、Changelog、RSS、external、GitHub-releases），6 層驗證管線，自動修復 feed（網站改版時），已為 21+ AI 部落格產生 feed。
- **LLM 角色**: 分析 HTML 結構，產出解析設定。

### 18. yinan-c/RSS-GPT

- **GitHub**: <https://github.com/yinan-c/RSS-GPT>
- **Stars**: 356
- **做法**: 先驅專案──用 GitHub Actions 定期執行 Python 腳本，呼叫 OpenAI API 為 RSS feed 文章生成 AI 摘要，然後將增強後的 feed 提交到 GitHub Pages。無需伺服器。
- **LLM 角色**: ChatGPT API 為文章添加 AI 摘要。

### 19. Olshansk/rss-feeds

- **GitHub**: <https://github.com/Olshansk/rss-feeds>
- **Stars**: 695
- **做法**: 爬取沒有 RSS 的 AI 公司部落格（Anthropic、OpenAI、Meta AI、Mistral 等），從 HTML 產生 RSS feed。每小時透過 GitHub Actions 執行。使用 Claude Code CLI 協助產生 feed 產生器。
- **LLM 角色**: Claude（Anthropic）協助撰寫 Python feed 爬取腳本。

### 20. thomd/rss-feeds

- **GitHub**: <https://github.com/thomd/rss-feeds>
- **做法**: 使用 GitHub Actions + GitHub Models（gpt-4o-mini）自動從任何公開網頁提取項目來產生 RSS 2.0 feed。只需設定 URL/ID/name──不需 CSS 選擇器。LLM 自動偵測頁面結構。
- **LLM 角色**: gpt-4o-mini 分析 HTML，自動識別文章/部落格結構，提取標題/連結/日期。

### 21. damien220/kompyla

- **GitHub**: <https://github.com/damien220/kompyla>
- **做法**: 自主研究 agent──指向一個領域後，定期拉入網頁、arXiv 預印本、GitHub repo、RSS feed、YouTube 逐字稿，用 LLM 編譯成結構化、附引用的 wiki。
- **LLM 角色**: 從多來源（包括 RSS）合成內容成知識庫。

### 22. fabiospampinato/rssa（RSS Anything，已封存）

- **GitHub**: <https://github.com/fabiospampinato/rssa>
- **Stars**: 49（已封存）
- **做法**: 監控任何 URL 的變更（非傳統 RSS）──定義 URL、CSS 選擇器或正規表達式模式，回報變更。更像是頁面變更監控。
- **LLM 角色**: 無（前 LLM 時代）。

---

## 關鍵洞見

### 目前缺少什麼

**「給一個主題或網址 → 自動找到並訂閱所有相關 RSS feed」的完整開源解決方案目前不存在。**

- 自動探索端（category A）的專案要不是純啟發式（無 LLM），就是極早期。
- 內容增強端（category B）已有成熟、高星數的專案（FeedMe 749★、UglyFeed 320★、RSSBrew 296★）。
- Feed 生成端（category C）也有成熟且高星數的專案（Olshansk 695★、RSS-GPT 356★）。

### 最接近「自動探索」的專案

| 專案 | 方法 | LLM 角色 | 成熟度 |
|------|------|----------|--------|
| SkyHustle/rss_feed_discovery_tool | LLM 建議網站 → 驗證 RSS | LLM 猜網站 | 極早期 |
| leontloveless/ai-rss-feeds | LLM 分析 HTML → 產生 feed | LLM 分析結構 | 成熟 |
| saptarshisama/rss-feed-discovery | 啟發式探索（無 LLM） | 無 | 基礎 |

### 可行方向

若要建立「人類懶得找 RSS，讓 LLM 幫忙」的工具，可組合：

1. **探索層**: 使用 `feedsearch-crawler`（啟發式，成熟）或 `SkyHustle/rss_feed_discovery_tool` 的 LLM 方法（早期）
2. **生成層**: 使用 `leontloveless/ai-rss-feeds`（LLM 分析 HTML 產 feed）或 `thomd/rss-feeds`（gpt-4o-mini 自動提取）
3. **增強層**: 使用 `FeedMe` 或 `UglyFeed` 做摘要/重寫
4. **交付層**: 任何標準 RSS reader（Miniflux、FreshRSS）或直接推送（Telegram、Email）

### LLM 在 RSS 領域的主要應用模式

| 模式 | 說明 | 代表專案 |
|------|------|---------|
| LLM 作為探索引擎 | 用 LLM 猜測相關網站或網頁結構 | SkyHustle, ai-rss-feeds, thomd/rss-feeds |
| LLM 作為摘要引擎 | 為文章或定期 digest 產生摘要 | FeedMe, RSSBrew, RLLM, Precis |
| LLM 作為重寫引擎 | 翻譯、改寫、角色扮演內容 | UglyFeed |
| LLM 作為分類引擎 | 自動標籤、分類、過濾 | freshrss-ai-assistant, digest-generator |
| LLM 作為編譯引擎 | 多來源合成為知識庫 | kompyla |

---

## 參考文獻

- Seanium. (n.d.). FeedMe. Retrieved 2026-10-01, from https://github.com/Seanium/FeedMe
- fabriziosalmi. (n.d.). UglyFeed. Retrieved 2026-10-01, from https://github.com/fabriziosalmi/UglyFeed
- yinan-c. (n.d.). RSSBrew. Retrieved 2026-10-01, from https://github.com/yinan-c/RSSbrew
- DanielZhangyc. (n.d.). RLLM. Retrieved 2026-10-01, from https://github.com/DanielZhangyc/RLLM
- leozqin. (n.d.). Precis. Retrieved 2026-10-01, from https://github.com/leozqin/precis
- yinan-c. (n.d.). RSS-GPT. Retrieved 2026-10-01, from https://github.com/yinan-c/RSS-GPT
- Olshansk. (n.d.). rss-feeds. Retrieved 2026-10-01, from https://github.com/Olshansk/rss-feeds
- leontloveless. (n.d.). ai-rss-feeds. Retrieved 2026-10-01, from https://github.com/leontloveless/ai-rss-feeds
- jaypetez. (n.d.). glean. Retrieved 2026-10-01, from https://github.com/jaypetez/glean
- maxwelljensen. (n.d.). llm_aggregator. Retrieved 2026-10-01, from https://github.com/maxwelljensen/llm_aggregator
- LeslieLeung. (n.d.). glean. Retrieved 2026-10-01, from https://github.com/LeslieLeung/glean
- cvlc. (n.d.). freshrss-ai-assistant. Retrieved 2026-10-01, from https://github.com/cvlc/freshrss-ai-assistant
- laplacef. (n.d.). digest-generator. Retrieved 2026-10-01, from https://github.com/laplacef/digest-generator
- tav607. (n.d.). rss-digest. Retrieved 2026-10-01, from https://github.com/tav607/rss-digest
- DBeath. (n.d.). feedsearch-crawler. Retrieved 2026-10-01, from https://github.com/DBeath/feedsearch-crawler
- saptarshisama. (n.d.). rss-feed-discovery. Retrieved 2026-10-01, from https://github.com/saptarshisama/rss-feed-discovery
- SkyHustle. (n.d.). rss_feed_discovery_tool. Retrieved 2026-10-01, from https://github.com/SkyHustle/rss_feed_discovery_tool
- thomd. (n.d.). rss-feeds. Retrieved 2026-10-01, from https://github.com/thomd/rss-feeds
- fabiospampinato. (n.d.). rssa. Retrieved 2026-10-01, from https://github.com/fabiospampinato/rssa
- damien220. (n.d.). kompyla. Retrieved 2026-10-01, from https://github.com/damien220/kompyla
- mixiaoluo88. (n.d.). AutoRSS. Retrieved 2026-10-01, from https://github.com/mixiaoluo88/AutoRSS