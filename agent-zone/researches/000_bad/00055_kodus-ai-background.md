# Kodus AI 背景調查報告

> [!WARNING] 對齊失敗
> "背景調查" 是指創投、資金背景，而非專案簡介。

## 專案概述

Kodus AI（GitHub: `kodustech/kodus-ai`）是一個**開源的 AI Code Review 平台**，定位為「CodeRabbit 的開源替代品」[^repo]。其 AI 審查員名為 **Kody**，能夠直接於 PR 上提供具備上下文感知的程式碼審查、標記風險等級並給出具體修復建議。

## 基本資料

| 項目 | 內容 |
|---|---|
| 倉儲位址 | https://github.com/kodustech/kodus-ai |
| 官方網站 | https://kodus.io |
| 文件網站 | https://docs.kodus.io |
| 雲端服務 | https://app.kodus.io |
| 社群 Discord | https://discord.gg/TFZBRk9fT6 |
| 組織 | kodustech（GitHub 組織） |
| 聯絡人 | Gabriel Malinosqui（可透過 Cal.com 預約對談）[^repo] |
| ⭐ GitHub Stars | 1.4k（截至 2026-09） |
| 🍴 Forks | 151 |
| 📝 Commits | 6,674+ |
| 📦 版本 | v2.1.8（package: `kodus-orchestrator`） |
| 📜 授權 | AGPL-3.0（核心功能開源）+ Enterprise 商業授權雙軌制 |

[^repo]: kodustech. (n.d.). *kodus-ai*. GitHub. Retrieved 2026-09-16, from https://github.com/kodustech/kodus-ai

## 核心特色

### 🔑 Bring Your Own Key（BYOK）
使用者可自行提供 OpenAI、Anthropic、Google Gemini、Vertex AI、Novita 或任何 OpenAI-compatible endpoint 的 API Key。Kodus 不收取任何 LLM Token 加成費用，使用者直接向模型供應商付款[^byok]。

[^byok]: Kodus. (n.d.). *README - Bring Your Own Key*. GitHub. Retrieved 2026-09-16, from https://github.com/kodustech/kodus-ai

### ⚙️ Kody Rules
團隊可以用自然語言定義審查規則，並套用至整個組織、特定儲存庫、路徑或審查範圍。Kody 在審查 PR 時會將這些規則作為上下文來執行[^rules]。

[^rules]: Kodus. (n.d.). *Kody Rules - AI Code Review Rules*. Retrieved 2026-09-16, from https://kodus.io/code-review-rules/

### 📊 Cockpit（儀表板）
提供工程指標儀表板，追蹤審查效果、規則健康度、儲存庫健康度與交付指標[^cockpit]。

[^cockpit]: Kodus. (n.d.). *README - Cockpit*. GitHub. Retrieved 2026-09-16, from https://github.com/kodustech/kodus-ai

### 🧩 Kody Issues
自動追蹤已關閉 PR 中未實作的建議，按狀態、嚴重性、類別與儲存庫管理，並在未來 PR 出現修復時自動解決[^issues]。

[^issues]: Kodus. (n.d.). *README - Kody Issues*. GitHub. Retrieved 2026-09-16, from https://github.com/kodustech/kodus-ai

### 🔎 審查範例
Kody 能夠在真實 PR 中捕獲如 IDOR（Insecure Direct Object Reference）等安全問題並給出明確的修復建議[^repo]。

## 技術架構

### 語言與框架
- **執行環境**: Node.js（TypeScript 6.0.3）
- **後端框架**: NestJS 11.1.19
- **前端**: Next.js（儀表板）
- **套件管理**: pnpm 11.9.0

### 資料庫與訊息佇列
- PostgreSQL（TypeORM）+ pgvector
- MongoDB（Mongoose）
- RabbitMQ（`@golevelup/nestjs-rabbitmq`）

### AI/ML 整合
使用 Vercel AI SDK（v7），支援 Anthropic、OpenAI、Google Gemini/Vertex、Amazon Bedrock、Azure 等多家模型供應商。

### 監控與可觀測性
Sentry、OpenTelemetry、Pyroscope（效能分析）、Langfuse、PostHog。

### 支援的 Git 平台
GitHub、GitLab、Bitbucket、Azure Repos、Forgejo（分別使用 Octokit、GitBeaker 等 SDK）。

## 專案結構（Monorepo）

```txt
kodus-ai/
├── apps/
│   ├── api/          # NestJS API — 認證、組織、團隊、規則、整合、審查編排
│   ├── web/          # Next.js 儀表板
│   ├── worker/       # 背景審查執行、佇列處理、自動化
│   ├── webhooks/     # Webhook 接收（GitHub, GitLab, Azure, Bitbucket, Forgejo）
│   ├── cli/          # CLI 工具（本機與 CI/CD 審查）
│   ├── mcp-manager/  # MCP（Model Context Protocol）管理服務
│   ├── analytics-cli/ # 分析 CLI
│   └── ast-cli/      # AST 工具
├── libs/             # 共用 NestJS 領域模組（含 libs/llm — LLM/BYOK 抽象層）
├── docs/             # Mintlify 文件
├── e2b-sandbox/      # E2B 沙箱環境
├── evals/            # AI 審查品質評估框架
└── tests/            # 端到端測試
```

## 定價方案

| 功能 | Community（免費） | Teams（$10/dev/月） | Enterprise（客製） |
|---|---|---|---|
| 部署方式 | 自架 **或** Kodus 代管 | Kodus 代管 | 自架 **或** Kodus 代管 |
| BYOK | ✅ | ✅ | ✅ |
| PR 使用量 | 無限（用自己的 Key） | 無限（用自己的 Key） | 無限（Kodus Token Key） |
| 使用者數 | 無限 | 無限 | 無限 |
| Kody Rules | 最多 10 條 | 無限 | 無限 |
| 外掛數量 | 最多 3 個 | 無限 | 無限 |
| 優先佇列 | ❌ | ✅ | ✅ |
| Cockpit 指標 | ❌ | ✅ | ✅ |
| SSO | ❌ | ❌ | ✅ |
| RBAC + 審計 | ❌ | ❌ | ✅ |
| SOC 2 | ❌ | ❌ | ✅ |
| 支援 | Discord 社群 | Discord + Email | Private Discord + Email + 5h/月專屬支援 |

費用計算方式: Teams 方案月費為 $10/開發者，另加 LLM Token 費用（直接支付給模型供應商，Kodus 不加價）[^pricing]。

[^pricing]: Kodus. (n.d.). *Kodus Pricing - AI Code Review*. Retrieved 2026-09-16, from https://kodus.io/pricing/

## 隱私與安全

- 原始碼不用於模型訓練
- 傳輸中與靜態資料均加密
- 支援自架 Runner
- 自架實例每日僅發送一次匿名心跳（聚合計數器，不含程式碼、名稱或識別碼），可透過環境變數 `KODUS_TELEMETRY_DISABLED=true` 完全關閉遙測[^telemetry]

[^telemetry]: Kodus. (n.d.). *Anonymous Telemetry*. Kodus Docs. Retrieved 2026-09-16, from https://docs.kodus.io/how_to_deploy/en/deploy_kodus/telemetry

## 競爭定位

Kodus 官網明確標示為 **「The open source alternative to CodeRabbit」**[^website]。官網設有專門的比較頁面：
- [Kodus vs CodeRabbit](https://kodus.io/kodus-vs-coderabbit/)
- [Kodus vs Cursor BugBot](https://kodus.io/kodus-vs-cursor-bugbot/)
- [Kodus vs GitHub Copilot](https://kodus.io/kodus-vs-github-copilot/)
- [Kodus vs Claude](https://kodus.io/kodus-vs-claude/)

[^website]: Kodus. (n.d.). *Kodus - Open Source AI Code Review*. Retrieved 2026-09-16, from https://kodus.io

## 開發活躍度

截至 2026 年 9 月，專案仍處於**高度活躍的開發狀態**：
- 累積超過 6,674 次提交
- 近期 Changelog 顯示頻繁更新至 2026 年 8 月（Linked Repository Context、Findings Sidebar、CLI 體驗改進等）[^changelog]
- 文件支援多國語言（英、葡、西、日、簡中、法）

[^changelog]: Kodus. (n.d.). *Changelog*. Kodus Docs. Retrieved 2026-09-16, from https://docs.kodus.io/changelog

## 使用方式

| 方式 | 說明 |
|---|---|
| ☁️ Kodus Cloud | 至 app.kodus.io 註冊，無需管理基礎設施 |
| 🏠 自架部署 | 支援 Docker Compose / Helm，可部署於 VM 或 Kubernetes |
| 💻 CLI | `kodus review`（審查 working tree）、`kodus review --staged`（暫存變更）、`kodus review --prompt-only` |
| 🤖 CI/CD | 可整合至流水線中自動執行審查 |

## 已知客戶／採用者

官網顯示的 Logo 包含：Rocket.Chat、Open Co、ClickBus、Lerian、QuintoAndar、Mainô 等多家公司[^website]。

## 來源評估

本報告資訊主要來自 GitHub 倉儲頁面與 README、官方網站 kodus.io、定價頁面、Changelog 及文件網站。這些均為一手的官方來源，可信度與即時性高。需要注意的是，SearXNG 搜尋引擎（yacy）在針對 Kodus 的特定查詢（如創辦人背景、第三方評論）時未能返回結果，因此本報告缺乏外部第三方媒體的報導或使用者評測作為佐證。若有需要，建議補充搜尋 Hacker News、Product Hunt 或技術部落格等外部來源以獲得更全面的評估。