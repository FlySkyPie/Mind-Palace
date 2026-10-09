# cq 知識庫的人類語言在地化研究

> 如何讓 cq (https://github.com/mozilla-ai/cq) 的 MCP 伺服器要求 Agent 以特定語言提出知識單元（Knowledge Unit），例如以 zh_TW 提出 proposal。

## 結論

**cq 目前沒有任何人類語言（locale/i18n）概念**。`languages` 欄位僅用於標示**程式語言**（Python、Go 等），與人類語言無關。要實現以特定語言提出知識單元，需透過**代理提示工程（skill prompt engineering）**或 **extensions 擴充欄位** 兩種途徑。

## 1. cq 的語言相關欄位：僅限程式語言

`KnowledgeUnit` 中的 `context.languages` 欄位完全針對程式語言設計，用於查詢過濾和相關性評分（權重 0.15）。[^schema]

```json
// schema/knowledge_unit.json — Context 物件
"Context": {
  "properties": {
    "languages": {
      "type": "array",
      "items": { "type": "string", "maxLength": 64 },
      "description": "Programming languages (e.g. python, go)"
    },
    "frameworks": { ... },
    "pattern": { ... }
  }
}
```

所有查詢和 CLI 工具都將 `--language` 視為程式語言過濾器。[^cli]

## 2. 方法 A（最簡單）：修改 Agent Skill Prompt

這是不需改動 cq 程式碼的解決方案。在 Crush 的 cq skill 提示中加入語言指示：

**修改 `mcp_global-cq_propose` 或對應 skill 的提示**，例如在 SKILL.md 中加入：

```
## 語言要求
- 所有知識單元的內容（summary、detail、action）必須使用繁體中文（zh_TW）撰寫
- 查詢時優先比對相同語言的知識單元
```

這直接控制 agent 的輸出語言，因為知識單元的文字內容完全由 agent 生成。

## 3. 方法 B（可擴展）：使用 extensions 欄位標記語言

cq 提供 `extensions` 欄位作為實作自訂欄位的命名空間，格式為 `namespace:key`。可用此欄位標記知識單元的人類語言：[^schema]

```json
{
  "extensions": {
    "crush:locale": "zh_TW"
  }
}
```

**查詢時**：Agent 可在查詢結果中過濾 `extensions["crush:locale"]` 符合目標語言的單元。

**提出時**：在 propose 請求中包含 `extensions` 即可。

限制：`extensions` 被明確定義為**非便攜（non-portable）**——不同實作間不保證互通。

## 4. 方法 C（深度整合）：修改 cq 架構

若需伺服器端原生支援語言過濾，需修改：

1. **Schema**：在 `KnowledgeUnit` 或 `Context` 中加入 `locale` 欄位（如 `zh_TW`、`en-US`）
2. **Query schema**：加入 `locale` 過濾參數
3. **Server 模型**：更新 `models/knowledge.py` 和 `repositories/_queries.py` 支援 locale 過濾
4. **SDK（Python/Go）**：更新 `Context` 模型加入 `locale` 欄位
5. **Relevance scoring**：在 `scoring.json` 中加入 `locale_weight`

這是最完整但工作量最大的方案。

## 5. 建議方案

| 方案 | 複雜度 | 可維護性 | 適用場景 |
|------|--------|----------|----------|
| A: Skill 提示修改 | 低 | 中（依賴 agent 理解力） | 單一語言團隊，不需伺服端過濾 |
| B: extensions 標記 | 中 | 高（顯式標記） | 多語言知識庫，agent 可自行過濾 |
| C: 架構修改 | 高 | 最高（原生支援） | 正式多語言部署 |

**建議從方法 A + B 混合開始**：在 skill prompt 中指定語言，同時在 propos 時帶入 `extensions["crush:locale"]` 以供後續過濾。

## 參考文獻

[^schema]: Mozilla.ai. (n.d.). cq Schema Reference. Retrieved 2026-10-09, from https://github.com/mozilla-ai/cq/blob/main/schema/README.md
[^cli]: Mozilla.ai. (n.d.). cq CLI — query command. Retrieved 2026-10-09, from https://github.com/mozilla-ai/cq/tree/main/cli
[^plugin]: Mozilla.ai. (n.d.). cq Plugin README. Retrieved 2026-10-09, from https://github.com/mozilla-ai/cq/blob/main/plugins/cq/README.md
[^models]: Mozilla.ai. (n.d.). cq Server — models/knowledge.py. Retrieved 2026-10-09, from https://github.com/mozilla-ai/cq/tree/main/server/backend/src/cq_server/models/knowledge.py
[^gotypes]: Mozilla.ai. (n.d.). cq Go SDK — types.go. Retrieved 2026-10-09, from https://github.com/mozilla-ai/cq/blob/main/sdk/go/types.go