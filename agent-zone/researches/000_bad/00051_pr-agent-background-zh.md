# PR-Agent 背景調查報告

> [!WARNING] 對齊失敗
> "背景調查" 是指創投、資金背景，而非專案簡介。

## 概述

PR-Agent（亦稱 The PR Agent）是一套開源、AI 驅動的程式碼審查（code review）代理工具，專為 Pull Request 自動化審查而設計。它目前擁有約 13,000 顆 GitHub Stars、1,900 個 Forks，採用 MIT 授權條款，主要開發語言為 Python。[^github-readme]

## 歷史沿革

PR-Agent 最初由 **CodiumAI** 公司開發與維護。該公司由 **Itamar Friedman**（CEO）與 **Dedy Kredo**（CPO）共同創立，後續公司品牌更名為 **Qodo**，定位為 AI 程式碼審查平台，提供從 IDE、Pull Request、CLI 到 Git 工作流程的全方位自動化審查服務。[^qodo-about]

PR-Agent 原名 **Qodo Merge**（Qodo 1.0），是 Qodo 的企業級版本。Qodo 後續演進為 Qodo 2.0，成為一個完整的 AI 程式碼審查平台。2020 年代中期，Qodo 將 PR-Agent 捐贈給開源社群，專案遷移至 `The-PR-Agent` GitHub 組織下，成為完全由社群擁有的專案。[^github-readme]

截至目前，PR-Agent 已有一位外部維護者 **Naor**（@naorpeled），並正在進行捐贈給開源基金會的程序。[^github-readme]

Docker Hub 命名空間亦經歷遷移：v0.34.2 以後的版本發布於 `pragent/pr-agent`，舊版（v0.31 以前）留存於舊的 `codiumai/pr-agent` 命名空間作為凍結存檔。[^github-readme]

## 架構設計

PR-Agent 是一個 CLI/Server 應用程式，其核心調度流程為 `pr_agent/agent/pr_agent.py` → `command2class` → 對應的 tool class。[^agents-md]

### 模組組織

- **`pr_agent/agent/`**：命令調度核心，透過 `pr_agent.py` 協調各項指令（review、describe、improve 等）
- **`pr_agent/tools/`**：各工具（reviewer、code suggestions、docs update、label generation 等）的實際實作
- **`pr_agent/algo/`**：共享演算法、模型處理器、提示詞/Token 處理、型別定義與工具函式
- **`pr_agent/git_providers/`**：Git 平台整合層，支援 GitHub、GitLab、Bitbucket（Cloud 與 Server）、Azure DevOps、Gitea、Gerrit、CodeCommit、本地 checkout 及純 diff 輸入
- **`pr_agent/settings/`**：以 Dynaconf 管理的預設設定（提示詞模板、設定範本、忽略清單）
- **`pr_agent/servers/`**：Webhook 與服務進入點
- **`pr_agent/identity_providers/`**：身份提供者
- **`pr_agent/secret_providers/`**： secrets 管理

### 提示詞系統

提示詞驅動的工具會建構一個 `self.vars` 字典，傳遞給 `TokenHandler` 搭配系統/使用者提示詞字串進行渲染。渲染引擎使用 **Jinja2** 搭配 `StrictUndefined` 模式。[^agents-md]

系統與使用者的提示詞字串以 **TOML** 格式儲存在 `pr_agent/settings/` 目錄下，透過 `pr_agent/config_loader.py` 載入至 `global_settings`。工具與提示詞檔案名稱通常對應，例如：
- `pr_reviewer.py` ↔ `pr_reviewer_prompts.toml`
- `pr_description.py` ↔ `pr_description_prompts.toml`
- `pr_code_suggestions.py` ↔ `code_suggestions/pr_code_suggestions_prompts.toml`

### 設定系統

使用 `get_settings()` 作為共享設定存取器，底層為 **Dynaconf**。預設值位於 `pr_agent/settings/configuration.toml`，個別儲存庫可透過 `.pr_agent.toml` 覆蓋設定。[^agents-md]

## 核心功能

PR-Agent 提供以下工具，可透過 PR 留言觸發或 CLI 執行：[^tools-docs]

| 工具 | 說明 | 觸發指令 |
|------|------|----------|
| PR Description（/describe） | 產生 PR 標題、類型、摘要、程式碼走讀與標籤 | `/describe` |
| PR Review（/review） | 產生 PR 審查意見，包含潛在問題、安全性疑慮、測試建議 | `/review` |
| Code Suggestions（/improve） | 產生可操作的程式碼改進建議 | `/improve` |
| Q&A（/ask） | 針對 PR 或特定程式碼行提問 | `/ask "問題"` |
| Add Documentation（/add_docs） | 為缺少文件的程式碼元件產生文件 | `/add_docs` |
| Generate Labels（/generate_labels） | 根據程式碼變更自動產生標籤 | `/generate_labels` |
| Similar Issues（/similar_issue） | 尋找相似 issue | `/similar_issue` |
| Update Changelog（/update_changelog） | 自動更新 CHANGELOG.md | `/update_changelog` |
| Help（/help） | 列出所有可用工具 | `/help` |

> ⚠️ `/help_docs` 自 v0.36.1 起因憑證暴露問題 (#2445) 暫時停用。

## Git 平台支援

PR-Agent 支援以下 Git 平台：[^docs-overview]

- GitHub（完整支援）
- GitLab（完整支援）
- Bitbucket（完整支援）
- Azure DevOps（完整支援）
- Gitea（完整支援）

## AI 模型支援

透過 **LiteLLM** 整合層，PR-Agent 支援多種 LLM 提供者：[^github-readme]

- OpenAI GPT
- Anthropic Claude
- Google Gemini
- DeepSeek
- Mistral
- 其他透過 LiteLLM 可串接的模型（Azure OpenAI、AWS Bedrock、Vertex AI、Databricks、OpenRouter、Ollama 等）

## 部署方式

- **GitHub Action**（推薦）：透過 `.github/workflows/pr-agent.yml` 自動化設定
- **CLI**：`pip install pr-agent` 後直接執行
- **Docker**：多種 Docker image 目標（`github_app`、`gitlab_webhook`、`gitea_app` 等）
- **Webhook**：自架伺服器，支援 GitHub App、GitLab webhook、Bitbucket 等
- **Lambda**：支援 GitHub 與 GitLab 的 AWS Lambda 部署

## 技術棧

- **語言**：Python ≥ 3.12
- **套件管理**：uv（搭配 uv.lock）
- **設定管理**：Dynaconf
- **提示詞模板**：Jinja2（StrictUndefined 模式）
- **測試框架**：pytest（asyncio_mode = "auto"）
- **Linting**：Ruff（規則 E、F、B、I）
- **靜態分析**：pre-commit hooks
- **文件**：MkDocs（mkdocs-material 主題）
- **專案結構**：pyproject.toml / setup.py

## 核心設計理念

1. **高效低成本**：每個工具僅需一次 LLM 呼叫（約 30 秒），成本低廉
2. **PR 壓縮策略**：有效處理大型 PR，將 diff 轉換為 LLM 可管理的提示詞
3. **高度可自訂**：基於 JSON 的提示詞系統，可輕鬆調整審查類別與行為
4. **平台無關**：支援多種 Git 平台與部署方式
5. **資料隱私**：自架版本直接用 OpenAI API key，資料僅在自架環境與 LLM 提供者之間傳遞

## 比較：開源 PR-Agent vs Qodo 平台

| 面向 | PR-Agent（開源） | Qodo（商業平台） |
|------|-----------------|-----------------|
| 維護方式 | 社群維護 | Qodo 公司 |
| 部署 | 自架（CLI/Docker/Action） | 託管服務 |
| 功能 | 基礎 PR 審查工具 | 完整平台（規則系統、儀表板、風險分析） |
| 資料控制 | 完全自控 | 由 Qodo 處理 |
| 價格 | 免費（自付 LLM 費用） | 有免費版與付費方案 |

## 結論

PR-Agent 是 AI 程式碼審查領域中最早也最具影響力的開源專案之一。從 CodiumAI（現 Qodo）的商業產品演化為社群維護的開源專案，它證明了 AI 輔助程式碼審查的實用價值。其架構設計使得開發團隊可以完全掌控審查流程、自訂提示詞行為，並選擇適合的 LLM 模型，是建置自動化程式碼審查基礎設施的重要參考項目。

[^github-readme]: The-PR-Agent. (n.d.). *PR-Agent README*. Retrieved 2026-09-13, from https://github.com/The-PR-Agent/pr-agent
[^agents-md]: The-PR-Agent. (n.d.). *AGENTS.md — Repository Guidelines*. Retrieved 2026-09-13, from https://github.com/The-PR-Agent/pr-agent/blob/main/AGENTS.md
[^docs-overview]: PR-Agent Contributors. (n.d.). *PR-Agent Documentation — Overview*. Retrieved 2026-09-13, from https://docs.pr-agent.ai/
[^tools-docs]: PR-Agent Contributors. (n.d.). *PR-Agent Documentation — Tools*. Retrieved 2026-09-13, from https://docs.pr-agent.ai/tools/
[^qodo-about]: Qodo. (n.d.). *About Us — Values, Mission and Team*. Retrieved 2026-09-13, from https://www.qodo.ai/about/