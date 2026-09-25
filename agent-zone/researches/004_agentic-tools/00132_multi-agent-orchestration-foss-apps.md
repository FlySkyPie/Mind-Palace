# 多代理協調 FOSS 應用程式市場調查

## 概述

本研究調查市面上符合以下條件的**開放原始碼多代理協調應用程式**：具備協調層（Orchestration Layer）、支援透過 ACP（Agent Communication/Control Protocol）或類似協定控制多個代理、**允許每個代理獨立設定各自的 MCP Server 與技能**，且為**可直接部署執行的應用程式**而非開發者函式庫。

排除純函式庫（如 LangChain、CrewAI、AutoGen）與僅為終端機多路復用工具（如 herdr）。最終鎖定 6 個符合條件的專案，按完整度排序。

---

## 1. SwarmClaw — 最完整的自託管控制面板

- **授權條款**：Apache-2.0
- **倉庫**：github.com/swarmclawai/swarmclaw
- **介面**：Web Dashboard + 桌面應用程式（macOS/Windows/Linux）+ REST API
- **語言**：未明確標示，推測為多語言堆疊

### 核心能力

**Web 儀表板**提供組織圖視覺化、代理對話面板與任務看板。管理者可透過「MCP Servers 面板」為每個代理單獨連接 stdio、SSE 或 streamable HTTP 的 MCP Server，並注入其工具至代理內建工具集中。[^swarmclaw-github]

**技能系統**支援執行期技能載入、OpenClaw 相容的 SKILL.md 匯入，以及「對話轉技能」功能——選取真實對話後可批准建立為新技能。[^swarmclaw-features]

支援 24+ LLM 提供商、代理記憶持久化、排程任務、心跳監控、Discord/Slack/Telegram/WhatsApp/Teams/Email 等連接器，以及結構化會話（Structured Sessions）——含模板、分支、迴圈與平行合併的可重複有界執行環境。[^swarmclaw-docs]

可透過 Docker、Render、Fly.io、Railway 自託管。

### 評析

SwarmClaw 是目前最接近「多代理協調層」需求的專案。其**每個代理可獨立設定 MCP、技能、記憶、LLM 提供商**的架構，加上完整的 Web 儀表板與組織圖視覺化，使其在管理複雜度與功能性上最全面。缺點是專案較新（~682 stars），生態系仍在成長中。

---

## 2. Synapse AI — 視覺化 DAG 代理管線

- **授權條款**：AGPL-3.0
- **網站**：synapseorch.com
- **倉庫**：github.com/synapseorch-ai/synapse-ai
- **介面**：視覺化 DAG 編輯器（ReactFlow Canvas）+ REST API

### 核心能力

提供**拖放式 DAG 工作流構建器**，可將代理、LLM 呼叫、工具、人工審核閘門串接成確定性管線。代理間可透過 MCP 用戶端（連接外部 MCP Server）與 MCP 服務端（內部工具以 MCP Server 運行）進行協作。[^synapse-docs]

每個代理在設定面板中可獨立設定系統提示詞、工具存取權限與模型覆蓋。支援 14+ LLM 提供商，管線中每個步驟可使用不同模型（如路由用低成本模型、推理用前沿模型）。[^synapse-features]

內建工具伺服器包括 Docker Python 沙箱、Vault、SQL Agent、瀏覽器（Playwright）、隱形網頁爬蟲、PDF/Excel 解析器等。支援人工介入閘門，可從 UI、Slack、Telegram、Teams 或 WhatsApp 恢復執行。

### 評析

Synapse AI 適合需要**確定性、可視覺化設計多代理工作流**的場景。其 DAG 架構讓每個代理的工具與模型設定完全獨立，AGPL-3.0 授權則限制了商業閉源使用。對不熟悉程式碼的操作者而言，視覺化編輯器是優勢。

---

## 3. AionUi — 桌面級多代理協作應用

- **授權條款**：Apache-2.0
- **倉庫**：github.com/iOfficeAI/AionUi（33k stars）
- **介面**：桌面應用（Electron 跨平台）+ WebUI + Telegram/Lark/WeChat 整合

### 核心能力

支援**並行執行 Claude Code、Codex、Gemini CLI、Hermes Agent 等 20+ CLI 代理**於同一統一介面。**MCP 統一管理**功能可集中管理多個 MCP 工具，並根據每個代理的能力自動注入或同步相容傳輸層。[^aionui-github]

**團隊模式（Team Mode）**：領導代理透過內建 Team MCP Server 將子任務委派給團隊成員代理，使用非同步信箱與共享任務看板進行平行執行。內建代理引擎無需外部 CLI 工具即可運作。

**三層技能系統**：內建技能、自訂技能與擴充技能。支援排程自動化（cron）、WebUI/手機遠端存取、30+ AI 平台（含 Ollama 本地模型）。

### 評析

AionUi 是**桌面端最成熟的選擇**，33k stars 顯示其社群活躍度極高。適合需要在單一桌面介面中並行執行與協調多個編碼代理的使用者。缺點是桌面應用侷限於單機操作，缺乏伺服器端集中管理能力。

---

## 4. Agent Control Plane (ACP) — Kubernetes 原生代理排程器

- **授權條款**：Apache-2.0
- **倉庫**：github.com/humanlayer/agentcontrolplane
- **語言**：Go（Kubernetes Operator）
- **介面**：Kubernetes CRD + API

### 核心能力

以 **Kubernetes Operator** 形式運作的代理協調層，將 LLM、Agent、MCP Server、Task、ContactChannel、ToolCall 定義為 Kubernetes CRD。支援**持久化代理執行**（檢查點/恢復）、子代理委派、人工審核（透過 Slack/Email）、OpenTelemetry 追蹤、完整 MCP 支援。[^acp-github]

子代理委派允許代理在執行中產生子任務，將子任務派給其他代理處理。人工審核機制可對 MCP 工具呼叫進行批准/拒絕。支援 OpenAI、Anthropic、Vertex、Mistral 等多個提供商。

### 評析

ACP 適合**已採用 Kubernetes 的團隊**，將代理管理融入現有基礎設施。其確定性執行語意與人工審核流程適用於需要嚴格管控的生產環境。缺點是需要 Kubernetes 叢集，非 K8s 環境部署成本較高。

---

## 5. AgentOven — 框架無關的企業級控制平面

- **授權條款**：Apache-2.0
- **倉庫**：github.com/agentoven/agentoven
- **語言**：Rust（CLI/SDKs）、Go（Control Plane）、Python/TypeScript SDKs
- **網站**：agentoven.dev

### 核心能力

標榜**「Kubernetes for AI agents」**，提供框架無關的代理控制平面，原生支援 **A2A**（Agent-to-Agent Protocol）與 **MCP**。功能涵蓋代理註冊表、模型路由器（含回退鍊）、DAG 工作流協調、RAG 管線（5 種策略）、提示詞管理、三層記憶系統（Pantry）、OpenTelemetry 可觀測性、成本追蹤、護欄與 RBAC。[^agentoven-github]

CLI 提供 55+ 指令，覆蓋代理生命週期管理。相容 LangChain、CrewAI、AutoGen、OpenAI SDK 等框架。支援全部 6 種協調模式。

### 評析

AgentOven 是**功能最全面的企業級方案**，但複雜度也最高。框架無關設計使其可整合既有代理，但也意味著較高的學習曲線。適合需要完整治理、可觀測性與成本控制的大型團隊。

---

## 6. ClawMatrix — 代理艦隊管理控制面板

- **授權條款**：MIT
- **倉庫**：github.com/somit/clawmatrix
- **語言**：Go
- **介面**：Web Dashboard + REST API + `/llms.txt` API

### 核心能力

提供**代理艦隊集中管理**：註冊代理、控制網路存取（每個代理獨立設定 egress 允許清單，透過 iptables 強制執行）、瀏覽/編輯代理工作區、排程 cron 任務、定義代理間委派連線（有向連線圖）。[^clawmatrix-github]

可作為 OpenClaw、PicoClaw 或任何自訂代理運行時的控制平面，透過 Sidecar 代理（Clutch）連接。支援 Let's Encrypt TLS 自動化憑證。

### 評析

ClawMatrix 與 herdr 最相似的類比物件——但 herdr 只是終端機多路復用器，ClawMatrix 則提供 Web 儀表板、排程、網路控制與代理間委派。MIT 授權是最自由的。缺點是專案極新（6 stars），成熟度有待驗證。

---

## 比較總表

| 應用程式 | 介面類型 | 每代理 MCP | 每代理技能/工具 | 協調模式 | 授權 |
|---|---|---|---|---|---|
| **SwarmClaw** | Web 儀表板 + 桌面應用 | ✅ 獨立 MCP 面板 | ✅ 技能 + 擴充 + 草稿 | 組織圖委派、結構化會話 | Apache-2.0 |
| **Synapse AI** | 視覺化 DAG + REST API | ✅ MCP 客戶端與服務端 | ✅ 內建工具伺服器 | DAG 管線、平行分支 | AGPL-3.0 |
| **AionUi** | 桌面應用 + WebUI | ✅ 統一 MCP 管理 | ✅ 21 內建 + 自訂 + 擴充 | 領導→成員團隊模式 | Apache-2.0 |
| **ACP** | Kubernetes CRD + API | ✅ 完整 MCP 支援 | ✅ 透過 CRD 設定 | K8s Operator、子代理委派 | Apache-2.0 |
| **AgentOven** | CLI + SDKs | ✅ 原生 MCP + A2A | ✅ 代理註冊表 | DAG 工作流、6 種協調模式 | Apache-2.0 |
| **ClawMatrix** | Web 儀表板 + API | ✅ 透通 MCP | ✅ 工作區管理 | 艦隊管理、有向委派 | MIT |

---

## 結論與建議

若需求為**「具有協調層、支援 ACP、可獨立控制每個代理的 MCP 與技能設定、FOSS 應用程式」**，則：

- **SwarmClaw** 是最全面的選擇——完整的 Web 儀表板、每代理 MCP 設定面板、技能系統與組織圖視覺化，Apache-2.0 授權也適合商業使用。
- **AionUi** 是桌面端最佳選擇，33k stars 的社群規模確保長期維護，適合個人開發者或小型團隊。
- **ACP（humanlayer/agentcontrolplane）** 適合已有 Kubernetes 基礎設施且需要確定性執行與人工審核流程的團隊。
- **AgentOven** 功能最全面但複雜度最高，適合大型企業。

上述專案均能在同一協調層下管理多個具有不同 MCP、技能與 LLM 設定的代理，符合原始查詢的所有條件。

[^swarmclaw-github]: SwarmClaw. (n.d.). SwarmClaw — Self-hosted AI agent runtime & dashboard. Retrieved 2026-09-25, from https://github.com/swarmclawai/swarmclaw
[^swarmclaw-features]: SwarmClaw. (n.d.). Features — MCP Servers, Skills, Structured Sessions. Retrieved 2026-09-25, from https://github.com/swarmclawai/swarmclaw
[^swarmclaw-docs]: SwarmClaw. (n.d.). Documentation — Deployment, Configuration, Connectors. Retrieved 2026-09-25, from https://github.com/swarmclawai/swarmclaw
[^synapse-docs]: Synapse Orchestrator. (n.d.). Synapse AI — Visual DAG-Based Multi-Agent Orchestration Platform. Retrieved 2026-09-25, from https://synapseorch.com
[^synapse-features]: Synapse Orchestrator. (n.d.). Documentation — Step Types, MCP Support, Providers. Retrieved 2026-09-25, from https://docs.synapseorch.com
[^aionui-github]: iOfficeAI. (n.d.). AionUi — Open-source multi-agent desktop app. Retrieved 2026-09-25, from https://github.com/iOfficeAI/AionUi
[^acp-github]: HumanLayer. (n.d.). Agent Control Plane — A distributed agent scheduler. Retrieved 2026-09-25, from https://github.com/humanlayer/agentcontrolplane
[^agentoven-github]: AgentOven. (n.d.). AgentOven — Framework-agnostic agent control plane. Retrieved 2026-09-25, from https://github.com/agentoven/agentoven
[^clawmatrix-github]: Somit. (n.d.). ClawMatrix — Fleet management for AI agents. Retrieved 2026-09-25, from https://github.com/somit/clawmatrix