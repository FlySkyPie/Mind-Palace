# Kodus AI 專案背景調查報告

## 專案概述

Kodus（kodustech/kodus-ai）是一個**開源 AI 程式碼審查工具**，定位為 CodeRabbit 的開源替代品。其核心產品名為 **Kody**，是一個 AI 程式碼審查代理，能夠直接在 GitHub、GitLab、Bitbucket、Azure Repos 等平台的 Pull Request 中執行程式碼審查。[^github-repo]

## 技術架構

Kodus 採用 **TypeScript** 開發的 monorepo 架構，包含以下主要元件：[^github-repo]

- `apps/api` — NestJS API（認證、組織、團隊、規則、整合）
- `apps/web` — Next.js 儀表板
- `apps/worker` — 背景審查執行與佇列處理
- `apps/webhooks` — Git 平台 Webhook 事件接收
- `libs/llm` — LLM/BYOK 抽象層

支援的模型提供商包括 OpenAI、Anthropic（Claude）、Google Gemini、Vertex AI、Novita，以及任何 OpenAI 相容端點。[^github-readme]

## 授權模式

採用雙重授權模式：[^license]
- **AGPL-3.0** — 除標記為 Enterprise 版本的檔案外，所有原始碼均為 AGPL-3.0
- **商用授權** — 包含 `.ee.` 或 `ee/` 路徑的檔案需另購商用授權

## 商業模式與定價

[^pricing]

| 方案 | 價格 | 適用對象 |
|------|------|----------|
| **Community** | 免費（自託管或 Kodus 代管） | 個人開發者、小型團隊 |
| **Teams** | $10/開發者/月 | 成長中團隊 |
| **Enterprise** | 客製化報價 | 需要 SSO、SOC 2 的大型組織 |

核心商業模式：**平台費 + BYOK（Bring Your Own Key）**—使用者自備 API Key 直接對模型供應商付費，Kodus 不抽取 LLM 費用加成。

專案最初建立於 **2025-03-28**。[^github-api]

截至 2026-09-16，GitHub 專案數據：[^github-api]
- **Stars**: 1,389
- **Forks**: 151
- **Open Issues**: 115
- **Watchers**: 8
- **貢獻者**: 超過 20 人
- **總提交數**: 6,674

## 團隊與創辦人

核心團隊成員主要來自 GitHub 貢獻紀錄：[^github-contributors]

| 貢獻者 | GitHub | 提交數 | 推測職責 |
|--------|--------|--------|----------|
| Wellington01 | @Wellington01 | 2,408 | 核心開發者 |
| sartorijr92 | @sartorijr92 | 1,604 | 後端與 prompt 工程 |
| **Gabriel Malinosqui** | @malinosqui | 1,555 | **創辦人/CEO** |
| jairo-litman | @jairo-litman | 770 | 工程師 |
| stelianok | @stelianok | 71 | 貢獻者 |

創辦人 **Gabriel Malinosqui**（GitHub: malinosqui），來自巴西。Kodus 官方部落格多為葡萄牙文，且眾多客戶案例來自巴西公司（如 Lerian、Notificações Inteligentes、Ikatec、Pilar、Brendi、Doji），強烈暗示團隊位於巴西。[^blog] [^github-contributors]

公司 GitHub 組織 **kodustech** 擁有 94 名追蹤者，旗下包含 53 個儲存庫。[^github-org]

## 社群規模

- **Discord 社群**: 對外開放的支援與討論頻道[^github-repo]
- **GitHub 組織**: 94 followers[^github-org]
- **文件網站**: docs.kodus.io（基於 Mintlify 建置）[^docs]
- **官方網站**: kodus.io

## 融資狀況

**未找到任何公開融資資訊。**

經過多輪搜尋（Crunchbase、YC、VC、funding/seed rounds），均未發現 Kodus 或 Kodustech 有任何外部募資紀錄。所有跡象指向該專案目前為 **bootstrap（自力更生）** 狀態，透過開源社群版吸引使用者，再以 Teams/Enterprise 方案獲利。

## 競品定位

Kodus 明確定位為 CodeRabbit 的開源替代方案，同時也與以下工具競爭：[^pricing]
- GitHub Copilot
- BugBot
- Claude（直接使用）

差異化優勢：
1. **BYOK 模式** — 直接對模型供應商付費，無中間加成
2. **AGPL-3.0 開源** — 完整透明可審視
3. **模型無關** — 自由選擇/更換 LLM 模型
4. **Kody Rules** — 自訂審查規則（純語言描述）

## 風險與注意事項

1. **融資不明** — 無公開募資紀錄，可能面臨資金壓力
2. **團隊規模不明** — 除 GitHub 貢獻者外，無法確認全職團隊人數
3. **競爭激烈** — AI 程式碼審查市場已有 CodeRabbit、Copilot Code Review、Amazon CodeGuru 等成熟產品
4. **AGPL 授權限制** — AGPL-3.0 對商業整合可能構成障礙

## 參考資料

[^github-repo]: kodustech. (n.d.). *kodus-ai*. GitHub. Retrieved 2026-09-16, from https://github.com/kodustech/kodus-ai

[^github-readme]: kodustech. (n.d.). *kodus-ai README*. GitHub. Retrieved 2026-09-16, from https://github.com/kodustech/kodus-ai#readme

[^license]: kodustech. (n.d.). *kodus-ai License*. GitHub. Retrieved 2026-09-16, from https://raw.githubusercontent.com/kodustech/kodus-ai/main/license.md

[^pricing]: Kodus. (n.d.). *Pricing*. Retrieved 2026-09-16, from https://kodus.io/pricing

[^github-api]: GitHub API. (2026). *Repository: kodustech/kodus-ai*. Retrieved 2026-09-16, from https://api.github.com/repos/kodustech/kodus-ai

[^github-contributors]: GitHub API. (2026). *Contributors: kodustech/kodus-ai*. Retrieved 2026-09-16, from https://api.github.com/repos/kodustech/kodus-ai/contributors

[^github-org]: kodustech. (n.d.). *Kodus GitHub Organization*. Retrieved 2026-09-16, from https://github.com/kodustech

[^docs]: Kodus. (n.d.). *Documentation*. Retrieved 2026-09-16, from https://docs.kodus.io

[^blog]: Kodus. (n.d.). *Blog*. Retrieved 2026-09-16, from https://kodus.io/blog