# 多代理協調框架調查報告：支援 ACP/A2A、MCP 異質性的開源工具

## 概述

本報告調查市面上具備協調層（Orchestration Layer）、支援代理間通訊協定（ACP/A2A）、允許不同代理搭載不同 MCP（Model Context Protocol）工具集與技能（Skill）的開源多代理框架。調查範圍專注於 FOSS（自由及開源軟體）方案。

## 背景名詞說明

- **ACP（Agent Communication Protocol）**：由 i-am-bee 提出的代理間通訊協定，定義了代理如何發現、互動與協作。目前已與 Google 的 A2A 協定合併，由 Linux Foundation 共同推動。[^acp]
- **A2A（Agent-to-Agent Protocol）**：Google 提出的代理間通訊開放標準，現已與 ACP 合併，成為 Linux Foundation 下的統一協定。[^a2a]
- **MCP（Model Context Protocol）**：由 Anthropic 提出的開放協定，定義 AI 模型如何與外部工具、資料來源、服務進行互動的標準化介面。[^mcp]
- **Orchestration Layer**：協調層，負責管理多個代理之間的工作流程、任務分配、狀態同步與生命週期管理。

## 研究發現

### 1. Microsoft Agent Framework (MAF)

**概述**：微軟最新的統一企業級代理框架，整合了 AutoGen 的多代理模式與 Semantic Kernel 的生產級基礎設施，支援 Python、.NET 與 Go。[^maf]

| 面向 | 狀態 |
|---|---|
| 授權 | MIT |
| 協調層 | ✅ 內建圖形化工作流（循序、並行、交接、群組協作） |
| ACP/A2A | ✅ 原生支援 A2A 協定，跨執行時期互通 |
| MCP 異質性 | ✅ 每個代理可獨立設定 MCP 伺服器與工具組 |
| 其他特色 | 中介軟體、持久化執行、人機協作、OpenTelemetry 可觀測性、檢查點/時光回溯、宣告式代理（YAML） |

**備註**：AutoGen（MIT，61.1k stars）已進入維護模式，現有用戶被鼓勵遷移至 MAF。AutoGen 本身不支援原生 A2A/ACP。[^autogen]

### 2. CrewAI

**概述**：高效能 Python 框架，提供角色扮演式自主代理協作。高層抽象為 Crews（自主代理團隊），低層控制為 Flows（事件驅動工作流）。[^crewai]

| 面向 | 狀態 |
|---|---|
| 授權 | MIT |
| 協調層 | ✅ Crews（角色協作）+ Flows（事件驅動工作流） |
| ACP/A2A | ✅ 原生 A2A 支援，代理可互相委派任務、請求資訊，或作為 A2A 伺服器 |
| MCP 異質性 | ✅ 一級支援，每個代理可設定不同 MCP 伺服器、工具與 LLM |
| 其他特色 | 順序/階層式流程、記憶/知識、人機協作、非同步執行、檢查點 |

### 3. LangGraph（LangChain）

**概述**：低階協調框架，用於構建長期運行、有狀態的代理系統。提供圖形化控制流（節點與邊），支援持久化執行。[^langgraph]

| 面向 | 狀態 |
|---|---|
| 授權 | MIT |
| 協調層 | ✅ 圖形化工作流，支援多重代理模式（監督者、階層式、代理即工具） |
| ACP/A2A | ⚠️ 非原生，但可透過自訂模式（如結構化訊息匯流排）實作 ACP 風格通訊 |
| MCP 異質性 | ✅ 透過 LangChain 的 MCP 整合，每個代理節點可有不同工具配置 |
| 其他特色 | 持久化執行（容錯、可回復）、完整記憶系統、人機協作（中斷）、LangSmith 除錯 |

### 4. OpenAI Agents SDK

**概述**：輕量級生產框架，OpenAI Swarm 的繼任者，提供代理交接、守衛閘道、MCP 工具支援，且為供應商中立。[^openai_sdk]

| 面向 | 狀態 |
|---|---|
| 授權 | MIT |
| 協調層 | ✅ 支援 LLM 驅動決策（交接）與程式碼驅動流程 |
| ACP/A2A | ❌ 使用自有交接機制，非 A2A/ACP |
| MCP 異質性 | ✅ 不同代理可設定不同 MCP 工具集 |
| 其他特色 | 守衛閘道（輸入/輸出驗證）、人機協作、沙箱代理（容器化執行）、即時/語音代理、100+ LLM |

### 5. Agno（原 Phidata）

**概述**：高效能 Python 框架與執行時期（AgentOS），內建記憶、知識庫、工具整合與 Web UI 控制面板。[^agno]

| 面向 | 狀態 |
|---|---|
| 授權 | Apache 2.0 |
| 協調層 | ✅ Agent Teams 多代理協作 + Workflows |
| ACP/A2A | ⚠️ 部分支援：可透過 A2A 暴露代理 |
| MCP 異質性 | ✅ MCP 一級支援，不同代理可有不同配置 |
| 其他特色 | 100+ 工具整合、人機核准、OpenTelemetry、JWT RBAC、AgentOS UI |

### 6. Google ADK（Agent Development Kit）

**概述**：Google 開源的程式碼優先框架，用於構建、評估與部署 AI 代理。原生多代理系統支援，為 A2A 協定的參考實作。[^google_adk]

| 面向 | 狀態 |
|---|---|
| 授權 | Apache 2.0 |
| 協調層 | ✅ 圖形化多代理工作流 |
| ACP/A2A | ✅ 原生 A2A 支援，ADK 是 A2A 協定的官方參考實作 |
| MCP 異質性 | ✅ 原生雙向 MCP 支援，不同代理可有不同 MCP 工具配置 |
| 其他特色 | 供應商中立（對 Gemini 最佳化但可接任何 LLM）、評估框架、代理卡片（能力探索）、多語言（Python、TS、Go、Java、Kotlin） |

## 綜合比較表

| 框架 | 授權 | 協調層 | ACP/A2A | MCP 異質性 | 語言 | GitHub Stars |
|---|---|---|---|---|---|---|
| **Microsoft Agent Framework** | MIT | ✅ 原生圖形化 | ✅ 原生 A2A | ✅ 每個代理獨立 | Python, .NET, Go | ~13.8k |
| **AutoGen**（維護模式） | MIT | ✅ 群組對話 | ❌ | ✅ MCP 支援 | Python, .NET | ~61.1k |
| **Semantic Kernel**（→ MAF） | MIT | ✅ 群組對話 | ❌ | ✅ 外掛/MCP | Python, .NET, Java | ~28.6k |
| **CrewAI** | MIT | ✅ Crews + Flows | ✅ 原生 A2A | ✅ 每個代理獨立 | Python | ~59.0k |
| **LangGraph** | MIT | ✅ 圖形化 | ⚠️ 自訂實作 | ✅ 透過 LangChain | Python, JS/TS | ~42.2k |
| **OpenAI Agents SDK** | MIT | ✅ 交接 + 程式碼 | ❌ 自有機制 | ✅ MCP 工具 | Python, JS/TS | ~29.7k |
| **Agno** | Apache 2.0 | ✅ Agent Teams | ⚠️ 部分 A2A | ✅ MCP 支援 | Python | ~42.3k |
| **Google ADK** | Apache 2.0 | ✅ 圖形工作流 | ✅ 原生 A2A | ✅ 原生 MCP | 多語言 | (新專案) |

## 結論與建議

### 前三名推薦

1. **CrewAI** — 若你需要快速上手的**高層次抽象**、角色扮演式代理協作，且要求完整的 A2A 與 MCP 異質性支援。授權 MIT，社群成熟（~59k stars）。
2. **Google ADK** — 若你重視**協定標準**（A2A 官方參考實作）、多語言支援、且 Google Cloud 生態對你有利。授權 Apache 2.0。
3. **Microsoft Agent Framework** — 若你需要**企業級生產部署**、複雜圖形化工作流、持久化執行與檢查點機制。授權 MIT。

### 選型考量

- **最純粹的開源**：以上除了 Amazon Bedrock（專有服務）外，全部為 MIT 或 Apache 2.0 授權。
- **ACP 轉向 A2A**：ACP 已與 A2A 合併至 Linux Foundation，未來選型時應以 A2A 相容性為標準。
- **MCP 異質性**：所有主要框架均已支援 MCP，差異在於整合深度（CrewAI、ADK 最深入，LangGraph 需透過 LangChain）。
- **低階控制 vs 高層抽象**：LangGraph 提供最大靈活度但需較多自訂程式碼；CrewAI 提供最多開箱即用的抽象。

[^acp]: i-am-bee. (n.d.). Agent Communication Protocol. Retrieved 2026-09-25, from https://github.com/i-am-bee/ACP
[^a2a]: Google & Linux Foundation. (n.d.). Agent-to-Agent Protocol (A2A). Retrieved 2026-09-25, from https://github.com/a2aproject/A2A
[^mcp]: Anthropic. (n.d.). Model Context Protocol. Retrieved 2026-09-25, from https://modelcontextprotocol.io
[^maf]: Microsoft. (n.d.). Microsoft Agent Framework. Retrieved 2026-09-25, from https://github.com/microsoft/agent-framework
[^autogen]: Microsoft. (n.d.). AutoGen. Retrieved 2026-09-25, from https://github.com/microsoft/autogen
[^crewai]: crewAI Inc. (n.d.). CrewAI. Retrieved 2026-09-25, from https://github.com/crewAIInc/crewAI
[^langgraph]: LangChain AI. (n.d.). LangGraph. Retrieved 2026-09-25, from https://github.com/langchain-ai/langgraph
[^openai_sdk]: OpenAI. (n.d.). OpenAI Agents SDK. Retrieved 2026-09-25, from https://github.com/openai/openai-agents-python
[^agno]: Agno AGI. (n.d.). Agno. Retrieved 2026-09-25, from https://github.com/agno-agi/agno
[^google_adk]: Google. (n.d.). Agent Development Kit (ADK). Retrieved 2026-09-25, from https://github.com/google/adk-python