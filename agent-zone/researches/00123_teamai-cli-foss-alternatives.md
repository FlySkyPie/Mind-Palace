# TeamAI CLI 的 FOSS 替代方案

## 概述

Tencent/teamai-cli 是一個基於 Git 的團隊 AI 代理設定同步與共享工具，允許團隊將 skills、rules、MCP servers、hooks、models、env、docs、agents 等 AI 代理資源透過 Git 儲存庫在 Claude Code、Cursor、Copilot CLI、Codex 等多種 AI 編碼工具之間共享[^teamai]。本報告調查其自由及開源替代方案。

## 替代方案總覽

| 工具 | 主要定位 | 同步機制 | 支援工具數 | 資產鎖定 | 授權 |
|---|---|---|---|---|---|
| **Capshelf** | 跨專案固定版本設定共享 | 從獨立資料 repo 複製 | 4+ | ✅ 每個專案 lockfile (Git tree SHA) | MIT |
| **AI Rules Sync** | 多工具聯邦式資產同步 | Symlink 指向全域 Git 快取 | 30+ | ❌ 自動跟隨最新版 | Unlicense |
| **GitAgent** | 完整代理框架與執行環境 | Agent 本身就是 Git repo | N/A (自身即代理) | ✅ Git 原生 | MIT |
| **OpenGAP** | 代理定義標準與轉換器 | 規範 + CLI 可攜式定義 | 匯出至 15+ 框架 | ✅ Git 原生 | MIT |

## 1. Capshelf

- **GitHub**: https://github.com/genged/capshelf
- **授權**: MIT
- **說明**: 以 Git 為後端的 CLI 工具，用來跨多個專案共享 AI 編碼代理設定（skills、Pi extensions、subagents、settings、MCP fragments），**每個專案各自有 lockfile 固定版本**。一個專案的更新不會影響其他專案，直到該專案明確要求更新。內建網頁儀表板（`capshelf ui`）並使用獨立於任何專案的「資料 repo」。[^capshelf]

**主要功能**:
- 管理 assets：skills、Pi extensions、subagents、settings、MCP fragments、Claude/Codex plugin 目錄、Codex config
- 每個專案的 `capshelf.lock.json` 鎖定 Git tree SHA，支援 drift 偵測、提升（promote）、復原（revert）、保留本地版本
- 互動式選取器（tab/enter 操作介面）
- `capshelf ui` — 唯讀儀表板檢視所有專案狀態
- 支援 Claude Code、Codex CLI、Cowork/claude.ai、Pi
- 系統級 asset 內建於 CLI 二進位檔中

**與 TeamAI 的差異**:
- **專案級別固定版本**：TeamAI 全域同步最新版；Capshelf 讓每個專案鎖定特定版本並自行決定何時更新
- **宣告式調解器**：`capshelf.lock.json` 為規格，`capshelf apply` 將檔案系統調解至該規格
- **獨立的資料 repo**：與專案 repo 分離，TeamAI 則將 Git repo 視為來源
- **資產類型更廣**：還管理 Pi extensions 和 Claude plugin marketplace

## 2. AI Rules Sync (AIS)

- **GitHub**: https://github.com/lbb00/ai-rules-sync
- **授權**: Unlicense（公有領域同等）
- **說明**: 命令列工具（`ais`），透過從集中式 Git 儲存庫建立**符號連結**的方式，在 **30 多種 AI 編碼工具**之間同步管理 AI 代理設定。上游變更即時生效於所有專案。[^ais]

**主要功能**:
- 支援 30+ 工具：Cursor、Claude Code、GitHub Copilot、OpenCode、Trae、Cline、Windsurf、Codex、Gemini CLI、Warp、Aider、Augment Code、Kiro 等
- Symlink 式同步 — 編輯一次，所有專案即時生效
- 多 repo 來源 — 不同 asset 可來自不同 Git repo
- 三層設定：專案本機（`ai-rules-sync.local.json`）、專案共享（`ai-rules-sync.json`）、使用者全域（`~/.config/ai-rules-sync/user.json`）
- `ais install` — 從設定還原所有 asset（團隊入職）
- `ais import` — 將本地 asset 拷貝至 Git repo 並以 symlink 取代
- `ais check` / `ais update` — repo 生命週期管理
- 僅 370 KB，5 個相依套件

**與 TeamAI 的差異**:
- **Symlink 而非拷貝**：AIS 使用 symlink 指向全域快取；TeamAI 將檔案拷貝進每個專案。Symlink 意味著上游變更無需執行 `teamai sync` 即立即生效
- **更廣泛的工具支援**：30+ 種工具 vs TeamAI 約 10 種。支援 Aider、Augment Code、Continue、Cline、Windsurf、Goose、Kiro 等
- **三層設定架構**：使用者層級、專案層級、共享層級。TeamAI 使用單一共享 Git repo
- **匯入工作流程**：AIS 可將現有本地 asset 匯入共享 repo

## 3. GitAgent

- **GitHub**: https://github.com/open-gitagent/gitagent
- **授權**: MIT
- **說明**: 一個**通用的 Git 原生 AI 代理框架**，代理程式本身就是一個 Git 儲存庫。身分（SOUL.md）、行為規則（RULES.md）、記憶（Git 提交）、工具（YAML）、skills、hooks、plugins、MCP servers、workflows — 全部是儲存庫內的版本控制檔案。同時提供 CLI 和 SDK。[^gitagent]

**主要功能**:
- 完整代理框架：agent.yaml（manifest）、SOUL.md（身分）、RULES.md（約束）、memory/、tools/、skills/、hooks/、plugins/、workflows/、compliance/
- MCP 客戶端 — 可連接任何 MCP server
- 多模型支援：Anthropic、OpenAI、Google、Groq、Mistral 等
- SDK（`query()`）— 程式內代理執行
- Plugin 系統 — 從 Git URL 安裝 plugin，manifest 自動發現
- 繼承與組合 — 代理可從其他 repo 擴展基礎代理
- 內建 OpenTelemetry 儀器
- 沙箱模式、GitHub repo 模式、語音/Web UI 模式
- 合規與審計：FINRA、SEC、Fed Reserve 合規支援

**與 TeamAI 的差異**:
- **範疇完全不同**：GitAgent 是完整的代理*執行環境/框架*，而非設定同步工具。它直接執行程式代理。TeamAI 是為現有代理分發設定的工具
- **代理即 repo** 的典範：repo 就是代理本身
- **多模型直接執行**：直接呼叫 LLM，而非委派給其他工具
- **Plugin 與繼承生態系統**：支援以可組合方式從其他 repo 擴展/匯入代理定義

## 4. OpenGAP (Open Git Agent Protocol)

- **GitHub**: https://github.com/open-gitagent/opengap
- **授權**: MIT
- **說明**: 一個**框架無關、Git 原生的標準**（規範 + 參考實作 CLI `opengap`），用於將 AI 代理定義為可攜式、版本控制的 Git 儲存庫。標準定義了可在任何代理框架（Claude Code、Cursor、CrewAI、OpenAI、Gemini 等）之間轉換的檔案結構（agent.yaml、SOUL.md、RULES.md 等）。也是 GitAgent 背後的標準化層。[^opengap]

**主要功能**:
- 正式規範（`spec/SPECIFICATION.md`）定義 Git 原生代理格式
- 多格式匯出轉接器：system-prompt、claude-code、openai、crewai、cursor、copilot、codex、gemini、lyzr、github、opencode、kiro、nanobot 等
- `opengap init` — 從範本建立代理（minimal、standard、full）
- `opengap validate --compliance` — 驗證規範 + 法規要求（FINRA、SEC、Fed Reserve）
- `opengap export --format <fmt>` — 匯出至任何支援的框架
- `opengap import --from <fmt> <path>` — 從 Claude、Cursor、CrewAI、OpenCode 匯入
- `opengap run` — 使用任何轉接器從 Git repo 執行代理
- 一等人合規支援：職責分離、人類參與審核、審計日誌、模型風險管理

**與 TeamAI 的差異**:
- **標準而非工具**：OpenGAP 主要是定義代理為 Git repo 的*規範*。TeamAI 是實用同步 CLI
- **雙向轉換**：可將代理定義匯出至任何框架，也可從任何框架匯入
- **合規優先**：內建 FINRA/SEC/SOD 合規
- **標準 vs 工具**：目標是生態系廣泛採用代理即 repo 的概念

## 建議

根據使用場景推薦：

- **需要精確版本控制**（類似套件管理）：選 **Capshelf**，它提供每個專案的 lockfile，變更不會意外影響其他專案
- **需要最廣泛的工具支援**（30+ 種工具）：選 **AI Rules Sync**，symlink 機制讓設定即時同步，Unlicense 授權最自由
- **想建立完整 AI 代理執行環境**：選 **GitAgent** 或 **OpenGAP**（前者是執行框架，後者是可攜式標準）

[^teamai]: Tencent. (n.d.). TeamAI CLI — Make Every Team AI Native. Retrieved 2026-09-25, from https://github.com/Tencent/teamai-cli
[^capshelf]: Genged. (n.d.). Capshelf — Git-backed team config sharing for AI coding agents. Retrieved 2026-09-25, from https://github.com/genged/capshelf
[^ais]: Lbb00. (n.d.). AI Rules Sync — Federated asset sync for 30+ AI coding tools. Retrieved 2026-09-25, from https://github.com/lbb00/ai-rules-sync
[^gitagent]: Open GitAgent. (n.d.). GitAgent — Universal git-native AI agent framework. Retrieved 2026-09-25, from https://github.com/open-gitagent/gitagent
[^opengap]: Open GitAgent. (n.d.). OpenGAP — Open Git Agent Protocol. Retrieved 2026-09-25, from https://github.com/open-gitagent/opengap