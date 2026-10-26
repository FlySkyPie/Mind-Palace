# Trilium Notes 是否支援 RSS？

## 摘要

Trilium Notes（現由 TriliumNext 維護，GitHub 上約 38.6k 星[^repo]）本身**沒有內建 RSS 功能**。它既不能原生訂閱外部 RSS 摘要，也不能把筆記發佈成 RSS 摘要。不過，社群維護的外掛 `rss-reader@beatlink` 提供了功能完整的 RSS 閱讀器，可以彌補訂閱端的需求；至於把筆記輸出成 RSS，則需要自行以 ETAPI 或靜態匯出加上外部工具來實作。

換句話說，「Trilium 支援 RSS 嗎？」的答案是：**官方不支援，但社群外掛可支援訂閱（讀取）方向，發佈（輸出）方向則無現成方案。**

---

## 一、原生支援狀況

### 1.1 訂閱外部 RSS（讀取）

Trilium 與 TriliumNext 都沒有內建的 RSS 閱讀器。官方匯入／匯出文件所列支援的格式為 HTML、Markdown 與 OPML，並未包含 RSS[^importexport]。

與 RSS 最相關的官方 GitHub 議題是 [TriliumNext/Trilium #2515](https://github.com/TriliumNext/Trilium/issues/2515)「新增可開啟外部網站 iframe 的筆記類型」。該議題於 2022-01-05 開啟、2022-06-19 關閉，請求者明確提到希望能在 Trilium 內使用 RSS 閱讀器。此需求最終以**實驗性的 WebView（iframe）筆記類型**實作：透過在筆記上設定 `#iframeSrc="https://..."` 屬性即可內嵌外部網站[^issue2515]。維護者 zadam 指出此功能有以下限制[^issue2515]：

- 僅限 Electron 桌面版
- 依賴 Electron/Chromium WebView，其未來支援性不確定
- 某些情況下可能不穩定
- 以實驗性功能形式發布

因此，嚴格來說這是「內嵌外部網頁」，而非原生解析及顯示 RSS/Atom 摘要。查閱整個 TriliumNext/Trilium 儲存庫的 issues 與 discussions，並沒有專門要求原生解析 RSS/Atom 的議題[^issue2515]。

### 1.2 將筆記發佈為 RSS（輸出）

官方並未提供把筆記樹產生 RSS/Atom 摘要的功能。搜尋 zadam/trilium 與 TriliumNext/Notes 的 issues，並未找到相關的功能請求[^issue2515]。

既有的相關功能為：

- **Sharing（分享）**：可將筆記發布成公開網站，具備可瀏覽的樹狀結構、自訂 CSS/JS 與 OpenGraph 中介資料，但**不會**產生 RSS/Atom 摘要[^sharing]。
- **Static HTML Export（靜態 HTML 匯出）**：可產生完整靜態網站（例如部署到 GitHub Pages），同樣沒有 RSS 輸出格式[^statichtml]。
- **ETAPI（REST API）**：理論上可透過 API 查詢筆記後，用外部腳本自行產生 RSS 摘要，但這是 DIY 方案，並非內建功能[^etapi]。

---

## 二、社群方案：`rss-reader@beatlink` 外掛

若需求是「在 Trilium 內訂閱及閱讀 RSS」，Trilium Community 組織維護的 `rss-reader@beatlink` 外掛是目前最完整的解法[^rssreader]。

### 2.1 功能

- 支援 **RSS 2.0**、**Atom**、**RSS 1.0/RDF** 與 **JSON Feed** 格式[^rssreader]。
- 可獨立運作（自行抓取摘要），也可作為完整的 **FreshRSS 客戶端**（透過 Google Reader API 雙向同步）[^rssreader]。
- 所有資料存放在持久化樹中的單一 JSON 筆記內[^rssreader]。
- 支援未讀／已加星號／已讀篩選、行內閱讀（HTML 經消毒處理）、資料夾管理與摘要搜尋[^rssreader]。

### 2.2 安裝與設定

- **必要條件**：必須啟用後端腳本（`backendScriptingEnabled=true` 或 `TRILIUM_SECURITY_BACKEND_SCRIPTING_ENABLED=true`）。原因是摘要來源不送 CORS 標頭，瀏覽器無法直接抓取，須由後端筆記代為抓取[^rssreader]。
- **僅訂閱摘要**：安裝並啟用外掛後，在 **Feeds** 頁籤貼上摘要 URL（每行一個）並按 **Add**[^rssreader]。
- **搭配 FreshRSS**：在 FreshRSS 的 Administration → Authentication 開啟 API 存取、於 Profile 設定 API 密碼，然後在外掛 Settings → FreshRSS 填入伺服器 URL、使用者名稱與 API 密碼，按 Refresh（會自動附加 `/api/greader.php`）[^rssreader]。

### 2.3 已知限制

| 限制 | 說明 |
|---|---|
| 無背景同步 | 只有在地化元件開啟時才會更新；因為唯一的 XML 解析器為瀏覽器的 `DOMParser`，無法在背景腳本執行[^rssreader] |
| API 密碼以明文儲存 | 與其他 libsettings 外掛相同，存放於設定筆記中；建議使用 FreshRSS 專用 API 密碼[^rssreader] |
| 無摘要自動探索 | 必須手動貼上摘要 URL，不能只給網站首頁[^rssreader] |
| 無 OPML 匯入 | 未搭配 FreshRSS 時須手動貼上 URL[^rssreader] |
| 重複摘要 | 同時在本機與 FreshRSS 訂閱的摘要會出現兩次，無去重[^rssreader] |
| 本機摘要歷史有限 | 本機訂閱只能看到摘要目前列出的內容（通常為最近 10–50 筆）[^rssreader] |
| 文章是 JSON 項目而非筆記 | 無法個別連結或複製，Trilium 的 collection 檢視也無法瀏覽[^rssreader] |
| 20,000 筆文章上限 | 帳號若超過 20,000 筆未讀／加星號文章，將跳過對帳並顯示警告[^rssreader] |

---

## 三、結論對照表

| RSS 能力 | 原生支援？ | 達成方式 |
|---|---|---|
| 在 Trilium 內訂閱／閱讀 RSS | 否 | 使用社群外掛 `rss-reader@beatlink`[^rssreader]；或以 iframe/WebView 筆記內嵌外部 RSS 閱讀器[^issue2515] |
| 將 Trilium 筆記輸出成 RSS | 否 | 自行以 ETAPI + 外部腳本產生，或使用 Sharing／靜態匯出（皆不含 RSS）[^etapi][^sharing][^statichtml] |

---

## 四、批判性反思

- **來源權威性**：本報告主要依據 TriliumNext 官方 GitHub 儲存庫、官方文件站（docs.triliumnotes.org）以及 Trilium Community 組織的外掛文件，屬於第一手或官方維護來源，可信度較高[^repo][^importexport][^sharing][^rssreader]。
- **時效性**：專案遷移頻繁。原始 `zadam/trilium` 已轉移至 TriliumNext 組織，`TriliumNext/Notes` 亦已於 2025-06-24 封存，目前活躍儲存庫為 `TriliumNext/Trilium`（最新版本 v0.106.1，2026-10-23）[^repo]。使用者在查證時應以現行儲存庫為準，避免引用舊資料。
- **反例空間**：本報告的「不支援」結論是「未找到」的否定命證，理論上仍可能存在未被索引的外掛或第三方工具。然就官方文件與主要儲存庫的搜尋結果而言，無原生 RSS 支援的可信度高。
- **行為變動風險**：iframe/WebView 功能被官方標示為實驗性，未來可能變動或移除；社群外掛則依賴後端腳本啟用，且無背景同步，實際體驗與專用 RSS 閱讀器仍有落差[^issue2515][^rssreader]。

---

## 參考文獻

[^repo]: TriliumNext. (2026). TriliumNext/Trilium. GitHub. Retrieved 2026-10-24, from https://github.com/TriliumNext/Trilium

[^importexport]: TriliumNext. (n.d.). Import & Export. Trilium Notes Documentation. Retrieved 2026-10-24, from https://docs.triliumnotes.org/user-guide/concepts/import-export

[^issue2515]: TriliumNext. (2022). New note type opening an iframe with an external website [Issue #2515]. GitHub. Retrieved 2026-10-24, from https://github.com/TriliumNext/Trilium/issues/2515

[^sharing]: TriliumNext. (n.d.). Sharing. Trilium Notes Documentation. Retrieved 2026-10-24, from https://docs.triliumnotes.org/user-guide/advanced-usage/sharing

[^statichtml]: TriliumNext. (n.d.). Static HTML Export. Trilium Notes Documentation. Retrieved 2026-10-24, from https://docs.triliumnotes.org/user-guide/advanced-usage/sharing/static-html

[^etapi]: TriliumNext. (n.d.). ETAPI discussion [Discussion #6601]. GitHub. Retrieved 2026-10-24, from https://github.com/orgs/TriliumNext/discussions/6601

[^rssreader]: Trilium Community. (n.d.). RSS Reader (rss-reader@beatlink). Trilium Scripts. Retrieved 2026-10-24, from https://trilium-community.github.io/trilium-scripts/rss-reader@beatlink/
