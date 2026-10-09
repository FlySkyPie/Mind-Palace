# OpenAPI 規格轉換為 MCP 程式碼生成工具研究

## 研究目的

尋找能將 **OpenAPI/Swagger 檔案**（openapi.json）轉換為 **MCP（Model Context Protocol）Server** 的開放原始碼解決方案。核心需求是「讀取 openapi.json 檔案後生成 MCP 工具定義的程式碼」，而非動態代理（proxy）或直接將 API 封裝成 MCP endpoint 的 runtime 方案。

## 背景說明

MCP 讓 AI Agent 能透過定義好的工具（tools）與外部服務互動。將 OpenAPI 規格轉換為 MCP 工具時，傳統 proxy 方案會把整個 API 的所有 endpoint 都註冊為 MCP tools，導致 Agent 消耗大量 token。理想的方案是「靜態生成」— 根據 openapi.json 生成 MCP Server 程式碼，讓 Agent 只取用需要的部分，而非載入整個規格檔。

## 研究發現：靜態程式碼生成器

以下為真正**生成 MCP Server 程式碼**的解決方案（非 runtime proxy）：

### 1. openapi-mcp-generator（harsha-iiiv）⭐ 637 ★

GitHub：https://github.com/harsha-iiiv/openapi-mcp-generator

最成熟、最多星星的專案。讀取 OpenAPI 規格後**生成完整的 TypeScript MCP Server 專案**，包含 Zod 驗證、HTTP client、多種傳輸方式（stdio、SSE、StreamableHTTP、Cloudflare Workers）。

**核心特點**：
- 支援 OpenAPI 3.0+ 規格
- 生成完整 Node.js 專案（tsconfig.json、package.json、entry point）
- 支援 API Key、Bearer、Basic、OAuth2 等認證方式
- 可透過 `x-mcp` 擴充欄位過濾要生成的 endpoint
- 內建 Zod schema 做執行時期輸入驗證
- 支援 `--input openapi.json --output ./mcp-server` 參數

**安裝**：`npm install -g openapi-mcp-generator`

**注意**：雖然官方描述提到「proxy behavior」，但其本質仍是**先從 openapi.json 生成完整 TypeScript 程式碼**，再執行該程式碼作為 MCP Server。生成後的程式碼是獨立專案，可由開發者修改與擴充，而非每個請求都去讀取 openapi.json。

### 2. dspachos/openapi2mcp ⭐ 3

GitHub：https://github.com/dspachos/openapi2mcp

採用 **Code Mode 模式**— Agent 寫 JavaScript，Server 在 `isolated-vm` 沙箱中安全執行。整份 API 只透過 2 個工具（`search`、`execute`）暴露，token 用量固定在 ~1-2k，不隨 API 規模成長。

**核心特點**：
- Token 效率極高（適合大 API，約 0.5% context 佔用）
- 使用 `isolated-vm` 沙箱確保安全
- 支援 OAuth 2.1
- 可輸出 Docker 化 MCP Server
- 早期專案（WIP），不建議生產使用

**適合場景**：API 極大（上千個 endpoint），token 預算有限的場景。

### 3. agentic-community/openapi-to-mcp ⭐ 17

GitHub：https://github.com/agentic-community/openapi-to-mcp

**AI 驅動的轉換器**。使用 LLM（Claude via Bedrock/Anthropic）分析、評估 OpenAPI 規格品質後，再生成 Python MCP Server。

**核心特點**：
- AI 自動評估規格完整性、安全性、AI 友善度
- 生成完整 Python MCP Server + Client
- 可設定的品質門檻
- Apache-2.0 授權
- 使用 `uv` 套件管理

**注意**：需要 LLM API 存取權（Amazon Bedrock 或 Anthropic），轉換流程較複雜。

### 4. @prodbybuddha/openapi-mcp-server ⭐ 0

npm：`@prodbybuddha/openapi-mcp-server`
GitHub：https://github.com/ProdByBuddha/openapi-mcp-server

萬用型 OpenAPI→MCP 工具生成器。可生成 tools JSON、完整 Server 專案（`--generate-server` flag），同時提供程式化 API（`generateMcpTools()`）。

**核心特點**：
- CLI 與程式化 API 並存
- 支援多服務同時運行
- 內建安全處理機制
- 可過濾特定 endpoint

**使用**：`npm run mcp:gen:server -- --from-file openapi.json --generate-server ./out`

### 5. mcp-from-openapi（npm/PyPI 函式庫）

npm：https://www.npmjs.com/package/mcp-from-openapi
PyPI：`mcp-from-openapi`

這不是 CLI 工具，而是**將 OpenAPI 轉換為 MCP 工具定義的核心函式庫**。適合整合到自有 pipeline 中。

**核心特點**：
- TypeScript 與 Python 版本皆可用
- 參數衝突解析
- HTTP request mapper 生成
- 無 CLI，需寫程式呼叫

### 6. FastMCP.from_openapi()（FastMCP 框架內建）

文件：https://gofastmcp.com/integrations/openapi

FastMCP（Python，3000+ stars）內建 `FastMCP.from_openapi()` 方法，可在啟動時載入 OpenAPI 規格並動態生成 MCP 工具。雖然偏 runtime，但生成邏輯內建於框架，適合 FastMCP 使用者。

## 設計考量與比較

| 專案 | 語言 | Stars | 生成方式 | Token 效率 | 成熟度 |
|------|------|-------|---------|-----------|-------|
| openapi-mcp-generator | TypeScript | 637 | 完整專案生成 | 一般（一個 endpoint 一個 tool） | ✅ 高 |
| dspachos/openapi2mcp | TypeScript | 3 | Code Mode（2 tools） | ✅ 極高（~1-2k） | ⚠️ WIP |
| openapi-to-mcp | Python | 17 | AI 強化生成 | 一般 | ⚠️ 實驗性 |
| @prodbybuddha/openapi-mcp-server | TypeScript | 0 | 多種輸出格式 | 一般 | ⚠️ 新 |
| mcp-from-openapi | TypeScript/Python | — | 無 CLI，函式庫 | 視使用方式 | ✅ 穩定 |
| FastMCP.from_openapi() | Python | 3k+ | Runtime 動態生成 | 一般 | ✅ 成熟 |

## 結論與建議

1. **首選：openapi-mcp-generator**（637 ★）— 最成熟的靜態程式碼生成方案。執行 `openapi-mcp-generator --input openapi.json --output ./mcp-server` 即可獲得完整的 TypeScript MCP Server 專案，再自行選擇只部署需要的部分。

2. **Token 敏感場景：dspachos/openapi2mcp** — 如果 API 規模極大（如 AWS、GitHub API），Code Mode 模式的 2 個工具設計可大幅節省 token。

3. **需要整合進 pipeline：mcp-from-openapi** 函式庫 — 適合有自訂流程、需要程式化控制轉換邏輯的場景。

4. **使用 FastMCP 的專案** — 直接使用內建 `from_openapi()` 方法，無需額外依賴。

所有上述方案都是**從 openapi.json 檔案讀取規格**來生成 MCP Server，而非 proxy 方案需要在 runtime 代理請求到真實 API。

---

[^harsha]: harsha-iiiv. (n.d.). *openapi-mcp-generator*. GitHub. Retrieved 2026-10-09, from https://github.com/harsha-iiiv/openapi-mcp-generator
[^dspachos]: dspachos. (n.d.). *openapi2mcp*. GitHub. Retrieved 2026-10-09, from https://github.com/dspachos/openapi2mcp
[^agentic]: agentic-community. (n.d.). *openapi-to-mcp*. GitHub. Retrieved 2026-10-09, from https://github.com/agentic-community/openapi-to-mcp
[^prodbuddha]: ProdByBuddha. (n.d.). *openapi-mcp-server*. GitHub. Retrieved 2026-10-09, from https://github.com/ProdByBuddha/openapi-mcp-server
[^npmfrom]: *mcp-from-openapi*. (n.d.). npm. Retrieved 2026-10-09, from https://www.npmjs.com/package/mcp-from-openapi
[^fastmcp]: FastMCP. (n.d.). *OpenAPI Integration*. Retrieved 2026-10-09, from https://gofastmcp.com/integrations/openapi
[^ivoto]: ivo-toby. (n.d.). *mcp-openapi-server*. GitHub. Retrieved 2026-10-09, from https://github.com/ivo-toby/mcp-openapi-server