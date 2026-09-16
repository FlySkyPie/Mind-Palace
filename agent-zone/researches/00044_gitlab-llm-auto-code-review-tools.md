# 可與 GitLab 整合的 FOSS LLM 自動 Code Review 工具調查

## 摘要

本報告調查市面上能以 LLM（大型語言模型）驅動、自動審查 GitLab Merge Request 的開放原始碼（FOSS）軟體，並特別著重 GitHub Star 數量作為社群成熟度與採用度的參考指標。

---

## 1. 前言

隨著 LLM 的發展，利用 AI 自動進行 Code Review 已成為 DevOps 領域的熱門方向。開發團隊希望在 GitLab CI/CD 或 Webhook 中整合 LLM，讓每次 Merge Request 都能自動獲得語意層面的程式碼審查意見。本報告聚焦於**真正開放原始碼（非僅 SaaS 前端開源）**且**能與 GitLab 整合**的工具。

---

## 2. 工具總覽

### 2.1 Open Code Review（阿里巴巴）

- **GitHub**: [alibaba/open-code-review](https://github.com/alibaba/open-code-review)
- **Stars**: ⭐ **30,200+**
- **Fork**: 2,200+
- **GitLab 支援**: ✅ 可透過 GitLab CI 執行 CLI
- **LLM 支援**: OpenAI、Anthropic 相容模型
- **部署方式**: CLI（Go 撰寫）

阿里巴巴內部的 battle-tested 方案，採用混合架構：確定性工程管線（靜態分析規則） + LLM Agent。內建多語言規則集（NPE、執行緒安全、XSS、SQL Injection）。號稱在特定測試中以 1/9 的 token 成本達到比 Claude Code 更高的精準度。[^alibaba-ocr]

### 2.2 PR-Agent（原 CodiumAI / Qodo）

- **GitHub**: [The-PR-Agent/pr-agent](https://github.com/The-PR-Agent/pr-agent)
- **Stars**: ⭐ **13,000+**
- **Fork**: 1,900+
- **GitLab 支援**: ✅ Webhook 整合
- **LLM 支援**: 10+ 種（OpenAI、Claude、Gemini、DeepSeek、Mistral、Ollama 等，透過 LiteLLM）
- **部署方式**: Docker / CLI（Python）

開源版 AI PR Reviewer 始祖，原由 Qodo（CodiumAI）開發，已捐贈給社群。支援 GitHub、GitLab、Bitbucket、Azure DevOps。功能涵蓋 PR 摘要、行內註解、改善建議、變更範圍分析。[^pragent]

### 2.3 Code Review GPT GitLab

- **GitHub**: [mimo-x/Code-Review-GPT-Gitlab](https://github.com/mimo-x/Code-Review-GPT-Gitlab)
- **Stars**: ⭐ **821**
- **Fork**: 260+
- **GitLab 支援**: ✅ 原生 GitLab MR 整合
- **LLM 支援**: GPT、DeepSeek 等
- **部署方式**: 自架 Bot

專為 GitLab 設計的輕量級自動審查 Bot，透過 GitLab API 讀取 MR 變更並交由 LLM 審查。支援多 Agent 協作模式開發中。[^code-review-gpt]

### 2.4 AI Review（Nikita-Filonov）

- **GitHub**: [Nikita-Filonov/ai-review](https://github.com/Nikita-Filonov/ai-review)
- **Stars**: ⭐ **574**
- **Fork**: 94
- **GitLab 支援**: ✅ 原生支援 GitLab CI/CD
- **LLM 支援**: 7+ 種（OpenAI、Claude、Gemini、Ollama、Bedrock、OpenRouter）
- **部署方式**: Docker / CLI

多平台 AI Code Review 工具，同時支援 GitLab、GitHub、Bitbucket、Azure DevOps、Gitea。特色是 Agent Mode（ReAct 風格的儲存庫探索），可產生行內註解、上下文審查與摘要審查。完全 Client-Side 運作，無中介伺服器。[^ai-review]

### 2.5 Kodus AI

- **GitHub**: [kodustech/kodus-ai](https://github.com/kodustech/kodus-ai)
- **Stars**: ⭐ **~1,000**
- **GitLab 支援**: ✅ 支援 GitLab
- **LLM 支援**: 多種 LLM
- **部署方式**: Docker

混合 AST（抽象語法樹）+ LLM 架構，旨在減少 LLM 幻覺。同時支援 GitHub、GitLab、Bitbucket、Azure Repos。[^kodus]

### 2.6 Gito

- **GitHub**: [Nayjest/Gito](https://github.com/Nayjest/Gito)
- **Stars**: ⭐ **430**
- **Fork**: 39
- **GitLab 支援**: ✅ Beta（可於 CI/CD 使用）
- **LLM 支援**: 任何 LLM（OpenAI、Anthropic、Google、Ollama、vLLM 等本地模型）
- **部署方式**: CLI（Python）/ CI/CD

強調無供應商綁定（No Vendor Lock-in），支援自訂審查規則、嚴重性分級，以及 Jira/Linear 整合。開發活躍（~1,099 commits）。[^gito]

### 2.7 Merge Mind

- **GitHub**: [omidbakhshi/merge-mind](https://github.com/omidbakhshi/merge-mind)
- **Stars**: ⭐ **25**
- **Fork**: 2
- **GitLab 支援**: ✅ 原生（特別針對 Self-Hosted GitLab）
- **LLM 支援**: OpenAI GPT-4
- **部署方式**: Docker

專為自架 GitLab 設計，使用 Qdrant 向量資料庫實現「學習」能力（可從既有程式碼庫學習），並具備框架感知審查（Laravel、Nuxt.js、React、Vue.js、Django 等）。另附 Web 儀表板、斷路器保護。[^merge-mind]

### 2.8 其他較小工具

| 工具 | Stars | GitLab 支援 | LLM |
|------|-------|-------------|-----|
| [rikvermeulen/co-op-gitlab](https://github.com/rikvermeulen/co-op-gitlab) | ⭐ 41 | Webhook | GPT-3/4 |
| [Evobaso-J/ai-gitlab-code-review](https://github.com/Evobaso-J/ai-gitlab-code-review) | ⭐ 34 | Webhook | GPT |
| [adraynrion/gitlab-cr-agent](https://github.com/adraynrion/gitlab-cr-agent) | ⭐ 7 | 原生 | Gemini、Anthropic、OpenAI |
| [KonstZiv/ai-code-reviewer](https://github.com/KonstZiv/ai-code-reviewer) | ⭐ 4 | GitHub Action / Docker | Gemini、Mistral |
| [alairjt/gitlab-gemini-reviewer](https://github.com/alairjt/gitlab-gemini-reviewer) | ⭐ 2 | CI/CD | Gemini |

---

## 3. 推薦排序

以下綜合考量 **GitLab 整合完善度**、**GitHub Star 數**、**LLM 彈性**與**文檔品質**：

```
等級  工具                 Stars      GitLab 支援      LLM 彈性
─────────────────────────────────────────────────────────────
S    Open Code Review     30.2k      ✅ CI/CD         中等
S    PR-Agent             13.0k      ✅ Webhook        極高
A    Kodus AI             ~1k        ✅ 原生            高
A    Code Review GPT       821       ✅ 原生            中高
B    AI Review             574       ✅ 原生            極高
B    Gito                  430       ✅ Beta            極高
C    Merge Mind             25       ✅ 原生(自架)      低
```

---

## 4. 選擇建議

- **追求最大社群成熟度**：選擇 **Open Code Review**（30.2k stars，阿里背書，但 LLM 支援較封閉）。
- **需要最多 LLM 選擇 + GitLab 支援**：選擇 **PR-Agent**（13k stars，支援 10+ 種 LLM，含本地 Ollama）。
- **純 GitLab 環境、輕量自架**：**Code Review GPT GitLab**（821 stars，最專注 GitLab）或 **AI Review**（574 stars，LLM 彈性最大）。
- **剛起步、需要客製化規則**：**Gito**（430 stars）或 **Merge Mind**（25 stars，適合小型團隊）。

---

## 5. 參考文獻

[^alibaba-ocr]: alibaba. (n.d.). *Open Code Review*. GitHub. Retrieved 2026-09-13, from https://github.com/alibaba/open-code-review

[^pragent]: The-PR-Agent. (n.d.). *PR-Agent*. GitHub. Retrieved 2026-09-13, from https://github.com/The-PR-Agent/pr-agent

[^code-review-gpt]: mimo-x. (n.d.). *Code Review GPT Gitlab*. GitHub. Retrieved 2026-09-13, from https://github.com/mimo-x/Code-Review-GPT-Gitlab

[^ai-review]: Nikita-Filonov. (n.d.). *AI Review*. GitHub. Retrieved 2026-09-13, from https://github.com/Nikita-Filonov/ai-review

[^kodus]: kodustech. (n.d.). *Kodus AI*. GitHub. Retrieved 2026-09-13, from https://github.com/kodustech/kodus-ai

[^gito]: Nayjest. (n.d.). *Gito*. GitHub. Retrieved 2026-09-13, from https://github.com/Nayjest/Gito

[^merge-mind]: omidbakhshi. (n.d.). *Merge Mind*. GitHub. Retrieved 2026-09-13, from https://github.com/omidbakhshi/merge-mind