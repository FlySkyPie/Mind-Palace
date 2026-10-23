# Trilium Notes 的 HTML 匯出：GUI、CLI 與 API 能力調查

## 摘要

Trilium Notes（目前由 TriliumNext 社群維護，儲存庫 `TriliumNext/Trilium`）確實支援將筆記匯出為 HTML，而且**不只透過圖形介面（GUI）**，也能透過官方對外 REST API（ETAPI）以程式化方式觸發匯出。不過，Trilium **沒有官方的命令列介面（CLI）**，也不存在原生的無介面（headless）匯出子命令；若要自動化，官方建議的做法是呼叫 ETAPI。

截至 2026-10-23，`TriliumNext/Trilium` 儲存庫約有 38,486 顆星，授權為 AGPL-3.0，主要語言為 TypeScript[^stars]。

## 一、匯出功能概觀

Trilium 原生支援以下格式，且**同時支援匯入與匯出**[^importexport]：

- **HTML**：Trilium 的主要格式，以標準標籤（如 `<strong>`、`<table>`、`<pre>`）表示基本格式與版面。由於 HTML 並非完全標準化的互通格式，部分 Trilium 專屬功能（如 admonitions、內部參照連結）在其他應用中可能無法完整呈現。此外，筆記也可匯出為可用於網路發布的**靜態 HTML**[^importexport]。
- **Markdown**：多數格式會被保留[^importexport]。
- **OPML**（Outliner Interchange Format）：同時支援純文字的 OPML v1.0 與含 HTML 的 v2.0[^importexport]。

匯出的粒度有兩種[^markdown]：

- **子樹匯出（subtree export）**：將整個子樹匯出為 ZIP 壓縮檔，目錄結構對應筆記樹結構。
- **單篇筆記匯出（single note export）**：僅匯出單一筆記。

若筆記受保護（protected），需先進入保護工作階段才能匯出，且匯出內容為**未加密**形式[^markdown]。

## 二、透過 GUI 匯出

在圖形介面中，匯出方式為[^importexport]：

- 在筆記樹（Note Tree）中對筆記按右鍵，選擇 **Export**。
- 或在「筆記按鈕（Note buttons）」中選擇 **Export note**。

此外，Trilium 提供「靜態 HTML 發布用」的匯出模式，能產生與 Sharing 功能相同版面與樣式的靜態 HTML 檔案，適合部署到 GitHub Pages 或 CloudFlare Pages 等無伺服器環境。官方文件本身（docs.triliumnotes.org）即是以此功能產生靜態 HTML 後部署而成[^statichtml]。

匯出格式的選擇（HTML／Markdown／OPML 等）都在 GUI 的匯出對話框中完成[^importexport]。

## 三、CLI 支援情況：無官方 CLI

Trilium **沒有官方的命令列介面**，也沒有原生的無介面匯出指令。這點在儲存庫的議題（issue）中有明確答覆：

- 議題 #1757（2021 年 3 月，詢問 CLI 指令）中，原主要作者 zadam 回覆：*「目前沒有 CLI，只能從 UI 觸發匯出。CLI 或類似介面（也許是 REST）的實作已有計畫，但沒有明確時程。」*[^cli1757]
- 議題 #3012（2022 年 7 月，請求「透過 CLI 自動匯出」）中，zadam 回覆：*「Trilium 並沒有真正的 CLI 介面，但它有適合自動化的 REST API，稱為 ETAPI。」* 該議題最終透過新增 ETAPI 匯出端點而結案[^cli3012]。

也就是說，官方對「如何在 GUI 之外匯出」的標準答案就是改用 ETAPI，而非提供 CLI。

社群另有第三方 CLI 封裝工具（例如 `triliumnext-cli`、`trilium-py`），但這些都是包裝 ETAPI 的社群專案，並非官方工具[^py]。

## 四、透過 ETAPI（對外 REST API）匯出

ETAPI 是 Trilium 的官方對外 REST API，自 v0.50 起提供[^etapi]。其匯出端點為：

```
GET /etapi/notes/{noteId}/export?format={html|markdown|share}
```

其中 `format` 為選填參數，**預設值為 `html`**；可接受的值為 `html`、`markdown`、`share`，若傳入其他值會回傳 400 錯誤 `UNRECOGNIZED_EXPORT_FORMAT`[^source]。端點會將指定筆記的子樹匯出為 ZIP 壓縮檔[^etapi]。

認證方式為在 `Authorization` 標頭帶入 ETAPI 權杖（相容格式 `Bearer ETAPITOKEN` 自 0.93.0 起支援），亦支援 Basic Auth[^etapi]。

官方文件提供了 Bash 腳本範例，說明如何取得筆記內容與匯出 ZIP[^etapi]：

```bash
#!/usr/bin/env bash
TOKEN=z1vA4fkGxjOR_ZXLrZeqHEFOv65yV3882iFCRtNIK9k9iWrHliITNSLQ=
SERVER=http://localhost:8080

# 取得筆記 HTML 內容
curl "$SERVER/etapi/notes/$NOTE_ID/content" -H "Authorization: $TOKEN"

# 匯出筆記為 ZIP（預設即 HTML 格式）
curl -H "Authorization: $TOKEN" \
  -X GET "$SERVER/etapi/notes/$NOTE_ID/export" \
  --output "out/$NOTE_ID.zip"
```

官方文件在討論靜態 HTML 匯出的「自動化（Automation）」時，也明確指出 *「ETAPI（REST API）可用於將匯出安排為排程任務。」*[^statichtml]

因此，**HTML 匯出可透過 GUI 與 API（ETAPI）達成，但目前沒有 CLI 途徑**（除非使用第三方封裝工具）。

## 五、內部 API 與 ETAPI 的差異

Trilium 內部另有供前端使用的 `/api/*` 路由（內部 API），與對外的 `/etapi/*` 是兩套不同的介面：內部 API 以瀏覽器工作階段 cookie 進行認證並附帶 CSRF 保護，並非為腳本或無介面使用而設計；而 ETAPI 則使用靜態權杖，適合自動化[^apiarch]。

| 面向 | GUI 匯出 | ETAPI 匯出 | CLI |
| --- | --- | --- | --- |
| 是否支援 HTML 匯出 | 是 | 是（預設格式） | 無官方 CLI |
| 認證方式 | 使用者登入 | ETAPI 權杖 | 不適用 |
| 可自動化 | 否 | 是（可排程） | 需第三方封裝 |
| 匯出格式 | HTML／Markdown／OPML | HTML／Markdown／share | 不適用 |

## 六、結論與注意事項

- **GUI 能否匯出 HTML**：能。透過筆記樹右鍵 Export 或筆記按鈕的 Export note[^importexport]。
- **API 能否匯出 HTML**：能。`GET /etapi/notes/{noteId}/export` 預設即 HTML 格式，並可透過權杖自動化與排程[^etapi][^source]。
- **CLI 能否匯出 HTML**：**沒有官方 CLI**，亦無原生 headless 匯出指令；官方建議改用 ETAPI[^cli1757][^cli3012]。

需要注意的限制：

- ETAPI 匯出產出的是一個 **ZIP 壓縮檔**（含筆記子樹），而非單一 HTML 檔案[^etapi]。
- ETAPI 的 `format` 參數僅支援 `html`、`markdown`、`share` 三種；OPML 不在 ETAPI 匯出選項內，但可從 GUI 匯出[^source][^importexport]。
- 靜態 HTML 匯出（發布用）需要一個可運作的網頁伺服器才能正確渲染，因為頁面使用 module scripts[^statichtml]。

## 參考文獻

[^stars]: GitHub. (2026). *TriliumNext/Trilium* [Repository metadata]. Retrieved 2026-10-23, from https://api.github.com/repos/TriliumNext/Trilium

[^importexport]: TriliumNext. (n.d.). Import & Export. In *Trilium Notes User Guide*. Retrieved 2026-10-23, from https://docs.triliumnotes.org/user-guide/concepts/import-export

[^markdown]: TriliumNext. (n.d.). Markdown. In *Trilium Notes User Guide*. Retrieved 2026-10-23, from https://docs.triliumnotes.org/user-guide/concepts/import-export/markdown

[^statichtml]: TriliumNext. (n.d.). Exporting static HTML for web publishing. In *Trilium Notes User Guide*. Retrieved 2026-10-23, from https://docs.triliumnotes.org/user-guide/advanced-usage/sharing/static-html

[^etapi]: TriliumNext. (n.d.). ETAPI (REST API). In *Trilium Notes User Guide*. Retrieved 2026-10-23, from https://docs.triliumnotes.org/user-guide/advanced-usage/etapi

[^source]: TriliumNext. (2026). *apps/server/src/etapi/notes.ts* [Source code]. GitHub. Retrieved 2026-10-23, from https://github.com/TriliumNext/Trilium/blob/main/apps/server/src/etapi/notes.ts

[^cli1757]: zadam. (2021, March). [Question] CLI commands [Issue #1757]. GitHub. Retrieved 2026-10-23, from https://github.com/TriliumNext/Trilium/issues/1757

[^cli3012]: zadam. (2022, July). [FR] Automatic Export via CLI [Issue #3012]. GitHub. Retrieved 2026-10-23, from https://github.com/TriliumNext/Trilium/issues/3012

[^apiarch]: TriliumNext. (n.d.). API. In *Trilium Notes Developer Guide*. Retrieved 2026-10-23, from https://docs.triliumnotes.org/developer-guide/architecture/api

[^py]: Nriver. (n.d.). *trilium-py* [Computer software]. GitHub. Retrieved 2026-10-23, from https://github.com/Nriver/trilium-py
