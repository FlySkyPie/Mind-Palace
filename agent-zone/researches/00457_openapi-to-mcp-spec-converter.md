# OpenAPI 規格轉 MCP 定義文件 — 解決方案調查

## 問題描述

當 Agent 需要與一個 API 互動時，傳統做法是讀取完整的 `openapi.json` 檔案。對於大型 API（如 Kubernetes、AWS），這份文件可能達數百 KB 甚至 MB，浪費寶貴的 context window。理想的解決方案是將 OpenAPI 規格**轉換為 MCP (Model Context Protocol) 格式的定義**，讓 Agent 可以透過 MCP 協定只 fetch 需要的 endpoint（工具／schema），而非一次讀完整份檔案。

**關鍵區別**：本調查尋找的是把 OpenAPI **文件描述本身**轉成 MCP 格式的工具，不是建立一個真正與 API 端點互動的 proxy 伺服器或 wrapper 伺服器。

---

## 解決方案比較

### 1. `mcp-from-openapi` — 純粹的 OpenAPI→MCP 定義轉譯器

- **GitHub**: [agentfront/mcp-from-openapi](https://github.com/agentfront/mcp-from-openapi)
- **Stars**: ⭐ 4
- **語言**: TypeScript
- **授權**: Apache 2.0
- **核心功能**：純粹的轉譯庫，將 OpenAPI 規格轉換為 MCP tool 定義物件（`name`, `inputSchema`, `outputSchema`, `mapper`, `metadata` 等）。不產生任何伺服器。[^mcp-from-openapi]

**最契合需求的理由**：

1. **純轉譯（No Server）**：`generateTools()` 回傳的是純粹的 MCP tool 定義 JSON 物件，不建立也不執行任何伺服器。使用者在 README 中被教導要自行手動註冊到 MCP SDK。[^mcp-from-openapi-arch]
2. **過濾機制**：支援 tag、method、path glob（`/admin/**`）、operationId 四種過濾方式，以及 `x-mcp` extension flag。Agent 可以只請求需要的工具。[^mcp-from-openapi-filter]
3. **Context Budget 分析**：`analyzeToolSet()` 可以估算每個 tool 的 token 成本，讓 Agent 在 context window 限制下決定載入哪些工具。[^mcp-from-openapi-budget]
4. **多來源輸入**：支援 URL、檔案、YAML 字串、JSON 物件輸入。[^mcp-from-openapi-input]
5. **Client 相容目標**：可指定 target 為 `claude` / `openai` / `gemini` / `strict`，產生的 schema 會自動適應該客戶端的限制。[^mcp-from-openapi-target]

**限制**：星星數低（4），Node.js >= 20.0.0，相依 zod@^4.0.0。

### 2. `openapi-mcp-generator` — 最受歡迎的選擇（637 ⭐），有程式化 API

- **GitHub**: [harsha-iiiv/openapi-mcp-generator](https://github.com/harsha-iiiv/openapi-mcp-generator)
- **Stars**: ⭐ 637
- **語言**: TypeScript
- **授權**: ISC
- **主要定位**：以 CLI 生成完整 MCP proxy 伺服器為主，但也提供程式化 API。[^openapi-mcp-generator]

**程式化 API `getToolsFromOpenApi()`**[^openapi-mcp-generator-programmatic]：

```typescript
import { getToolsFromOpenApi } from 'openapi-mcp-generator';

// 從檔案／URL 直接取得 MCP tool 定義
const tools = await getToolsFromOpenApi('./petstore.json');

// 進階選項
const filtered = await getToolsFromOpenApi(spec, {
  baseUrl: 'https://api.example.com',
  dereference: true,
  excludeOperationIds: ['deletePet'],
  filterFn: (tool) => tool.method.toLowerCase() === 'get',
});
```

每個回傳的 `McpToolDefinition` 包含 `name`、`description`、`inputSchema`、`method`、`pathTemplate`、`parameters`、`operationId` 等屬性。[^openapi-mcp-generator-props]

**優點**：社群驗證度高（637 ⭐），支援 `filterFn` 和 `x-mcp` extension 過濾，成熟度最高。

**限制**：本質上是 proxy 伺服器生成器，程式化 API 雖然可用但非主要設計目標。

### 3. `API-to-MCP-Generator` — Python 生態的選擇

- **PyPI**: [API-to-MCP-Generator](https://pypi.org/project/API-to-MCP-Generator/) (v0.2.2)
- **語言**: Python
- **核心功能**：`convert_spec_to_mcp(url)` 回傳 `MCPConversionResult`，包含 `server.tools`（MCPTool 物件列表）和 `server.resources`（MCPResource 物件列表）。[^api-to-mcp-generator]

**適合**：如果 Agent 生態是 Python-based，這是最簡單的路徑。

**限制**：專案較新、社群較小。

### 4. AWS OpenAPI MCP Server — 動態查詢支援的伺服器方案

- **GitHub**: [awslabs/mcp/servers/openapi-mcp-server](https://awslabs.github.io/mcp/servers/openapi-mcp-server)
- **語言**: Python
- **特色**：動態從 OpenAPI 規格建立 MCP 工具，支援 tag 過濾、多規格合併、豐富的工具描述。[^aws-openapi-mcp]

**雖然是伺服器方案**，但其動態查詢能力（只載入指定 tag 的工具）與「按需 fetch」的需求相符。若可接受一個輕量背景程序，這是最成熟的選擇。

---

## 推薦排名

| 排名 | 工具 | 推薦原因 | 適用場景 |
|------|------|---------|---------|
| 🥇 | **`mcp-from-openapi`** | 純轉譯、不產生伺服器、有過濾與 token 預算分析 | 需要純粹的規格轉換，不想要任何伺服層 |
| 🥈 | **`openapi-mcp-generator`**（程式化 API） | 637 ⭐ 高信任度、`getToolsFromOpenApi()` 可直接取定義 | 需要社群驗證且接受其程式化 API 的專案 |
| 🥉 | **`API-to-MCP-Generator`** | Python 原生、API 簡單直接 | Agent 為 Python 生態的專案 |
| 備選 | **AWS OpenAPI MCP Server**（動態模式） | 最成熟的動態載入方案 | 可接受背景伺服程序的場景 |

---

## 總結

如果你的目標是「將 OpenAPI 文件本身轉換為 MCP 格式的定義文件，讓 Agent 透過 MCP 協定按需 fetch 個別工具定義」，**`mcp-from-openapi`** 是最純粹的選擇——它只做規格轉換不做伺服器。如果專案需要更多社群支援與成熟度，**`openapi-mcp-generator`** 的程式化 API `getToolsFromOpenApi()` 是次佳選擇。

兩者都可以搭配 MCP 的 `resources/tools/list` 和 `tools/call` 機制實現：Agent 先列出可用工具名稱 → 只請求需要的工具定義 → 節省 context window。

---

[^mcp-from-openapi]: Agentfront. (n.d.). *mcp-from-openapi: Convert OpenAPI to MCP Tool Definitions*. GitHub. Retrieved 2026-09-10, from https://github.com/agentfront/mcp-from-openapi

[^mcp-from-openapi-arch]: Agentfront. (n.d.). *mcp-from-openapi — Architecture*. GitHub. Retrieved 2026-09-10, from https://github.com/agentfront/mcp-from-openapi#serving-with-the-official-mcp-sdk

[^mcp-from-openapi-filter]: Agentfront. (n.d.). *mcp-from-openapi — Filtering*. GitHub. Retrieved 2026-09-10, from https://github.com/agentfront/mcp-from-openapi#filtering

[^mcp-from-openapi-budget]: Agentfront. (n.d.). *mcp-from-openapi — Context-Budget Reports*. GitHub. Retrieved 2026-09-10, from https://github.com/agentfront/mcp-from-openapi#context-budget-reports

[^mcp-from-openapi-input]: Agentfront. (n.d.). *mcp-from-openapi — Multiple Input Sources*. GitHub. Retrieved 2026-09-10, from https://github.com/agentfront/mcp-from-openapi#loading-the-spec

[^mcp-from-openapi-target]: Agentfront. (n.d.). *mcp-from-openapi — Client Compatibility Targets*. GitHub. Retrieved 2026-09-10, from https://github.com/agentfront/mcp-from-openapi#target

[^openapi-mcp-generator]: harsha-iiiv. (n.d.). *openapi-mcp-generator*. GitHub. Retrieved 2026-09-10, from https://github.com/harsha-iiiv/openapi-mcp-generator

[^openapi-mcp-generator-programmatic]: harsha-iiiv. (n.d.). *openapi-mcp-generator — Programmatic API*. GitHub. Retrieved 2026-09-10, from https://github.com/harsha-iiiv/openapi-mcp-generator/blob/main/PROGRAMMATIC_API.md

[^openapi-mcp-generator-props]: harsha-iiiv. (n.d.). *openapi-mcp-generator — Return Type*. GitHub. Retrieved 2026-09-10, from https://github.com/harsha-iiiv/openapi-mcp-generator

[^api-to-mcp-generator]: PyPI. (n.d.). *API-to-MCP-Generator 0.2.2*. Retrieved 2026-09-10, from https://pypi.org/project/API-to-MCP-Generator/

[^aws-openapi-mcp]: AWS Labs. (n.d.). *OpenAPI MCP Server*. GitHub. Retrieved 2026-09-10, from https://awslabs.github.io/mcp/servers/openapi-mcp-server