# TeamAI CLI 替代方案調查

## 概述

TeamAI CLI 是騰訊開發的開源 CLI 工具，以 Git 為基礎，作為團隊級 AI Agent 設定的分發層。其核心理念為「讓每個團隊都成為 AI Native」。它將個人 AI Agent 設定（技能、規則、MCP 伺服器、Hooks、環境變數、文件、Agent、模型）存放在共用 Git 儲存庫中，管理員發布更新後，團隊成員執行一次 `teamai init` 即可讓本機工具連結至此儲存庫，後續每次 AI 工作階段都會自動同步最新設定。其流程遵循標準 Git 工作流程：**push → review & merge → pull (session 啟動時自動同步)**。[^teamai]

本報告旨在找出其 FOSS 替代方案，並以 GitHub Star 數量作為重要篩選標準。

## TeamAI CLI 的功能層

TeamAI CLI 包含三個架構層：[^teamai]

| 層級 | 用途 | 涵蓋內容 |
|------|------|----------|
| **Team Execution** | 讓每個 Agent 以團隊方式運作 | 技能、規則、文件、環境變數、Agent、Hooks、MCP、模型 — 跨所有成員同步 |
| **Team Context** (beta) | 讓每個 Agent 理解團隊背景 | 摩擦導向的經驗分享、子 Agent 知識召回、程式碼知識圖譜 (AST-based) |
| **Team Improvement** (beta) | 讓每次執行改善團隊 | 使用追蹤、Session 摘要、週報、互動儀表板 |

## 直接替代方案（同一類別 — 小型專案）

經過廣泛搜尋，**目前沒有任何直接替代 TeamAI CLI 的專案擁有顯著 Star 數量**。這一類別（團隊級 Git-based AI 設定同步/管理）是新興領域，現有專案均低於 40 顆 Star。[^versionman]

| 專案 | Stars | 語言 | 說明 |
|------|-------|------|------|
| [ai-rules-sync](https://github.com/lbb00/ai-rules-sync) | 38 ⭐ | TypeScript | 透過 symlink 從 Git 儲存庫同步 Agent 規則、技能、指令至 34 種工具。採用資產聯邦 (Asset Federation) 方式。[^airulessync] |
| [Plexus](https://github.com/miniLV/Plexus) | 30 ⭐ | TypeScript | 本機儀表板，在 Claude Code、Codex、Gemini CLI、Qwen Code 等工具間同步規則、MCP 與技能。支援一鍵配置。[^plexus] |
| [agent-dotfiles](https://github.com/saqibameen/agent-dotfiles) | 13 ⭐ | Python | 讀取現有的 AGENTS.md/CLAUDE.md 並傳播至所有 Agent。較 TeamAI 簡化。[^agentdotfiles] |
| [team-ai-sync](https://github.com/paladini/team-ai-sync) | 10 ⭐ | YAML | CI/CD Action (GitHub Action、GitLab Component、Bitbucket Pipe)，透過 PR 跨儲存庫同步團隊 AI 資產。[^teamaisync] |
| [ai-agent-config](https://github.com/dongitran/ai-agent-config) | 3 ⭐ | TypeScript | 通用技能與工作流程管理器，支援 Claude Code、Cursor、Codex 等的雙向 GitHub 同步。[^aiagentconfig] |
| [agentsync](https://github.com/chrisleekr/agentsync) | 1 ⭐ | TypeScript | 加密 Git 備份庫，跨機器管理 AI Agent 設定。支援快照、加密、還原。[^agentsync] |
| [agent-sync](https://github.com/JustinBeaudry/agent-sync) | 1 ⭐ | Go | Agent 版 dotfiles。從單一 Git manifest 將技能、規則、指令、MCP 同步至多種工具。[^agentsync2] |
| [repo-agents-sync](https://github.com/redirwin/repo-agents-sync) | 2 ⭐ | Shell | 單一 `.agents/` 資料夾鏡像至 Cursor、Claude Code、Copilot。簡易複製/symlink 腳本。[^repoagentssync] |

## 相關但不同類別（高 Star 專案）

以下專案**並非直接替代方案** — 它們解決不同問題 — 但屬於更廣泛的 AI Agent 設定生態系：

| 專案 | Stars | 類別 | 說明 |
|------|-------|------|------|
| [Superpowers (obra)](https://github.com/obra/superpowers) | 282k ⭐ | Agentic 技能框架 | 技能框架 + 開發方法論，非團隊設定同步工具。[^superpowers] |
| [mattpocock/skills](https://github.com/mattpocock/skills) | 250k ⭐ | 技能集合 | 可重複使用的 Agent 技能集合，非團隊分發基礎設施。[^mattpocock] |
| [anthropics/skills](https://github.com/anthropics/skills) | 175k ⭐ | 官方技能庫 | Claude Code 官方技能儲存庫，純技能內容，非同步基礎設施。[^anthropic] |
| [Spec-Kit (github)](https://github.com/github/spec-kit) | 132k ⭐ | 規格驅動開發工具集 | 規格驅動開發工具包，非設定同步。[^speckit] |
| [OpenSpec (Fission-AI)](https://github.com/Fission-AI/OpenSpec) | 69k ⭐ | 規格驅動開發框架 | 規格驅動開發框架，非團隊設定分發。[^openspec] |
| [GSD Core](https://github.com/open-gsd/gsd-core) | 61k ⭐ | 規格驅動上下文工程 | Session 管理框架，非團隊設定同步。[^gsdcore] |

## 分析與結論

### 直接替代方案現狀

TeamAI CLI（~5k Stars）所處的「團隊級 Git-based AI Agent 設定同步」類別非常新，2026 年 4 月才誕生。目前這一類別中沒有任何專案達到可被認為「不冷門」的 Star 數。最接近的替代專案（ai-rules-sync 38 ⭐、Plexus 30 ⭐）在功能範圍、團隊協作支援、Agent 支援數量上均遠不如 TeamAI CLI。

### 使用場景建議

1. **若需要完整團隊級 AI 設定同步**：TeamAI CLI 是目前此類別中唯一成熟的選擇（5k Stars、騰訊背書、支援 16+ Agent）。
2. **若只需個人跨工具同步**：可考慮 Plexus（30 ⭐）或 ai-rules-sync（38 ⭐），但需注意其規模與成熟度有限。
3. **若需要高 Star 的生態系工具**：Superpowers、Spec-Kit、OpenSpec 等高 Star 專案屬於不同類別 — 它們是技能框架或開發方法論，而非設定同步工具，無法取代 TeamAI CLI 的核心功能。

### 關於 Star 數量的方法論反思

本報告將 GitHub Star 數量作為「不冷門」的篩選標準。需注意 Star 數量本身有其限制：[^starwarning]
- 高 Star 不一定代表專案成熟或維護良好
- 低 Star 不一定代表專案品質差 — 某些優質專案只是尚未被廣泛發現
- 在此新興類別中，Star 數量未能反映專案的實用性；所有現有專案都處在早期階段

## 參考資料

[^teamai]: Tencent. (2026). TeamAI CLI - Make Every Team AI Native. Retrieved 2026-09-25, from https://github.com/Tencent/teamai-cli

[^airulessync]: lbb00. (2025). ai-rules-sync - Synchronize, manage, and share your AI rules. Retrieved 2026-09-25, from https://github.com/lbb00/ai-rules-sync

[^plexus]: miniLV. (2026). Plexus - One-click local setup for MCP servers, skills, and rules. Retrieved 2026-09-25, from https://github.com/miniLV/Plexus

[^agentdotfiles]: saqibameen. (2026). agent-dotfiles. Retrieved 2026-09-25, from https://github.com/saqibameen/agent-dotfiles

[^teamaisync]: paladini. (2026). team-ai-sync. Retrieved 2026-09-25, from https://github.com/paladini/team-ai-sync

[^aiagentconfig]: dongitran. (2026). ai-agent-config. Retrieved 2026-09-25, from https://github.com/dongitran/ai-agent-config

[^agentsync]: chrisleekr. (2026). agentsync. Retrieved 2026-09-25, from https://github.com/chrisleekr/agentsync

[^agentsync2]: JustinBeaudry. (2026). agent-sync. Retrieved 2026-09-25, from https://github.com/JustinBeaudry/agent-sync

[^repoagentssync]: redirwin. (2026). repo-agents-sync. Retrieved 2026-09-25, from https://github.com/redirwin/repo-agents-sync

[^superpowers]: obra. (2025). Superpowers. Retrieved 2026-09-25, from https://github.com/obra/superpowers

[^mattpocock]: mattpocock. (2025). skills. Retrieved 2026-09-25, from https://github.com/mattpocock/skills

[^anthropic]: Anthropic. (2025). skills. Retrieved 2026-09-25, from https://github.com/anthropics/skills

[^speckit]: GitHub. (2025). Spec-Kit. Retrieved 2026-09-25, from https://github.com/github/spec-kit

[^openspec]: Fission-AI. (2025). OpenSpec. Retrieved 2026-09-25, from https://github.com/Fission-AI/OpenSpec

[^gsdcore]: open-gsd. (2025). GSD Core. Retrieved 2026-09-25, from https://github.com/open-gsd/gsd-core

[^versionman]: VersionMan. (2026). AI Agent Skills Ecosystem 2026. Retrieved 2026-09-25, from https://versionman.com/blog/tools/agent-skills-ecosystem-2026.html

[^starwarning]: 本報告使用 GitHub API 於 2026-09-25 擷取各專案 Star 數據。Star 數量為動態指標，可能隨時間變化。