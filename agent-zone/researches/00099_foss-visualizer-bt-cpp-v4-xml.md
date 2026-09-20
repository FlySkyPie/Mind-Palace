# 支援 BehaviorTree.CPP V4 XML 格式之 FOSS 視覺化工具調查

## 背景

BehaviorTree.CPP 是一套廣為使用的 C++ 行為樹函式庫。V4 版引入了新版 XML 格式（`BTCPP_format="4"`），與 V3.8 版不相容。本報告調查目前市面上**完全自由開源（FOSS）** 且支援 V4 XML 格式的行為樹視覺化/編輯工具。

## 調查結果

### 1. Groot 1 — 官方原版視覺化工具

- **授權：** MIT（完全開源）
- **V4 支援：** ❌ **不相容。** 官方已明確公告「Groot 1.0 僅相容 BehaviorTree.CPP 3.8.x，不預期能與 4.x 正常運作」[^groot1-deprecation]
- **狀態：** 已棄用，僅維護模式
- **技術棧：** C++/Qt 桌面應用

### 2. Groot1Publisher — 社群橋接工具

- **授權：** MIT（完全開源）
- **V4 支援：** ✅ 可將 V4 樹序列化為 Groot 1 協定，藉此在已棄用的 Groot 1 中即時監控 V4 行為樹
- **說明：** 作為 `Groot2Publisher` 的替代品，透過 ZMQ/FlatBuffers 協定串流 V4 樹[^groot1publisher]；不需修改原始碼即可以 ROS2 或獨立 CMake 編譯使用
- **狀態：** 小型專案（約 20 stars）

### 3. Groot 2 — 官方新版編輯器 ⚠️ 非 FOSS

- **授權：** ❌ **閉源。** GitHub 僅存放更新日誌與釋出說明[^groot2-repo]，原始碼未公開。Free 方案（€0）為「免費使用」但非開源；PRO 方案（€590/年浮動授權）解鎖無限監控節點、黑板可視化、中斷點、故障注入等功能
- **V4 支援：** ✅ 相容 BT.CPP 3 與 4，提供拖放編輯器、即時 XML 預覽、ZMQ 監控、日誌回放
- **限制：** Free 方案監控與日誌視覺化上限為 **20 個節點**
- **狀態：** 積極維護（最新版 1.9.0，2026-02-14），為官方推薦的 V4 方案
- **結論：** 功能最完整，但因非開源而**不符合 FOSS 需求**

### 4. BTView — VS Code 擴充功能

- **授權：** Apache-2.0（完全開源）
- **V4 支援：** ✅ 自動偵測 V3.8 與 V4（`BTCPP_format="4"`），雙向 XML 同步（圖形編輯即時更新 XML），支援 include 解析（相對路徑、絕對路徑、ROS `ros_pkg`），另提供 V3→V4 遷移輔助與差異預覽[^btview]
- **狀態：** 積極開發（v0.9.0），VS Code Marketplace 與 Open VSX 皆可安裝；約 74 安裝數
- **適合：** 習慣在 VS Code 中工作的開發者

### 5. BehaviorTree Viewer — VS Code 擴充功能

- **授權：** MIT（完全開源）
- **V4 支援：** ✅ **純 V4** — 解析 `<root BTCPP_format="4">`、`<BehaviorTree ID=...>`、`<TreeNodesModel>`、`<SubTree/>`（可內聯展開）；提供互動式樹狀渲染、黑板檢查器、節點調色盤、搜尋功能與**即時 ZMQ 監控**（狀態疊加）[^bt-viewer]
- **狀態：** 早期版本（v0.1.2），「積極測試中，預期有 bug」；約 485 安裝數
- **定位：** 輕量級 FOSS 替代 Groot 2 的方案

### 6. BT Viz — 瀏覽器端離線單頁工具

- **授權：** MIT（完全開源）
- **V4 支援：** ✅ 明確標榜支援「BehaviorTree.CPP format 4 XML」，包含選用的 `TreeNodesModel` 區段以自訂節點顏色
- **功能：** 貼上/拖放 XML、平移/縮放、拖曳節點、摺疊子樹、即時 XML 編輯器、版面切換、小地圖、節點搜尋、多樹下拉選單[^bt-viz]
- **技術特性：** 零依賴，僅需開啟單一 HTML 檔案即可離線運作，無需伺服器
- **狀態：** 小型專案（11 commits）

### 7. bt_visualizer_pkg — ROS 2 視覺化套件

- **授權：** Apache-2.0（完全開源）
- **V4 支援：** ✅ 可直接載入與解析 BehaviorTree.CPP V4 格式 XML 檔案
- **功能：** Python/ROS 2（Jazzy）GUI，自動佈局、平移/縮放、彩色編碼節點類型、白色主題、高解析度（300 DPI）PNG 匯出[^bt-visualizer-pkg]
- **限制：** 需 ROS 2 Jazzy、Python3-PIL、Ghostscript
- **狀態：** 約 37 stars

### 8. Behavior Tree Editor（bteditor.dev）— Web 編輯器

- **授權：** MIT（開源），基於 Drawflow
- **V4 支援：** 🟡 部分 — 提供 BTCPP XML 匯出模式與「自動偵測 BTCPP」匯入，另支援 NDJSON 日誌回放與 WebSocket 即時監控
- **限制：** 屬通用行為樹編輯器，非 BT.CPP 專用，對 V4 格式（ports、`TreeNodesModel`）的解析完整度低於上述專用工具
- **狀態：** 持續維護的網站工具

## 結論與建議

| 工具 | 授權 | V4 支援 | 形式 | 特色 |
|------|------|---------|------|------|
| Groot 1 | MIT | ❌ | 桌面 | 已被官方棄用 |
| Groot1Publisher | MIT | ✅（橋接） | 桌面 | 讓 Groot 1 可看 V4 |
| Groot 2 | **閉源** | ✅ | 桌面 | 功能最完整但非 FOSS |
| BTView | Apache-2.0 | ✅ | VS Code | 雙向同步、V3→V4 遷移 |
| BehaviorTree Viewer | MIT | ✅ | VS Code | ZMQ 即時監控、黑板檢查 |
| BT Viz | MIT | ✅ | 瀏覽器（離線） | 零依賴、單檔離線 |
| bt_visualizer_pkg | Apache-2.0 | ✅ | ROS 2 GUI | ROS 2 生態專用 |
| bteditor.dev | MIT | 🟡 部分 | 瀏覽器 | 通用型 BT 編輯器 |

**最推薦的 FOSS 方案：**

1. **BTView**（VS Code）— 功能最完整、積極開發、支援編輯與雙向同步
2. **BehaviorTree Viewer**（VS Code）— 輕量、支援 ZMQ 即時監控、MIT 授權
3. **BT Viz**（瀏覽器）— 零依賴離線使用，最簡單的快速預覽方案

---

[^groot1-deprecation]: BehaviorTree/Groot. (n.d.). *Groot — Behavior Tree Editor.* Retrieved 2026-09-20, from https://github.com/BehaviorTree/Groot

[^groot1publisher]: Vishnu-Kr. (n.d.). *Groot1Publisher — BehaviorTree.CPP V4 Publisher for Groot 1.* Retrieved 2026-09-20, from https://github.com/Vishnu-Kr/Groot1Publisher

[^groot2-repo]: BehaviorTree/Groot2. (n.d.). *Groot2 — Behavior Tree Editor.* Retrieved 2026-09-20, from https://github.com/BehaviorTree/Groot2

[^btview]: guilyx. (n.d.). *BTView — Visual graph editor for BehaviorTree.CPP v3.8 and v4 XML files.* Retrieved 2026-09-20, from https://github.com/guilyx/btview-vscode-plugin

[^bt-viewer]: Neoxra. (n.d.). *BehaviorTree Viewer — VS Code extension.* Retrieved 2026-09-20, from https://github.com/Neoxra/vscode-bt-viewer

[^bt-viz]: iameijaz. (n.d.). *BT Viz — Browser-based BehaviorTree.CPP format 4 visualizer.* Retrieved 2026-09-20, from https://github.com/iameijaz/bt-viz

[^bt-visualizer-pkg]: shivcc. (n.d.). *bt_visualizer_pkg — Behavior Tree Visualizer for ROS 2.* Retrieved 2026-09-20, from https://github.com/shivcc/bt_visualizer_pkg