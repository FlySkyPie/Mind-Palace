# Crush 的 fetch 功能是否已被提取並重構為 MCP 伺服器？

## 調查摘要

截至 **2026 年 10 月**，**沒有發現任何專案** 將 Crush (https://github.com/charmbracelet/crush) 內建的 fetch 相關功能提取出來，並重構成獨立的 MCP (Model Context Protocol) 伺服器或工具。

## 調查方法

- 使用多組搜尋字詞組合進行網路搜尋，包括："crush fetch mcp"、"charmbracelet crush fetch extracted"、"crush agentic_fetch mcp server standalone"、"crush fetch tool refactor mcp" 等。
- 檢視 Crush 的 GitHub forks 頁面前兩頁（約 50 個 fork），逐一確認有無提取工具的行為。
- 搜尋相關部落格文章與討論串。

## Crush 內建的 fetch 工具

Crush 原始碼中（Go 語言）內建三種 fetch 相關工具，均位於 `internal/agent/tools/` 目錄：[^crush-fetch-source]

1. **`fetch`** (`fetch.go`) — 直接取得 URL 內容，支援 `text`、`markdown`、`html` 三種格式，有 100KB 限制與權限提示系統。
2. **`web_fetch`** (`web_fetch.go`) — 簡化版本，專為子代理設計，無權限提示；超過 50KB 的內容會存到暫存檔。
3. **`agentic_fetch`** (`fetch_types.go` + UI 在 `agent.go`) — 使用子代理進行推理式搜尋，能自行決定搜尋方式並反覆擷取直到找到資訊為止。

三者均深度整合在 Crush 的 `fantasy.AgentTool` 架構中，提取難度較高。

## 相近但無關的專案

### Crush 的分支 (forks)

| Fork | 說明 |
|------|------|
| **bwl/cliffy** (3★) [^cliffy] | 無 TUI、無資料庫、無 session 的 headless 分支，重用 Crush 整個工具系統，但**並未提取單一工具**為 MCP 伺服器。 |
| **example-git/crux** (1★) [^crux] | 獨立維護的衍生版，新增自訂 provider 插件系統與遠端工作區，保留 Crush 內部工具架構。 |
| **amarbel-llc/trapeze** (1★) [^trapeze] | 改名鏡像分支，無結構性變更。 |
| **unleg1t/crush-plus** (1★) [^crush-plus] | 極簡 rebrand，無功能變化。 |
| **meistro57/crushed** (1★) [^crushed] | 同樣是改名分支。 |

### MCP 官方 fetch 參考伺服器

MCP 官方組織 (`modelcontextprotocol`) 提供了一個標準的 fetch 參考伺服器（Python/Node.js 實作）[^mcp-fetch]，但此專案：
- **並非**衍生自 Crush，也與 Crush 無關
- 是通用的 HTTP 轉 Markdown 工具
- 以 PyPI / npm 套件形式發佈

## 結論

目前在 GitHub、網路搜尋及已知分支中，**不存在將 Crush 的 fetch 功能提取並重構成 MCP 伺服器的專案**。若要實現此目標，需要從 Crush 的 Go 原始碼（`fetch.go` / `web_fetch.go`）移植其使用 `goquery` 進行 HTML 解析、以及 `html-to-markdown` 進行格式轉換的邏輯，重新實作為 MCP 伺服器。

## 參考來源

[^crush-fetch-source]: charmbracelet. (n.d.). Crush — internal/agent/tools/fetch.go, web_fetch.go, fetch_types.go. Retrieved 2026-10-01, from https://github.com/charmbracelet/crush/tree/main/internal/agent/tools
[^cliffy]: bwl. (n.d.). cliffy — a headless Crush fork. Retrieved 2026-10-01, from https://github.com/bwl/cliffy
[^crux]: example-git. (n.d.). crux — independently maintained Crush derivative. Retrieved 2026-10-01, from https://github.com/example-git/crux
[^trapeze]: amarbel-llc. (n.d.). trapeze — renamed Crush mirror fork. Retrieved 2026-10-01, from https://github.com/amarbel-llc/trapeze
[^crush-plus]: unleg1t. (n.d.). crush-plus — minimal rebrand of Crush. Retrieved 2026-10-01, from https://github.com/unleg1t/crush-plus
[^crushed]: meistro57. (n.d.). crushed — renamed Crush fork. Retrieved 2026-10-01, from https://github.com/meistro57/crushed
[^mcp-fetch]: modelcontextprotocol. (n.d.). MCP fetch reference server. Retrieved 2026-10-01, from https://github.com/modelcontextprotocol/servers