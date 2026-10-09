# Blender MCP 伺服器一覽

Blender 的 MCP (Model Context Protocol) 生態系已有數個成熟專案，讓 AI 編程助手（如 Claude、Cursor、VSCode 等）能直接操控 Blender 進行 3D 建模、場景管理、材質設定與渲染。

## 共通架構

多數 Blender MCP 採用**三層架構**：

```
AI 客戶端 (stdio/HTTP) ↔ MCP 伺服器 (Python) ↔ Blender 附加元件 (TCP/JSON-RPC) ↔ Blender bpy API
```

MCP 伺服器作為橋樑，將 AI 的請求以 JSON-RPC 傳送至 Blender 內的附加元件，後者在 Blender 主執行緒上執行指令並回傳結果。

## 專案列表

### 1. ahujasid/mcp-for-blender — 原創、最受歡迎

- **GitHub**: <https://github.com/ahujasid/mcp-for-blender>
- **Stars**: ⭐ 30,300+
- **授權**: MIT
- **PyPI**: `mcp-for-blender`

**亮點**: 原始 Blender MCP 專案，社群最大。單鍵安裝 (`uvx mcp-for-blender setup`)，自動偵測 Claude Desktop、Claude Code、Cursor、VSCode 等客戶端。支援多角度視埠螢幕截圖、相機渲染、Poly Haven/Sketchfab/Poly Pizza 免費素材整合，以及 Hyper3D Rodin / Tripo / Hunyuan3D 等 AI 模型生成[^ahujasid]。

### 2. RFingAdam/mcp-blender — 工具最多（218 個）

- **GitHub**: <https://github.com/RFingAdam/mcp-blender>
- **Stars**: ⭐ 21
- **授權**: AGPL-3.0

**亮點**: 全生態系工具最多的專案，涵蓋 18 類別。獨有**自我優化迴圈**：`render_multi_angle` → `analyze_viewport`（Ollama 視覺辨識）→ `refine_iteration`。另含 MSFS 航班模擬器素材管線（LOD、碰撞網格、材質、塗裝轉移）及 21 種 AI 3D 生成後端[^rfingadam]。

### 3. seehiong/blender-mcp-bridge — 錄影回放、3D 列印

- **GitHub**: <https://github.com/seehiong/blender-mcp-bridge>
- **Stars**: ⭐ 66
- **授權**: MIT
- **網站**: <https://seehiong.github.io/blender-mcp-bridge/>

**亮點**: Bridge Sessions 可記錄所有工具呼叫為 JSON 並回放（含參數分支）。專注 3D 列印管線：網格檢查、修復、體素重構、多色 3D 列印。附 **Blender Studio**（Vite+React 視覺編輯器）及 n8n 整合[^seehiong]。

### 4. PatrykIti/blender-ai-mcp — 目標導向路由

- **GitHub**: <https://github.com/PatrykIti/blender-ai-mcp>
- **Stars**: ⭐ 65
- **授權**: Apache 2.0

**亮點**: 獨特**目標優先**（Goal-First）設計——先以 `router_set_goal` 設定目標，系統再選取工作流程。巨集工具（`macro_cutout_recess`、`macro_relative_layout` 等）為高層次操作。含確定性驗證工具（`scene_measure_*`、`scene_assert_*`）與可插拔視覺執行環境[^patrykiti]。

### 5. Blender Foundation（官方）— Blender Lab 專案

- **網站**: <https://www.blender.org/lab/mcp-server/>
- **原始碼**: <https://projects.blender.org/lab/blender_mcp>
- **最低版本**: Blender 5.1

**亮點**: Blender Foundation 官方專案。提供場景分析（多邊形計數、效能瓶頸）、資料區塊重新命名、自然語言查詢資料關係、Geometry Nodes 文件自動生成等。拖曳安裝 `.mcpb` 套件即可使用[^blenderlab]。

### 6. niewinny/bMCP — Blender 擴充功能，無需 Python 安裝

- **GitHub**: <https://github.com/niewinny/bMCP>
- **Stars**: ⭐ 5
- **授權**: GPL-3.0
- **最低版本**: Blender 4.2+

**亮點**: 以 Blender 擴充功能（`.bmcp`）安裝，使用 Blender 自帶 Python，不需額外環境。支援三種傳輸模式：Streamable HTTP（建議）、SSE、Stdio。內建安全機制（localhost-only、可選授權 token）[^bMcp]。

### 7. naab007/blender_mcp — 擴充分支（92 工具）

- **GitHub**: <https://github.com/naab007/blender_mcp>
- **Stars**: ⭐ 2
- **授權**: MIT（從 ahujasid 分支）

**亮點**: 延伸原版至 92 個工具，涵蓋 26 類別。加入本端 TripoSR 影像轉 3D 推論、Blender 4.x 相容性修正、移除遙測[^naab007]。

### 8. ooMike1/BlenderMCP — 輕量簡潔

- **GitHub**: <https://github.com/ooMike1/BlenderMCP>
- **Stars**: ⭐ 3
- **授權**: MIT

**亮點**: 極簡實作，約 8 個檔案。支援基本幾何體、布林運算、修飾器、材質、網格操作及多種匯出格式（OBJ、FBX、STL、PLY）[^oomike]。

## 比較總表

| 專案 | Stars | 工具數 | 授權 | 關鍵特色 |
|---|---|---|---|---|
| **ahujasid/mcp-for-blender** | 30,300+ | ~30–40 | MIT | 最受歡迎、單鍵安裝、免費素材整合 |
| **RFingAdam/mcp-blender** | 21 | 218 | AGPL-3.0 | 工具最多、MSFS 管線、視覺自我優化 |
| **seehiong/blender-mcp-bridge** | 66 | 106 | MIT | 錄影回放、3D 列印、Blender Studio |
| **PatrykIti/blender-ai-mcp** | 65 | ~80+ | Apache 2.0 | 目標路由、巨集工具、確定性驗證 |
| **Blender Foundation 官方** | — | — | — | 官方專案、場景分析、Blender 5.1+ |
| **niewinny/bMCP** | 5 | — | GPL-3.0 | 無需 Python、三種傳輸模式 |
| **naab007/blender_mcp** | 2 | 92 | MIT | 擴充分支、本端 TripoSR |
| **ooMike1/BlenderMCP** | 3 | ~15 | MIT | 輕量簡潔 |

## 推薦

- **入門/快速上手**: `mcp-for-blender`（ahujasid）— 單鍵安裝，社群最大，文件完整。
- **最大工具量**: `mcp-blender`（RFingAdam）— 218 工具，涵蓋航班模擬、AI 生成。
- **3D 列印需求**: `blender-mcp-bridge`（seehiong）— 錄影回放、多色列印管線、Blender Studio。
- **結構化/生產級**: `blender-ai-mcp`（PatrykIti）— 目標路由、巨集、確定性驗證。
- **官方支援**: Blender Foundation 官方 MCP — 適合 Blender 5.1+ 使用者，需要場景分析最佳化。

---

[^ahujasid]: ahujasid. (n.d.). MCP for Blender. Retrieved 2026-10-09, from <https://github.com/ahujasid/mcp-for-blender>
[^rfingadam]: RFingAdam. (n.d.). mcp-blender. Retrieved 2026-10-09, from <https://github.com/RFingAdam/mcp-blender>
[^seehiong]: seehiong. (n.d.). Blender MCP Bridge. Retrieved 2026-10-09, from <https://github.com/seehiong/blender-mcp-bridge>
[^patrykiti]: PatrykIti. (n.d.). blender-ai-mcp. Retrieved 2026-10-09, from <https://github.com/PatrykIti/blender-ai-mcp>
[^blenderlab]: Blender Foundation. (n.d.). Blender Lab — MCP Server. Retrieved 2026-10-09, from <https://www.blender.org/lab/mcp-server/>
[^bMcp]: niewinny. (n.d.). bMCP. Retrieved 2026-10-09, from <https://github.com/niewinny/bMCP>
[^naab007]: naab007. (n.d.). blender_mcp. Retrieved 2026-10-09, from <https://github.com/naab007/blender_mcp>
[^oomike]: ooMike1. (n.d.). BlenderMCP. Retrieved 2026-10-09, from <https://github.com/ooMike1/BlenderMCP>