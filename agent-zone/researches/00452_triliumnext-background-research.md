# TriliumNext／Trilium Notes 專案背景調查報告

## 摘要

TriliumNext／Trilium Notes 是一款開源、跨平台、可自架的階層式筆記應用程式，專注於建立大型個人知識庫。原專案由獨立開發者 Zadam 於 2017 年創建，累積逾 38,000 顆 GitHub 星標後因開發者個人因素進入維護模式。2024 年 1 月社群成員成立 TriliumNext 分支，最終說服原開發者將官方儲存庫轉移至該組織，成為事實上的官方延續版本。本報告調查其團隊結構、資金來源、社群規模與治理模式。

## 專案起源與歷史

Trilium Notes（原始名稱 Trilium）由化名 Zadam（Adam）的獨立開發者於 2017 年 5 月 23 日建立[^repo]。該專案以 Node.js 為基礎，使用 Electron 作為桌面端框架，SQLite 作為資料庫後端，提供富文本編輯（CKEditor 5）、程式碼編輯（CodeMirror）、白板繪圖（Excalidraw）、心智圖（Mind Elixir）、地理地圖（Leaflet）、Mermaid 圖表等豐富功能，並支援每則筆記的 AES-128-CBC 加密、JavaScript API 腳本擴充、REST API、關聯圖與看板等進階功能[^repo]。

經過約兩年密集開發後，Zadam 因個人因素（育兒與工作壓力）將專案轉為維護模式。2024 年 1 月，社群成員在 Elian Doran 領導下建立了 TriliumNext 分支，決心維持專案生命[^dosu][^fork_story]。

## 儲存庫合併事件

TriliumNext 團隊經過近一年的持續聯繫（透過 Git 提交歷史追蹤、公司電子郵件等管道），最終成功說服 Zadam 將原始儲存庫轉移至 TriliumNext 組織。這是開源社群中罕見的「分支反轉」案例——分支不僅存活，還獲得了官方地位。此後 `zadam/trilium` Docker 映像已自動追蹤 `triliumnext/trilium`，遷移無需任何特殊步驟[^docker_migration][^dosu]。

## 團隊結構

TriliumNext 組織目前公開列出 6 名成員[^org_people]：

| 姓名 | GitHub ID | 角色 |
|---|---|---|
| Elian Doran | eliandoran | **主要維護者與主開發者** |
| Jon Fuller | perfectra1n | DevOps、安全、搜尋功能 |
| Orsiris de Jong | deajan | 維護者 |
| Adorian Doran | adoriandoran | 看板、備份還原等功能 |
| Arne Keller | FliegendeWurst | TriliumDroid（Android 客戶端） |
| JYC333 | JYC333 | 成員 |
| SngAbc (SiriusXT) | SiriusXT | 成員 |

此團隊完全由志願者組成，無全職受薪員工。

## 資金與商業模式

TriliumNext **沒有**任何公司實體、創投投資或商業營運背景。專案採用 AGPL-3.0 授權，社群有強烈共識反對引入付費方案或「企業版」[^funding]。

資金完全依賴社群捐贈，主要流向主開發者 Elian Doran：

- GitHub Sponsors[^sponsors]
- PayPal[^paypal]
- Buy Me a Coffee[^coffee]
- LiberaPay[^liberapay]

此外獲得以下組織的實物贊助：

- **NetPerfect** — 提供 EV 憑證與 Windows CI
- **CKEditor** — 提供進階編輯器功能授權
- **Dosu** — 提供自動化 GitHub issue 分類與去重處理

## 社群規模

根據 GitHub API 資料（調查日期 2026-10-09）[^repo]：

| 指標 | 數值 |
|---|---|
| GitHub 星標 | 38,254 |
| Forks | 2,578 |
| 開放 Issues | 639 |
| 訂閱者 | 282 |
| 授權 | AGPL-3.0 |
| 主要語言 | TypeScript |
| 專案健康評分 | 100/100 |

社群溝通管道[^repo][^website]：

- **Matrix**：`#triliumnext:matrix.org`（亦橋接至 XMPP）
- **GitHub Discussions**：非同步討論與問答
- **GitHub Issues**：錯誤回報與功能請求
- **We blate**：翻譯協作平台
- **第三方社群**：awesome-trilium、TriliumRocks!
- **行動端**：TriliumDroid（Android）、Pocket Trilium（Android）、Trinote（iOS，開源）

## 技術概覽

- **前端框架**：Electron 桌面應用
- **後端語言**：Node.js (TypeScript)
- **資料庫**：SQLite
- **主要整合元件**：CKEditor 5（富文本）、CodeMirror（程式碼）、Excalidraw（白板）、Mind Elixir（心智圖）、Leaflet（地圖）、Mermaid（圖表）
- **支援最大筆記量**：經測試可有效處理 100,000+ 則筆記
- **安全功能**：每則筆記獨立 AES-128-CBC 加密
- **擴充性**：JavaScript API 腳本系統、REST API
- **AI 整合**：可選的 LLM 聊天整合（選用、預設關閉）

## 結論

TriliumNext／Trilium Notes 是一個典型的社群驅動開源專案，無公司、創投或商業資助背景。其獨特之處在於成功從原開發者手中接掌官方地位，在開源生態中實屬罕見。專案維持純志願者治理、全開放授權、反對商業化的社群文化。對於需要自架知識庫系統的使用者而言，這是一條成熟且社群活躍的技術路線。

---

[^repo]: TriliumNext. (n.d.). *Trilium* [GitHub repository]. Retrieved 2026-10-09, from https://github.com/TriliumNext/Trilium

[^dosu]: Dosu. (n.d.). *How TriliumNext revitalized an abandoned open-source project with Dosu's help*. Retrieved 2026-10-09, from https://dosu.dev/customers/how-triliumnext-revitalized-an-abandoned-open-source-project-with-dosus-help

[^fork_story]: TriliumNext. (2024). *TriliumNext — The community fork keeping Trilium alive* [GitHub Discussions]. Retrieved 2026-10-09, from https://github.com/orgs/TriliumNext/discussions

[^docker_migration]: TriliumNext. (n.d.). *Docker migration Q&A* [GitHub Discussions]. Retrieved 2026-10-09, from https://github.com/orgs/TriliumNext/discussions/7322

[^org_people]: GitHub. (n.d.). *TriliumNext organization people*. Retrieved 2026-10-09, from https://github.com/orgs/TriliumNext/people

[^funding]: TriliumNext. (2024). *Funding discussion: keeping Trilium free and open source* [GitHub Discussions]. Retrieved 2026-10-09, from https://github.com/TriliumNext/Notes/discussions/27

[^sponsors]: GitHub. (n.d.). *Sponsor @eliandoran*. Retrieved 2026-10-09, from https://github.com/sponsors/eliandoran

[^paypal]: PayPal. (n.d.). *Donate to Elian Doran*. Retrieved 2026-10-09, from https://paypal.me/eliandoran

[^coffee]: Buy Me a Coffee. (n.d.). *Elian Doran*. Retrieved 2026-10-09, from https://buymeacoffee.com/eliandoran

[^liberapay]: LiberaPay. (n.d.). *Elian Doran*. Retrieved 2026-10-09, from https://liberapay.com/ElianDoran

[^website]: TriliumNext. (n.d.). *Trilium Notes — Official website*. Retrieved 2026-10-09, from https://triliumnotes.org