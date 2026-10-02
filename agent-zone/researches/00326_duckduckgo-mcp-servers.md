# DuckDuckGo 搜尋引擎 MCP 伺服器一覽

本報告整理目前（2026-09-30）可用的 DuckDuckGo MCP（Model Context Protocol）整合方案，包括社群熱門實作、付費託管服務、多功能搜尋引擎聚合器，以及研究型工作流程工具。

> **重要背景**：DuckDuckGo **並未提供官方 MCP 伺服器**。以下所有方案皆為第三方建置。DuckDuckGo 官方 Instant Answer API (`api.duckduckgo.com`) 回傳的是百科式摘要，**非排名網頁搜尋結果**。因此這些 MCP 伺服器多數透過 HTML 爬蟲或第三方後端實作。[^ddg-api]

---

## 1. 主要 DuckDuckGo MCP 方案

### 1.1. `nickclyde/duckduckgo-mcp-server`（⭐ 1.5k | PyPI 下載量 15.5k+）

| 項目 | 說明 |
|---|---|
| 語言 | Python |
| 安裝 | `uvx duckduckgo-mcp-server` |
| 授權 | MIT |
| 倉儲 | https://github.com/nickclyde/duckduckgo-mcp-server |

**功能特色**：DuckDuckGo 網頁搜尋、內容提取與解析、長網址縮短（移除追蹤 token）、速率限制（滑動視窗或 Token Bucket）、SafeSearch 設定、37 個區域代碼、TLS 代理支援、自動在 `httpx` 與 `curl_cffi` 間降級（處理 Cloudflare/TLS 指紋封鎖）、多種內容解析模式（text/main/markdown）、內容快取、Docker 支援。支援 stdio、SSE 與 Streamable HTTP 三種傳輸方式。[^nickclyde]

### 1.2. `HasData/duckduckgo-mcp`（付費託管）

| 項目 | 說明 |
|---|---|
| 語言 | TypeScript / Python |
| 安裝 | `npx @hasdata/duckduckgo-mcp` 或 `pip install hasdata-duckduckgo-mcp` |
| 授權 | MIT |
| 倉儲 | https://github.com/HasData/duckduckgo-mcp |

**功能特色**：遠端託管 MCP 伺服器，無需本機安裝。回傳結構化 JSON，包含排名有機搜尋結果、獨立廣告陣列、DuckDuckGo AI 答案（`searchAssist`）、37 個區域代碼。定價：每月 1,000 免費額度（100 次搜尋），付費方案 $59/月起。可合併多個搜尋引擎（如 `?apis=duckduckgo,google_serp,bing_serp`）。[^hasdata]

### 1.3. `Aas-ee/open-webSearch`（⭐ 1.8k | 多引擎聚合）

| 項目 | 說明 |
|---|---|
| 語言 | TypeScript |
| 安裝 | `npx open-websearch@latest` |
| 授權 | Apache 2.0 |
| 倉儲 | https://github.com/Aas-ee/open-webSearch |

**功能特色**：多引擎搜尋伺服器，DuckDuckGo 為其中一個後端（另含 Bing、Baidu、Brave、Exa、Startpage、Sogou、HackerNews、Juejin、CSDN）。無需 API 金鑰。提供 CLI、本地常駐程式與 MCP 伺服器三種使用模式。支援 HTTP 代理、Playwright 瀏覽器降級、多來源內容提取。[^open-websearch]

### 1.4. `zhsama/duckduckgo-mcp-server`（⭐ 87 | npm 下載量 1.9k+）

| 項目 | 說明 |
|---|---|
| 語言 | TypeScript |
| 安裝 | `npx -y duckduckgo-mcp-server` |
| 授權 | MIT |
| 倉儲 | https://github.com/zhsama/duckduckgo-mcp-server |

**功能特色**：DuckDuckGo 搜尋，可設定回傳數量（1-20）與 SafeSearch。輕量、無需 API 金鑰。速率限制 1 req/sec，每月 15,000 次。是 npm 上最早且最受歡迎的 DuckDuckGo MCP 之一。[^zhsama]

### 1.5. `OEvortex/ddg_search`（⭐ 42 | 內建 AI 回答）

| 項目 | 說明 |
|---|---|
| 語言 | TypeScript |
| 安裝 | `npx -y @oevortex/ddg_search@latest` |
| 授權 | Apache 2.0 |
| 倉儲 | https://github.com/OEvortex/ddg_search |

**功能特色**：單一工具 `web-search`，整合 DuckDuckGo 網頁搜尋 + IAsk AI 與 Monica AI 後端之 AI 回答。無需 API 金鑰。亦可作為 CLI 使用（`ddg` 指令）。支援環境變數代理設定。[^ddg-search]

### 1.6. `CyranoB/web-forager`（研究型工作流程）

| 項目 | 說明 |
|---|---|
| 語言 | Python |
| 安裝 | `uvx --python ">=3.10,<3.14" web-forager serve` |
| 授權 | MIT |
| 倉儲 | https://github.com/CyranoB/web-forager |

**功能特色**：MCP 伺服器 + **7 個 Agent Skills**（深度研究、事實查核、新聞監控、競爭情報、技術顧問、地緣政治分析、文章稽核）。底層使用 DuckDuckGo（`ddgs` 套件）搜尋，Jina Reader 內容提取。專為結構化、有引用、多來源的研究工作流程設計。無需 API 金鑰。[^web-forager]

---

## 2. 其他 DuckDuckGo MCP 伺服器

| 名稱 | 語言 | 特點 |
|---|---|---|
| `AstroCorp/MCP-DuckDuckGo` | TypeScript | 網頁搜尋 + 內容提取。GPL 3.0 |
| `artivus-labs/ddg-mcp` | TypeScript | 文字、圖片、新聞、影片搜尋 + AI 對話 |
| `Nipurn123/duckduckgo-mcp` | TypeScript | 免費、無限制、無需 API 金鑰 |
| `varlabz/duckduckgo-mcp` | TypeScript | 區域、安全搜尋、時間過濾、研究提示 |
| `sourav-spd/duckduckgo-mcp` | Python | DuckDuckGo 瀏覽器搜尋 |
| `T1ckbase/duckduckgo-mcp` | Bun/TypeScript | 最小化、快取、機器人偵測重試 |
| `joohyukjung/duckduckgo-mcp` | TypeScript | 網頁搜尋 + 內容解析 |
| `LLLeoLi/duckduckgo-mcp-server` | TypeScript | 搜尋 + 內容提取、速率限制 |
| `988664li-star/duckduckgo-mcp-server` | TypeScript | 搜尋 + 網頁內容提取 |
| `cploutarchou/duckduckgo-mcp-agent` | TypeScript | 最小化、SSE 串流、LM Studio 相容 |
| `shaheen2013/duckduckgo-search-mcp-server` | TypeScript | 使用 DuckDuckGo Instant Answer API |
| `zopalz/websearch_mcp` | Python | 網頁搜尋、圖片搜尋、下載 |
| `lucagioacchini/duckduckgo-mcp` | TypeScript | 網頁搜尋 + 內容提取 |
| `rsimd/duckduckgo-mcp-server` | TypeScript | 基本搜尋 |

---

## 3. 套件可用性

| 方案 | npm | PyPI |
|---|---|---|
| nickclyde/duckduckgo-mcp-server | — | `duckduckgo-mcp-server` |
| HasData/duckduckgo-mcp | `@hasdata/duckduckgo-mcp` | `hasdata-duckduckgo-mcp` |
| Aas-ee/open-webSearch | `open-websearch` | — |
| zhsama/duckduckgo-mcp-server | `duckduckgo-mcp-server` | — |
| OEvortex/ddg_search | `@oevortex/ddg_search` | — |
| CyranoB/web-forager | — | `web-forager` |
| T1ckbase/duckduckgo-mcp | `duckduckgo-mcp` | — |

---

## 4. 使用場景建議

| 場景 | 建議方案 |
|---|---|
| 最簡設定、立即使用 | `npx -y duckduckgo-mcp-server`（zhsama）或 `uvx duckduckgo-mcp-server`（nickclyde） |
| 生產環境/高流量/結構化資料 | HasData 託管方案 — 可靠、JSON 輸出、37 區域、付費（有免費額度） |
| 多引擎搜尋（DDG + Bing/Brave 等） | `Aas-ee/open-webSearch` — 11 個引擎、無需 API 金鑰 |
| 完整研究工作流程 | `CyranoB/web-forager` — 7 個 Agent Skills |
| 搜尋 + AI 回答 | `@oevortex/ddg_search` — DuckDuckGo 搜尋 + IAsk/Monica AI |
| 規避 Cloudflare/TLS 封鎖 | `nickclyde/duckduckgo-mcp-server` 搭配 `[browser]` extras（`curl_cffi` 後端） |

---

## 5. 注意事項

- 多數方案依賴 HTML 爬蟲，DuckDuckGo 可能變更頁面結構導致相容性問題
- 部分託管方案涉及付費，使用前請詳閱定價
- 速率限制與機器人偵測是常見課題，建議選用具降級機制的方案（如 nickclyde 版本）
- 各方案維護活躍度不一，選擇時請參考 GitHub 最近更新時間

---

[^ddg-api]: DuckDuckGo. (n.d.). DuckDuckGo Instant Answer API. Retrieved 2026-09-30, from https://duckduckgo.com/api
[^nickclyde]: nickclyde. (2024-2025). duckduckgo-mcp-server. https://github.com/nickclyde/duckduckgo-mcp-server. Retrieved 2026-09-30.
[^hasdata]: HasData. (2025). HasData DuckDuckGo MCP Server. https://github.com/HasData/duckduckgo-mcp. Retrieved 2026-09-30.
[^open-websearch]: Aas-ee. (2024-2025). open-webSearch. https://github.com/Aas-ee/open-webSearch. Retrieved 2026-09-30.
[^zhsama]: zhsama. (2024). duckduckgo-mcp-server. https://github.com/zhsama/duckduckgo-mcp-server. Retrieved 2026-09-30.
[^ddg-search]: OEvortex. (2025). ddg_search. https://github.com/OEvortex/ddg_search. Retrieved 2026-09-30.
[^web-forager]: CyranoB. (2025). web-forager. https://github.com/CyranoB/web-forager. Retrieved 2026-09-30.