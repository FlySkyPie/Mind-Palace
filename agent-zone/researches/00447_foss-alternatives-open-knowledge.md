# Open Knowledge 之 FOSS 替代方案研究

本報告調査了 [Open Knowledge](https://github.com/inkeep/open-knowledge)（一款 AI-native Markdown IDE 與 LLM Wiki，4.4k stars，GPL-3.0）的自由開源替代方案，涵蓋知識庫管理、WYSIWYG Markdown 編輯、LLM Wiki 與 AI 記憶系統等類別。

## 替代方案一覽

### 1. Tolaria（最接近的全能替代品）

- **GitHub：** https://github.com/refactoringhq/tolaria
- **Stars：** 20,000+
- **授權：** AGPL-3.0
- **技術棧：** Tauri + React + TypeScript

跨平台桌面應用，使用區塊式編輯器管理 Markdown 知識庫，可作為第二大腦、公司文件中心及 AI 記憶存儲。每個知識庫都是 Git 儲存庫，支援原生 Git 版本歷史。直接整合 MCP、Claude Code、Codex CLI 等 AI 模型提供者。離線優先、無需帳號。[^tolaria]

**vs. Open Knowledge：** Tolaria 使用區塊式編輯而非真正的 WYSIWYG，也沒有內建團隊共享功能。

### 2. LLM Wiki（nashsu/llm_wiki）——最佳 LLM Wiki 實作

- **GitHub：** https://github.com/nashsu/llm_wiki
- **Stars：** 20,300+
- **授權：** GPL-3.0
- **技術棧：** Tauri v2 + React 19 + TypeScript + Rust

基於 Andrej Karpathy 的 LLM Wiki 模式，但以完整桌面應用形式大幅強化。採用兩階段思緒鏈（分析→生成）來建立高品質 Wiki 頁面。具備知識圖譜（4 信號關聯模型、Louvain 社群偵測）、多格式文件解析（PDF、DOCX、PPTX、XLSX、EPUB、MOBI）、LanceDB 向量語意搜尋、MCP 伺服器、深入研究模式。[^nashsu_llm_wiki]

**vs. Open Knowledge：** 更偏重「LLM Wiki」模式——AI 為你閱讀來源並寫出結構化 Wiki——而非通用 Markdown IDE。在知識圖譜視覺化與社群偵測方面勝出。

### 3. Basic Memory——最佳 MCP/AI Agent 整合

- **GitHub：** https://github.com/basicmachines-co/basic-memory
- **Stars：** 4,100+
- **授權：** AGPL-3.0
- **技術棧：** Python + FastMCP

MCP-native 知識管理系統，讓 AI Agent 透過結構化 Markdown 檔案維持持久記憶。支援知識圖譜（觀測、WikiLinks、語意搜尋）、人類與 AI 雙向寫入同一個檔案、可選的交叉編碼器重排序。[^basic_memory]

**vs. Open Knowledge：** 沒有視覺化編輯器——需搭配 Obsidian 或 VS Code 使用。更偏向 AI 記憶/持久層，而非編輯體驗。

### 4. Markra——最佳 WYSIWYG 編輯體驗

- **GitHub：** https://github.com/markrahq/markra
- **Stars：** 908
- **授權：** AGPL-3.0
- **技術棧：** Tauri + React + TypeScript

本地優先的 WYSIWYG Markdown 編輯器，原生 AI 整合。支援行內 AI 編輯（選取文字後）、側欄文件級 AI 任務、15+ AI 提供者（OpenAI、Anthropic、Gemini、DeepSeek、Ollama 等）、KaTeX 數學、Mermaid 圖表、Typora 主題相容。[^markra]

**vs. Open Knowledge：** 編輯體驗最接近，但缺少知識庫/wiki 平台功能（無 Git 同步、無團隊共享、無 MCP 伺服器）。

### 5. Snaptium——全能 AI 筆記應用

- **GitHub：** https://github.com/jetyu/Snaptium
- **Stars：** 63
- **授權：** Apache-2.0
- **技術棧：** Vue 3 + TypeScript + Electron + CodeMirror 6

跨平台 AI Markdown 筆記知識管理應用。內建 CodeMirror 6 編輯器、基於 LangChain 的向量知識索引、RAG 知識庫 Q&A、端對端加密同步（WebDAV/S3/Cloudflare R2/MinIO）、AES-256-GCM 加密、混合搜尋（全文+語意）。[^snaptium]

**vs. Open Knowledge：** 明星數量少但功能豐富。加密與同步功能更強。使用預覽模式而非真正 WYSIWYG。

### 6. nvk/llm-wiki——最佳 AI Agent 外掛模式

- **GitHub：** https://github.com/nvk/llm-wiki
- **Stars：** ~858
- **授權：** MIT
- **技術棧：** Python + Claude Code/Codex/OpenCode 外掛

作為 Claude Code、OpenAI Codex、OpenCode 的外掛來運作。支援並行多 Agent 研究、論文驅動的調查、跨對話的會話記憶、Wiki 健康審計/檢查。輸出相容 Obsidian。[^nvk_llm_wiki]

**vs. Open Knowledge：** 不是獨立應用——它是一個外掛/工作流程。沒有 GUI。如果你已重度使用 Claude Code/Codex 則非常適合。

### 7. SwarmVault——最佳知識圖譜 + RAG 編譯器

- **GitHub：** https://github.com/swarmclawai/swarmvault
- **Stars：** ~640
- **授權：** MIT
- **技術棧：** Node.js CLI + 桌面應用

本地優先的 LLM Wiki、知識圖譜建構器與 RAG 知識庫。採用模式引導編譯（`swarmvault.schema.md`）、圖形化 Wiki（節點、邊、溯源追蹤）、反漂移 linting（過時來源、衝突聲明、覆蓋缺口）。離線啟發式工作流程無需 API 金鑰。[^swarmvault]

**vs. Open Knowledge：** 更專注於編譯流程（原始來源→Wiki+知識圖譜），而非 Markdown 編輯體驗。無 WYSIWYG。離線-first 表現出色。

### 8. MindBase——最簡 Karpathy LLM Wiki

- **GitHub：** https://github.com/frankchu91/mindbase-llm-wiki
- **Stars：** 128
- **授權：** MIT
- **技術棧：** TypeScript monorepo（MCP 伺服器 + Web UI）

Karpathy LLM Wiki 的產品化實作。AI 從你的筆記與來源建立並維護 Markdown Wiki。完全可透過 Ollama 在免費本地模型上執行——無需 API 金鑰。AI 在寫入前會先展示要點與核取清單以供人類核准。包含 Web UI WYSIWYG 編輯器。[^mindbase]

**vs. Open Knowledge：** 規模小但理念一致。AI 在每次變更前都需人類批准。對隱私極為重視。

### 9. tonbistudio/llm-wiki——最佳範本/起始套件

- **GitHub：** https://github.com/tonbistudio/llm-wiki
- **Stars：** 265
- **授權：** MIT
- **技術棧：** 目錄結構 + CLAUDE.md 範本

Karpathy 模式 LLM Wiki 的起始範本。提供現成目錄結構（`raw/` + `wiki/`）、預設 CLAUDE.md 架構、資料儀表板、間隔重複、Marp 投影片、研究日誌範本。MCP 相容。[^tonbistudio]

**vs. Open Knowledge：** 這是範本/樣板，不是應用程式。最簡單的 Karpathy LLM Wiki 入門方式，無 GUI。

### 10. AIOS-Brain——最佳 Obsidian + AI 第二大腦

- **GitHub：** https://github.com/rmarji/AIOS-Brain
- **授權：** 開源，基於 Karpathy LLM Wiki 模式

自我組織的 AI 第二大腦，整合 Obsidian 與 Claude Code。丟入任何來源，Claude 即可閱讀、關聯、歸檔至你擁有的純文字 Markdown 知識圖譜中。[^aios_brain]

**vs. Open Knowledge：** 最適合已使用 Obsidian 的使用者。非獨立產品——屬於工作流程/外掛組合。

## 比較彙總

| 專案 | 類別 | Stars | 編輯器類型 | AI 整合 | MCP | Git 同步 |
|---|---|---|---|---|---|---|
| **Open Knowledge** | 基準 | 4.4k | ✅ WYSIWYG | ✅ 原生 | ✅ 內建 | ✅ 內建 |
| **Tolaria** | 知識庫 | 20k | 區塊式 | ✅ MCP 途徑 | ✅ | ✅ 原生 |
| **LLM Wiki (nashsu)** | LLM Wiki | 20.3k | Milkdown | ✅ 原生 | ✅ | ❌ 手動 |
| **Basic Memory** | AI 記憶 | 4.1k | 外部編輯器 | ✅ MCP-native | ✅ 核心 | ❌ 手動 |
| **Markra** | Markdown 編輯器 | 908 | ✅ WYSIWYG | ✅ 原生 | ❌ | 選擇性 WebDAV |
| **Snaptium** | AI 筆記 | 63 | 即時預覽 | ✅ 原生 | ❌ | E2EE WebDAV/S3 |
| **nvk/llm-wiki** | Agent 外掛 | ~858 | 無（CLI） | ✅ Agent-native | ✅ | ❌ |
| **SwarmVault** | KG+RAG | ~640 | 無（CLI） | ✅ 選擇性 | ✅ | ❌ |
| **MindBase** | LLM Wiki | 128 | Web UI WYSIWYG | ✅ Ollama 優先 | ✅ | ❌ |
| **tonbistudio/llm-wiki** | 範本 | 265 | 無（檔案） | ⚠️ Agent 途徑 | ✅ | ❌ |

## 推薦摘要

- **最接近的全能替代方案：** Tolaria（最成熟、Git-native、AI-ready）
- **最佳 WYSIWYG 體驗：** Markra（編輯感受最接近 Open Knowledge）
- **最佳自建 LLM Wiki：** nashsu/llm_wiki（功能最豐富的 Karpathy 實作）
- **最佳 MCP/Agent 記憶整合：** Basic Memory（MCP-native，適用任何 AI 用戶端）
- **最大隱私/離線優先：** SwarmVault 或 MindBase（皆可完全離線使用）
- **Claude Code/Codex 使用者：** nvk/llm-wiki（最深入的 Agent 整合）

---

[^tolaria]: Refactoring HQ. (n.d.). Tolaria. Retrieved 2026-10-09, from https://github.com/refactoringhq/tolaria
[^nashsu_llm_wiki]: nashsu. (n.d.). LLM Wiki. Retrieved 2026-10-09, from https://github.com/nashsu/llm_wiki
[^basic_memory]: Basic Machines Co. (n.d.). Basic Memory. Retrieved 2026-10-09, from https://github.com/basicmachines-co/basic-memory
[^markra]: Markra HQ. (n.d.). Markra. Retrieved 2026-10-09, from https://github.com/markrahq/markra
[^snaptium]: Jet Yu. (n.d.). Snaptium. Retrieved 2026-10-09, from https://github.com/jetyu/Snaptium
[^nvk_llm_wiki]: nvk. (n.d.). LLM Wiki. Retrieved 2026-10-09, from https://github.com/nvk/llm-wiki
[^swarmvault]: SwarmCLAI. (n.d.). SwarmVault. Retrieved 2026-10-09, from https://github.com/swarmclawai/
[^mindbase]: Frank Chu. (n.d.). MindBase LLM Wiki. Retrieved 2026-10-09, from https://github.com/frankchu91/mindbase-llm-wiki
[^tonbistudio]: tonbistudio. (n.d.). LLM Wiki. Retrieved 2026-10-09, from https://github.com/tonbistudio/llm-wiki
[^aios_brain]: Rmarji. (n.d.). AIOS-Brain. Retrieved 2026-10-09, from https://github.com/rmarji/AIOS-Brain