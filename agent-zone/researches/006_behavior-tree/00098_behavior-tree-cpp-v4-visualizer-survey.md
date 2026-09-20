# BehaviorTree.CPP V4 XML 格式可視化工具調查

## 概述

BehaviorTree.CPP 是基於 C++ 的行為樹函式庫，從第 4 版 (`BTCPP_format="4"`) 開始引入了新的 XML 格式（如 `TreeNodesModel`、`<include>`、SubTree port remapping 等），與舊的 v3 格式不相容。本報告調查支援 v4 XML 格式的可視化工具，提供選擇參考。

## 1. 官方工具

### Groot2（官方推薦）

- **類型**：官方 IDE（編輯、監控、日誌回放）
- **V4 XML 支援**：✅ 完整支援。相容 BT.CPP 3 與 4，是 v4 格式的參考工具
- **平台**：桌面版 — Windows x86_64 安裝包、Linux x86_64 安裝包、AppImage（x86_64 / aarch64）。無網頁版
- **功能**：拖放式 BT 編輯器、即時監控（透過 ZMQ 連接執行中的 BT.CPP）、日誌視覺化回放（含速度控制）、分屏雙樹檢視、即時 XML 預覽；**PRO 版**（€590/年浮動授權）額外提供黑板可視化、互動斷點、錯誤注入、執行時節點取代
- **安裝**：從 <https://www.behaviortree.dev/groot/> 下載（最新版 1.9.0，2026-02-14）。啟用即時監控需在 BT.CPP 程式碼中使用 `BT::Groot2Publisher`
- **狀態**：✅ 積極維護（頻繁發版）
- **限制**：免費版 Monitor & Log Visualizer 限制 20 節點；PRO 功能需付費授權

### Groot 1（Groot 第一代 — 已棄用）

- **V4 XML 支援**：❌ **不相容**。官方文件明確指出「Groot 1.0 只相容 BehaviorTree.CPP 3.8.x，預期無法正確搭配 BT.CPP 4.x 使用」
- **狀態**：⚠️ 維護模式／已存檔，作者不再處理 Issue
- **注意**：若仍想用 Groot 1 觀看 v4 樹，需搭配第三方橋接工具 Groot1Publisher（見後述）

### BehaviorTree.CPP 內建日誌／監控鉤子

- 函式庫本身提供 `BT::Groot2Publisher`、`BT::FileLogger2`、`BT::StdCoutLogger`，作為與 Groot2 及其他工具的整合點

## 2. VS Code / Cursor 擴充套件

### BTView（`rangonomics.btview`）

- **類型**：VS Code / Cursor 圖形編輯器
- **V4 XML 支援**：✅ 完整支援 v3.8 與 v4。提供**v3 → v4 遷移工具並附 diff 預覽**
- **功能**：互動圖形（縮放、平移、小地圖、節點檢查器）、**雙向 XML 同步**（圖形編輯即時更新 XML 檔案）、完整鍵盤操作、節點類型顏色/圖標、驗證問題面板、子樹鑽研（breadcrumb）、自動檢測 v3 vs v4、include 解析（相對/絕對路徑/ROS `ros_pkg`）、版本忠實的往返序列化
- **平台**：VS Code ≥1.85 及 Cursor（透過 Open VSX）
- **安裝**：`ext install rangonomics.btview` 或搜尋「BTView」
- **狀態**：✅ 積極維護（142 commits，CI，有路線圖文件）
- **限制**：無即時監控模式（純編輯器/視覺化工具）；ROS `ros_pkg` include 解析需要 ROS 2 工作空間

### BehaviorTree Viewer（`NicholasJamesBell.behaviortree-viewer`）

- **類型**：VS Code 視覺化 + ZMQ 即時監控
- **V4 XML 支援**：✅ 僅支援 v4（解析 `BTCPP_format="4"`、`TreeNodesModel`、SubTrees、內建節點類型）
- **功能**：樹狀渲染、節點搜尋、黑板檢查器、節點調色板、拖放佈局、**即時 ZMQ 狀態疊加**（與 Groot2 相同協定）
- **平台**：VS Code 擴充套件（桌面）
- **安裝**：從 Marketplace 安裝；開啟 v4 XML 檔案後按 `Ctrl+Shift+T` 或執行「Open Behavior Tree Viewer」
- **狀態**：⚠️ **早期開發階段**（8 stars、21 commits）— 文件說明「正積極測試中，預期有 bug」
- **限制**：監控功能需 ZMQ 依賴；成熟度較低

### Nav2 BT Editor（`DavidG-Develop.vscode-nav2-bt-editor`）

- **類型**：VS Code 圖形編輯器（ROS 2 Nav2 生態系）
- **V4 XML 支援**：✅ 支援「BehaviorTree.CPP 3.8 風格與 4.x 風格」XML（含 `root BTCPP_format="4"`、`SubTree`/`SubTreePlus`、BT.CPP 4.x script、pre/post-conditions）
- **功能**：互動圖形編輯、拖放節點排序/移動、子樹導航或內聯展開、從檔案/URL 匯入 `TreeNodesModel` 和外部 BT 作為 SubTree 模板、畸形 XML 檢測
- **平台**：VS Code 擴充套件（桌面）
- **安裝**：從 Marketplace 搜尋「Nav2 BT Editor」
- **狀態**：✅ 積極維護（75 commits，Apache-2.0）
- **限制**：Nav2/ROS 導向；無執行時監控；未知標籤保留但不做語意驗證

## 3. 獨立工具／ROS 工具

### bt_visualizer_pkg（shivcc）

- **類型**：ROS 2 視覺化工具
- **V4 XML 支援**：✅ 「直接載入並解析 BehaviorTree.CPP **v4 格式** .xml 檔案」
- **功能**：互動式樹狀圖、縮放/平移、三種主題（Dark/Light/Forest）、節點類型顏色標記、自動佈局、高解析度 PNG 匯出（300 DPI）
- **平台**：桌面（Linux / ROS 2 Jazzy；Python + Tkinter GUI）。37 stars
- **安裝**：需要 ROS 2 Jazzy、Pillow、Ghostscript。clone 至 `~/ros2_ws/src`，`colcon build` 後 `ros2 run bt_visualizer_pkg visualizer`
- **狀態**：⚠️ 小型但功能完整（7 commits）
- **限制**：需要 ROS 2 環境；純檢視器（無編輯、無即時監控）

### Groot1Publisher（Vishnu-Kr）— v4→Groot1 橋接

- **類型**：執行時橋接函式庫
- **V4 XML 支援**：✅ 間接 — 將 v4 樹序列化為 v3/Groot1 線路協定，使 Groot 1 得以顯示
- **功能**：取代 `BT::Groot2Publisher`，無需修改原始程式碼；處理 SubTree port remapping
- **平台**：C++ 函式庫（支援 ROS 2 colcon build 或獨立 CMake）
- **安裝**：`git clone https://github.com/Vishnu-Kr/Groot1Publisher.git`；CMake 或 colcon 建置
- **狀態**：⚠️ 小型社群專案（20 stars）
- **限制**：同時只能執行一個實例；限速 25 msg/sec；需 ZeroMQ；僅執行時橋接（無 XML 編輯）

## 4. 線上／網頁工具

### bteditor.dev

- **類型**：網頁編輯器 + 日誌回放 + WebSocket 即時監控
- **V4 XML 支援**：✅ 可匯入/匯出 BehaviorTree.CPP XML（含 `TreeNodesModel` 和 ports），有專門的 BTCPP XML 匯入匯出工作流程
- **功能**：拖放節點、自訂節點類型、子樹折疊、水平/垂直佈局、NDJSON **日誌回放**（含逐步/暫停）、**WebSocket 即時監控**、離線可用、無需註冊
- **平台**：網頁（所有現代瀏覽器）
- **安裝**：直接開啟 <https://bteditor.dev/>，無需安裝
- **狀態**：✅ 積極維護（文件最後更新 2026-07-16）
- **限制**：BTCPP XML 往返可能遺失編輯器中繼資料（節點位置、縮放）；自訂節點庫需手動重新匯入；首次載入需要網路

### BT-Viz（iameijaz/bt-viz）

- **類型**：輕量單一 HTML 網頁視覺化工具
- **V4 XML 支援**：✅ 專門支援格式 4（`BTCPP_format="4"` 和 `TreeNodesModel`）
- **功能**：拖放/貼上 XML、互動式畫布（縮放/平移/拖曳/折疊）、即時 XML 編輯器（樹狀結構隨輸入即時更新）、垂直/水平佈局、小地圖、節點搜尋、節點類型顏色、多 `BehaviorTree` 支援（下拉選單）。零依賴，~700 行純 Canvas 2D + Reingold–Tilford 佈局
- **平台**：網頁（Demo 於 iameijaz.github.io/bt-visualizer；亦可以 `open index.html` 離線執行）
- **安裝**：開啟連結，或 `git clone` 後直接開啟 `index.html`
- **狀態**：⚠️ 新專案（11 commits）但功能完整
- **限制**：純檢視器（無編輯/儲存功能）；無即時監控

### elliewlh behavior-tree-visualization-tool

- **類型**：網頁編輯器/驗證器
- **V4 XML 支援**：✅ 支援 SubTree 引用（範例使用 v4 風格），但文件較不完整
- **平台**：網頁（GitHub Pages）
- **安裝**：開啟 <https://elliewlh2094.github.io/behavior-tree-visualization-tool/>
- **狀態**：⚠️ 小型專案，文件有限

## 5. 不支援 V4 的工具（供參照，避免錯誤選擇）

- **behaviortrees.com** — 免費線上 AI 遊戲 BT 編輯器，僅匯出 JSON，不支援 BT.CPP V4 XML
- **通用 XML 樹狀檢視器**（如 xmltools.github.io/xml-viewer）— 可渲染 BT.CPP 檔案，但無 BT 語意（無節點類型顏色、無 ports 處理、無監控）
- **Groot 1 的第三方 fork**（如 kswlt/Groot、HPpaper/Groot）— 同樣僅支援 v3

## 比較總表

| 工具 | 類型 | V4 支援 | 平台 | 狀態 |
|---|---|---|---|---|
| **Groot2** | 官方 IDE（編輯/監控/回放） | ✅ 3+4 | 桌面（Win/Linux/AppImage） | ✅ 積極維護，免費版 + PRO €590/年 |
| **Groot 1** | 官方舊版編輯器 | ❌ 僅 v3 | 桌面（原始碼編譯） | ⚠️ 已棄用 |
| **BTView** | VS Code/Cursor 編輯器 | ✅ v3.8+v4，含遷移工具 | 桌面（VS Code 擴充） | ✅ 積極維護 |
| **BehaviorTree Viewer** | VS Code 檢視器+ZMQ監控 | ✅ 僅 v4 | 桌面（VS Code 擴充） | ⚠️ 早期開發 |
| **Nav2 BT Editor** | VS Code 編輯器 | ✅ v3.8+v4 | 桌面（VS Code 擴充） | ✅ 積極維護 |
| **bt_visualizer_pkg** | ROS 2 檢視器 | ✅ v4 | 桌面（Linux/ROS2） | ⚠️ 小型但功能完整 |
| **Groot1Publisher** | v4→Groot1 橋接 | ✅ 間接 | 桌面（C++/ROS2） | ⚠️ 小型 |
| **bteditor.dev** | 網頁編輯器+回放+監控 | ✅ BTCPP XML 匯入匯出 | 網頁 | ✅ 積極維護 |
| **BT-Viz** | 網頁檢視器（單一 HTML） | ✅ 格式 4 | 網頁 | ⚠️ 新專案 |
| **elliewlh 視覺化工具** | 網頁編輯器/驗證器 | ✅（風格未正式文件化） | 網頁 | ⚠️ 小型 |

## 選擇建議

- **完整功能需求（編輯 + 監控 + 回放）**：**Groot2** 為官方首選
- **VS Code / Cursor 整合編輯工作流**：**BTView** 最為完整（含雙向同步與 v3→v4 遷移）
- **快速檢視、無需安裝**：**bteditor.dev**（功能最全面）或 **BT-Viz**（最輕量）
- **ROS 2 生態系**：**Nav2 BT Editor**（編輯）或 **bt_visualizer_pkg**（檢視）
- **僅需執行時監控、已有 Groot 1 環境**：**Groot1Publisher** 橋接方案

## 參考來源

- BehaviorTree.CPP. (n.d.). *Groot2*. Retrieved 2026-09-20, from <https://www.behaviortree.dev/groot/> [^groot2]
- BehaviorTree.CPP. (n.d.). *Groot2 Integration*. Retrieved 2026-09-20, from <https://www.behaviortree.dev/docs/tutorial-basics/tutorial_11_groot2/> [^groot2-tut]
- BehaviorTree.CPP. (n.d.). *XML Format*. Retrieved 2026-09-20, from <https://www.behaviortree.dev/docs/learn-the-basics/xml_format/> [^xml-format]
- BehaviorTree. (n.d.). *Groot — Graphical Editor for BehaviorTree.CPP*. GitHub. Retrieved 2026-09-20, from <https://github.com/BehaviorTree/Groot> [^groot1]
- BehaviorTree. (n.d.). *Groot2*. GitHub. Retrieved 2026-09-20, from <https://github.com/BehaviorTree/Groot2> [^groot2-repo]
- guilyx. (n.d.). *btview-vscode-plugin*. GitHub. Retrieved 2026-09-20, from <https://github.com/guilyx/btview-vscode-plugin> [^btview]
- Neoxra. (n.d.). *vscode-bt-viewer*. GitHub. Retrieved 2026-09-20, from <https://github.com/Neoxra/vscode-bt-viewer/> [^bt-viewer-ext]
- DavidG-Develop. (n.d.). *vscode-nav2-bt-editor*. GitHub. Retrieved 2026-09-20, from <https://github.com/DavidG-Develop/vscode-nav2-bt-editor> [^nav2-editor]
- shivcc. (n.d.). *bt_visualizer_pkg*. GitHub. Retrieved 2026-09-20, from <https://github.com/shivcc/bt_visualizer_pkg> [^ros2-viz]
- Vishnu-Kr. (n.d.). *Groot1Publisher*. GitHub. Retrieved 2026-09-20, from <https://github.com/Vishnu-Kr/Groot1Publisher> [^groot1publisher]
- Vishnu-Kr. (2025). *Groot1Publisher: Visualize and Monitor BehaviorTree V4 Trees in Groot 1*. ROS Discourse. Retrieved 2026-09-20, from <https://discourse.openrobotics.org/t/groot1publisher-visualize-and-monitor-behaviortree-v4-trees-in-groot-1/52994> [^groot1pub-discourse]
- *Behavior Tree Editor*. (n.d.). Retrieved 2026-09-20, from <https://bteditor.dev/> [^bteditor-web]
- iameijaz. (n.d.). *bt-viz*. GitHub. Retrieved 2026-09-20, from <https://github.com/iameijaz/bt-viz> [^bt-viz]
- elliewlh2094. (n.d.). *behavior-tree-visualization-tool*. GitHub Pages. Retrieved 2026-09-20, from <https://elliewlh2094.github.io/behavior-tree-visualization-tool/> [^elliewlh-viz]
- Ros Navigation. (n.d.). *Nav2 Groot Tutorials*. Retrieved 2026-09-20, from <https://docs.nav2.org/rolling/tutorials/general_tutorials/groot_tutorials/> [^nav2-groot]
- BehaviorTree.CPP. (n.d.). *BehaviorTree.CPP Logging and Visualization*. Retrieved 2026-09-20, from <https://behaviortree.github.io/BehaviorTree.CPP/> [^bt-cpp-docs]

[^groot2]: BehaviorTree.CPP. (n.d.). Groot2. Retrieved 2026-09-20, from https://www.behaviortree.dev/groot/
[^groot2-tut]: BehaviorTree.CPP. (n.d.). Groot2 Integration. Retrieved 2026-09-20, from https://www.behaviortree.dev/docs/tutorial-basics/tutorial_11_groot2/
[^xml-format]: BehaviorTree.CPP. (n.d.). XML Format. Retrieved 2026-09-20, from https://www.behaviortree.dev/docs/learn-the-basics/xml_format/
[^groot1]: BehaviorTree. (n.d.). Groot — Graphical Editor for BehaviorTree.CPP. GitHub. Retrieved 2026-09-20, from https://github.com/BehaviorTree/Groot
[^groot2-repo]: BehaviorTree. (n.d.). Groot2. GitHub. Retrieved 2026-09-20, from https://github.com/BehaviorTree/Groot2
[^btview]: guilyx. (n.d.). btview-vscode-plugin. GitHub. Retrieved 2026-09-20, from https://github.com/guilyx/btview-vscode-plugin
[^bt-viewer-ext]: Neoxra. (n.d.). vscode-bt-viewer. GitHub. Retrieved 2026-09-20, from https://github.com/Neoxra/vscode-bt-viewer/
[^nav2-editor]: DavidG-Develop. (n.d.). vscode-nav2-bt-editor. GitHub. Retrieved 2026-09-20, from https://github.com/DavidG-Develop/vscode-nav2-bt-editor
[^ros2-viz]: shivcc. (n.d.). bt_visualizer_pkg. GitHub. Retrieved 2026-09-20, from https://github.com/shivcc/bt_visualizer_pkg
[^groot1publisher]: Vishnu-Kr. (n.d.). Groot1Publisher. GitHub. Retrieved 2026-09-20, from https://github.com/Vishnu-Kr/Groot1Publisher
[^groot1pub-discourse]: Vishnu-Kr. (2025). Groot1Publisher: Visualize and Monitor BehaviorTree V4 Trees in Groot 1. ROS Discourse. Retrieved 2026-09-20, from https://discourse.openrobotics.org/t/groot1publisher-visualize-and-monitor-behaviortree-v4-trees-in-groot-1/52994
[^bteditor-web]: Behavior Tree Editor. (n.d.). Retrieved 2026-09-20, from https://bteditor.dev/
[^bt-viz]: iameijaz. (n.d.). bt-viz. GitHub. Retrieved 2026-09-20, from https://github.com/iameijaz/bt-viz
[^elliewlh-viz]: elliewlh2094. (n.d.). behavior-tree-visualization-tool. GitHub Pages. Retrieved 2026-09-20, from https://elliewlh2094.github.io/behavior-tree-visualization-tool/
[^nav2-groot]: Ros Navigation. (n.d.). Nav2 Groot Tutorials. Retrieved 2026-09-20, from https://docs.nav2.org/rolling/tutorials/general_tutorials/groot_tutorials/
[^bt-cpp-docs]: BehaviorTree.CPP. (n.d.). BehaviorTree.CPP. Retrieved 2026-09-20, from https://behaviortree.github.io/BehaviorTree.CPP/