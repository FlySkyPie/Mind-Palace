# Blockbench MCP 生態系調查

> Blockbench (https://github.com/JannisX11/blockbench) 是一款跨平台的 3D 模型編輯器，主要用於 Minecraft 模型製作。本報告調查其 MCP (Model Context Protocol) 相關整合方案。

## 概述

Blockbench 生態系中有多個獨立開發的 MCP 伺服器/外掛，讓 AI 工具能直接透過 MCP 協定來操作 Blockbench 進行 3D 模型的建立、編輯、動畫、紋理繪製等工作。

---

## 1. jasonjgardner/blockbench-mcp-plugin (493★)

**倉儲：** https://github.com/jasonjgardner/blockbench-mcp-plugin

目前最成熟、最多人採用的方案，提供 **兩種 MCP 伺服器模式**：

- **無頭 `.bbmodel` (stdio)** — 無需開啟 Blockbench 即可直接編輯 `.bbmodel` 檔案。可透過 `npx -y github:jasonjgardner/blockbench-mcp-plugin --root ./models` 執行，支援多個 Agent 平行操作。
- **桌面外掛 (HTTP)** — 以 HTTP 伺服器直接在 Blockbench 內執行（透過外掛），設定為 Streamable HTTP MCP 伺服器，端點為 `http://localhost:3000/bb-mcp`。

**支援客戶端：** Claude Desktop、Claude Code、VS Code、Cline、Cursor、Codex、OpenCode、Ollama、Antigravity

**主要特色：** Agent skills 協調、外掛擴充性（其他外掛可透過 `MCP_QUEUE` 註冊 MCP tools）、risky_eval、螢幕截圖

**授權：** GPL-3.0

**相關範例專案：** https://github.com/jasonjgardner/blockbench-mcp-project

---

## 2. sosadly/blockbench-mcp (28★)

**倉儲：** https://github.com/sosadly/blockbench-mcp

提供 **70 個以上 MCP tools**，是工具數量最豐富的方案。採用雙組件架構：一個橋接外掛在 Blockbench 內執行（`plugin/blockbench_mcp.js`），於 `127.0.0.1:8787` 開啟本機 HTTP 端點；另一個 Node.js MCP 伺服器（stdio transport）由 AI 客戶端啟動。

**獨特工具：**
- 程序性細節工具：`voxelize_matrix`、`add_hollow_volume`、`generate_array`、`extrude_chain`、`add_wing`
- 品質閘門：`audit_complexity`（方塊預算評分）、`check_rig`、`check_model`
- 人工審查閘門：`request_review`（會阻塞直到使用者按下按鈕）
- 參考比對：`compare_reference`（計算輪廓 IoU 分數）
- 骨架：`create_rig`（建立分段式人類/四足骨架）
- 動畫：`generate_animation`（完整基礎動畫循環：idle/walk/run/attack/cast/jump/hurt/death/fly，含步態相位）

**支援格式：** Minecraft Java Edition、Bedrock、GeckoLib、通用 glTF 匯出

**授權：** MIT

**需求：** Blockbench 4.8+、Node.js 18+

---

## 3. vasyacullin/Blockbench-mcp (1★)

**倉儲：** https://github.com/vasyacullin/Blockbench-mcp

採用 **TCP bridge** 架構。外掛（`plugin/blockbench_mcp_bridge.js`）在 Blockbench 內執行，透過 Blockbench 的 `net` 模組在 `127.0.0.1:19888` 開啟原始 TCP socket。

**獨特特色：**
- **認證 token 機制** — 需共用 token 才能連線，防止 CSRF
- **通用動作橋接** — `bb_list_actions` 列舉所有 Blockbench 指令，`bb_run_action` 依 ID 執行任意指令
- `bb_execute_script` — 任意 JavaScript 逃脫機制
- 完整覆蓋建模、紋理、動畫、渲染工具

---

## 4. enfpdev/blockbench-mcp (49★)

**倉儲：** https://github.com/enfpdev/blockbench-mcp
**LobeHub 登錄：** https://lobehub.com/mcp/enfpdev-blockbench-mcp

Monorepo 結構，包含 `apps/mcp-server`（Node.js MCP 伺服器）與 `apps/mcp-plugin`（Blockbench 外掛，使用 Socket.IO 連接埠 9999）。採用 WebSocket 為基礎的雙向通訊架構。

**主要特色：** 即時雙向通訊、專屬歷史面板、可擴充架構

**授權：** MIT

**需求：** Blockbench 4.0+、Node.js 18+

---

## 5. bbmcp

**登錄頁：** https://mcpmarket.com/server/bbmcp

以 AI 自然語言與 3D 資產創作為核心的 Blockbench 外掛。特色包括自動 UV atlas 管理與像素重投影、自動 UV 密度降低以防止 atlas 溢位、以及安全並行編輯的修訂防護機制。

---

## 6. 相關資源一覽

| 資源 | 網址 |
|---|---|
| Agent Skill：blockbench-mcp-overview | https://eliteai.tools/agent-skills/blockbench-mcp-overview |
| Claude Marketplace 登錄 | https://claudemarketplaces.com/mcp/jasonjgardner/blockbench-mcp-plugin |
| MCP Repository (CyperNexus) | https://mcprepository.com/cypernexus/blockbench-mcp |
| DeepWiki 技術解說 | https://deepwiki.com/jasonjgardner/blockbench-mcp-plugin |
| Glama.ai 部署頁 | https://glama.ai/mcp/servers/jasonjgardner/blockbench-mcp-plugin |
| MCP World | https://www.mcpworld.com/en/detail/e78dda4567e06cb0c58d5c1a511258fa |
| Find MCP Servers | https://findmcpservers.com/server/blockbench-mcp-plugin |
| MCPlane | https://www.mcplane.com/mcp_servers/blockbenchplugin |

## 7. 真實應用案例

一篇日英雙語部落格文章（Minecraft 附加元件開發者）記錄了使用 **Blockbench MCP + Claude Code** 來產生「袋中金魚」自訂物品的過程[^note]。AI 成功建立了：
- 9 方塊的金魚 3D 模型
- 袋子與水面模型
- 128×128 像素藝術紋理
- 閒置動畫（袋子搖晃）
- 16×16 物品圖示
- 玩家手持顯示設定

開發者回報整個流程耗時「幾分鐘」而非手動的半日，結果僅需約 5 分鐘微調。

[^note]: mirinhoshi. (n.d.). Blockbench MCP で遊ぶ. Retrieved 2026-10-09, from https://note.com/mirinhoshi/n/n283d219b037d?hl=en

---

## 比較總結

| 專案 | Stars | 架構 | 主要優勢 |
|---|---|---|---|
| **jasonjgardner/blockbench-mcp-plugin** | 493 | HTTP + stdio | 最受歡迎；雙模式（無頭/外掛）；Agent skills 協調 |
| **sosadly/blockbench-mcp** | 28 | stdio + HTTP bridge | 工具數量最多（70+）；程序性細節；品質閘門；人工審查 |
| **vasyacullin/Blockbench-mcp** | 1 | TCP bridge + 認證 | 完整 Blockbench API 覆蓋；認證 token 安全機制 |
| **enfpdev/blockbench-mcp** | 49 | WebSocket (Socket.IO) | Monorepo；即時雙向通訊 |
| **bbmcp** | N/A | 外掛 | UV atlas 管理；安全並行編輯 |