# TiddlyRAG 專案

## 專案起源

- 因為 LLM 會幻覺，所以人們需要 RAG (Retrieval-Augmented Generation)。
- Chunking, Embedding, Graph 是現代 RAG 的基本要素。
- 早在 LLM 時代以前，DDD (Domain-driven design) 就已經被提出。
- 因為 LLM 降低程式碼成本，重心將轉移到領域知識上，事實是 DDD 不是 LLM 時代發明的新東西，而是原本就已經存在的概念與方法論，只是在 LLM 時代更顯重要。
- TiddlyWiki 也不是新東西，只是它巧妙的具備以下特性：
  - Tiddler 和 Wiki 特性剛好對應了 Chunking 和 Graph。
  - 檔案即程式與資料帶來了遠比 llm.txt 更好的特性：
    - 人類可讀。
    - 人類可編輯。
    - 同時仍然方便被程序化讀寫，因此適合跟 LLM/RAG 系統整合。
  - 卡片盒筆記 (Zettelkasten) 是天然的領域知識載體，TiddlyWiki 本身跟卡片盒筆記契合，因此 TiddlyWiki 是天然的 DDD 載體。

## 具體作為

- https://github.com/FlySkyPie/tiddlyrag-poc
  - 建立各種 POC，包含：
    - Embedding + RAG + MCP + Tiddler 結構實驗
    - 試圖重構 tiddlyweb
    - 試圖重構 MWS
- https://github.com/FlySkyPie/github-issue-simple-etl
  - 使用 ETL 工具調查 [MWS](https://github.com/TiddlyWiki/MultiWikiServer)
- https://github.com/FlySkyPie/tiddlyrag-planning
  - 建立計畫用 TiddlyWiki
- https://github.com/FlySkyPie/tiddlyweb-tw5-sample
  - 以 Docker 再現 Tiddlyweb 與 TiddlyWiki5 的整合範例
- https://github.com/FlySkyPie/tiddlyweb-spec
  - 將 tiddlyweb 的行為使用 LLM 抽出變成 OpenAPI Spec
