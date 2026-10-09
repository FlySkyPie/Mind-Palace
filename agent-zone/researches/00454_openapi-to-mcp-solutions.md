# 將 OpenAPI 轉換為 MCP 的開放原始碼解決方案

## 背景

當 AI Agent 需要與 REST API 互動時，傳統方式是將完整的 OpenAPI 規格檔餵入 LLM 上下文。這對大型 API（如 Kubernetes、AWS）會造成大量 token 浪費、超出上下文限制、以及降低回應品質。本報告調查能將 OpenAPI 規格轉換為 MCP (Model Context Protocol) Server 的 FOSS 工具，重點在於 **讓 Agent 只擷取所需端點**，而非讀取整個檔案。

## 重點推薦工具

### 1. 🥇 ivo-toby/mcp-openapi-server ⭐298

- **GitHub**: https://github.com/ivo-toby/mcp-openapi-server
- **語言**: TypeScript (Node.js)
- **授權**: MIT
- **套件**: `@ivotoby/openapi-mcp-server` (npm)

**最符合需求的方案**。`--tools dynamic` 模式不將每個端點註冊為獨立 tool，而是僅暴露 3 個 meta-tool：`list-api-endpoints`、`get-api-endpoint-schema`、`invoke-api-endpoint`。Agent 透過這些 tool 動態探索和呼叫 API，**永遠不需載入完整規格**。

其他篩選方式：
- `--tools explicit` + `--tool <toolId>`：只暴露特定端點
- `--tag <tag>` / `--exclude-tag <tag>`：依 OpenAPI tag 過濾
- `--resource <path>`：依 URL 路徑前綴過濾
- `--operation <method>`：依 HTTP 方法過濾
- 支援動態 AuthProvider，適合有 token 時效的 API

---

### 2. 🥈 harsha-iiiv/openapi-mcp-generator ⭐637

- **GitHub**: https://github.com/harsha-iiiv/openapi-mcp-generator
- **語言**: TypeScript (Node.js)
- **授權**: MIT
- **套件**: `openapi-mcp-generator` (npm)

**靜態程式碼生成方案**。從 OpenAPI 3.0+ 規格生成完整的 TypeScript MCP Server 專案（含 Zod validation、型別定義、package.json）。

關鍵篩選機制：
- **`x-mcp` 擴充屬性**：在 OpenAPI 路徑/操作上標記 `x-mcp: true/false`，精準控制哪些端點變成 MCP tool
- **`--default-include false`**：預設不納入任何端點，只納入有 `x-mcp: true` 的端點
- **`getToolsFromOpenApi()` API**：支援 `excludeOperationIds`、`filterFn` 自訂過濾邏輯
- 支援 stdio、SSE、StreamableHTTP、Cloudflare Worker 四種傳輸層

---

### 3. 🥉 salacoste/openapi-mcp-swagger ⭐18

- **GitHub**: https://github.com/salacoste/openapi-mcp-swagger
- **語言**: Python
- **授權**: MIT
- **套件**: `swagger-mcp-server` (PyPI)

**搜尋式架構**。建立 Whoosh 全文搜尋索引 + SQLite，Agent 透過以下 MCP methods 精準查詢：
- `searchEndpoints(keywords, httpMethods)`：自然語言搜尋端點
- `getSchema(componentName)`：取得特定 schema
- `getExample(endpoint, format)`：取得特定端點的程式碼範例
- 支援大檔案（stream 式 JSON 解析，上限 10MB）
- 回應時間 < 200ms

適合大型規格（例如上千個端點的 API），搜尋取代枚舉。

---

### 4. Vizioz/Swagger-MCP ⭐168

- **GitHub**: https://github.com/Vizioz/Swagger-MCP
- **語言**: TypeScript (Node.js)
- **授權**: MIT

**LLM 驅動的逐步發現模式**。不將每個端點註冊為 tool，而是提供 `getSwaggerDefinition`、`listEndpoints`、`listEndpointModels`、`generateModelCode`、`generateEndpointToolCode` 等工具，讓 AI 自行探索規格，選擇要產生的端點。

---

## 其他工具一覽

| 工具 | 星星 | 語言 | 篩選載入方式 |
|------|------|------|-------------|
| agentic-community/openapi-to-mcp | 17 | Python | AI 評分後批次處理（計劃中） |
| sssdavid529/openapi-mcp-bridge | 2 | Python | `--include-tags` / `--exclude-tags` |
| tadata-org/openapi-mcp | - | Python | 零配置，支援 FastAPI 一鍵掛載 |
| rmasters/mcp-openapi | 8 | Python | 簡潔 CLI，尚無過濾功能（PoC） |
| openapi/mcp-server | 22 | Python | openapi.com 專用，非通用工具 |

## 結論

若要 **Agent 只擷取所需端點而不載入完整 OpenAPI 檔案**，前三推薦：

1. **ivo-toby/mcp-openapi-server** — `dynamic` 模式是唯一從設計上就不讓 Agent 看到完整規格的方案，最適合 token 敏感場景
2. **harsha-iiiv/openapi-mcp-generator** — 適合需要靜態生成、精準控制暴露範圍的生產環境
3. **salacoste/openapi-mcp-swagger** — 適合超大型 OpenAPI 規格（如 Kubernetes），搜尋式存取避免枚舉開銷

## 參考文獻

- ivo-toby. (n.d.). *mcp-openapi-server*. GitHub. Retrieved 2026-10-09, from https://github.com/ivo-toby/mcp-openapi-server [^ivo-toby]
- harsha-iiiv. (n.d.). *openapi-mcp-generator*. GitHub. Retrieved 2026-10-09, from https://github.com/harsha-iiiv/openapi-mcp-generator [^harsha]
- salacoste. (n.d.). *openapi-mcp-swagger*. GitHub. Retrieved 2026-10-09, from https://github.com/salacoste/openapi-mcp-swagger [^salacoste]
- Vizioz. (n.d.). *Swagger-MCP*. GitHub. Retrieved 2026-10-09, from https://github.com/Vizioz/Swagger-MCP [^vizioz]
- agentic-community. (n.d.). *openapi-to-mcp*. GitHub. Retrieved 2026-10-09, from https://github.com/agentic-community/openapi-to-mcp [^agentic]
- sssdavid529. (n.d.). *openapi-mcp-bridge*. GitHub. Retrieved 2026-10-09, from https://github.com/sssdavid529/openapi-mcp-bridge [^bridge]
- tadata-org. (n.d.). *openapi-mcp*. GitHub. Retrieved 2026-10-09, from https://github.com/tadata-org/openapi-mcp [^tadata]
- rmasters. (n.d.). *mcp-openapi*. GitHub. Retrieved 2026-10-09, from https://github.com/rmasters/mcp-openapi [^rmasters]
- Openapi. (n.d.). *mcp-server*. GitHub. Retrieved 2026-10-09, from https://github.com/openapi/mcp-server [^openapi]

[^ivo-toby]: ivo-toby. (n.d.). *mcp-openapi-server*. GitHub. Retrieved 2026-10-09, from https://github.com/ivo-toby/mcp-openapi-server
[^harsha]: harsha-iiiv. (n.d.). *openapi-mcp-generator*. GitHub. Retrieved 2026-10-09, from https://github.com/harsha-iiiv/openapi-mcp-generator
[^salacoste]: salacoste. (n.d.). *openapi-mcp-swagger*. GitHub. Retrieved 2026-10-09, from https://github.com/salacoste/openapi-mcp-swagger
[^vizioz]: Vizioz. (n.d.). *Swagger-MCP*. GitHub. Retrieved 2026-10-09, from https://github.com/Vizioz/Swagger-MCP
[^agentic]: agentic-community. (n.d.). *openapi-to-mcp*. GitHub. Retrieved 2026-10-09, from https://github.com/agentic-community/openapi-to-mcp
[^bridge]: sssdavid529. (n.d.). *openapi-mcp-bridge*. GitHub. Retrieved 2026-10-09, from https://github.com/sssdavid529/openapi-mcp-bridge
[^tadata]: tadata-org. (n.d.). *openapi-mcp*. GitHub. Retrieved 2026-10-09, from https://github.com/tadata-org/openapi-mcp
[^rmasters]: rmasters. (n.d.). *mcp-openapi*. GitHub. Retrieved 2026-10-09, from https://github.com/rmasters/mcp-openapi
[^openapi]: Openapi. (n.d.). *mcp-server*. GitHub. Retrieved 2026-10-09, from https://github.com/openapi/mcp-server