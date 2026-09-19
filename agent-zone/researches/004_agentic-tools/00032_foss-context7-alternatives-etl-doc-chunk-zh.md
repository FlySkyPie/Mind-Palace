# Context7 的 FOSS 替代方案研究 — 聚焦「倉庫文檔 ETL → 資訊塊」方向

## 研究範圍

本報告聚焦於「將專案倉庫（repository）中的文檔擷取、轉換為結構化資訊塊（chunk）以供 LLM / RAG 使用的 ETL 管線工具」，作為 Context7 的 FOSS 替代方案。與先前報告[^prev-report]不同，本次特別關注**完整的 ETL 流程**：clone／讀取倉庫 → 萃取文檔 → 分割 chunk → 向量嵌入（embedding）→ 儲存 → 透過 MCP 或 API 提供查詢。

## Context7 的 ETL 核心流程回顧

Context7 的封閉後端本質上執行以下管線：

1. **爬取** — 爬取程式庫註冊表（npm、PyPI、crates.io 等）與 GitHub 倉庫
2. **萃取** — 解析文檔頁面、README、API 參考文件
3. **分塊** — 將文檔分割為語意單元（chunk）
4. **嵌入** — 使用向量模型產生嵌入
5. **檢索** — 透過語意匹配注入 agent prompt

其 MCP 前端（MIT 授權）已開源，但上述 ETL 後端為私有[^context7-disclaimer]。

## FOSS 替代方案（完整 ETL 管線）

### 1. ContextMine ⭐17

| 屬性 | 內容 |
|------|------|
| 授權 | MIT |
| 倉庫 | mayflower/contextmine[^contextmine] |
| 語言 | 未公開（推測 Python/TypeScript） |
| 核心機制 | 自託管文檔與程式碼索引系統 + MCP 整合 |

**ETL 管線：**
- **來源** — Web 爬蟲（`spider_rs`）+ GitHub 倉庫索引，支援增量更新
- **萃取** — Tree-sitter 符號萃取（支援 Python、TS、JS、Go、Rust、Java、C、C++、Ruby、PHP），程式碼大綱、結構導航
- **分塊** — 內建混合檢索管線
- **嵌入** — BM25 全文檢索 + 向量相似度（RRF 排序），支援 OpenAI、Gemini embeddings
- **儲存** — PostgreSQL + pgvector + Apache AGE（圖資料庫能力）
- **提供** — MCP 協定工具：`get_markdown`、`search`、`outline`、`find_symbol`、`definition`、`references`、`deep_research`

**特色：** 深度研究 agent（多步驟 AI agent + LSP + Tree-sitter）、架構儀表板（C4 視圖）、嚴格真實度量（LOC／複雜度／耦合／覆蓋率）[^contextmine]。

> **Context7 相似度：★★★★★** — 目前最完整的 FOSS 替代方案，管線涵蓋所有 ETL 階段。

### 2. open-context7 ⭐9

| 屬性 | 內容 |
|------|------|
| 授權 | MIT |
| 倉庫 | rakuv3r/open-context7[^open-context7] |
| 語言 | Next.js + FastAPI + Qdrant |
| 核心機制 | 隱私優先的自託管 Context7 替代方案，Docker Compose 部署 |

**ETL 管線（三元件架構）：**
1. **Web UI**（Next.js）— 管理倉庫與設定
2. **FastAPI 後端** — 協調索引流程
3. **Qdrant 向量資料庫** — 儲存嵌入向量
4. **MCP 伺服器** — 提供文檔給 Cursor、Claude 等

**啟動方式：** `docker-compose up -d`，約 3 分鐘即可啟動。透過 `.env.dev` 設定。Nginx 反向代理[^open-context7]。

> **Context7 相似度：★★★★★** — 名稱與目標都直接指向 Context7 替代，完整 ETL 管線，但專案仍新。

### 3. ContextMCP

| 屬性 | 內容 |
|------|------|
| 授權 | 開源 |
| 網站 | contextmcp.ai[^contextmcp] |
| 部署 | Cloudflare Workers |
| 核心機制 | 專為 Context7 替代設計的自託管 MCP 伺服器 |

**ETL 管線：**
- **來源** — GitHub、GitLab、本機檔案或 URL
- **解析器** — MDX、Markdown、OpenAPI 或 HTML
- **分塊器** — 智慧內容分割，最佳化檢索
- **嵌入** — OpenAI、Gemini、Cohere、Voyage 或本機 Ollama
- **儲存** — Pinecone 向量資料庫
- **提供** — Cloudflare Workers（MCP + REST API）

**特色：** YAML 設定（無需改程式碼）、語意搜尋、邊緣部署、可擴展的解析器／分塊器架構。可搭配 ContextChat 作為「Ask AI」元件[^contextmcp]。

> **Context7 相似度：★★★★★** — 管線完整，且支援本機 Ollama 嵌入，邊緣部署速度快。

### 4. Context（by Neuledge）⭐401

| 屬性 | 內容 |
|------|------|
| 授權 | Apache 2.0 |
| 倉庫 | neuledge/context[^context] |
| 語言 | TypeScript |
| 核心機制 | 本機優先的 MCP 文檔伺服器，社群註冊表 100+ 套件 |

**ETL 管線：**
- **來源** — `context add <git-url>` clone 倉庫，自動偵測文檔資料夾（docs/、documentation/、doc/）
- **萃取** — 支援 Markdown、MDX、AsciiDoc、reStructuredText、HTML；使用 readability 萃取器（defuddle）
- **分塊** — 內建分塊（透過 SQLite FTS5）
- **嵌入** — ❌ 無向量嵌入（使用 BM25 全文檢索）
- **儲存** — SQLite 搭配 FTS5 全文檢索（約 1-5MB .db 檔案）
- **提供** — MCP 伺服器（stdio 或 HTTP），`get_docs` 工具，查詢延遲 <10ms

**特色：** 完全離線（下載後）、無速率限制、社群註冊表（100+ 套件）、可攜檔案分享團隊、支援私有倉庫、Token 上限控制（~2,000 tokens）[^context]。

> **Context7 相似度：★★★★☆** — 輕量、離線優先，但無向量嵌入，檢索品質依賴 FTS。

### 5. Context Engine ⭐48

| 屬性 | 內容 |
|------|------|
| 授權 | 開源 |
| 倉庫 | Kirachon/context-engine[^context-engine] |
| 核心機制 | 本機工作區索引 + 語意檢索 + MCP |

**ETL 管線：**
- **索引** — 本機工作區索引（自動偵測 git root，啟動時背景索引）
- **檢索** — 語意搜尋、程式碼審查、記憶操作
- **提供** — MCP 工具：計畫、執行、程式碼審查、記憶管理

**特色：** Agent 無關（支援 Codex、Claude、Cursor）、5 層整潔架構（索引 → 服務 → MCP → agents → 狀態）、Windows 支援[^context-engine]。

> **Context7 相似度：★★★☆☆** — 本機索引，但非遠端倉庫 ETL。

## 輕量替代方案（部分 ETL 或特定階段）

### 6. GitMCP ⭐8.4k

| 屬性 | 內容 |
|------|------|
| 授權 | Apache 2.0 |
| 倉庫 | idosal/git-mcp[^gitmcp] |
| 核心機制 | 零設定將 GitHub 倉庫變為 MCP 端點 |

**運作方式：** 直接讀取倉庫中的 `llms.txt`、`llms-full.txt`、`README.md`。提供 `fetch_documentation`、`search_documentation`、`search_code`、`fetch_url_content` 工具。只需將 `github.com` 改為 `gitmcp.io` 即可使用[^gitmcp]。

**限制：** 僅限公開倉庫、需要網路、無分塊／嵌入（原始文檔提取）。

> **Context7 相似度：★★★☆☆** — 無 ETL 管線，但零設定即可獲得倉庫文檔，適合簡單場景。

### 7. Repomix ⭐28.3k（原名 Repopack）

| 屬性 | 內容 |
|------|------|
| 授權 | MIT |
| 倉庫 | yamadashy/repomix[^repomix] |
| 核心機制 | 將整個倉庫打包為單一 AI 友好檔案 |

**特色：** Token 計數、gitignore／repomixignore 感知、可選 Tree-sitter 程式碼壓縮、Secretlint 安全掃描、MCP 整合。執行方式：`npx repomix`[^repomix]。

**限制：** 輸出單一檔案（XML／Markdown），非向量 chunk 管線。

> **Context7 相似度：★★☆☆☆** — 不同方向（打包 vs 即時注入），但可作為 ETL 的「萃取」階段元件。

## 基礎元件（可自建 ETL 管線）

### 8. Docling（IBM）⭐60.1k

| 屬性 | 內容 |
|------|------|
| 授權 | MIT |
| 倉庫 | docling[^docling] |
| 核心能力 | PDF、HTML、Office 文件 → Markdown 轉換 |

IBM 開發的文件理解引擎，Red Hat 稱其為「文件智慧第一名的開源倉庫」。可將 PDF 等複雜文件轉換為乾淨的 Markdown，供 LLM 管線使用[^docling]。

### 9. MarkItDown（Microsoft）

| 屬性 | 內容 |
|------|------|
| 授權 | MIT |
| 倉庫 | microsoft/markitdown[^markitdown] |
| 核心能力 | 多種檔案格式 → Markdown 轉換，附官方 MCP 伺服器 |

支援 HTML、PDF、Word、Excel、PowerPoint 等格式轉換為 Markdown。有官方 `markitdown-mcp` 套件，agent 可直接在 session 中轉換檔案[^markitdown]。

### 10. Unstructured ⭐15.2k

| 屬性 | 內容 |
|------|------|
| 授權 | Apache 2.0 |
| 倉庫 | Unstructured-IO/unstructured[^unstructured] |
| 核心能力 | 複雜文件 → 結構化資料的 ETL 解決方案 |

專為 LLM 設計的開源 ETL 解決方案，將複雜文件（PDF、HTML、Office 等）轉換為乾淨的結構化格式。可作為 Context7 管線中的**文件解析與分塊元件**[^unstructured]。

## 比較總結

| 方案 | Git 讀取 | 文檔萃取 | 分塊 | 向量嵌入 | 向量儲存 | MCP | ⭐ | 授權 |
|------|:--------:|:--------:|:---:|:--------:|:--------:|:---:|:-:|:----:|
| **ContextMine** | ✅ Git | ✅ Tree-sitter | ✅ | ✅ OpenAI/Gemini | ✅ pgvector | ✅ | 17 | MIT |
| **open-context7** | ✅ Git | ✅ | ✅ | ✅ | ✅ Qdrant | ✅ | 9 | MIT |
| **ContextMCP** | ✅ Git/GitLab | ✅ MDX/MD/HTML | ✅ | ✅ Ollama/Gemini/OpenAI | ✅ Pinecone | ✅ | - | 開源 |
| **Context (Neuledge)** | ✅ Git/URL | ✅ MD/MDX/AsciiDoc | ✅(FTS5) | ❌ BM25 | ✅ SQLite FTS5 | ✅ | 401 | Apache 2.0 |
| **GitMCP** | ✅ GitHub | ✅ llms.txt/README | ❌ | ❌ | ❌ | ✅ | 8.4k | Apache 2.0 |
| **Context Engine** | ✅ 本機 | ✅ | ✅ | ✅ 本機 | ✅ 本機 | ✅ | 48 | 開源 |
| **Repomix** | ✅ Git | ✅ | ❌(單檔) | ❌ | ❌ | ✅ | 28.3k | MIT |
| **Docling** | ❌ | ✅ PDF/Office→MD | ❌ | ❌ | ❌ | ❌ | 60.1k | MIT |
| **MarkItDown** | ❌ | ✅ 通用→MD | ❌ | ❌ | ❌ | ✅ | - | MIT |
| **Unstructured** | ❌ | ✅ 通用→結構化 | ✅ | ❌ | ❌ | ❌ | 15.2k | Apache 2.0 |

## 結論與建議

針對「ETL 專案倉庫文檔轉換為資訊塊」方向，以下推薦排序：

1. **ContextMine** — 目前 FOSS 生態中最完整的 ETL 管線，從 git clone、Tree-sitter 萃取、向量嵌入到 MCP 查詢一站式解決，但 ⭐17 仍屬早期專案
2. **open-context7** — 直接以 Docker Compose 封裝完整管線（FastAPI + Qdrant + MCP），是最直接的 Context7 替代，3 分鐘即可啟動
3. **Context（Neuledge）** ⭐401 — 最成熟的選項，本機優先、SQLite FTS5 低延遲、社群註冊表 100+ 套件，但無向量嵌入（BM25 在語意檢索上可能不如向量搜尋）
4. **自建組合** — Docling／MarkItDown（解析）+ Unstructured（分塊）+ Ollama（嵌入）+ Qdrant（儲存）+ 自訂 MCP 伺服器 → 完全可控但建置成本最高

若目標是**立即替代 Context7 的核心功能**，`open-context7` 提供最接近的開箱即用體驗；若需要**生產級穩定性與社群支援**，`Context（Neuledge）` 雖無向量嵌入，但成熟度與文件品質最佳；若需要**完整 ETL + 向量檢索**，`ContextMine` 是目前唯一涵蓋所有階段的專案。

---

[^prev-report]: 先前研究. (2026). Context7 的 FOSS 替代方案研究. *agent-zone/researches/00031_foss-alternatives-of-context7.md*. Retrieved 2026-09-13.
[^context7-disclaimer]: Upstash. (n.d.). context7 README. GitHub. Retrieved 2026-09-13, from https://github.com/upstash/context7
[^contextmine]: mayflower. (n.d.). contextmine. GitHub. Retrieved 2026-09-13, from https://github.com/mayflower/contextmine
[^open-context7]: rakuv3r. (n.d.). open-context7. GitHub. Retrieved 2026-09-13, from https://github.com/rakuv3r/open-context7
[^contextmcp]: ContextMCP. (n.d.). ContextMCP Docs. Retrieved 2026-09-13, from https://contextmcp.ai/docs
[^context]: Neuledge. (n.d.). context. GitHub. Retrieved 2026-09-13, from https://github.com/neuledge/context
[^context-engine]: Kirachon. (n.d.). context-engine. GitHub. Retrieved 2026-09-13, from https://github.com/Kirachon/context-engine
[^gitmcp]: idosal. (n.d.). git-mcp. GitHub. Retrieved 2026-09-13, from https://github.com/idosal/git-mcp
[^repomix]: yamadashy. (n.d.). repomix. GitHub. Retrieved 2026-09-13, from https://github.com/yamadashy/repomix
[^docling]: IBM. (n.d.). docling. GitHub. Retrieved 2026-09-13, from https://github.com/docling
[^markitdown]: Microsoft. (n.d.). markitdown. GitHub. Retrieved 2026-09-13, from https://github.com/microsoft/markitdown
[^unstructured]: Unstructured-IO. (n.d.). unstructured. GitHub. Retrieved 2026-09-13, from https://github.com/Unstructured-IO/unstructured