# CodiMD / HedgeDoc 與 TriliumNext Notes 比較研究報告

## 概述

本報告比較兩款開源自架設（self-hosted）筆記工具：**CodiMD（HedgeDoc）** 與 **TriliumNext Notes**。兩者皆採用 AGPL-3.0 授權、支援自架部署，但它們解決的是截然不同的使用場景，適合不同類型的使用者。

| 面向 | CodiMD / HedgeDoc | TriliumNext Notes |
|------|-------------------|-------------------|
| **本質** | 即時協作 Markdown 編輯器 | 個人知識庫（hierarchical note-taking） |
| **核心定位** | 多人同時編輯一份文件 | 單人建立龐大知識體系 |
| **即時協作** | ✅ 原生支援（Operational Transformation） | ❌ 尚未實作（ roadmap 中） |
| **多使用者** | ✅ 原生多人 | ❌ 目前單人單伺服器 |
| **GitHub Stars** | ~7,500 | ~38,300 |
| **最新穩定版** | v1.12.0 (2026-08-21) | v0.106.0 (2026-09-25) |
| **開發狀態** | 1.x 維護模式 / 2.x Alpha | 活躍開發（~週更） |

---

## 架構與技術棧

### CodiMD / HedgeDoc

- **類型**：純 Web 應用，無桌面客戶端
- **後端**：Node.js（1.x）/ NestJS（2.x）
- **前端**：React / Next.js（2.x）
- **資料庫**：PostgreSQL、MySQL、MariaDB、SQLite
- **運作方式**：瀏覽器開啟即用，不須安裝軟體
- **部署**：Docker、手動 Node.js、Kubernetes

### TriliumNext Notes

- **類型**：桌面應用 + 伺服器端
- **後端**：Node.js / Express.js
- **前端**：客製化 UI（jQuery → React/Preact 遷移中）
- **資料庫**：SQLite（單一檔案）
- **運作方式**：Electron 桌面應用、或獨立伺服器 + Web UI
- **部署**：Docker、桌面安裝檔（Electron）、手動 Node.js

---

## 功能比較

| 功能 | CodiMD / HedgeDoc | TriliumNext Notes |
|------|------------------|-------------------|
| **即時協作編輯** | ✅ Operational Transformation | ❌ 未支援 |
| **多人同時使用** | ✅ 原生支援 | ❌ 單一使用者 |
| **Markdown 編輯** | ✅ 左寫右看雙欄 | ✅ Markdown auto-format + WYSIWYG |
| **WYSIWYG 編輯** | ❌ 純 Markdown | ✅ CKEditor 5 |
| **階層式筆記** | ❌ 單層文件 | ✅ 無限深度樹狀結構 |
| **筆記複製（Cloning）** | ❌ 無 | ✅ 一份筆記存在多個位置 |
| **全文搜尋** | ✅ 基礎搜尋 | ✅ 全文 + 模糊搜尋 + 屬性查詢 |
| **簡報模式** | ✅ reveal.js | ❌ 無 |
| **圖表支援** | ✅ Mermaid, PlantUML, Graphviz, Vega-Lite | ✅ Mermaid, Excalidraw, 心智圖 |
| **數學公式** | ✅ KaTeX | ✅ KaTeX |
| **程式碼高亮** | ✅ highlight.js | ✅ CodeMirror 6 |
| **看板（Kanban）** | ❌ 無 | ✅ 完整看板功能 |
| **地理地圖** | ❌ 無 | ✅ Leaflet 整合 |
| **腳本/自動化** | ❌ 無 | ✅ 內建 JavaScript API |
| **筆記版本控制** | ✅ 完整修訂歷史 | ✅ Git-style 快照 |
| **端對端加密** | ❌ 傳輸加密 | ✅ 逐筆記 AES-128-CBC |
| **雙因素認證** | ❌ 無 | ✅ TOTP |
| **OpenID Connect** | ✅ 多種認證方式 | ✅ |
| **離線編輯** | ❌ 需連線 | ✅ 桌面應用完全離線 |
| **行動裝置支援** | ⚠️ 瀏覽器 Responsive | ✅ PWA + 原生 Android/iOS |
| **匯入** | 有限 | ✅ Evernote, Notion, Markdown, OPML, HTML, JSON |
| **匯出** | Markdown, HTML, PDF | Markdown, HTML, JSON |

---

## 適用場景分析

### CodiMD / HedgeDoc 適合

1. **團隊會議記錄** — 多人即時共筆，開會時所有人同時編輯
2. **技術文件協作** — 開發團隊撰寫 runbook、ADR、API 文件
3. **教育與學術** — 學生共筆筆記、研究實驗室協作草稿
4. **開放原始碼社群** — 研討會筆記、社群共筆
5. **簡報製作** — Markdown 轉 reveal.js 投影片
6. **需要資料主權的組織** — 政府、醫療、金融等合規需求

### TriliumNext Notes 適合

1. **個人知識庫 / 第二大腦** — 累積數年以上、數萬則筆記
2. **Zettelkasten / 卡片盒筆記法** — 原子化筆記 + 雙向連結
3. **開發者技術筆記** — 程式碼片段、架構圖、自動化腳本
4. **研究實驗筆記本** — 長期累積的大量研究資料
5. **專案管理** — 看板 + 屬性命題查詢 + 自動化
6. **日記與每日筆記** — 自動產生模板、版本追蹤
7. **離開 Notion/Obsidian 的隱私需求** — 資料完全自主

---

## GitHub Stars 比較

```
CodiMD / HedgeDoc:   ~7,500 ★
TriliumNext Notes:   ~38,300 ★
```

TriliumNext 的星數顯著較高，反映其作為個人知識庫的廣泛需求，以及在原始 Trilium Notes 基礎上累積的社群基礎。CodiMD/HedgeDoc 雖然星數較少，但在即時協作編輯器領域是成熟穩定的選擇。

---

## 開發狀態與成熟度

### CodiMD / HedgeDoc

- **1.x 穩定版**（v1.12.0）: 維護模式，僅修 Bug 不開發新功能
- **2.x Alpha**（v2.0.0-alpha.3）: 完整重寫中，尚未達到功能對等
- 短期內無 stable 的重大更新
- 適合不需要新功能、只需要穩定協作編輯的團隊

### TriliumNext Notes

- **v0.106.0**（2026-09-25）: 每週發布、功能持續增加
- 從原始 Trilium 接管後持續活躍開發
- roadmap 包含多使用者支援、即時協作、外掛市場
- 適合追求功能持續進化的長期使用者

---

## 總結

| 你的需求 | 推薦選擇 |
|---------|---------|
| 需要多人即時共筆、會議記錄 | **CodiMD / HedgeDoc** |
| 需要個人知識庫、數萬則筆記管理 | **TriliumNext Notes** |
| 需要簡報模式（Markdown → 投影片） | **CodiMD / HedgeDoc** |
| 需要看板、地圖、腳本自動化 | **TriliumNext Notes** |
| 不想架伺服器、只想簡單用 | 兩者都不適合（皆需自架） |
| 團隊協作 + 個人知識庫都要 | 兩者互補，可同時部署 |

兩者並非直接競爭關係，而是定位互補。CodiMD/HedgeDoc 是**即時協作編輯器**，TriliumNext 是**個人知識庫管理系統**。若團隊同時有文件協作與知識管理需求，可考慮同時部署兩者。

---

## 參考資料

[^h1]: HedgeDoc. (n.d.). HedgeDoc — Ideas grow better together. Retrieved 2026-10-09, from https://hedgedoc.org/
[^h2]: HedgeDoc. (n.d.). GitHub repository. Retrieved 2026-10-09, from https://github.com/hedgedoc/hedgedoc
[^h3]: HedgeDoc. (n.d.). Releases. Retrieved 2026-10-09, from https://hedgedoc.org/releases/
[^h4]: Gappsy. (n.d.). HedgeDoc Review. Retrieved 2026-10-09, from https://www.gappsy.com/tools/hedgedoc/
[^h5]: Pistack. (2026). HedgeDoc vs Etherpad: Self-hosted Collaborative Editors Guide 2026. Retrieved 2026-10-09, from https://www.pistack.xyz/posts/hedgedoc-vs-etherpad-self-hosted-collaborative-editors-guide-2026/
[^t1]: TriliumNext. (n.d.). TriliumNext/Trilium — GitHub repository. Retrieved 2026-10-09, from https://github.com/TriliumNext/Trilium
[^t2]: TriliumNext. (n.d.). TriliumNext/Notes — Archived fork repository. Retrieved 2026-10-09, from https://github.com/TriliumNext/Notes
[^t3]: TriliumNext. (n.d.). Releases. Retrieved 2026-10-09, from https://github.com/TriliumNext/Trilium/releases
[^t4]: TriliumNext. (n.d.). Developer Guide — Architecture. Retrieved 2026-10-09, from https://docs.triliumnotes.org/developer-guide/architecture
[^t5]: XDA Developers. (2025). TriliumNext Notes guide. Retrieved 2026-10-09, from https://www.xda-developers.com/triliumnext-notes-guide/
[^t6]: XDA Developers. (2025). TriliumNext — The Notion alternative. Retrieved 2026-10-09, from https://www.xda-developers.com/trilium-next-notion-alternative/
[^t7]: Homelab Router. (n.d.). Self-hosted notes app comparison. Retrieved 2026-10-09, from https://homelabrouter.com/self-hosted-notes-app-comparison/
[^t8]: Bright Coding Dev. (2025-09-20). TriliumNext Notes — The Open Source Powerhouse for Building a Personal Knowledge Base. Retrieved 2026-10-09, from https://www.blog.brightcoding.dev/2025/09/20/triliumnext-notes-the-open-source-powerhouse-for-building-a-personal-knowledge-base