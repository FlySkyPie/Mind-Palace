# LLM 驅動之 RSS 改善：開源專案調查

RSS (Really Simple Syndication) 是經典的內容訂閱格式，但其原始設計缺乏智慧過濾、摘要、個人化等能力。本報告調查當前使用大型語言模型 (LLM) 改善 RSS 體驗的開放原始碼專案。

## 調研結果

以下為 GitHub 上活躍的開源專案，皆以 LLM 強化 RSS 流程：

### 1. Folo (原 RSSNext)

跨平台 (Web/iOS/Android/桌面) 的 AI 驅動 RSS 閱讀器，將內容組織為單一無噪聲時間軸。內建 AI 翻譯與摘要功能[^folo]。

- **LLM 功能**：AI 摘要、AI 翻譯、AI 降噪
- **授權**：AGPL-3.0
- **Stars**: ~39,100

### 2. Glean (LeslieLeung)

自託管 RSS 閱讀器與個人知識管理工具。使用 AI 嵌入向量 (Milvus 向量資料庫) 實作語意搜尋與「智慧 feed」——超越關鍵字比對，智慧發掘相關內容[^glean-ll]。

- **LLM 功能**：語意搜尋、智慧 feed (AI 內容推薦)；規劃中：AI 摘要、自動標籤、關鍵字萃取
- **授權**：AGPL-3.0
- **Stars**: ~870

### 3. FeedMe

輕量化 AI RSS 閱讀器，可部署至 GitHub Pages、Docker 或 Vercel。使用 LLM 自動產生文章摘要[^feedme]。

- **LLM 功能**：可設定 LLM 摘要（API key、base URL、模型名稱皆可調）、自訂摘要提示、語言/長度控制
- **授權**：MIT
- **Stars**: ~749

### 4. Saga Reader

跨平台 AI 驅動「智庫閱讀器」。使用雲端或本地 LLM 自動依使用者關鍵字檢索、摘要、提供導讀。具備 AI 互動伴讀功能[^saga]。

- **LLM 功能**：AI 摘要（雲端 + 本地 LLM）、AI 互動伴讀/問答、智慧內容檢索、多語言翻譯
- **授權**：MIT
- **Stars**: ~537

### 5. RSS-GPT

使用 GitHub Actions 排程 ChatGPT API 摘要 RSS feed。聚合多個 feed、去重、以 AI 摘要前綴於文章前。無需伺服器，託管於 GitHub Pages[^rssgpt]。

- **LLM 功能**：ChatGPT 摘要、自訂摘要長度/語言/過濾規則
- **授權**：MIT
- **Stars**: ~356

### 6. RSSBrew

RSS-GPT 的後繼者，Django 為基底的自託管 RSS 工具，包含 Web GUI。可聚合、過濾、摘要文章，並產生每日/每週 AI 摘要[^rssbrew]。

- **LLM 功能**：AI 摘要（OpenAI 相容模型）、自訂提示、AI 每日/每週摘要、基於過濾的摘要範圍控制
- **授權**：AGPL-3.0
- **Stars**: ~296

### 7. Precis (Précis)

可延伸的自託管 AI RSS 閱讀器，著重通知。使用 LLM 摘要與跨來源資訊綜合。支援 Ollama、OpenAI 與可延伸 LLM 處理器[^precis]。

- **LLM 功能**：LLM 摘要（Ollama/OpenAI/可延伸）、跨來源資訊綜合
- **授權**：MIT
- **Stars**: ~94

### 8. glean (jaypetez)

自託管、可插拔的個人代理 daemon，從 RSS、爬蟲、Hacker News、Reddit、網頁搜尋收集訊號——以 LLM 處理後，排程摘要推送至 Telegram、Discord、Slack、Email、ntfy 等多種 sink[^glean-jp]。

- **LLM 功能**：可設定 LLM pipeline (去重 → 排名 → 摘要 → 技能萃取 → 摘要生成)、per-source LLM 指派、JSON 結構化技能輸出
- **授權**：MIT
- **Stars**: ~7

### 9. Readfine

自託管 Web RSS 閱讀器，具網頁擷取、過濾、可讀性萃取與選擇性 AI 功能。自帶 API key：[Anthropic、OpenAI、Gemini]或任何 OpenAI 相容端點（含本地 Ollama）[^readfine]。

- **LLM 功能**：AI 摘要、模型化相關性評分、文章問答、「補上進度」簡報與排程簡報
- **授權**：AGPL-3.0
- **Stars**: ~19

### 10. rssnook / Readr (AdrienLF)

自託管、單一使用者 RSS 閱讀器，具全文萃取與本地 AI 摘要。使用本地 LLM（Ollama + Qwen3.5:9b）產生每日主題摘要、隨選文章摘要、批量 feed 匯入 AI 分類、實體萃取。完全本地執行，無雲端依賴[^rssnook]。

- **LLM 功能**：每日 AI 摘要（本地 Ollama）、單篇文章摘要、批量 feed AI 分類、命名實體萃取
- **授權**：MIT
- **Stars**: ~0 (新專案)

## 專案比較總覽

| 專案 | LLM 核心功能 | 授權 | Stars |
|---|---|---|---|
| **Folo** | AI 翻譯 + 摘要 + 降噪 | AGPL-3.0 | ~39,100 |
| **Glean (LeslieLeung)** | 語意搜尋、智慧 feed (嵌入向量) | AGPL-3.0 | ~870 |
| **FeedMe** | LLM 文章摘要 | MIT | ~749 |
| **Saga Reader** | AI 伴讀、摘要、翻譯 | MIT | ~537 |
| **RSS-GPT** | ChatGPT 摘要 | MIT | ~356 |
| **RSSBrew** | AI 摘要 + 摘要 + 過濾 | AGPL-3.0 | ~296 |
| **Precis** | LLM 摘要 (Ollama/OpenAI) | MIT | ~94 |
| **Readfine** | AI 摘要、評分、簡報 | AGPL-3.0 | ~19 |
| **glean (jaypetez)** | LLM pipeline (排名、摘要、技能、推送) | MIT | ~7 |
| **rssnook** | 本地 AI 摘要、分類、實體萃取 | MIT | ~0 |

## 觀察與趨勢

1. **雲端 vs 本地 LLM**：早期專案 (RSS-GPT, RSSBrew) 依賴 OpenAI API；新興專案 (rssnook, Precis, Readfine) 逐漸支援 Ollama 等本地模型，反映了開源 LLM 與隱私意識的崛起。

2. **從摘要到 pipeline**：最早僅止於「AI 摘要文章標題/內文」，較成熟的專案（glean/jaypetez、Readfine）已實作完整 pipeline：去重 → 排名 → 摘要 → 技能萃取 → 多通路推送。

3. **向量資料庫的引入**：Glean (LeslieLeung) 引入 Milvus 等向量資料庫，使 RSS 閱讀從關鍵字比對進化至語意搜尋與智慧推薦，這是傳統 RSS 無法提供的。

4. **授權趨勢**：多數專案選擇 MIT（寬鬆）或 AGPL-3.0（強 Copyleft），反映了開源社群對兩極授權策略的偏好。

---

[^folo]: RSSNext. (n.d.). Folo — AI-powered RSS reader. Retrieved 2026-01-10, from https://github.com/RSSNext/Folo
[^glean-ll]: LeslieLeung. (n.d.). Glean — Self-hosted RSS reader with AI Smart Feeds. Retrieved 2026-01-10, from https://github.com/LeslieLeung/glean
[^feedme]: Seanium. (n.d.). FeedMe — Lightweight, AI-powered RSS reader. Retrieved 2026-01-10, from https://github.com/Seanium/feedme
[^saga]: sopaco. (n.d.). Saga Reader — AI-driven think tank reader. Retrieved 2026-01-10, from https://github.com/sopaco/saga-reader
[^rssgpt]: yinan-c. (n.d.). RSS-GPT — ChatGPT-powered RSS summarizer. Retrieved 2026-01-10, from https://github.com/yinan-c/RSS-GPT
[^rssbrew]: yinan-c. (n.d.). RSSBrew — Self-hosted RSS tool with AI summaries. Retrieved 2026-01-10, from https://github.com/yinan-c/RSSBrew
[^precis]: leozqin. (n.d.). Precis — Extensible self-hosted AI-enabled RSS reader. Retrieved 2026-01-10, from https://github.com/leozqin/precis
[^glean-jp]: jaypetez. (n.d.). glean — Personal agent daemon for RSS with LLM pipeline. Retrieved 2026-01-10, from https://github.com/jaypetez/glean
[^readfine]: jakublibik. (n.d.). Readfine — Self-hosted RSS reader with AI features. Retrieved 2026-01-10, from https://github.com/jakublibik/readfine
[^rssnook]: AdrienLF. (n.d.). rssnook — Self-hosted RSS reader with local AI digests. Retrieved 2026-01-10, from https://github.com/AdrienLF/rssnook