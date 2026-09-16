# PR-Agent (The-PR-Agent/pr-agent) 使用方式

PR-Agent 是一套開源的 AI 驅動程式碼審查工具，由 Qodo 捐贈給社群維護，可整合 GitHub、GitLab、Bitbucket、Azure DevOps 及 Gitea 等多個 Git 平台。[^pr-agent-overview]

## 安裝方式

### 1. GitHub Action（推薦）

在 repository 新增 `.github/workflows/pr_agent.yml`：

```yaml
name: PR Agent
on:
  pull_request:
    types: [opened, reopened, ready_for_review, synchronize]
  issue_comment:
jobs:
  pr_agent_job:
    if: ${{ github.event.sender.type != 'Bot' }}
    runs-on: ubuntu-latest
    permissions:
      issues: write
      pull-requests: write
      contents: write
    steps:
      - name: PR Agent action step
        uses: the-pr-agent/pr-agent@main
        env:
          OPENAI_KEY: ${{ secrets.OPENAI_KEY }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

需在 GitHub Secrets 設定 `OPENAI_KEY`（你的 OpenAI API Key），`GITHUB_TOKEN` 由 GitHub 自動提供。[^github-action-setup]

若需支援 fork 來的 PR（外部貢獻者），改用 `pull_request_target` 事件。[^github-action-fork]

### 2. CLI 本地執行

```bash
pip install pr-agent
export OPENAI_KEY=your_key_here
pr-agent --pr_url https://github.com/owner/repo/pull/123 review
```

或使用 Python module 方式：

```bash
python -m pr_agent.cli --pr_url=<PR_URL> review
```

[^cli-setup]

### 3. 其他平台

- **GitLab**：透過 webhook 整合 [^gitlab-setup]
- **Bitbucket**：安裝 App [^bitbucket-setup]
- **Azure DevOps**：設定整合 [^azure-setup]

## 支援的 AI 模型

除 OpenAI GPT 外，也可透過 LiteLLM 使用 Anthropic Claude、Google Gemini、DeepSeek、Mistral、Azure OpenAI、AWS Bedrock、Vertex AI、OpenRouter、Ollama 等。[^changing-model]

範例：改用 Gemini

```yaml
env:
  OPENAI_KEY: ${{ secrets.OPENAI_KEY }}
  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  # Gemini 設定請參考官方文件
```

[^gemini-setup]

## 主要工具 (Commands)

所有工具可透過 **PR 留言** 或 **CLI** 觸發：[^tools]

| 工具 | 留言指令 | CLI 範例 | 功能說明 |
|------|----------|----------|----------|
| **Describe** | `/describe` | `pr-agent --pr_url=<URL> describe` | 產生 PR 標題、類型、摘要、程式碼導覽與標籤 |
| **Review** | `/review` | `pr-agent --pr_url=<URL> review` | 產生程式碼審查意見（問題、安全、測試、工作量） |
| **Improve** | `/improve` | `pr-agent --pr_url=<URL> improve` | 提供可操作的程式碼改善建議 |
| **Ask** | `/ask "你的問題"` | `pr-agent --pr_url=<URL> ask "你的問題"` | 針對 PR 進行自由問答，也可針對特定程式碼行提問 |
| **Add Docs** | `/add_docs` | `pr-agent --pr_url=<URL> add_docs` | 為缺少說明的程式碼元件產生文件 |
| **Generate Labels** | `/generate_labels` | `pr-agent --pr_url=<URL> generate_labels` | 根據程式碼變更自動產生 PR 標籤 |
| **Similar Issues** | `/similar_issue` | `pr-agent --issue_url=<URL> similar_issue` | 尋找類似的 Issue |
| **Update Changelog** | `/update_changelog` | `pr-agent --pr_url=<URL> update_changelog` | 自動更新 CHANGELOG.md |
| **Help** | `/help` | `pr-agent --pr_url=<URL> help` | 列出所有可用工具 |

> ⚠️ `/help_docs` 自 v0.36.1 起暫時停用（憑證暴露問題 [#2445](https://github.com/the-pr-agent/pr-agent/issues/2445)）。

## 自動化觸發

可在 GitHub Actions 中設定自動執行特定工具，例如 PR 開啟時自動 review：

```yaml
env:
  github_action_config.pr_actions: '["opened", "reopened", "synchronize", "ready_for_review", "review_requested"]'
```

[^automation]

## 設定方式

可透過環境變數覆蓋 `configuration.toml` 中的任何設定：[^configuration]

```yaml
env:
  PR_REVIEWER.REQUIRE_TESTS_REVIEW: "false"    # 停用測試審查
  PR_CODE_SUGGESTIONS.NUM_CODE_SUGGESTIONS: 6  # 增加程式碼建議數量
```

完整的設定參考位於：[pr_agent/settings/configuration.toml](https://github.com/the-pr-agent/pr-agent/blob/main/pr_agent/settings/configuration.toml)

## Docker Hub 注意事項

v0.34.2 起 Docker image 已遷移至新 namespace：
- **新**：`pragent/pr-agent`
- **舊**（v0.31 以下）：`codiumai/pr-agent`（凍結存檔，不再推送新 image）

升級時需更新 `image:` / `docker pull` / `uses: docker://` 等參考。[^docker-migration]

---

[^pr-agent-overview]: The-PR-Agent. (n.d.). PR-Agent Overview. Retrieved 2026-09-16, from https://docs.pr-agent.ai/
[^github-action-setup]: The-PR-Agent. (n.d.). GitHub Integration — Run as a GitHub Action. Retrieved 2026-09-16, from https://docs.pr-agent.ai/installation/github/#run-as-a-github-action
[^github-action-fork]: The-PR-Agent. (n.d.). GitHub Integration — Using with pull_request_target. Retrieved 2026-09-16, from https://docs.pr-agent.ai/installation/github/#using-with-pull_request_target-forkcontribution-support
[^cli-setup]: The-PR-Agent. (n.d.). CLI Usage. Retrieved 2026-09-16, from https://docs.pr-agent.ai/usage-guide/automations_and_usage/#local-repo-cli
[^gitlab-setup]: The-PR-Agent. (n.d.). GitLab Integration. Retrieved 2026-09-16, from https://docs.pr-agent.ai/installation/gitlab/
[^bitbucket-setup]: The-PR-Agent. (n.d.). BitBucket Integration. Retrieved 2026-09-16, from https://docs.pr-agent.ai/installation/bitbucket/
[^azure-setup]: The-PR-Agent. (n.d.). Azure DevOps Integration. Retrieved 2026-09-16, from https://docs.pr-agent.ai/installation/azure/
[^changing-model]: The-PR-Agent. (n.d.). Changing a Model. Retrieved 2026-09-16, from https://docs.pr-agent.ai/usage-guide/changing_a_model/
[^gemini-setup]: The-PR-Agent. (n.d.). GitHub Integration — Gemini Setup. Retrieved 2026-09-16, from https://docs.pr-agent.ai/installation/github/#gemini-setup
[^tools]: The-PR-Agent. (n.d.). Tools. Retrieved 2026-09-16, from https://docs.pr-agent.ai/tools/
[^automation]: The-PR-Agent. (n.d.). Usage and Automation. Retrieved 2026-09-16, from https://docs.pr-agent.ai/usage-guide/automations_and_usage/
[^configuration]: The-PR-Agent. (n.d.). Configuration Options. Retrieved 2026-09-16, from https://docs.pr-agent.ai/usage-guide/configuration_options/
[^docker-migration]: The-PR-Agent. (n.d.). Getting Started — Docker Hub namespace migration. Retrieved 2026-09-16, from https://github.com/marketplace/actions/the-pr-agent