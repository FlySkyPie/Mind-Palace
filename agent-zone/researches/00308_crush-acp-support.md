# Crush（charmbracelet/crush）對 ACP（Agent Communication / Client Protocol）的支援情況

## 概述

Crush 是一款由 Charmbracelet 開發的 CLI AI 輔助工具。截至 2026 年 9 月，Crush 的正式版本（`main` 分支）**並未內建支援任何一種 ACP 協定**。但在社群層面，已有多次功能請求、草案 PR 以及第三方套件嘗試補足此缺口。

## 兩種不同的「ACP」

在研究過程中必須先釐清：Crush 社群討論中提及的「ACP」實際上指涉**兩種不同的協定**：

| 協定 | 用途 | 主導者 | 狀態 |
|------|------|--------|------|
| **Agent Client Protocol** | 標準化程式碼編輯器／IDE 與程式碼生成 Agent 之間的通訊 | Zed、Gemini CLI 等 | Crush 社群多數 issue/PR 以此為目標 |
| **Agent Communication Protocol** | Agent 與 Agent 之間的通訊與協作 | IBM / BeeAI（Linux Foundation） | 已合併入 A2A 協定 |

目前 Crush 社群中較活躍的討論集中於**前者（Agent Client Protocol）**，即讓 IDE（如 Zed）能夠透過標準協定呼叫 Crush 作為後端 agent。

## 官方支援狀態

### main 分支

Crush 在 `README.md` 中僅提及支援 **MCP（Model Context Protocol）**，供工具整合使用，完全未提及 ACP。[^readme]

### ACP 分支

存在一個 `acp` 分支，內含貢獻者 Amolith 實作的 ACP 伺服器程式碼，但從未被合併進 `main`。[^acp-branch]

## 相關 Issue 與 PR

### 功能請求

| 編號 | 標題 | 狀態 | 提出日期 |
|------|------|------|----------|
| #990 | Support Agent Client Protocol to integrate crush with IDE's | 開啟 | 2025-09 |
| #2091 | Crush as an ACP client | 開啟 | 2026-02 |

[^issue990]: Charmbracelet. (2025). Support Agent Client Protocol to integrate crush with IDE's [Issue #990]. Retrieved 2026-09-30, from https://github.com/charmbracelet/crush/issues/990
[^issue2091]: Charmbracelet. (2026). Crush as an ACP client [Issue #2091]. Retrieved 2026-09-30, from https://github.com/charmbracelet/crush/issues/2091

### Pull Request

| 編號 | 標題 | 狀態 | 說明 |
|------|------|------|------|
| #1302 | [Draft] ACP mode | 草稿（未合併） | 由 plandem 提出，停留在草稿階段 |
| #1769 | feat: run Crush as an ACP server | **已關閉** | 作者刪除 fork 後關閉；程式碼保留於 `acp` 分支 |

[^pr1302]: plandem. (2025). [Draft] ACP mode [Pull Request #1302]. Retrieved 2026-09-30, from https://github.com/charmbracelet/crush/pull/1302
[^pr1769]: Amolith. (2026). feat: run Crush as an ACP server [Pull Request #1769]. Retrieved 2026-09-30, from https://github.com/charmbracelet/crush/pull/1769

### Bug 回報

| 編號 | 標題 | 狀態 |
|------|------|------|
| #2587 | session/new fails in ACP stdio mode（OpenCode 整合） | 開啟 |

[^issue2587]: Charmbracelet. (2026). session/new fails in ACP stdio mode [Issue #2587]. Retrieved 2026-09-30, from https://github.com/charmbracelet/crush/issues/2587

## 社群第三方套件

外部開發者 aleksclark 提供了一個 Go 套件 `github.com/aleksclark/crush-modules/acp`，可作為 Crush 的外掛使用，支援以下模式：[^community-module]

- **伺服器模式**：將 Crush 暴露為 ACP agent，提供 HTTP API（`POST /runs`、串流等）
- **客戶端模式**：提供 LLM 工具（`acp_list_agents`、`acp_run_agent`、`acp_resume_run`），讓 agent 能發現並呼叫遠端 ACP agent
- 支援 hub-and-spoke 與 peer-to-peer 兩種多 agent 架構

## 維護團隊態度

Crush 維護者在 Issue #2091 中表示：[^maintainer-comment]

> 「我們對 ACP 作為 client 及 server 都持開放態度……但這需要在 client/server 模式下運作，而該模式目前仍在 feature flag（`CRUSH_CLIENT_SERVER=1`）之後。」

顯示團隊**不排斥**納入 ACP 支援，但尚未將其列為優先事項。

## 結論

| 面向 | 狀態 |
|------|------|
| 官方 main 分支支援 | ❌ 不支援 |
| ACP 分支（Amolith 實作） | 存在但未合併 |
| 社群第三方套件 | ✅ 可用（`crush-modules/acp`） |
| 維護團隊態度 | 開放但非優先 |
| 兩種 ACP 的區別 | 社群多數討論針對 Agent Client Protocol（IDE 整合），而非 Agent Communication Protocol（agent 間通訊） |

若使用者現在需要在 Crush 中使用 ACP，可選擇：
1. 使用 `acp` 分支的程式碼自行編譯（Agent Client Protocol 實作）
2. 安裝社群套件 `github.com/aleksclark/crush-modules/acp`
3. 等待官方正式納入支援（尚無明確時間表）

## 參考資料

[^readme]: Charmbracelet. (n.d.). Crush — a CLI tool for working with AI. Retrieved 2026-09-30, from https://github.com/charmbracelet/crush
[^acp-branch]: Charmbracelet. (n.d.). Crush ACP branch. Retrieved 2026-09-30, from https://github.com/charmbracelet/crush/tree/acp
[^community-module]: aleksclark. (n.d.). crush-modules/acp. Retrieved 2026-09-30, from https://pkg.go.dev/github.com/aleksclark/crush-modules/acp
[^maintainer-comment]: Charmbracelet. (2026). Crush as an ACP client [Issue #2091]. Retrieved 2026-09-30, from https://github.com/charmbracelet/crush/issues/2091