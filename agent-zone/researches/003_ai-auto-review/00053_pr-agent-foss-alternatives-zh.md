# PR-Agent 自由開源替代方案

## 概述

[The-PR-Agent/pr-agent](https://github.com/The-PR-Agent/pr-agent)（即 PR-Agent）是一套由 Qodo 捐贈給社群的開源 AI 程式碼審查工具，採用 MIT 授權條款，主要語言為 Python。[^github-readme] 然而，PR-Agent 的社群維護模式進展緩慢，功能迭代有限，促使許多開發者轉向功能更豐富、更活躍的替代方案。

本報告調查 7 個可自行託管（self-hosted）的自由開源替代方案，涵蓋從 AI 審查引擎到非 AI 的 linter 整合、流程規範與安全掃描等各類選擇。

---

## 替代方案詳述

### 1. Mira — 功能最完整、含儀表板的 AI 審查引擎

| 屬性 | 內容 |
|------|------|
| GitHub Stars | ⭐ 298 |
| 授權 | Apache 2.0 |
| 倉庫 | [github.com/miracodeai/mira](https://github.com/miracodeai/mira)[^mira-repo] |
| 支援平台 | GitHub、GitLab、Forgejo（Bitbucket 開發中） |
| LLM 支援 | 任意模型，透過 OpenRouter、Ollama、vLLM、AWS Bedrock 等 |
| 部署方式 | Docker |

Mira 提供完整的審查引擎、程式碼庫索引、漏洞掃描、自訂規則，以及組織層級的儀表板（含套件庫存、CVE 警報、成本追蹤、審查健康度）。其「學習循環」（learning loop）功能可從已合併 PR 與人工回饋中提煉規則。據開發者聲稱在 Code Review Bench 基準測試中平均約 77 秒完成單次 PR 審查，為同類工具中最快。[^mira-features]

**無付費版本、無授權金鑰、無 SaaS 引導，完全開放原始碼。**

---

### 2. Kodus — 平行多代理架構、氣隙環境適用

| 屬性 | 內容 |
|------|------|
| GitHub Stars | ⭐ 1.4k |
| 授權 | AGPLv3 |
| 倉庫 | [github.com/kodustech/kodus-ai](https://github.com/kodustech/kodus-ai)[^kodus-repo] |
| 支援平台 | GitHub、GitLab、Bitbucket、Azure DevOps（含各平台自管版本） |
| LLM 支援 | OpenAI、Anthropic、Google、Groq、Cerebras、Together AI，或任何 OpenAI 相容端點（含本機 vLLM/Ollama） |
| 部署方式 | Docker Compose（約 15–30 分鐘安裝） |

Kodus 的核心特色是 4 個專業代理平行運作：Bug Agent、Security Agent、Performance Agent、KodyRules Agent。語意去重（semantic dedup）與嚴重性排序（critical/high/medium/low）減少噪音。支援 SSO（SAML）、RBAC、稽核日誌（Enterprise 版）。氣隙部署模式下程式碼完全不出 VPC。[^kodus-features]

---

### 3. OpenReview (Vercel Labs) — 沙箱執行、自動修復

| 屬性 | 內容 |
|------|------|
| GitHub Stars | ⭐ 1.7k |
| 授權 | MIT |
| 倉庫 | [github.com/vercel-labs/openreview](https://github.com/vercel-labs/openreview)[^openreview-repo] |
| 支援平台 | GitHub |
| LLM 支援 | Anthropic Claude（透過 AI SDK） |
| 部署方式 | Vercel |

OpenReview 可部署至 Vercel，透過 GitHub App 整合。使用者可透過 `@openreview` 留言觸發按需審查。其獨特之處在於利用 **Vercel Sandbox** 沙箱執行 linter/test，並能直接產生 GitHub suggestion block 實現一鍵修復，甚至可直接推送格式化/lint 修正至 PR 分支。[^openreview-features]

**注意：此為測試中專案，原為 Vercel 內部工具。**

---

### 4. Gito — 最高設定彈性、供應商中立的 Python 工具

| 屬性 | 內容 |
|------|------|
| GitHub Stars | ⭐ 430 |
| 授權 | MIT |
| 倉庫 | [github.com/Nayjest/Gito](https://github.com/Nayjest/Gito)[^gito-repo] |
| 支援平台 | GitHub、GitLab、CLI／任意 CI |
| LLM 支援 | 任意供應商：OpenAI、Anthropic、Google、Ollama、vLLM、llama.cpp、LM Studio 等 |
| 部署方式 | `pip install gito.bot` 或獨立 Windows 安裝程式 |

Gito 採無狀態、零保留設計，程式碼直接傳送至 LLM 供應商。完整設定透過 `.gito/config.toml`，支援自訂審查提示詞、範本、後處理。可整合 Jira 與 Linear，適用於本機 CLI、CI/CD 管線、GitHub Action 或 GitLab CI 任務。[^gito-features]

---

### 5. Reviewdog — 最受歡迎的 Linter 結果 PR 評論工具

| 屬性 | 內容 |
|------|------|
| GitHub Stars | ⭐ 9.6k |
| 授權 | MIT |
| 倉庫 | [github.com/reviewdog/reviewdog](https://github.com/reviewdog/reviewdog)[^reviewdog-repo] |
| 支援平台 | GitHub、GitLab、Bitbucket 等 |
| 特色 | 非 AI，但可整合任意 linter 至 PR 評論 |
| 部署方式 | GitHub Action／CLI／Docker |

Reviewdog 是一款以 Go 撰寫的 PR 評論發布工具，可將任意 linter（如 ESLint、golangci-lint、Ruff）的輸出結果發布為 PR 中的行內評論（inline comment）。支援 `reviewdog` 本身也內建數十種執行器（runner），涵蓋主流語言。它不僅能自動建議修復（autofix），還支援 diff 範圍過濾，只評論修改行。[^reviewdog-features]

**不依賴任何 LLM，完全免費、輕量、快速。**

---

### 6. Danger — PR 規範強制引擎

| 屬性 | 內容 |
|------|------|
| GitHub Stars | ⭐ 5.7k（Ruby）+ 5.5k（JS） |
| 授權 | MIT |
| 倉庫 | [github.com/danger/danger](https://github.com/danger/danger)（Ruby）、[github.com/danger/danger-js](https://github.com/danger/danger-js)（JS）[^danger-repo] |
| 支援平台 | GitHub、GitLab、Bitbucket 等 |
| 特色 | 透過程式碼強制 PR 規範（描述長度、標籤、變更範圍等） |
| 部署方式 | CLI 或 CI 整合（GitHub Action／Jenkins 等） |

Danger 讓團隊透過程式碼（Swift、Ruby、JavaScript 等語言）來定義 PR 的自動化檢查規則，例如：要求 PR 描述必須包含特定格式、禁止超過一定行數的變更、確保 CHANGELOG 有更新、要求關聯 Issue 等。執行結果會以評論形式發布至 PR 中。支援豐富的 plugin 生態系。[^danger-features]

**非 AI 工具，專注於 PR 流程自動化規範。**

---

### 7. Semgrep — 靜態分析與安全掃描引擎

| 屬性 | 內容 |
|------|------|
| GitHub Stars | ⭐ 16.6k |
| 授權 | LGPL-2.1（社群版） |
| 倉庫 | [github.com/semgrep/semgrep](https://github.com/semgrep/semgrep)[^semgrep-repo] |
| 支援平台 | GitHub、GitLab、Bitbucket、CI/CD 任意平台 |
| 特色 | 30+ 語言、3000+ 社群規則、CI 掃描約 10 秒 |
| 部署方式 | CLI／CI 整合／自行託管 |

Semgrep 是 r2c 開發的靜態分析引擎，社群版（Community Edition）採用 LGPL-2.1 授權，完全可自行託管。支援 30 種以上語言，內建 3000+ 社群規則。掃描速度極快，約 10 秒即可完成單次 CI 掃描。能抓出 linter 無法發現的邏輯漏洞與安全弱點，整合至 CI 管線後可自動對 PR 發布評論。[^semgrep-features]

**付費版含 AI 分類功能，社群版為規則引擎。**

---

## 比較總表

| 工具 | GitHub Stars | 授權 | 自託管 | 支援平台 | LLM 控制 | 核心差異 |
|------|-------------|------|--------|----------|----------|---------|
| **Mira** | ⭐ 298 | Apache 2.0 | ✅ Docker | GitHub、GitLab、Forgejo | 自帶金鑰（任意模型） | 儀表板 + 學習循環 + 最快基準 |
| **Kodus** | ⭐ 1.4k | AGPLv3 | ✅ Docker Compose | GitHub、GitLab、Bitbucket、Azure DevOps | 自帶金鑰（任意模型） | 4 代理平行審查 + 氣隙環境 |
| **OpenReview** | ⭐ 1.7k | MIT | ✅ Vercel | GitHub | Claude 限定 | 沙箱執行 + 自動修復 |
| **Gito** | ⭐ 430 | MIT | ✅ pip／CLI | GitHub、GitLab | 自帶金鑰（任意模型） | 最高設定彈性 + Jira/Linear 整合 |
| **Reviewdog** | ⭐ 9.6k | MIT | ✅ CLI／Action | GitHub、GitLab、Bitbucket | 非 AI，Linter 整合 | 最受歡迎的 PR 評論發布工具 |
| **Danger** | ⭐ 5.7k（Ruby）、5.5k（JS） | MIT | ✅ CLI／CI | GitHub、GitLab、Bitbucket | 非 AI，流程規範 | PR 規範強制引擎 |
| **Semgrep** | ⭐ 16.6k | LGPL-2.1 | ✅ CLI／CI | GitHub、GitLab、Bitbucket | 非 AI（付費版含 AI） | 30+ 語言靜態分析安全掃描 |

---

## 選擇建議

- **追求完整 AI 平台體驗**：Mira（儀表板 + 學習循環）或 Kodus（多代理平行審查 + 氣隙）
- **輕量非 AI 方案**：Reviewdog（將任意 linter 輸出發布為 PR 評論）
- **PR 流程自動化**：Danger（程式化定義 PR 規範）
- **安全掃描整合**：Semgrep（CI 內 10 秒完成 30+ 語言靜態分析）
- **最高設定彈性**：Gito（pip 安裝、Jira/Linear 整合、自訂提示詞）
- **快速原型或自動修復**：OpenReview（Vercel 部署、沙箱執行）

[^github-readme]: The-PR-Agent. (n.d.). PR-Agent: The Original Open-Source PR Reviewer. Retrieved 2026-09-13, from https://github.com/The-PR-Agent/pr-agent
[^mira-repo]: MiraCodeAI. (n.d.). Mira: AI Code Review Agent with Dashboard. Retrieved 2026-09-13, from https://github.com/miracodeai/mira
[^mira-features]: MiraCodeAI. (n.d.). Mira Features Overview. Retrieved 2026-09-13, from https://github.com/miracodeai/mira
[^kodus-repo]: KodusTech. (n.d.). Kodus AI: multi-agent code review. Retrieved 2026-09-13, from https://github.com/kodustech/kodus-ai
[^kodus-features]: Kodus. (n.d.). Self-Hosted AI Code Review. Retrieved 2026-09-13, from https://kodus.io/self-hosted-ai-code-review/
[^openreview-repo]: Vercel Labs. (n.d.). OpenReview: AI-powered code review. Retrieved 2026-09-13, from https://github.com/vercel-labs/openreview
[^openreview-features]: Vercel Labs. (n.d.). OpenReview README. Retrieved 2026-09-13, from https://github.com/vercel-labs/openreview
[^gito-repo]: Nayjest. (n.d.). Gito: Vendor-agnostic AI code review. Retrieved 2026-09-13, from https://github.com/Nayjest/Gito
[^gito-features]: Nayjest. (n.d.). Gito README. Retrieved 2026-09-13, from https://github.com/Nayjest/Gito
[^reviewdog-repo]: reviewdog. (n.d.). reviewdog: Automated code review tool. Retrieved 2026-09-13, from https://github.com/reviewdog/reviewdog
[^reviewdog-features]: reviewdog. (n.d.). reviewdog README. Retrieved 2026-09-13, from https://github.com/reviewdog/reviewdog
[^danger-repo]: Danger. (n.d.). Danger: Automate your team's conventions. Retrieved 2026-09-13, from https://github.com/danger/danger
[^danger-features]: Danger. (n.d.). Danger README. Retrieved 2026-09-13, from https://github.com/danger/danger
[^semgrep-repo]: r2c. (n.d.). Semgrep: Static analysis engine. Retrieved 2026-09-13, from https://github.com/semgrep/semgrep
[^semgrep-features]: r2c. (n.d.). Semgrep documentation. Retrieved 2026-09-13, from https://semgrep.dev/