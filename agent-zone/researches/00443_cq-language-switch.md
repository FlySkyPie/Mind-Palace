# cq 語言切換功能調查

## 概要

Mozilla AI 的 [cq][repo] 是一個開放標準，旨在讓 AI 代理之間共享結構化知識，避免重複犯錯。本報告調查 cq 是否支援人類語言（UI/內容語言）切換。

**結論：cq 目前沒有任何語言切換功能或國際化（i18n）基礎設施。**

## 詳細說明

### 不存在任何語言/地區相關設定

經過對 cq 專案全部元件（CLI、Server、Schema、SDK、Plugin、文件）的完整搜尋，以下專案中**完全不存在**：

- ❌ `CQ_LOCALE` 或類似環境變數
- ❌ `--lang`、`--locale` 等 CLI 旗標
- ❌ 任何 i18n 框架（如 gettext、ICU MessageFormat 等）
- ❌ 翻譯檔案（`.po`、`.mo`、`.json` locale 檔案）
- ❌ 設定檔中的語言/地區欄位
- ❌ 文件中任何關於 locale、l10n、翻譯的提及

### cq 中的「language」實際含義

程式碼中出現的「language」全部指**程式語言標記**，用於知識單元（Knowledge Unit）的分類與查詢過濾：

1. **`context.languages`**（`schema/knowledge_unit.json`）— 字串陣列，標記該知識適用的程式語言（如 `["python", "go"]`）
2. **`cq query --language go`** — CLI 查詢過濾參數，僅回傳指定程式語言的知識
3. **`scoring.json` 中的 `language_weight`** — 程式語言匹配在相關性評分中的權重（0.15）

這些都與人類語言（UI 顯示）完全無關。[^unitschema][^scoring][^cli_flags]

### 專案現狀

- CLI 所有輸出為英文
- Server 前端（React/TypeScript）使用硬編碼英文字串
- 文件與計畫目錄中未見任何 locale 支援的 roadmap 項目
- 無自架設或 cq.exchange 服務提供語言切換選項

## 結論

如果需要 cq 支援繁體中文或其他語言的 UI/內容，需要從零開始對以下元件進行國際化改造：CLI 的 Go 原始碼、Server 前端的 React/TypeScript UI、Server 後端的 FastAPI 錯誤訊息，以及可能的 SDK 與 Schema 文件。

[repo]: https://github.com/mozilla-ai/cq
[^unitschema]: mozilla-ai/cq. (n.d.). `schema/knowledge_unit.json`. Retrieved 2026-10-09, from https://github.com/mozilla-ai/cq/blob/main/schema/knowledge_unit.json
[^scoring]: mozilla-ai/cq. (n.d.). `schema/scoring.json`. Retrieved 2026-10-09, from https://github.com/mozilla-ai/cq/blob/main/schema/scoring.json
[^cli_flags]: mozilla-ai/cq. (n.d.). `cli/cmd/query.go`. Retrieved 2026-10-09, from https://github.com/mozilla-ai/cq/blob/main/cli/cmd/query.go