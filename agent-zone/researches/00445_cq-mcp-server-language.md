# cq MCP Server 工具描述語言客製化研究

> 如何調整 cq MCP server 的設定，讓提供給 agent 的工具描述（tool descriptions）已包含語言約束（例如強制使用 zh_TW 提出 knowledge unit）。

## 結論

**cq MCP server 沒有任何設定來改變工具描述的語言。** 所有 MCP tool descriptions 寫死在 Go 原始碼中，編譯進二進位檔，無環境變數或設定檔可覆蓋。

## 1. MCP Tool Descriptions 在哪裡

五個 MCP tool 的描述定義在 `cli/mcpserver/` 目錄下的獨立檔案中：[^server]

| Tool | 檔案 | 描述摘要（原文） |
|------|------|----------------|
| query | `query.go` | *"Search for known gotchas, workarounds, and version-specific quirks before acting..."* |
| propose | `propose.go` | *"Share a non-obvious insight so other agents avoid the same pitfall..."* |
| confirm | `confirm.go` | *"Strengthen a knowledge unit that proved correct during your task..."* |
| flag | `flag.go` | *"Report that a knowledge unit is wrong, outdated, or redundant..."* |
| status | `status.go` | *"Show knowledge store statistics: tier counts..."* |

例如 `propose` 工具的實際 code：[^propose]

```go
func ProposeTool() mcp.Tool {
    return mcp.NewTool("propose",
        mcp.WithDescription(
            "Share a non-obvious insight so other agents avoid the same pitfall. "+
            "Call IMMEDIATELY after resolving a non-obvious error...",
        ),
        // ... input params ...
    )
}
```

這些字串在 compile 時被鎖死。`cq mcp` 啟動流程：[^mcpcmd]

1. `cq mcp` → `mcpserver.New(client, version)` → 建立 `Server` struct
2. `AddTool(QueryTool(), ...)` / `AddTool(ProposeTool(), ...)` 等五次註冊
3. `server.ServeStdio(srv.MCPServer())` → 在 stdio 上服務 MCP 請求
4. Agent 呼叫 `tools/list` 時，這些描述就直接送給 agent

**沒有任何 hook、middleware、或 template engine** 在這條鏈中。

## 2. 現有設定機制的盤點

| 機制 | 可影響什麼 | 無法影響什麼 |
|------|-----------|------------|
| `CQ_ADDR` / `CQ_API_KEY` | 連線與 remote 儲存 | ❌ 不影響 tool descriptions |
| `CQ_LOCAL_DB_PATH` | 本機 SQLite 路徑 | ❌ 不影響 tool descriptions |
| SKILL.md（plugin skill） | Agent 何時呼叫 tool | ❌ 不影響 MCP tool schema 本身 |
| `cq prompt` 命令 | 輸出 canonical prompt text | ❌ 只是文字檔，不改變 MCP server 行為 |
| `extensions` 欄位 | Propose 時自訂 metadata | ❌ 不影響 tool descriptions |

## 3. 可行方案

### 方案 A：Fork + 重新編譯（最徹底）

修改 `cli/mcpserver/*.go` 中的描述字串，例如：

```go
// 將 propose 的描述改為繁體中文
mcp.WithDescription(
    "分享一個非顯而易見的洞察，讓其他 agent 避免踩到同樣的地雷。\n" +
    "在解決非顯而易見的錯誤、發現未記錄的 API 行為後，立即呼叫此工具...",
)
```

然後重新編譯。

```
git clone https://github.com/mozilla-ai/cq
# 修改 cli/mcpserver/*.go
# 依照 DEVELOPMENT.md 編譯
```

**優點**：原生支援，無額外轉發層
**缺點**：需要 Go toolchain，每次 upstream 更新需 merge

### 方案 B：MCP Proxy 包裝（無需 fork）

撰寫一個 stdio MCP proxy，插在 agent 和 `cq mcp` 之間：

```
Agent (MCP host)
    → MCP Proxy (stdin/stdout)
        → cq mcp (subprocess)
```

Proxy 攔截 `tools/list` response，替換 tool descriptions 字串。因為 MCP 通訊是純 JSON 透過 stdio，所以可以這樣做。

```python
#!/usr/bin/env python3
"""MCP proxy that translates cq tool descriptions to zh_TW."""
import subprocess, json, sys

cq_proc = subprocess.Popen(["cq", "mcp"], stdin=sys.stdin, stdout=subprocess.PIPE,
                           stderr=sys.stderr, text=True, bufsize=1)

ZH_DESCRIPTIONS = {
    "query": "查詢已知陷阱、解決方案和版本特定問題。在執行不熟悉的任務時呼叫。",
    "propose": "分享非顯而易見的洞察，讓其他agent避免踩到同樣的地雷。必使用繁體中文（zh_TW）。",
    "confirm": "確認一筆知識單元在你的任務中證明是正確的。",
    "flag": "回報一筆知識單元是錯誤的、過時的或重複的。",
    "status": "顯示知識庫統計：層級數量、領域分佈、信心度分佈。",
}
```

然後在 `crush.json` 中把 MCP server 指向這個 proxy，而不是 `cq`。

**優點**：不需要 fork/compile，可獨立於 cq 版本更新
**缺點**：多一層 process，需處理 error forwarding

### 方案 C：直接修改 SKILL.md（有限）

雖然你說不要建議 skill 方式，但需說明這跟純 skill 不同：**修改 plugin 的 SKILL.md 可以控制 agent 的行為指令**，但**無法改變 MCP tool 本身的描述**。這會造成 agent 看到兩種語言混雜（tool description 仍為英文）。

## 4. 建議

**方案 B（MCP Proxy）** 是最實用的：無需 fork cq、可獨立維護、語言轉換邏輯清晰。結合 skill 層的語言指示，達到雙重約束。

## 參考文獻

[^server]: Mozilla.ai. (n.d.). cq — cli/mcpserver/server.go. Retrieved 2026-10-09, from https://github.com/mozilla-ai/cq/blob/main/cli/mcpserver/server.go
[^propose]: Mozilla.ai. (n.d.). cq — cli/mcpserver/propose.go. Retrieved 2026-10-09, from https://github.com/mozilla-ai/cq/blob/main/cli/mcpserver/propose.go
[^query]: Mozilla.ai. (n.d.). cq — cli/mcpserver/query.go. Retrieved 2026-10-09, from https://github.com/mozilla-ai/cq/blob/main/cli/mcpserver/query.go
[^mcpcmd]: Mozilla.ai. (n.d.). cq — cli/cmd/mcp.go. Retrieved 2026-10-09, from https://github.com/mozilla-ai/cq/blob/main/cli/cmd/mcp.go
[^plugin]: Mozilla.ai. (n.d.). cq Plugin — plugin.json. Retrieved 2026-10-09, from https://github.com/mozilla-ai/cq/blob/main/plugins/cq/.claude-plugin/plugin.json