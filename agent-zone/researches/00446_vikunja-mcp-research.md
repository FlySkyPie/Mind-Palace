# Vikunja MCP 生態系統研究報告

## 概觀

Vikunja（一套開放原始碼的代辦事項管理應用）擁有豐富的 MCP（Model Context Protocol）生態支援。從 **Vikunja 2.7.0** 開始，核心專案已內建原生 MCP 端點，此外社群也建構了至少 5 個以上的第三方 MCP 伺服器。

---

## 1. 官方內建 MCP 支援（Vikunja 核心）

| 項目 | 說明 |
|------|------|
| **說明** | 自 Vikunja 2.7.0 起，核心專案提供內建 MCP 端點，位於 `https://your-instance/api/v2/mcp`。使用者在 Settings → MCP 建立 MCP token，可選擇三種權限預設（唯讀、Typed tools 讀寫、完整存取）。使用 Streamable HTTP + bearer token 認證。 |
| **Url** | https://vikunja.io/help/mcp/ |
| **Repo** | https://github.com/go-vikunja/vikunja |
| **支援用戶端** | Claude Code、Codex、Claude Desktop、Mistral Vibe |
| **限制** | ChatGPT 尚不支援（需 MCP OAuth） |

[^builtin]: Vikunja. (n.d.). MCP support for AI tools. Retrieved 2026-10-09, from https://vikunja.io/help/mcp/

---

## 2. 社群 MCP 伺服器

### 2a. democratize-technology/vikunja-mcp（最受歡迎）

| 項目 | 說明 |
|------|------|
| **GitHub** | https://github.com/democratize-technology/vikunja-mcp |
| **Stars** | ⭐ 109（最高） |
| **語言** | TypeScript |
| **安裝** | `npx -y @democratize-technology/vikunja-mcp` |
| **說明** | 提供 42 個工具涵蓋 tasks、comments、labels、projects、teams、users、webhooks、batch import/export。支援 JWT + API token 認證、circuit breaker 容錯、Zod 輸入驗證、rate limiting。 |
| **支援用戶端** | Claude Desktop、Claude Code、OpenCode |

[^demo-tech]: democratize-technology. (n.d.). vikunja-mcp. Retrieved 2026-10-09, from https://github.com/democratize-technology/vikunja-mcp

### 2b. Wosh-i/vikunja-mcp

| 項目 | 說明 |
|------|------|
| **GitHub** | https://github.com/Wosh-i/vikunja-mcp |
| **Stars** | ⭐ 2 |
| **語言** | TypeScript |
| **安裝** | `npx -y vikunja-mcp` |
| **說明** | 完整 CRUD 支援 projects、tasks、comments、task relations、subtasks、attachments、assignees、labels。支援環境變數或 CLI 標記。 |

[^wosh-i]: Wosh-i. (n.d.). vikunja-mcp. Retrieved 2026-10-09, from https://github.com/Wosh-i/vikunja-mcp

### 2c. acidvegas/vikunja-mcp（AI 記憶焦點）

| 項目 | 說明 |
|------|------|
| **GitHub** | https://github.com/acidvegas/vikunja-mcp |
| **Stars** | ⭐ 4 |
| **語言** | Go（單一二進位檔） |
| **說明** | 將 Vikunja 轉變為 AI agent 的長期記憶與專案管理系統。projects = containers、tasks = memory entries、labels = namespaced tags、buckets = state machines。支援 **stdio** 和 **Streamable HTTP** 兩種傳輸方式。可作為 systemd 服務執行。 |
| **支援用戶端** | Claude Code、Claude Desktop、Cursor、Continue |

[^acidvegas]: acidvegas. (n.d.). vikunja-mcp. Retrieved 2026-10-09, from https://github.com/acidvegas/vikunja-mcp

### 2d. 13rac1/vikunja-mcp（Go 實作）

| 項目 | 說明 |
|------|------|
| **GitHub** | https://github.com/13rac1/vikunja-mcp |
| **Stars** | ⭐ 0 |
| **語言** | Go |
| **說明** | 完整 CRUD + **views/kanban buckets**、**power queries**（overdue_tasks、due_today、high_priority_tasks、focus_now、task_summary）、**batch operations**。支援 `vikunja://projects`、`vikunja://tasks/{id}` MCP 資源 URI。支援 stdio 和 HTTP 傳輸。 |

[^13rac1]: 13rac1. (n.d.). vikunja-mcp. Retrieved 2026-10-09, from https://github.com/13rac1/vikunja-mcp

### 2e. rmc-vikunja-mcp（Python/PyPI）

| 項目 | 說明 |
|------|------|
| **PyPI** | https://pypi.org/project/rmc-vikunja-mcp/ |
| **說明** | Python 實作的 MCP 伺服器，連接 LLM 助手與 Vikunja 任務管理平台。 |

[^rmc]: rmc-vikunja-mcp. (n.d.). PyPI. Retrieved 2026-10-09, from https://pypi.org/project/rmc-vikunja-mcp/

### 2f. FelixCoal/AI_Vikunja（AI 助手 + MCP）

| 項目 | 說明 |
|------|------|
| **GitHub** | https://github.com/FelixCoal/AI_Vikunja |
| **Stars** | ⭐ 0 |
| **語言** | Python（FastAPI） |
| **說明** | 一套小型 FastAPI 服務，將自然語言任務描述透過 Together AI 的 LLM 轉換為結構化 Vikunja 任務。**也包含自己的 MCP 伺服器**（`mcp_server.py`），提供單一 `add_task` 工具。包含 Docker Compose 設定。 |

[^felixcoal]: FelixCoal. (n.d.). AI_Vikunja. Retrieved 2026-10-09, from https://github.com/FelixCoal/AI_Vikunja

---

## 3. 其他目錄列出的 MCP 伺服器

以下伺服器在 MCP 目錄中有列表但無獨立 GitHub repo 或詳細資訊：

| 目錄 URL | 說明 |
|----------|------|
| https://mcp.pub/servers/vikunja/ | 提供 project 和 task 操作 |
| https://lobehub.com/mcp/0xk3vin-vikunja-mcp | 需 `VIKUNJA_URL` 和 `VIKUNJA_API_TOKEN` |
| https://lobehub.com/mcp/kuuhakudev-vikunja-mcp | Go 實作，提供 projects、tasks、labels |
| https://mcprepository.com/ugoi/vikunja-mcp | 基本任務管理 |
| https://mcprepository.com/ufna/vikunja-mcp | 支援設定檔 + `VIKUNJA_PROJECT_ID` |
| https://mcpservers.com/servers/lowlyocean-vikunja | 任務建立、取得、管理 |

---

## 4. 非 MCP 整合

Vikunja 官方整合頁面列出其他工具：n8n、Vja（CLI）、Cria（TUI）、mDone（iOS/macOS）、tui-do（Rust TUI，含離線支援）、Home Assistant 整合、Taskwarrior 遷移工具等。[^integrations]

[^integrations]: Vikunja. (n.d.). Integrations. Retrieved 2026-10-09, from https://vikunja.io/docs/integrations/

---

## 總結

Vikunja 擁有目前同類開源專案中最完善的 MCP 生態：

| # | 方案 | 語言 | Stars | 類型 | 特色 |
|---|------|------|-------|------|------|
| 1 | **Vikunja 內建 MCP** | Go（核心） | N/A | 官方 | 自 v2.7.0 起內建，無需額外伺服器 |
| 2 | **democratize-technology/vikunja-mcp** | TypeScript | ⭐ 109 | 社群 | 最受歡迎，42 tools，npm 套件 |
| 3 | **Wosh-i/vikunja-mcp** | TypeScript | ⭐ 2 | 社群 | 完整 CRUD，`npx vikunja-mcp` |
| 4 | **acidvegas/vikunja-mcp** | Go | ⭐ 4 | 社群 | AI agent 記憶/持久化 |
| 5 | **13rac1/vikunja-mcp** | Go | ⭐ 0 | 社群 | Power queries、kanban、batch ops |
| 6 | **FelixCoal/AI_Vikunja** | Python | ⭐ 0 | AI 助手 | LLM 自然語言任務建立 |
| 7 | **rmc-vikunja-mcp** | Python | N/A | 社群 | PyPI 套件 |

### 建議

- **最簡單**：直接使用 Vikunja 2.7.0+ 內建 MCP，在 Settings → MCP 建立 token，指向 Claude/Cursor 即可。
- **功能最豐富**：使用 `@democratize-technology/vikunja-mcp`（TypeScript，npm）。
- **AI agent 記憶**：使用 `acidvegas/vikunja-mcp`（Go，單一二進位檔，systemd 整合）。