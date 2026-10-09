# FOSS 替代方案：OpenKnowledge（Web/MCP 面向）

> 研究 OpenKnowledge（[inkeep/open-knowledge](https://github.com/inkeep/open-knowledge)）的 FOSS 替代方案，重點在於提供 **Web UI** 或 **MCP（Model Context Protocol）** 存取方式，**排除純桌面應用**。

[^ok]: inkeep. (n.d.). open-knowledge — Beautiful, AI-native markdown IDE and LLM wiki. GitHub. Retrieved 2026-10-09, from https://github.com/inkeep/open-knowledge

---

## 背景

OpenKnowledge 是一款 AI-native Markdown IDE / LLM wiki，具備 WYSIWYG 編輯器、MCP 整合、Git-based 同步，以及桌面/Web 雙模式。其功能組合（Markdown 筆記 + AI 協作 + MCP 知識庫）在 FOSS 生態中並非唯一，但能做到**Web UI + MCP 原生支援 + 知識管理**三者的成熟方案不多。

以下按使用場景分組評比。

---

## 第一組：Web UI + MCP + 知識管理（三項全備）

這組是 OpenKnowledge 最直接的替代方案。

### 1. SiYuan（思源筆記）

| 項目 | 內容 |
|---|---|
| GitHub | [siyuan-note/siyuan](https://github.com/siyuan-note/siyuan) |
| Stars | ~46.7k ⭐ |
| 授權 | AGPL-3.0 |
| Web UI | ✅ Docker 自我托管，瀏覽器存取（port 6806），另有桌面/行動客戶端 |
| MCP | ✅ 原生支援，另有第三方 [siyuan-mcp-server](https://github.com/leolulu/siyuan-mcp-server)（MIT）提供 22+ 工具 |
| AI 整合 | ✅ 內建 AI 寫作 / Q&A（OpenAI API），block-based 編輯器 |
| Markdown | ✅ Block-level WYSIWYG |

SiYuan 是目前唯一同時具備**自我托管 Web UI、原生 MCP server、AI 整合**三者且為真正 FOSS 的方案。Docker 一鍵部署，適合個人或團隊作為 LLM wiki / 第二知識庫使用。[^siyuan]

[^siyuan]: siyuan-note. (n.d.). SiYuan — Privacy-first, self-hosted knowledge workspace. GitHub. Retrieved 2026-10-09, from https://github.com/siyuan-note/siyuan

### 2. TriliumNext（原 Trilium Notes 社群分支）

| 項目 | 內容 |
|---|---|
| GitHub | [TriliumNext/Trilium](https://github.com/TriliumNext/Trilium) |
| Stars | ~38.3k ⭐ |
| 授權 | AGPL-3.0 |
| Web UI | ✅ 原生 Web 伺服器，瀏覽器完整存取 |
| MCP | ✅ 內建 MCP 支援（docs.triliumnotes.org），另有社群 MCP server |
| AI 整合 | ❌ 無內建 AI，但提供可腳本化 REST API |
| Markdown | ✅ 層級筆記、WYSIWYG 編輯器 |

TriliumNext 從 Trilium Notes 社群分支繼承，具備成熟的層級筆記、加密、腳本 API，官方文件中明確說明如何透過 MCP 讓 LLM 讀取筆記。可以擴展至十萬條以上筆記而不降低效能。[^trilium]

[^trilium]: TriliumNext. (n.d.). Trilium — Hierarchical note-taking application. GitHub. Retrieved 2026-10-09, from https://github.com/TriliumNext/Trilium

### 3. SecondBrain（JonasKruegerCode）

| 項目 | 內容 |
|---|---|
| GitHub | [JonasKruegerCode/SecondBrain](https://github.com/JonasKruegerCode/SecondBrain) |
| Stars | 新專案（低 ⭐） |
| 授權 | MIT |
| Web UI | ✅ 內建知識圖譜可視化 + remember/recall 介面 |
| MCP | ✅ 原生 MCP server，提供 `remember()` 與 `recall()` 工具 |
| AI 整合 | ✅ HybridRAG（向量 + 知識圖譜 + Markdown） |
| Markdown | ✅ Obsidian 相容 Markdown Vault，支援 `[[wikilinks]]` |

這是一個以 AI 記憶層為核心的專案，使用 Python + Neo4j + Qdrant 實作。每個小時會自動執行的 review agent 維護 wiki。支援 Docker Compose 一鍵部署。如果需求更接近「AI 的持久記憶 + Web 視覺化」而非傳統編輯器，這是最佳選。[^secondbrain]

[^secondbrain]: JonasKruegerCode. (n.d.). SecondBrain — Knowledge graph-backed memory layer for AI agents. GitHub. Retrieved 2026-10-09, from https://github.com/JonasKruegerCode/SecondBrain

---

## 第二組：Web UI + 可透過社群 MCP 整合

這些方案具備優秀的 Web UI，雖然沒有原生 MCP，但社群已建立第三方 MCP server。

### 4. Logseq

| 項目 | 內容 |
|---|---|
| GitHub | [logseq/logseq](https://github.com/logseq/logseq) |
| Stars | ~45.2k ⭐ |
| 授權 | AGPL-3.0 |
| Web UI | ✅ Web 版（app.logseq.com），Docker 可自我托管 |
| MCP | ✅ 社群：[ergut/mcp-logseq](https://github.com/ergut/mcp-logseq)（341 ★，17 工具） |
| Markdown | ✅ 原生 Markdown + Org-mode 支援 |
| 特點 | 日記式（journal-first）、知識圖譜、反向連結、間隔重複 |

Logseq 的使用者基數龐大，生態成熟。其資料格式為純文字 Markdown/Org，可跨平台遷移。[^logseq]

[^logseq]: Logseq. (n.d.). Logseq — Privacy-first knowledge management platform. GitHub. Retrieved 2026-10-09, from https://github.com/logseq/logseq



---

## 第三組：純 Web 知識庫/維基（無原生 MCP，可透過 API 整合）

這些是成熟的 Web 知識管理系統，但需要自行串接 MCP。

### 6. Wiki.js

| 項目 | 內容 |
|---|---|
| GitHub | [requarks/wiki](https://github.com/requarks/wiki) |
| Stars | ~29k ⭐ |
| 授權 | AGPL-3.0 |
| Web UI | ✅ 全功能 Web 應用 |
| MCP | ❌ 但有 REST API |
| Markdown | ✅ 支援 Markdown 與 WYSIWYG 混合編輯 |

Wiki.js 模組化程度高，支援多種認證、Git 儲存、圖形化頁面編輯器。適合團隊文件/wiki 使用場景。[^wikijs]

[^wikijs]: requarks. (n.d.). Wiki.js — Next-generation open source wiki. GitHub. Retrieved 2026-10-09, from https://github.com/requarks/wiki

### 7. BookStack

| 項目 | 內容 |
|---|---|
| GitHub | [BookStackApp/BookStack](https://github.com/BookStackApp/BookStack)（主站在 Codeberg） |
| Stars | ~19.1k ⭐ |
| 授權 | MIT |
| Web UI | ✅ PHP Web 應用，易於部署 |
| MCP | ❌ 但有 REST API |
| Markdown | ✅ WYSIWYG 或 Markdown 編輯 |

BookStack 以 Shelves → Books → Chapters 層級組織內容，適合結構化的文件/知識庫。[^bookstack]

[^bookstack]: BookStackApp. (n.d.). BookStack — Opinionated documentation platform. GitHub. Retrieved 2026-10-09, from https://github.com/BookStackApp/BookStack

### 8. SilverBullet

| 項目 | 內容 |
|---|---|
| GitHub | [silverbulletmd/silverbullet](https://github.com/silverbulletmd/silverbullet) |
| Stars | ~6.3k ⭐ |
| 授權 | MIT |
| Web UI | ✅ 自我托管 Web app |
| MCP | ❌ 但內建 Lua 腳本引擎與 SLIQ 查詢語言 |
| Markdown | ✅ Markdown 即時預覽編輯器，Wiki 風格連結 |

SilverBullet 輕量但可擴充性強。其程式化架構（Space Lua 腳本）可實現自訂命令、模板與動態內容生成。簡潔、易於修改。[^silverbullet]

[^silverbullet]: silverbulletmd. (n.d.). SilverBullet — Programmable knowledge management. GitHub. Retrieved 2026-10-09, from https://github.com/silverbulletmd/silverbullet

### 9. HedgeDoc

| 項目 | 內容 |
|---|---|
| GitHub | [hedgedoc/hedgedoc](https://github.com/hedgedoc/hedgedoc) |
| Stars | ~7.5k ⭐ |
| 授權 | AGPL-3.0 |
| Web UI | ✅ 即時協作 Web app |
| MCP | ❌ |
| Markdown | ✅ 即時協作 Markdown、圖示、數學公式、投影片模式 |

HedgeDoc 更偏向即時協作筆記而非長期知識庫管理。HedgeDoc 2 正在重寫，需關注後續發展。[^hedgedoc]

[^hedgedoc]: HedgeDoc. (n.d.). HedgeDoc — Real-time collaborative Markdown editor. GitHub. Retrieved 2026-10-09, from https://github.com/hedgedoc/hedgedoc

---

## 第四組：純 MCP 記憶層（無或弱 Web UI）

若 Web UI 不是硬需求，純 MCP server 的方案在某些場景下更輕量。

### 10. llm-wiki-mcp（sumitroajiprabowo）

| 項目 | 內容 |
|---|---|
| GitHub | [sumitroajiprabowo/llm-wiki-mcp](https://github.com/sumitroajiprabowo/llm-wiki-mcp) |
| Stars | 低 ⭐ |
| 授權 | MIT |
| Web UI | ❌ 檔案可被 Obsidian 瀏覽，無原生 Web UI |
| MCP | ✅ 原生 MCP server：wiki_init/ingest/create/read/update/delete/search/lint |
| Markdown | ✅ Obsidian 相容 Markdown vault |

實作 Karpathy 「LLM Wiki」模式——讓 LLM 自行維護維基。[^llmwikimcp]

[^llmwikimcp]: sumitroajiprabowo. (n.d.). llm-wiki-mcp — LLM Wiki pattern via MCP. GitHub. Retrieved 2026-10-09, from https://github.com/sumitroajiprabowo/llm-wiki-mcp

### 11. jainil3010/second-brain-mcp

| 項目 | 內容 |
|---|---|
| GitHub | [jainil3010/second-brain-mcp](https://github.com/jainil3010/second-brain-mcp) |
| Stars | 低 ⭐ |
| 授權 | MIT |
| Web UI | ❌ 但有 Flutter 行動 app + API |
| MCP | ✅ 4 工具：search_memory, get_item, list_recent, add_note |
| Markdown | ✅ SQLite-backed，Markdown 匯出 |

BYOA（Bring Your Own AI）架構，可在手機分享選單中快速捕捉內容。[^jainilmcp]

[^jainilmcp]: jainil3010. (n.d.). second-brain-mcp — Mobile-capture + MCP memory layer. GitHub. Retrieved 2026-10-09, from https://github.com/jainil3010/second-brain-mcp

---

## 比較彙總

| 項目 | Web UI | MCP | AI 整合 | 授權 | 適合場景 |
|---|---|---|---|---|---|
| **SiYuan** | ✅ Docker | ✅ 原生 | ✅ 內建 | AGPL-3.0 | 最接近 OpenKnowledge 的全功能替代 |
| **TriliumNext** | ✅ 原生 | ✅ 原生 | ⚠️ 可腳本化 | AGPL-3.0 | 大型層級知識庫 + LLM 讀取 |
| **SecondBrain** | ✅ 內建 | ✅ 原生 | ✅ HybridRAG | MIT | AI 記憶層 + 知識圖譜視覺化 |
| **Logseq** | ✅ Docker | ✅ 社群 | ✅ 插件 | AGPL-3.0 | 日記式個人知識管理 |
| **Wiki.js** | ✅ 原生 | ❌ API | ❌ | AGPL-3.0 | 團隊文件/wiki |
| **BookStack** | ✅ 原生 | ❌ API | ❌ | MIT | 結構化文件管理 |
| **SilverBullet** | ✅ 原生 | ❌ 腳本 | ❌ | MIT | 輕量、可程式化個人 wiki |
| **HedgeDoc** | ✅ 原生 | ❌ | ❌ | AGPL-3.0 | 即時協作筆記 |
| **llm-wiki-mcp** | ❌ | ✅ 原生 | ✅ 面向 LLM | MIT | 純 MCP LLM wiki |
| **jainil3010/mcp** | ❌ 行動 | ✅ 原生 | ❌ | MIT | 輕量 MCP 記憶捕捉 |

---

## 建議

1. **若要最直接的 OpenKnowledge 替代** → **SiYuan**。唯一的 Web UI + 原生 MCP + AI 整合三者皆備的成熟 FOSS 專案，stars 近 47k，生態活躍。

2. **若重視 AI 記憶層而非編輯器體驗** → **SecondBrain**。HybridRAG、知識圖譜、定時 review agent，純正 MIT 授權，非常適合讓 LLM agent 擁有持久記憶。

3. **若偏好純 MCP 架構、不要 Web UI** → **llm-wiki-mcp**。最輕量的 Karpathy LLM wiki 實作，完全以 MCP 為中心。

4. **若需要團隊協作的知識庫** → **Wiki.js** 或 **BookStack**。雖然沒有原生 MCP，但 REST API 可以自行串接，且 Web UI 體驗成熟。

---

## 排除說明

下列工具因為是純桌面應用而被排除在本研究之外：

- **Obsidian** — 封閉原始碼
- **Joplin** — 桌面優先，Web Clipper 但無自我托管 Web UI
- **Zettlr** — 桌面學術筆記應用
- **Foam** (VS Code 擴充) — 依賴桌面編輯器