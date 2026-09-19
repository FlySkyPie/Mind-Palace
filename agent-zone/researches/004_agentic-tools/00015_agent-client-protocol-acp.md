# Agent Client Protocol (ACP) 研究報告

> **撰寫日期：** 2026 年 9 月 12 日

---

## 目錄

1. [ACP 是什麼](#1-acp-是什麼)
2. [為什麼需要 ACP](#2-為什麼需要-acp)
3. [設計原理與架構](#3-設計原理與架構)
4. [ACP v2 的演進](#4-acp-v2-的演進)
5. [ACP 與 MCP 的關係](#5-acp-與-mcp-的關係)
6. [ACP 與其他協定的區別](#6-acp-與其他協定的區別)
7. [目前發展狀況](#7-目前發展狀況)
8. [總結](#8-總結)
9. [參考資料](#9-參考資料)

---

## 1. ACP 是什麼

**Agent Client Protocol (ACP)** 是一個開放的 **JSON-RPC 2.0** 通訊協定，由 **Zed Industries** 於 **2025 年 8 月** 提出，旨在標準化**程式碼編輯器 (IDE)** 與 **AI 編碼代理 (Coding Agent)** 之間的通訊。[^introduction]

ACP 的設計靈感來自 **LSP (Language Server Protocol)**——正如 LSP 標準化了編輯器與語言伺服器之間的通訊，ACP 標準化了編輯器與 AI 代理之間的通訊。[^introduction]

> **核心概念：** LSP 之於程式語言，如同 ACP 之於 AI Agent。[^comparison]

ACP 同時支援本地與遠端場景。本地 Agent 以子程序形式在編輯器中執行，透過 stdio 使用 JSON-RPC 通訊；遠端 Agent 可託管於雲端或獨立基礎設施，透過 HTTP 或 WebSocket 通訊——但遠端 Agent 的完整支援仍在開發中。[^architecture]

---

## 2. 為什麼需要 ACP

在 ACP 出現之前，AI 編碼工具的整合面臨以下關鍵問題：[^introduction]

- **整合開銷**：每對新的 Agent-Editor 組合都需要從頭打造自訂整合
- **相容性限制**：Agent 只能與少數編輯器搭配運作
- **供應商鎖定**：選擇一個 Agent 往往意味著接受它所支援的有限介面
- **N×M 整合噩夢**：N 個編輯器 × M 個 Agent，理論上需要 N×M 個自訂整合

ACP 將此轉變為 **N + M** 的模式：
- Agent 只要實作 ACP 就能在任何相容編輯器中運作
- 編輯器只要支援 ACP 就能使用整個生態系中的所有相容 Agent

ACP 也重用了 MCP 的 JSON 表示法，以避免常見資料類型需要另一套表示法；同時納入自訂類型來支援有用的 UX 元素（如顯示 diff）。使用者可讀文字預設使用 Markdown 格式。[^introduction]

---

## 3. 設計原理與架構

### 3.1 傳輸層與訊息格式

| 特性 | 說明 |
|------|------|
| **底層協定** | JSON-RPC 2.0 [^overview] |
| **傳輸方式** | 本地：stdin/stdout（行分隔 JSON）[^transports]<br>遠端：Streamable HTTP（草案討論中） |
| **角色** | **Client** = 程式碼編輯器 (IDE)<br>**Agent** = AI 編碼代理程序 |
| **版本協商** | 透過 `initialize` 階段的 `protocolVersion` 動態協商 |

### 3.2 工作階段生命週期（v1）

ACP v1 定義了清晰的工作流程：[^v1overview]

```
1. 初始化 (Initialization)
   └─ Client → Agent: initialize → 協商 protocolVersion 與能力
   └─ Client → Agent: authenticate（若需身分驗證）

2. 工作階段建立 (Session Setup)
   ├─ session/new   → 建立新的對話工作階段
   └─ session/load  → 載入已有的工作階段（需 Agent 支援）

3. 提示回合 (Prompt Turn)
   ├─ Client → Agent: session/prompt → 發送用戶提示
   ├─ Agent → Client: session/update → 串流回增量輸出
   ├─ Agent → Client: session/request_permission → 請求用戶授權
   └─ Agent → Client: 返回 session/prompt 回應（含 stopReason）

4. 取消與清理 (Cancellation & Cleanup)
   └─ session/cancel → 中斷進行中的操作
```

### 3.3 雙向通訊設計

ACP 的關鍵設計是**雙向性**——Agent 可以反過來向 Client 請求資源。在 v1 中，Agent 可透過 Client 的方法包括：[^v1overview]

- `fs/read_text_file`：讀取檔案（含編輯器中尚未儲存的內容）
- `fs/write_text_file`：寫入檔案
- `terminal/create`、`terminal/output` 等：啟動與管理終端命令
- `session/request_permission`：請求用戶授權執行敏感操作
- `elicitation/create`：向用戶請求結構化資訊輸入

這種設計使編輯器可以充當**守門員 (gatekeeper)**——Agent 無法直接存取檔案系統或執行命令，必須透過 Client 請求，而 Client 可以在過程中加入權限檢查與用戶確認。

**值得注意的是，在 v2 中，fs/ 與 terminal/ 相關的 Client 方法已被移除**，改為由 Agent 透過 Client 提供的 MCP 伺服器來處理這些操作。[^v2migration]

### 3.4 權限與安全性模型

- Agent 執行敏感操作前，可透過 `session/request_permission` 請求用戶許可[^v1toolcalls]
- 用戶可以選擇「允許一次」或「始終允許」，但「始終允許」應嚴格限定範圍
- 檔案路徑必須是絕對路徑，且應做路徑正則化以防止遍歷攻擊
- 雙向授權：ACP 的批准不等於對後端服務的授權，每個 MCP 呼叫仍需自己的身份驗證

### 3.5 工具呼叫分類

ACP 將工具呼叫分類為多種 `kind`：[^v1toolcalls]

- `read`：讀取檔案或資料
- `edit`：修改檔案或內容
- `delete`：移除檔案或資料
- `move`：移動或重新命名檔案
- `search`：搜尋資訊
- `execute`：執行命令或程式碼
- `think`：內部推理或規劃
- `fetch`：擷取外部資料
- `other`：其他類型（預設值）

Agent 透過 `session/update` 通知回報工具呼叫的狀態轉變：`pending` → `in_progress` → `completed` / `failed`

---

## 4. ACP v2 的演進

ACP v2 是一個整合性 (consolidation) 版本，主要改變包括：[^v2migration]

### 4.1 核心變更

1. **新的提示生命週期**：`session/prompt` 的回應不再終止回合，而是確認接收。前台進度與完成透過 `state_update` 通知傳遞，停止原因也移至此處。
2. **更新即 upsert**：訊息、工具呼叫與計劃透過 ID 進行增量更新。省略的欄位 = 不變，`null` = 清除，值 = 取代，chunks = 附加。
3. **Client 端 API 移除**：`fs/read_text_file`、`fs/write_text_file`、`terminal/*`、`session/set_mode` 均被移除。Agent 應使用 Client 提供的 MCP 伺服器來處理檔案與終端操作。
4. **能力重新組織**：統一的 `capabilities` + `info` 欄位，session 範圍的能力群組，物件支援標記取代布林值。
5. **前向相容性**：列舉與 tagged unions 接受未知值，`_` 前綴的值保留給實作自訂擴充。

### 4.2 v1 到 v2 方法對照

| v1 方法 | v2 對應 |
|---------|---------|
| `authenticate` | 更名為 `auth/login` |
| `logout` | 更名為 `auth/logout` |
| `session/load` | **移除**，改用 `session/resume` 搭配 `replayFrom` |
| `session/set_mode` | **移除**，改用 `session/set_config_option` |
| `fs/read_text_file`, `fs/write_text_file` | **移除** |
| `terminal/*` | **移除** |

### 4.3 版本協商策略

v1 與 v2 將長期共存。實作者應同時支援兩個版本：在 `initialize` 中協商版本，保留 v1 支援，並在 feature flag 後添加 v2。單一連線在 `initialize` 後只使用一個協商版本。[^v2migration]

### 4.4 架構設計原則

ACP 架構遵循以下核心原則：[^architecture]

1. **MCP 友善**：基於 JSON-RPC，重用 MCP 類型以減少整合者的負擔
2. **UX 優先**：專注於解決與 AI 代理互動的 UX 挑戰，確保足夠的靈活性來清晰呈現代理意圖
3. **信任基礎**：ACP 假設使用者在可信的編輯器中與可信的模型對話

編輯器可將使用者設定的 MCP 伺服器配置轉發給 Agent，使 Agent 能直接連接到 MCP 伺服器。編輯器自身也可透過小型代理將自己的 MCP 伺服器公開給 Agent。[^architecture]

---

## 5. ACP 與 MCP 的關係

**ACP 和 MCP 是互補而非競爭的協定**，它們作用於 AI 堆疊的不同層次：[^comparison]

| 維度 | ACP (Agent Client Protocol) | MCP (Model Context Protocol) |
|------|----------------------------|------------------------------|
| **主要關係** | 編輯器 (Client) ↔ 編碼 Agent | LLM 應用 (Host) ↔ 工具/資料伺服器 |
| **標準化內容** | 編輯器如何啟動、驅動、渲染 Agent 的工作階段 | LLM 應用如何發現並呼叫外部工具/資源 |
| **控制方向** | Client（編輯器）是整合面；Agent 插入其中 | Agent/Host 是客戶端；MCP Server 暴露能力 |
| **創建者** | Zed Industries（2025 年 8 月） | Anthropic（2024 年 11 月） |
| **典型用途** | 讓 Claude Code、Gemini CLI 等可在任何 IDE 中使用 | 給 Agent 存取資料庫、API、檔案系統等工具 |
| **傳輸方式** | JSON-RPC over stdio/HTTP | JSON-RPC over stdio/SSE |

在實際應用中，它們會堆疊在一起：[^architecture]

```
開發者 ↔ IDE (ACP Client) ↔ 編碼 Agent (ACP Agent) ↔ MCP Server ↔ 資料/工具
```

具體流程：
1. IDE 透過 **ACP** 啟動 Agent 並發送提示
2. Agent 處理提示時，透過 **MCP** 呼叫外部工具（資料庫、API 等）
3. Agent 將結果透過 **ACP** 串流回 IDE 顯示

---

## 6. ACP 與其他協定的區別

| 協定 | 主要關係 | 創建者 | 說明 |
|------|---------|--------|------|
| **ACP** | 編輯器 ↔ 編碼 Agent | Zed Industries | 標準化 IDE 與 AI 編碼代理的通訊 |
| **MCP** | AI ↔ 工具/資料 | Anthropic | 標準化 AI 模型與外部工具的連接 |
| **A2A** | Agent ↔ Agent | Google | 標準化自主 Agent 之間的同級通訊與任務委派 |
| **ANP** | Agent ↔ Agent (去中心化) | 社群 | 基於 W3C DID 的去中心化 Agent 網路協定 |
| **LSP** | 編輯器 ↔ 語言伺服器 | Microsoft | 標準化編輯器與語言智慧功能的通訊 |

> **注意：** IBM 也曾提出名為 Agent Communication Protocol（也稱 ACP）的協定，但其目標是標準化 **Agent 與 Agent 之間** 的通訊，已於 2025 年 8 月合併至 Google 的 A2A 協定。這與本文討論的 Zed 的 Agent Client Protocol 是完全不同的協定。[^comparison]

---

## 7. 目前發展狀況

### 7.1 版本狀態

| 項目 | 狀態 |
|------|------|
| **穩定協定版本** | v1（Production）[^github] |
| **下一版本** | v2（草案中）[^v2overview] |
| **GitHub 星數** | 4,200+（主倉庫）；384（Registry 倉庫）[^github][^registry] |
| **授權** | Apache License 2.0 [^github] |
| **Registry 註冊 Agent** | 46+ 個已註冊 Agent，支援身分驗證[^registry] |

### 7.2 編輯器採用情況

| 編輯器 | 支援狀態 |
|--------|---------|
| **Zed** | ✅ 原生支援（創始者） |
| **JetBrains** (IntelliJ, PyCharm, WebStorm, GoLand 等) | ✅ 原生支援，專屬 Registry 索引[^registry] |
| **Neovim** | ✅ 社群插件 (CodeCompanion.nvim, avante.nvim, agentic.nvim, hermes.nvim) |
| **Emacs** | ✅ 社群插件 (agent-shell.el, acp.el) |
| **VS Code** | ✅ 多個擴充功能 (ACP Client, ACP Patchbay, ACP Pro, Multicoder, Poolside Assistant)[^clients] |
| **Cursor** | ⚠️ 支援 ACP Agent 但不完全 |
| **Sublime Text** | ✅ 社群插件[^clients] |
| **Qt Creator** | ✅ 官方 ACP Client Plugin[^clients] |
| **Pulsar** | ✅ 社群套件[^clients] |
| **Visual Studio** | ✅ Poolside Assistant 擴充功能[^clients] |
| **Unity** | ✅ UnityACP Client 與 UnityAgent Client[^clients] |
| **Obsidian** | ✅ 多個外掛（Agent Client, Agent Console, Copilot for Obsidian, Obsidian Harness）[^clients] |
| **Anycode** | ✅ 網頁版 IDE[^clients] |
| **Chrome ACP** | ✅ Chrome 擴充功能/PWA[^clients] |

### 7.3 支援 ACP 的 Agent

截至 2026 年 9 月，官方文件列出 **40+ 個** Agent，包括：[^agents]

| Agent | 開發者 |
|-------|--------|
| AgentPool | 社群 |
| Augment Code | Augment |
| AutoDev | 社群 |
| Blackbox AI | Blackbox |
| Claude Agent | Anthropic（透過 Zed SDK 轉接器） |
| Cline | 社群 |
| Codex CLI | OpenAI（透過官方轉接器） |
| Cursor | Cursor |
| Docker cagent | Docker |
| fast-agent | 社群 |
| Factory Droid | Factory |
| fount | 社群 |
| Gemini CLI | Google |
| GitHub Copilot | GitHub（公開預覽） |
| Goose | Block（社群） |
| Junie | JetBrains |
| Kimi CLI | MoonshotAI |
| Kiro CLI | kiro.dev |
| Mistral Vibe | Mistral |
| OpenCode | 社群 (SST) |
| OpenHands | 社群 |
| Poolside | Poolside |
| Qoder CLI | Qoder |
| Qwen Code | Alibaba |
| 及其他 15+ 個 Agent |

### 7.4 Desktop、Web 與 CLI 用戶端

ACP 生態系不僅包含傳統編輯器，還擴展到：[^clients]

- **Desktop 應用**：Braide、Capsule、Codeg、CompozyOS、Devin Desktop、DeepChat、Gold Band、Kepler、Lody、Panda 等
- **Web 應用**：ACP UI、aizen、AgentRQ、AgentConnect、Casper、Ghosty Teams 等
- **CLI/TUI**：acpx、Hash、Hydra、Martty、Nori CLI、Toad 等

### 7.5 官方 SDK 支援

| 語言 | 套件名稱 | 狀態 |
|------|---------|------|
| Rust | `agent-client-protocol` (crates.io) | Production [^github] |
| TypeScript | `@agentclientprotocol/sdk` (npm) | Production |
| Python | `python-sdk` (PyPI) | Production |
| Java | `java-sdk` (Maven Central) | Production |
| Kotlin | `acp-kotlin` (Maven Central) | Production（支援 JVM，其他平台進行中）[^github] |

### 7.6 ACP Registry

ACP Registry 是一個在編輯器內建的 Agent 目錄，使用者可直接在 IDE 中瀏覽、發現並安裝相容的 Agent。Registry 只收錄支援身分驗證的 Agent，並透過 CI 驗證所有 Agent 是否能回傳有效的 `authMethods`。[^registry]

Registry 使用 CDN 分發索引：
- 標準索引：`https://cdn.agentclientprotocol.com/registry/v1/latest/registry.json`
- JetBrains 專屬索引：含專用與預覽頻道

### 7.7 尚在發展中的功能

- **v2 協定穩定化**：v2 目前仍為草案，需在 feature flag 後使用[^v2migration]
- **Streamable HTTP 傳輸**：遠端 Agent 支援仍在討論中[^transports]
- **Registry 品質訊號**：目前缺少使用統計、評論和相容性標記
- **Microsoft 原生 VS Code 支援**：仍在協商中

---

## 8. 總結

**ACP = LSP 之於 AI Agent。**[^comparison]

ACP 讓開發者可以：

1. **在同一個編輯器中使用多個 AI Agent**（Claude Code、Gemini CLI、Codex CLI 等）
2. **同一個 AI Agent 在任意編輯器中運作**（Zed、JetBrains、Neovim、VS Code 等）
3. **透過一份 JSON 設定檔搞定所有整合**

ACP 是當前 AI 編碼工具生態系中**最實務導向、最快速獲得採用**的協定之一，尤其適合需要跨編輯器、跨 Agent 工作的開發者。從 2025 年 8 月提出至今僅約一年，已發展出包含 40+ Agent、20+ 用戶端、46+ Registry 註冊、5 種官方 SDK 的完整生態系。

v2 的推出顯示協定仍在積極演進中，移除 v1 中過於緊耦合的 Client 端 API（檔案系統、終端），轉而透過 MCP 生態系來處理這些功能，體現了 ACP 與 MCP 明確分工、協同運作的設計哲學。

---

## 9. 參考資料

[^introduction]: Zed Industries. (2026). ACP Introduction. Retrieved 2026-09-12, from https://agentclientprotocol.com/get-started/introduction
[^architecture]: Zed Industries. (2026). ACP Architecture. Retrieved 2026-09-12, from https://agentclientprotocol.com/get-started/architecture
[^overview]: Zed Industries. (2026). ACP v1 Overview. Retrieved 2026-09-12, from https://agentclientprotocol.com/protocol/v1/overview
[^v1overview]: Zed Industries. (2026). ACP v1 Overview - Protocol. Retrieved 2026-09-12, from https://agentclientprotocol.com/protocol/v1/overview
[^v2overview]: Zed Industries. (2026). ACP v2 Overview. Retrieved 2026-09-12, from https://agentclientprotocol.com/protocol/v2/overview
[^v2migration]: Zed Industries. (2026). Migrating from v1. Retrieved 2026-09-12, from https://agentclientprotocol.com/protocol/v2/migration
[^transports]: Zed Industries. (2026). ACP Transports. Retrieved 2026-09-12, from https://agentclientprotocol.com/protocol/v1/transports
[^v1toolcalls]: Zed Industries. (2026). ACP Tool Calls. Retrieved 2026-09-12, from https://agentclientprotocol.com/protocol/v1/tool-calls
[^agents]: Zed Industries. (2026). ACP Agents. Retrieved 2026-09-12, from https://agentclientprotocol.com/get-started/agents
[^clients]: Zed Industries. (2026). ACP Clients. Retrieved 2026-09-12, from https://agentclientprotocol.com/get-started/clients
[^registry]: Zed Industries. (2026). ACP Registry. Retrieved 2026-09-12, from https://agentclientprotocol.com/get-started/registry
[^github]: agentclientprotocol. (2026). Agent Client Protocol Repository. Retrieved 2026-09-12, from https://github.com/agentclientprotocol/agent-client-protocol
[^comparison]: Jitendra Zaa. (2025). MCP vs A2A vs ACP vs ANP: Complete AI Agent Protocol Guide. Retrieved 2026-09-12, from https://www.jitendrazaa.com/blog/ai/mcp-vs-a2a-vs-acp-vs-anp-complete-ai-agent-protocol-guide/