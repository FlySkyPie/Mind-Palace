# Context7 的 FOSS 替代方案研究

## Context7 是什麼

Context7 是由 Upstash 公司開發的文檔注入工具，專門為 LLM 與 AI 程式碼編輯器（如 Claude Code、Cursor、OpenCode 等）提供**即時、版本特定的程式庫文檔**，直接注入到 LLM 的提示上下文中，解決 LLM 依賴過時訓練資料所產生的幻覺 API 與過時代碼問題[^context7]。

Context7 的運作模式有兩種：
- **CLI + Skills 模式**：透過 `ctx7` CLI 指令安裝一個 skill，引導 AI agent 在需要時自動擷取文檔
- **MCP 模式**：註冊 Context7 MCP 伺服器，讓 agent 可以原生呼叫文檔工具

目前 Context7 的 **MCP 伺服器原始碼**以 MIT 授權在 GitHub 上開源[^context7-gh]，但其**核心後端（API 後端、解析引擎、爬蟲引擎）為私有、未開源**[^context7-disclaimer]。這意味著 Context7 本質上是一個 **部分開源、雲端依賴的服務**，而非真正的端到端自託管方案。

**重要釐清**：Context7 並非程式碼審查（Code Review）工具，而是**即時文檔注入 / RAG（檢索增強生成）工具**，透過為 AI 編碼 agent 提供最新的程式庫文檔來提升程式碼生成品質。

## FOSS 替代方案列表

以下方案均為自由開源軟體（FOSS），可自託管（self-hosted）或本機執行。

### 1. mcp-ragdocs (⭐265)

| 屬性 | 內容 |
|------|------|
| 授權 | MIT |
| 語言 | TypeScript |
| 倉庫 | hannesrudolph/mcp-ragdocs[^mcp-ragdocs] |
| 核心機制 | 向量檢索 + Qdrant + OpenAI Embeddings |

一個 MCP 伺服器，提供向量搜尋（vector search）的文檔檢索與處理工具，讓 AI agent 能透過語意搜尋存取文檔。使用 Qdrant 作為向量資料庫、OpenAI Embeddings 進行向量化。

**工具列表**：
- `search_documentation`：自然語言查詢文檔，回傳相關片段
- `list_sources`：列出所有已建立索引的文檔來源
- `extract_urls`：爬取網頁並分析 URL
- `remove_documentation`：移除特定文檔來源
- `run_queue` / `clear_queue`：批次處理／清除文檔佇列

> **Context7 相似度**：★★★★☆ — 同為文檔注入工具，但依賴外部向量資料庫與 API。

### 2. Repocks (⭐9)

| 屬性 | 內容 |
|------|------|
| 授權 | MIT |
| 語言 | TypeScript |
| 倉庫 | boke0/repocks[^repocks] |
| 核心機制 | 本機向量檢索 + Ollama |

將 Markdown 文檔轉換為可搜尋的知識庫，透過 MCP 伺服器提供 AI 輔助搜尋與問答功能。完全本機執行，使用 Ollama 作為 embedding 與推理引擎。

**特色**：
- 本機運作，資料不離開機器
- 自動掃描指定目錄的 `.md` 檔案
- 使用 Ollama（如 `qwen3:4b`、`mxbai-embed-large` 等模型）
- 支援自訂索引目標（透過 `repocks.config.json`）
- 與任何支援 MCP 的用戶端相容（Claude Code、Cline 等）

> **Context7 相似度**：★★★★☆ — 本機優先，完全自託管，但限於 Markdown 文檔（非即時網頁爬取）。

### 3. npm-package-docs-mcp

| 屬性 | 內容 |
|------|------|
| 授權 | 開源（具體授權待確認） |
| 語言 | TypeScript | 
| 核心機制 | 即時擷取 NPM 套件文檔 |

一個 MCP 伺服器，能夠即時擷取任意 NPM 套件的最新文檔[^npm-package-docs]。與 Context7 的核心使用情境（「取得套件 X 的 API 文檔」）最為接近。

> **Context7 相似度**：★★★★★（針對 NPM 套件） — 功能匹配度最高，但僅限 NPM 生態系。

### 4. godoc-mcp-server

| 屬性 | 內容 |
|------|------|
| 授權 | 開源 |
| 語言 | TypeScript |
| 核心機制 | 查詢 Go 套件文檔 |

查詢 `pkg.go.dev` 上的 Go 套件資訊，為 Go 開發者提供即時文檔查詢能力[^godoc-mcp]。

> **Context7 相似度**：★★★☆☆ — 限定 Go 語言生態系。

### 5. mcp-typescribe

| 屬性 | 內容 |
|------|------|
| 語言 | TypeScript |
| 核心機制 | TypeScript API 資訊查詢 |

MCP 伺服器，為 agent 提供 TypeScript API 資訊，使其能夠與未受訓練的 API 互動[^mcp-typescribe]。

> **Context7 相似度**：★★★☆☆ — 限定 TypeScript 生態系。

### 6. Docustore

| 屬性 | 內容 |
|------|------|
| 語言 | TypeScript |
| 核心機制 | 向量化技術文檔儲存 |

向量化的技術文檔儲存庫，用來將技術文檔向量化儲存並提供檢索能力[^docustore]。

> **Context7 相似度**：★★★☆☆ — 廣義向量檢索方案，但未預設與 coding agent 整合。

### 7. wigolo (⭐數未明)

| 屬性 | 內容 |
|------|------|
| 授權 | 開源（公測中） |
| 語言 | TypeScript |
| 倉庫 | KnockOutEZ/wigolo[^wigolo] |
| 核心機制 | 本機搜尋 + 爬蟲 + MCP |

一個「為 AI coding agent 打造的 Web 搜尋工具」— 本機優先的搜尋、擷取、爬蟲與研究工具，透過 MCP 運作。無需 API key、無雲端、每次查詢零成本[^wigolo]。

> **Context7 相似度**：★★★☆☆ — 非專門的文檔工具，但能透過即時網頁擷取提供類似功能。

## 自建方案（DIY Alternatives）

若以上專案無法完全滿足需求，可基於以下開源元件自建 Context7 替代方案：

### 8. Headroom

| 屬性 | 內容 |
|------|------|
| 授權 | MIT |
| 倉庫 | headroomlabs-ai/headroom[^headroom] |
| 核心能力 | Token 壓縮（60-95% 減少） |

雖然 Headroom 主要是「壓縮工具輸出、日誌與 RAG chunks」的 MCP 伺服器，而非文檔注入工具，但它可作為 Context7 流程中的**元件**：在注入大量文檔時壓縮 token 使用量[^headroom]。

### 9. Not Human Search

| 屬性 | 內容 |
|------|------|
| 核心能力 | 以 agentic readiness 排名網站的搜尋引擎 |

一個專為 AI agent 設計的搜尋引擎，依照 `llms.txt`、OpenAPI、MCP、`ai-plugin` 等標準對網站進行排名，目前已索引超過 8,000 個網站，透過 MCP 伺服器、REST API 與全文搜尋公開存取[^not-human-search]。

### 10. Qdrant + Ollama 自建

最靈活的方案是自行組合以下元件的管線（pipeline）：

```
網頁爬蟲 → Markdown/HTML 解析 → Ollama Embeddings → Qdrant 向量庫 → MCP 伺服器
```

- **Qdrant**（開源向量資料庫）作為文檔儲存層
- **Ollama** 作為本機 embedding 與推理引擎
- **llm.txt / llms.txt** 標準作為文檔來源（llmstxt 是一項 emerging standard，讓網站公開 LLM 可讀摘要）

此方案可完全複製 Context7 的核心功能，但需要較高的初期建置成本。

## 比較總結

| 方案 | 自託管 | 即時擷取 | 多語言支援 | MCP 原生 | 維護程度 | Context7 相似度 |
|------|--------|----------|-----------|---------|---------|----------------|
| Context7 (本體) | ❌（部分） | ✅ | ✅ | ✅ | 活躍（Upstash） | ★★★★★ |
| mcp-ragdocs | ✅（需 Qdrant） | ✅（需設定 URL） | ✅（通用） | ✅ | 中等 | ★★★★☆ |
| Repocks | ✅（完全本機） | ❌（本地 md） | ❌（限 md） | ✅ | 新專案 | ★★★★☆ |
| npm-package-docs-mcp | ✅ | ✅ | ❌（限 NPM） | ✅ | 新專案 | ★★★★★(限 NPM) |
| Docustore | ✅ | ✅ | ✅ | ❌（無 MCP） | 新專案 | ★★★☆☆ |
| wigolo | ✅（完全本機） | ✅ | ✅（通用） | ✅ | 活躍 | ★★★☆☆ |
| 自建 (Qdrant+Ollama+MCP) | ✅（完全自控） | ✅ | ✅ | ✅ | 自負 | ★★★★★ |

## 結論

目前**沒有任何單一 FOSS 專案能完全複製 Context7 的所有功能**（即時多語言程式庫文檔擷取 + 智慧語意匹配 + 注入 agent prompt），但以下組合最接近：

1. **針對 NPM 生態系**：`npm-package-docs-mcp` + `mcp-typescribe` — 已可取代大部分日常使用場景
2. **通用用途，需自建**：`mcp-ragdocs` + Ollama（替代 OpenAI Embeddings）+ 自訂爬蟲 → 可達 Context7 90% 功能
3. **完全本機、注重隱私**：`Repocks` — 適合內部文件庫，但不具備即時網頁擷取能力

若社群需要一個真正的 Context7 FOSS 替代品，理想規格應為：**一個 MCP 伺服器，使用本機 Ollama embeddings，支援 llms.txt / OpenAPI / 程式庫註冊表（npm、PyPI、crates.io）作為文檔來源，能在無雲端依賴的情況下實現「即時文檔注入」**。截至 2026 年 9 月，此專案尚不存在於 FOSS 生態系中。

---

[^context7]: Context7 官方網站. (n.d.). Context7 - Up-to-date documentation for LLMs and AI code editors. Retrieved 2026-09-13, from https://context7.com
[^context7-gh]: Upstash. (n.d.). context7. GitHub. Retrieved 2026-09-13, from https://github.com/upstash/context7
[^context7-disclaimer]: Upstash. (n.d.). context7 README. GitHub. Retrieved 2026-09-13, from https://github.com/upstash/context7 — Section "Disclaimer" 指出後端元件為私有。
[^mcp-ragdocs]: hannesrudolph. (n.d.). mcp-ragdocs. GitHub. Retrieved 2026-09-13, from https://github.com/hannesrudolph/mcp-ragdocs
[^repocks]: boke0. (n.d.). repocks. GitHub. Retrieved 2026-09-13, from https://github.com/boke0/repocks
[^npm-package-docs]: meanands. (n.d.). npm-package-docs-mcp. Show HN. Retrieved 2026-09-13, from https://news.ycombinator.com/item?id=47388646
[^godoc-mcp]: yikakia. (n.d.). godoc-mcp-server. Retrieved 2026-09-13, from https://github.com/punkpeye/awesome-mcp-servers
[^mcp-typescribe]: yWorks. (n.d.). mcp-typescribe. Retrieved 2026-09-13, from https://github.com/punkpeye/awesome-mcp-servers
[^docustore]: PAndreew. (n.d.). docustore. Show HN. Retrieved 2026-09-13, from 相關 HN 討論。
[^wigolo]: KnockOutEZ. (n.d.). wigolo. GitHub. Retrieved 2026-09-13, from https://github.com/KnockOutEZ/wigolo
[^headroom]: headroomlabs-ai. (n.d.). headroom. GitHub. Retrieved 2026-09-13, from https://github.com/headroomlabs-ai/headroom
[^not-human-search]: bradAGI. (n.d.). awesome-cli-coding-agents. GitHub. Retrieved 2026-09-13, from https://github.com/bradAGI/awesome-cli-coding-agents