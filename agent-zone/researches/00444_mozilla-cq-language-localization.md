# Mozilla cq 的語言處理機制與自然語言本地化分析

## 概述

Mozilla cq 是一個開放標準的共享 agent 知識庫系統，讓 AI agent 之間可以查詢、提交、確認、標記知識單元 (Knowledge Units)。本報告分析 cq 目前的語言處理機制，特別是**程式語言**與**人類自然語言**的區別，以及如何讓 MCP server 要求 agent 以特定語言提出知識。

## 核心發現：`languages` 指的是程式語言，非人類語言

cq 的知識單元 schema (`knowledge_unit.json`) 在 `Context` 區塊中定義了一個 `languages` 欄位，**但這是指程式語言 (programming languages)**，例如 `"python"`、`"go"`、`"rust"`，而非人類自然語言（如繁體中文、英文）。[^1]

```json
"Context": {
  "type": "object",
  "description": "Language, framework, and pattern context.",
  "properties": {
    "languages": {
      "type": "array",
      "items": { "type": "string", "maxLength": 64 },
      "maxItems": 16
    },
    "frameworks": { ... },
    "pattern": { ... }
  }
}
```

整個 cq 專案中**不存在**任何 `locale`、`human_language`、`natural_language` 或類似的人類自然語言欄位。

## MCP 工具中的語言參數

### `propose` 工具

MCP `propose` 工具接受 `languages` 作為選擇性陣列參數，文件說明為：「Programming languages the insight applies to (e.g. "python", "go").」[^2]

```go
params := cq.ProposeParams{
    Summary:    summary,
    Detail:     detail,
    Action:     action,
    Domains:    domains,
    Languages:  req.GetStringSlice("languages", nil),  // optional
    Frameworks: req.GetStringSlice("frameworks", nil),
    Pattern:    req.GetString("pattern", ""),
    Extensions: extensions,
}
```

### `query` 工具

MCP `query` 工具同樣接受 `languages` 作為選擇性陣列參數： [^3]

```go
params := cq.QueryParams{
    Domains:    domains,
    Languages:  req.GetStringSlice("languages", nil),
    Frameworks: req.GetStringSlice("frameworks", nil),
    Pattern:    req.GetString("pattern", ""),
    Limit:      limit,
}
```

### `confirm` 與 `flag` 工具

這兩個工具**沒有任何語言相關的參數**。`confirm` 只接受 `unit_id`，`flag` 接受 `unit_id`、`reason`、`detail`、`duplicate_of`。[^4]

## Schema 驗證限制

`schema/limits.go` 與 `schema/values.go` 定義了相關限制：[^5]

| 欄位 | 最大值 | 說明 |
|-------|-------|------|
| `languages` 單項長度 (maxLength) | 64 字符 | 每個語言標籤上限 |
| `languages` 陣列長度 (maxItems) | 16 項 | 最多可標註 16 種語言 |
| `language_weight` (scoring) | 0.15 | 語言匹配在相關性評分中的權重 |

語言標籤在存儲時會自動轉為小寫並去重。[^6]

## 如何在 MCP 層面加入自然語言支援

由於 cq 目前**完全沒有**人類自然語言的概念，要實現「讓 MCP server 要求 agent 以特定語言提出知識」，有以下可行路徑：

### 方案 A：使用 `extensions` 擴充欄位（立即可行，無需改動 schema）

`KnowledgeUnit` 的 `extensions` 欄位是 `additionalProperties: true` 的開放物件，可用於存放實作特定的額外資料：[^7]

```json
{
  "summary": "某個發現",
  "detail": "詳細說明",
  "action": "具體操作",
  "domains": ["database"],
  "extensions": {
    "myimpl:language": "zh-TW",
    "myimpl:locale": "zh-TW"
  }
}
```

但 `extensions` 的特性是「實作特定、不可攜帶」，非 schema 原生欄位不會被 query 的相關性評分考慮，也不保證跨實作可互通。

### 方案 B：修改 schema，添加原生 `language`/`locale` 欄位（需 fork/PR）

在 `knowledge_unit.json` 的 `Context` 中新增 `locale` 或 `naturalLanguage` 欄位：

```json
"Context": {
  "properties": {
    "languages": { ... },
    "naturalLanguage": {
      "type": "string",
      "maxLength": 10,
      "description": "Human language/locale of the insight content, e.g. 'zh-TW', 'en'"
    },
    "frameworks": { ... },
    "pattern": { ... }
  }
}
```

同時需修改：
- MCP `propose` 工具定義（`propose.go`）加入新參數
- MCP `query` 工具定義（`query.go`）加入新參數
- Server API route（`knowledge.py`）加入查詢過濾
- SDK 相應更新
- 相關性評分演算法加入 `naturalLanguage` 匹配邏輯

### 方案 C：利用 `domains` 標籤作為語言標記（最簡單，但非正規）

將語言標記放入 `domains` 中：

```
query(domains=["zh-TW", "database"])
propose(domains=["zh-TW", "database"], ...)
```

這種方式不修改 schema，但違反了 `domains` 的原設計意圖（描述知識主題），且與 skill 文件中的規則衝突（「Do not repeat languages or frameworks here」）。

### 方案 D：修改 cq skill 的指令（最實用，但需對每個使用者環境修改）

修改 `plugins/cq/skills/cq/SKILL.md`，在 `propose` 與 `query` 的指令中加入關於自然語言的段落。但這不會影響其他的 cq MCP server 實作。

## CLI 的語言處理

CLI 層面的 `propose` 與 `query` 子命令也支援 `--language` 旗標：[^8]

```bash
cq propose --language go --language python ...
cq query --language go --language python ...
```

這同樣是針對程式語言。

## 結論

| 面向 | 現狀 | 
|-------|--------|
| `languages` 欄位 | 僅代表**程式語言**，非人類自然語言 |
| 人類語言欄位 | 不存在於任何 schema/MCP tool/API 中 |
| 國際化支援 | 無，且無相關 roadmap |
| 實作特定擴充 | 可透過 `extensions` 欄位臨時儲存，但不具可攜性 |
| 最可行的改動路徑 | 方案 B（修改 schema 加 `naturalLanguage` 欄位 + MCP tool 參數 + server 查詢邏輯） |

若要讓 MCP server 能要求 agent 以特定人類語言提出/查詢知識，需要**修改 cq 的 schema、MCP tool 定義、以及 server 的查詢邏輯**，加入人類語言/地區欄位，並在相關性評分中考慮該欄位的匹配。

---

[^1]: Mozilla AI. (2025). *knowledge_unit.json — cq schema*. Retrieved 2026-10-09, from https://github.com/mozilla-ai/cq/blob/main/schema/knowledge_unit.json
[^2]: Mozilla AI. (2025). *propose.go — MCP server tool definition*. Retrieved 2026-10-09, from https://github.com/mozilla-ai/cq/blob/main/cli/mcpserver/propose.go
[^3]: Mozilla AI. (2025). *query.go — MCP server tool definition*. Retrieved 2026-10-09, from https://github.com/mozilla-ai/cq/blob/main/cli/mcpserver/query.go
[^4]: Mozilla AI. (2025). *Schema directory — confirm.json, flag.json*. Retrieved 2026-10-09, from https://github.com/mozilla-ai/cq/tree/main/schema
[^5]: Mozilla AI. (2025). *limits.go — schema limits*. Retrieved 2026-10-09, from https://github.com/mozilla-ai/cq/blob/main/schema/limits.go
[^6]: Mozilla AI. (2025). *Scoring values — language_weight*. Retrieved 2026-10-09, from https://github.com/mozilla-ai/cq/blob/main/schema/scoring.values.json
[^7]: Mozilla AI. (2025). *KnowledgeUnit — extensions field schema*. Retrieved 2026-10-09, from https://github.com/mozilla-ai/cq/blob/main/schema/knowledge_unit.json
[^8]: Mozilla AI. (2025). *CLI propose command — language flag*. Retrieved 2026-10-09, from https://github.com/mozilla-ai/cq/blob/main/cli/cmd/propose.go