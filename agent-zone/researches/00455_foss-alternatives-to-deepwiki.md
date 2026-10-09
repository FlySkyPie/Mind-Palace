# DeepWiki.com 的 FOSS 替代方案

## 關於 DeepWiki

DeepWiki 是由 Cognition AI（Devin 的開發團隊）推出的 AI 驅動程式碼 documentation 平台，於 2025 年 5 月上線。其核心功能是：將 `github.com` 網域替換為 `deepwiki.com`，即可將任何公開 GitHub 儲存庫自動轉換為結構化的互動式 Wiki，包含架構圖、檔案摘要與基於 RAG 的對話式問答。[^deepwiki]

主要功能包括：
- **AI 自動生成文件** — 掃描程式碼庫後自動產生結構化 Wiki 頁面
- **架構圖** — Mermaid 格式的依賴關係與資料流程圖
- **Ask Devin 問答** — 基於程式碼庫的自然語言聊天問答，附引用來源
- **Deep Research 模式** — 跨檔案深度技術分析
- **MCP Server** — 提供 AI 代理工具（Cursor、Claude Code 等）使用的 MCP 協定端點
- **50,000+ 預索引儲存庫** — 熱門開源專案即開即用

但 DeepWiki 是封閉 SaaS，有**幻覺問題**（如 LibreOffice 專案發現它錯誤宣稱使用 Buck 建置系統）、**無需維護者許可**即產生文件、以及**資料主權疑慮**。[^concerns]

---

## FOSS 替代方案一覽

| 專案 | Stars | 語言 | 授權 | 部署方式 | 特點 |
|---|---|---|---|---|---|
| **DeepWiki-Open** | 18.2k | Python + Next.js | MIT | Docker Compose | 最受歡迎社群復刻，支援 GitHub/GitLab/BitBucket |
| **RepoWiki** | 298 | Python | MIT | `pip install` | CLI 優先、支援本機儲存庫、無需 Docker |
| **OpenDeepWiki** | 3.6k | C# + TypeScript | MIT | Docker | Semantic Kernel 整合、知識圖譜、模組化設計 |
| **Litho (deepwiki-rs)** | 3.1k | Rust | MIT | 二進位檔 | 高效能、C4 模型架構文件、已演化為 Terrain |

---

## 各方案詳細介紹

### 1. DeepWiki-Open (Grok-Wiki) ⭐ 18.2k

**GitHub**: https://github.com/AsyncFuncAI/deepwiki-open  
**授權**: MIT  
**語言**: Python + Next.js  
**部署**: Docker Compose

目前最受歡迎的 DeepWiki 開源復刻版。支援 GitHub、GitLab、BitBucket 三大平台，具備 Mermaid 架構圖、RAG 問答、多輪研究等功能。支援多種 LLM 後端（OpenAI、Gemini、Ollama 本地模型、OpenRouter）。提供桌面版應用（grok-wiki.com）。[^deepwiki-open]

**優點**: 專案成熟度高、社群活躍、功能最接近 DeepWiki  
**缺點**: 需要 Docker 環境、不支援本機儲存庫

---

### 2. RepoWiki ⭐ 298

**GitHub**: https://github.com/he-yufeng/RepoWiki  
**授權**: MIT  
**語言**: Python  
**安裝**: `pip install repowiki`

輕量級 CLI 為主的方案，最大的特色是**支援本機儲存庫**，可直接對本地專案產生文件，無需 Docker 或資料庫。內建 PageRank 分析自動偵測重要檔案，支援增量重新掃描，以及 Git diff 審查排序功能。提供終端機問答（`repowiki chat`）基於 TF-IDF 無需 embeddings 服務。[^repowiki]

**優點**: 零依賴（只需 Python + SQLite）、CLI 體驗佳、支援本機與遠端儲存庫、可輸出 Markdown/JSON/HTML  
**缺點**: 專案較新、社群較小、Web UI 功能較基本

---

### 3. OpenDeepWiki ⭐ 3.6k

**GitHub**: https://github.com/AIDotNet/OpenDeepWiki  
**授權**: MIT  
**語言**: C# (.NET 10) + TypeScript  
**部署**: Docker

由 AIDotNet 開發的 C# 生態系方案，整合 Semantic Kernel 與知識圖譜技術。設計上強調模組化，支援 Git 儲存庫、ZIP 壓縮檔與本機目錄。內建 MCP server 可供 AI 代理工具使用，並提供公共站台（opendeep.wiki）。[^opendeepwiki]

**優點**: 知識圖譜能力強、MCP 原生支援、模組化設計易擴展  
**缺點**: 需要 .NET 執行環境、社群規模中等

---

### 4. Litho (deepwiki-rs) ⭐ 3.1k

**GitHub**: https://github.com/sopaco/deepwiki-rs  
**授權**: MIT  
**語言**: Rust  
**安裝**: 編譯二進位檔

高效能的 Rust 實作，使用 C4 模型（Context/Container/Component/Code）產生架構文件。支援 Rust、Python、Java、Go 等多種語言。作者已將該專案進一步演化為 **Terrain**，提供 AI 代理所需的即時程式碼地圖與 ACP 協定支援。[^litho]

**優點**: 極高效能、C4 模型專業文件、二進位檔零依賴部署  
**缺點**: 偏重架構文件而非完整 Wiki、設定較複雜

---

## 選擇建議

- **想要功能最接近 DeepWiki** → DeepWiki-Open（社群最大、功能最齊全）
- **輕量 CLI 派、離線使用** → RepoWiki（`pip install` 即用、本機儲存庫）
- **C#/.NET 生態系** → OpenDeepWiki（知識圖譜 + MCP 原生支援）
- **高效能架構文件生成** → Litho/Terrain（Rust、C4 模型）

---

## 參考資料

[^deepwiki]: Cognition AI. (2025). DeepWiki — AI documentation you can talk to. Retrieved 2026-10-09, from https://deepwiki.com/

[^concerns]: BigGo Finance. (2025-08-27). DeepWiki Backlash Over Inaccurate Docs: LibreOffice and LLVM Communities Raise Accuracy Concerns. Retrieved 2026-10-09, from https://finance.biggo.com/news/202508270142_DeepWiki_Accuracy_Concerns

[^deepwiki-open]: AsyncFuncAI. (n.d.). deepwiki-open: Open Source DeepWiki. GitHub. Retrieved 2026-10-09, from https://github.com/AsyncFuncAI/deepwiki-open

[^repowiki]: he-yufeng. (n.d.). RepoWiki: Open-source DeepWiki alternative. GitHub. Retrieved 2026-10-09, from https://github.com/he-yufeng/RepoWiki

[^opendeepwiki]: AIDotNet. (n.d.). OpenDeepWiki: Open-source version of DeepWiki. GitHub. Retrieved 2026-10-09, from https://github.com/AIDotNet/OpenDeepWiki

[^litho]: sopaco. (n.d.). deepwiki-rs (Litho): High-performance AI documentation generator. GitHub. Retrieved 2026-10-09, from https://github.com/sopaco/deepwiki-rs