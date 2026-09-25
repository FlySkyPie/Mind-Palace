# FOSS 團隊 AI 協作工具替代方案：聚焦 Skills/MCP 共享能力

> 本報告旨在調查 teamai-cli 的開放原始碼替代方案，重點關注**團隊間共享 AI Skills、MCP 伺服器、以及代理商配置**的功能。

## 1. 背景：什麼是 teamai-cli？

[teamai-cli](https://github.com/Tencent/teamai-cli) 是騰訊推出的開源 CLI 工具，核心解決方案是「透過 Git 倉庫作為骨幹，將團隊的 AI 代理配置管理集中化」。它將 Skills、Rules、MCP 伺服器、環境變數、Hooks 等統一存放在一個團隊 Git 倉庫中，團隊成員透過 `teamai pull` 自動同步。[^teamai]

在共享 Skills 方面，使用者可執行 `teamai push` 將本地 Skills 推送至團隊倉庫；MCP 伺服器則集中定義於 `mcp/mcp.yaml`，並支援根據角色（Role）、專案（Project）進行權限控管。各 AI 工具（Claude Code、Cursor、Codex 等）的配置會自動轉譯為該工具的原生格式。[^teamai]

## 2. 評選標準

- **開放原始碼（FOSS）**：採用 OSI 核准授權條款（MIT、Apache 2.0 等）
- **團隊共享能力**：支援團隊間共享 Skills、MCP 配置、Prompt 或代理設定
- **GitHub Stars**：優先納入高星數、社群活躍的專案，排除低知名度的專案
- **功能相關性**：與 teamai-cli 的核心功能（技能/MCP/配置共享）有直接可比性

## 3. 替代方案分類與比較

### 3.1 最直接替代方案（與 teamai-cli 核心功能最接近）

| 專案 | Stars | 授權 | 語言 | 共享機制 |
|------|-------|------|------|----------|
| [OpenWork](https://github.com/different-ai/openwork) | 23.7k | MIT + EE | TypeScript | MCP Gateway 集中管控 |
| [AgentTeams](https://github.com/agentscope-ai/AgentTeams) | 5.7k | Apache 2.0 | Go | K8s CRDs + Matrix 聊天室 |
| [Agenta](https://github.com/Agenta-AI/agenta) | 4.8k | 自訂授權 | TypeScript | 工作區（Workspace）共享 |

#### OpenWork（23.7k ⭐）

OpenWork 是最接近 teamai-cli 的替代方案。它是一個開源的 MCP 控制平面，團隊只需將一個 OpenWork MCP 伺服器 URL 加入任何 MCP 相容的代理（Claude Code、Codex、Cursor 等），即可讓該代理使用團隊已配置的所有 Skills、Plugin 和服務，無需逐一設定。[^openwork]

**團隊共享機制**：OpenWork Den（組織控制平面）讓管理者發布能力、管理存取權限、邀請成員、在組織/團隊/個人層級指派 Skills 與 Plugin，並設定政策。與 teamai-cli 的關鍵差異在於：OpenWork 使用 MCP 協議（而非 Git 同步）作為分發機制。

#### AgentTeams（5.7k ⭐）

由阿里巴巴雲端/AgentScope 團隊開發，是一個 Kubernetes 原生的協作式多代理執行平台。它讓多個 AI 代理在基於 Matrix 協議的聊天室中協作，具備完整的人機協同（Human-in-the-loop）與稽核能力。[^agentteams]

**團隊共享機制**：Skills 透過 Kubernetes CRD（Custom Resource Definitions）宣告式交付，支援從 skills.sh（80,000+ 社群技能）拉取生態系技能。MCP 配置同樣以 CRD 定義於 Worker/Manager/Team 層級。內建 Higress AI Gateway 進行憑證管理。

#### Agenta（4.8k ⭐）

Agenta 是一個團隊工作區，專注於建置與共享 AI 代理與自動化流程。它支援多種代理 Harness（Claude Code、Pi、Codex），具備 Skills 管理、MCP 伺服器整合、版本歷史、團隊角色權限控制、以及背景代理（Background Agent）功能。[^agenta]

**團隊共享機制**：團隊成員在同一個工作區中協作，Agents、Skills 和 MCP 配置皆可共享。支援 AGENTS.md 格式。可自託管。

### 3.2 成熟的團隊 AI 協作平台（功能更廣泛但非專注於 CLI 同步）

| 專案 | Stars | 授權 | 語言 | 共享機制 |
|------|-------|------|------|----------|
| [n8n](https://github.com/n8n-io/n8n) | 206k | Fair-code | TypeScript | 工作流程 + 團隊协作 |
| [Dify](https://github.com/langgenius/dify) | 157k | 自訂授權 | Python/TS | 工作區 + 市場（Marketplace） |
| [CrewAI](https://github.com/crewAIInc/crewAI) | 59k | MIT | Python | Skills CLI + JSON 配置 |

#### n8n（206k ⭐）

n8n 是功能最全面的開源工作流程自動化平台，近年加入原生 AI 代理支援。支援 MCP 伺服器、1500+ 整合、可視化編輯器、以及團隊協作功能。團隊可共享 AI 工作流程與代理設定。[^n8n]

**團隊共享機制**：工作流程（含 AI 代理）可透過 Git、n8n 雲端或自託管實例進行團隊共享。支援 RBAC 權限控制。

#### Dify（157k ⭐）

Dify 是一個完整的 LLM 應用開發平台，內建提示詞 IDE、RAG 管線、代理能力、以及工具/模型市場。團隊可在同一工作區中協作開發與共享 AI 應用。[^dify]

**團隊共享機制**：支援協作工作區、提示詞版本管理、代理市場（Marketplace）。團隊可發布與發現擅寫的提示詞、工具和工作流程。

#### CrewAI（59k ⭐）

CrewAI 是一個多代理協調框架，支援 JSON/YAML 格式的代理與任務配置。官方提供 Skills 系統（`npx skills add crewaiinc/skills`），可將 Skills 安裝至 Claude Code、Cursor 等工具。配置可透過 Git 版本控制與團隊共享。[^crewai]

### 3.3 Prompt/Skill/工具發現與共享平台

| 專案 | Stars | 授權 | 語言 | 共享機制 |
|------|-------|------|------|----------|
| [prompts.chat](https://github.com/f/prompts.chat) | 171k | 自訂授權 | TypeScript | Web UI 共享與發現 |
| [agentgateway](https://github.com/agentgateway/agentgateway) | 5k | Apache 2.0 | Rust | MCP Gateway + RBAC |
| [Promptfoo](https://github.com/promptfoo/promptfoo) | 25.4k | MIT | TypeScript | CLI 測試結果共享 |

#### prompts.chat（171k ⭐）

前身為 Awesome ChatGPT Prompts，是全世界最大的開源提示詞資料庫。支援自託管，組織可部署私有實例搭配自訂品牌、主題和驗證（GitHub/Google/Azure AD）。提供 Claude Code Plugin 和 MCP 伺服器整合。[^promptschat]

**團隊共享機制**：本質上是一個提示詞發布與發現平台。團隊可自託管私有實例，僅限組織成員使用。

#### agentgateway（5k ⭐）

Linux Foundation 專案，是「Agentic AI 的首個完整連接解決方案」。作為基於 MCP 與 A2A 協議的開放原始碼代理，提供安全、可觀測性與治理能力。團隊可透過單一 Gateway 端點安全地共享 MCP 工具，並支援 RBAC 權限控管。[^agentgateway]

#### Promptfoo（25.4k ⭐）

提示詞測試 CLI 與函式庫，支援測試 Prompt、Agent 和 RAG 系統。測試結果可與團隊分享。支援 `.claude`、`.cursor`、`.agents` 配置目錄。現已成為 OpenAI 的一部分，但原始碼仍為 MIT 授權。[^promptfoo]

### 3.4 其他相關專案（部分功能重疊但非核心聚焦）

| 專案 | Stars | 簡要說明 |
|------|-------|----------|
| [Activepieces](https://github.com/activepieces/activepieces) | 24.7k | AI 代理 + 400+ MCP 伺服器，開源 Zapier 替代品 |
| [ChatDev](https://github.com/OpenBMB/ChatDev) | 34.4k | 零程式碼多代理平台，支援 MCP 與 Skills 目錄 |
| [OpenAgents](https://github.com/openagents-org/openagents) | 4.1k | AI 代理協作 OS，所有代理連接到同一個工作區 |
| [kagent](https://github.com/kagent-dev/kagent) | 3.8k | CNCF 專案，K8s 原生代理框架，MCP 工具為一等公民 |
| [smolagents](https://github.com/huggingface/smolagents) | 25k+ | Hugging Face 代理函式庫，透過 HF Hub 共享代理/工具 |

## 4. 功能對照表

| 功能 | teamai-cli | OpenWork | AgentTeams | Agenta | Dify | n8n |
|------|:----------:|:--------:|:----------:|:------:|:----:|:---:|
| Skill 共享 | ✅ Git Push | ✅ MCP Gateway | ✅ CRD + Skills.sh | ✅ 工作區 | ✅ 市場 | ⚠️ Template |
| MCP 伺服器共享 | ✅ mcp.yaml | ✅ 原生 MCP | ✅ CRD 定義 | ✅ 內建 | ✅ 工具市場 | ✅ 內建 |
| 角色/權限控管 | ✅ Role/Tag | ✅ Den RBAC | ✅ K8s RBAC | ✅ 團隊角色 | ✅ 工作區 | ✅ RBAC |
| 跨工具支援 | 9+ 工具 | 任何 MCP 客戶端 | 多執行時期 | 多 Harness | Web App | Web App |
| 自託管 | ✅ Git | ✅ Docker | ✅ Helm/K8s | ✅ Docker | ✅ Docker | ✅ Docker |
| CLI 工具 | ✅ teamai | ✅ openwork | ❌ Web 為主 | ✅ agenta-cli | ❌ Web 為主 | ✅ n8n CLI |
| 版本歷史 | ✅ Git | ❌ | ❌ | ✅ 內建 | ✅ 提示詞版本 | ⚠️ Git |

## 5. 分析與建議

### 最推薦方案

依據使用情境不同，推薦方案如下：

**情境一：需要最接近 teamai-cli 的 CLI 體驗**
→ **OpenWork**（23.7k ⭐）

OpenWork 採用 MCP 協議作為分發機制，團隊只需管理一個 MCP 端點即可讓所有代理獲得團隊技能與工具。它擁有完整的組織控制平面，支援精細的權限管理。對於已經使用 MCP 相容代理（Claude Code、Cursor、Codex 等）的團隊而言，整合成本最低。

**情境二：需要 Kubernetes 原生、企業級方案**
→ **AgentTeams**（5.7k ⭐）

由阿里巴巴雲端支援，基於 K8s CRD 的技能交付與 Matrix 聊天室協作機制，非常適合已經採用 Kubernetes 的企業團隊。內建技能生態系（skills.sh）提供 80,000+ 社群技能。

**情境三：需要可視化工作區與快速原型**
→ **Agenta**（4.8k ⭐）

提供圖形化介面管理代理、技能與 MCP，支援版本歷史與團隊權限，適合非技術團隊成員參與 AI 代理配置管理。

**情境四：需要全方位的 LLM 應用開發平台**
→ **Dify**（157k ⭐）或 **n8n**（206k ⭐）

若團隊的需求不只有技能/MCP 共享，還包含完整的 LLM 應用開發（RAG、提示詞工程、工作流程自動化），這兩個成熟平台是更全面的選擇。

### 注意事項

1. **Lock-in 風險**：OpenWork 的 EE 授權對 5 人以上團隊有限制；n8n 採用 Fair-code 授權，部分企業功能需付費。
2. **GitHub Stars 並非唯一指標**：低星數（如 AgentTeams 5.7k）不代表品質不佳，可能僅因較晚發布或生態系尚未成熟。
3. **團隊規模**：本報告所列方案均適合中小型團隊（5-50 人），大型企業可能需要考慮更完整的授權方案。
4. **agentregistry**（497 ⭐）因星數過低，未列入主要評比，但其功能定位（MCP/Agent/Skill/Prompt 註冊中心）與 teamai-cli 最為接近，值得關注未來發展。

## 參考資料

[^teamai]: Tencent. (n.d.). teamai-cli: Make Every Team AI Native. Retrieved 2026-09-25, from https://github.com/Tencent/teamai-cli

[^openwork]: different-ai. (2026). OpenWork: The open-source alternative to Claude Cowork. Retrieved 2026-09-25, from https://github.com/different-ai/openwork

[^agentteams]: agentscope-ai. (2026). AgentTeams: An open-source Collaborative Multi-Agent OS. Retrieved 2026-09-25, from https://github.com/agentscope-ai/AgentTeams

[^agenta]: Agenta-AI. (2023). Agenta: AI Agent Workspace. Retrieved 2026-09-25, from https://github.com/Agenta-AI/agenta

[^n8n]: n8n-io. (n.d.). n8n: Workflow automation with AI capabilities. Retrieved 2026-09-25, from https://github.com/n8n-io/n8n

[^dify]: langgenius. (2023). Dify: LLM Application Development Platform. Retrieved 2026-09-25, from https://github.com/langgenius/dify

[^crewai]: crewAIInc. (2023). CrewAI: Multi-agent orchestration framework. Retrieved 2026-09-25, from https://github.com/crewAIInc/crewAI

[^promptschat]: f. (2022). prompts.chat: Share, discover, and collect prompts. Retrieved 2026-09-25, from https://github.com/f/prompts.chat

[^agentgateway]: agentgateway. (2025). AgentGateway: Next Generation Agentic Proxy. Retrieved 2026-09-25, from https://github.com/agentgateway/agentgateway

[^promptfoo]: promptfoo. (2023). Promptfoo: Prompt testing and evaluation. Retrieved 2026-09-25, from https://github.com/promptfoo/promptfoo