# 使用 LLM 自動審查 Jira 票務內容的 FOSS 軟體調查

## 概述

本報告調查了市面上能夠使用大型語言模型（LLM）自動審查 Jira 票務（Ticket）內容的自由及開源軟體（FOSS）。重點關注 GitHub Star 數量、授權條款、Jira 整合方式以及 LLM 支援情況。

---

## 一、直接審查 Jira 票務內容的工具

這類工具**直接分析 Jira 票務的內容**（描述、評論、格式、完整性等）並使用 LLM 進行審查。

### 1.1 jira-ticket-reviewer

| 項目 | 內容 |
|---|---|
| **GitHub** | [DevMetwaly/jira-ticket-reviewer](https://github.com/DevMetwaly/jira-ticket-reviewer) |
| **Stars** | 0 |
| **License** | MIT |
| **npm** | `@dev-metwaly/jira-ticket-reviewer` |
| **整合方式** | Jira REST API（API Token + Email） |

CLI 工具，支援多種 AI 提供者（OpenAI、Anthropic、Gemini、本地 Ollama、自訂端點）。可透過 Jira API 獲取票務，或接受手動 JSON 輸入。使用可自訂的提示模板，支援 `{{placeholder}}` 語法（summary、description、priority 等變數），輸出結構化的 Markdown 審查報告。[^jtr]

[^jtr]: DevMetwaly. (n.d.). jira-ticket-reviewer. Retrieved 2026-09-16, from https://github.com/DevMetwaly/jira-ticket-reviewer

### 1.2 ai_jira_ticket_validator

| 項目 | 內容 |
|---|---|
| **GitHub** | [zilongqiu/ai_jira_ticket_validator](https://github.com/zilongqiu/ai_jira_ticket_validator) |
| **Stars** | 0 |
| **License** | MIT |
| **整合方式** | Jira URL + Project Key + Username + Password（Basic Auth） |

Web 應用（Next.js），可讓使用者定義**自然語言驗證規則**（例如「票務描述應已填寫且優先級已設定」），連接 Jira 執行個體後，使用 OpenAI（gpt-4o-mini）對所有票務進行驗證。每個票務會獲得評分（X/10）、問題列表以及改進建議。[^aijtv]

[^aijtv]: zilongqiu. (n.d.). ai_jira_ticket_validator. Retrieved 2026-09-16, from https://github.com/zilongqiu/ai_jira_ticket_validator

### 1.3 jira-quality-check

| 項目 | 內容 |
|---|---|
| **GitHub** | [ns-skuncha/jira-quality-check](https://github.com/ns-skuncha/jira-quality-check) |
| **Stars** | 0 |
| **License** | 未明確標示（內部工具） |
| **整合方式** | Jira API Token，依元件/標籤或修正版本查詢 |

Python 腳本，針對特定元件的 Jira 票務進行自動化品質評估。支援 **Ollama（本地 AI）** 或規則評分兩種模式。對開啟中的票務檢查重現步驟、預期/實際結果、日誌；對已解決的票務檢查 RCA、修正詳情、測試執行。產生 HTML 與 JSON 報表。可透過 cron/systemd 定時執行。[^jqc]

[^jqc]: ns-skuncha. (n.d.). jira-quality-check. Retrieved 2026-09-16, from https://github.com/ns-skuncha/jira-quality-check

---

## 二、Jira 票務上下文提取工具

這類工具**提取並結構化 Jira 票務的上下文**（包含關聯票務、Confluence 文件、附件等），供 LLM 使用——這是自動審查的前置步驟。

### 2.1 ticket-miner

| 項目 | 內容 |
|---|---|
| **GitHub** | [gulliverhan/ticket-miner](https://github.com/gulliverhan/ticket-miner) |
| **Stars** | 0 |
| **License** | MIT |
| **PyPI** | `ticket-miner` |
| **整合方式** | Jira API + Confluence API + 網頁爬取 |

Python 函式庫，**模擬人類調查 Jira 票務的流程**——自動挖掘票務、追蹤相關票務連結、爬取 Confluence 頁面、Help Center 文章及外部 URL，遞迴進行。輸出結構化 JSON（最佳化供 LLM 處理），包含循環偵測以防止無限迴圈。[^tm]

[^tm]: gulliverhan. (n.d.). ticket-miner. Retrieved 2026-09-16, from https://github.com/gulliverhan/ticket-miner

### 2.2 jira-context-mcp

| 項目 | 內容 |
|---|---|
| **GitHub** | [pdudzinsky/jira-context-mcp](https://github.com/pdudzinsky/jira-context-mcp) |
| **Stars** | 1 |
| **License** | MIT |
| **整合方式** | Jira API Token，以 stdio MCP Server 運作 |

**MCP（Model Context Protocol）伺服器**，提供四項工具將 Jira 票務上下文引入 LLM 開發環境（Claude Desktop、Cursor 等）：`get_issue_tree`（層級結構）、`get_ticket_content`（完整票務資料）、`get_smart_checklist`（僅驗收條件）、`get_ticket_attachment`（以原生 MCP Payload 獲取檔案）。純唯讀設計。[^jcmcp]

[^jcmcp]: pdudzinsky. (n.d.). jira-context-mcp. Retrieved 2026-09-16, from https://github.com/pdudzinsky/jira-context-mcp

---

## 三、整合 Jira 上下文的程式碼審查工具

這類工具主要進行 **AI 程式碼審查（Pull Request）**，但會拉取 Jira 票務上下文以比對需求。

### 3.1 codereview-agent

| 項目 | 內容 |
|---|---|
| **GitHub** | [SuperscriptSystems/codereview-agent](https://github.com/SuperscriptSystems/codereview-agent) |
| **Stars** | ⭐ **20**（本次調查最高） |
| **License** | Apache 2.0 |
| **整合方式** | Jira URL + Email + API Token；Bitbucket Pipelines、GitHub Actions |

AI 驅動的程式碼審查代理，**會擷取 Jira 任務上下文**並用以審查程式碼變更。執行多階段 LLM 分析，合併後將審查結果發回 Jira。支援任何 OpenAI 相容 API（預設 OpenRouter），透過 Tree-sitter 進行靜態分析，並以 Docker 映像支援 CI/CD 整合。[^cra]

[^cra]: SuperscriptSystems. (n.d.). codereview-agent. Retrieved 2026-09-16, from https://github.com/SuperscriptSystems/codereview-agent

### 3.2 suseek/ai-agent

| 項目 | 內容 |
|---|---|
| **GitHub** | [suseek/ai-agent](https://github.com/suseek/ai-agent) |
| **Stars** | 0 |
| **License** | 未指定 |
| **整合方式** | Jira API + GitLab API |

CLI 工具，包含 **Triage Assistant**（協助 AI 分類 Jira 票務）與 **Jira Assistant**（檢索/更新票務資訊），同時支援跨系統分析（Jira + GitLab Wiki）。[^sai]

[^sai]: suseek. (n.d.). ai-agent. Retrieved 2026-09-16, from https://github.com/suseek/ai-agent

---

## 四、總結與建議

| 工具 | GitHub Stars | 審查對象 | LLM 支援 | FOSS 授權 |
|---|---|---|---|---|
| jira-ticket-reviewer | 0 | Jira 票務內容（自訂提示） | OpenAI、Anthropic、Gemini、本地 Ollama | ✅ MIT |
| ai_jira_ticket_validator | 0 | Jira 票務內容（自訂規則） | OpenAI（gpt-4o-mini） | ✅ MIT |
| jira-quality-check | 0 | Jira 票務欄位完整性 | Ollama（本地）+ 規則 | ❌ 未標示 |
| ticket-miner | 0 | 挖掘票務及所有關聯（資料管線） | 不適用（資料層） | ✅ MIT |
| jira-context-mcp | 1 | 票務上下文（供 LLM Agent 使用） | 不適用（MCP 提供者） | ✅ MIT |
| codereview-agent | **20** | 程式碼 diff vs Jira 需求 | 任何 OpenAI 相容 API | ✅ Apache 2.0 |
| ai-agent (suseek) | 0 | 票務分類/分析 | OpenAI | ❌ 未指定 |

**最直接相關的三個工具**（針對 Jira 票務內容審查）：

1. **jira-ticket-reviewer** — 最佳 CLI 工具，支援多 LLM、自訂提示模板、本地模型
2. **ai_jira_ticket_validator** — 最佳 Web UI 工具，以自然語言定義規則進行批次驗證
3. **jira-quality-check** — 最佳排程工具，支援本地 AI（Ollama）定時品質評分

**關於 GitHub Stars**：以上 FOSS 專案均相當新且尚無大量關注，但 **codereview-agent（20 stars）** 是目前此領域中最成熟的開源專案。

**關於 Jira 整合方式**：所有工具均透過 **Jira REST API**（需服務帳號與 API Token）整合，可透過 **Jira Webhook** 觸發自動審查流程（如 jira-quality-check 支援排程），或由 CI/CD pipeline 觸發（如 codereview-agent 支援 GitHub Actions）。

---

## 參考資料

DevMetwaly. (n.d.). jira-ticket-reviewer. Retrieved 2026-09-16, from https://github.com/DevMetwaly/jira-ticket-reviewer

zilongqiu. (n.d.). ai_jira_ticket_validator. Retrieved 2026-09-16, from https://github.com/zilongqiu/ai_jira_ticket_validator

ns-skuncha. (n.d.). jira-quality-check. Retrieved 2026-09-16, from https://github.com/ns-skuncha/jira-quality-check

gulliverhan. (n.d.). ticket-miner. Retrieved 2026-09-16, from https://github.com/gulliverhan/ticket-miner

pdudzinsky. (n.d.). jira-context-mcp. Retrieved 2026-09-16, from https://github.com/pdudzinsky/jira-context-mcp

SuperscriptSystems. (n.d.). codereview-agent. Retrieved 2026-09-16, from https://github.com/SuperscriptSystems/codereview-agent

suseek. (n.d.). ai-agent. Retrieved 2026-09-16, from https://github.com/suseek/ai-agent

Cubic. (n.d.). The AI Reviewer That Understands Ticket Intent. Retrieved 2026-09-16, from https://www.cubic.dev/blog/the-ai-reviewer-that-understands-ticket-intent-cubic

Git AutoReview. (n.d.). Jira Integration. Retrieved 2026-09-16, from https://gitautoreview.com/integrations/jira

Waterline. (n.d.). Product Homepage. Retrieved 2026-09-16, from https://www.getwaterline.dev

MCP Market. (n.d.). Jira Ticket Quality Validator. Retrieved 2026-09-16, from https://mcpmarket.com/tools/skills/jira-ticket-quality-validator