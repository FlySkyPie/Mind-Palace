# FOSS 車輛建造遊戲綜合調查

本報告調查目前可用的自由及開源（FOSS）車輛建造遊戲，這類遊戲的核心玩法是讓玩家從零開始組合零件或區塊，建造出車輛、飛機、船艦、機械等載具。

## 什麼是車輛建造遊戲？

車輛建造遊戲（Vehicle Building Game）是沙盒遊戲的一個子類型，玩家透過組合零件（方塊、梁、輪子、引擎、武器等）來設計並建造可運作的載具。代表性商業作品包括《From the Depths》、《Scrap Mechanic》、《Besiege》、《Stormworks》、《Trailmakers》等，而本報告聚焦於其 FOSS 替代方案。

## 完整遊戲列表

### 1. Rigs of Rods

- **描述：** 歷史悠久的開源物理沙盒車輛模擬器，使用軟體物理（質量-彈簧-阻尼模型）。玩家可從相互連接的節點建造汽車、卡車、火車、船隻、飛機、直升機和重型機械。車輛會根據物理結構即時彎曲和變形。支援多人連線、AngelScript 模組和內建模組瀏覽器。
- **授權條款：** GNU GPL v3（或更新版本）
- **儲存庫：** https://github.com/RigsOfRods/rigs-of-rods
- **平台：** Windows、Linux、macOS
- **狀態：** **活躍開發中** — 1,200+ 星標，從 2005 年起累積 6,700+ 提交。最新穩定版為 2022.12，GitHub 上持續開發。[^ror-github]

### 2. Principia

- **描述：** 基於物理的沙盒遊戲，提供 200+ 物件（梁、輪子、馬達、齒輪、電子元件、Lua 腳本）。可建造遙控車、卡車、挖土機、機械裝置、彈珠台等。原為 2013 年發行的商業遊戲，2022 年開源。包含解謎/冒險模式和社群關卡分享網站。
- **授權條款：** BSD 3-Clause
- **儲存庫：** https://github.com/Bithack/principia
- **平台：** Windows、Linux、Android（實驗性：Haiku OS、macOS、Web）
- **狀態：** **活躍開發中** — 414+ 星標，867+ 提交。社群驅動開發，提供 nightly build 版本。[^principia-github]

### 3. Tinybox

- **描述：** 以 Godot 4（GDScript）建構的 3D 物理沙盒遊戲。支援線上多人連線、世界編輯器，以及基於物理的建造系統，可打造世界、機械裝置、車輛等。內建世界資料庫 API 供分享創作。
- **授權條款：** GNU AGPL v3
- **儲存庫：** https://github.com/caelan-douglas/tinybox
- **平台：** Windows、Linux、macOS（Godot 匯出目標）
- **狀態：** **活躍開發中** — 54 星標，426 提交。預覽測試版可供下載。[^tinybox-github]

### 4. Blocks Beyond the Stars

- **描述：** 跨星系的 3D 方塊建造沙盒遊戲，玩家逐塊建造太空船、太空站和基地。探索程式生成的星系和行星、採礦、製作、馴服生物。支援單人與多人模式（自架或官方伺服器）。使用 Unity 6（客戶端）+ .NET 10（權威伺服器）建構。特色包括 29 種行星類型、14 種語言、類 Lua 體素建造、持久多人世界。由一個父親和 10 歲兒子的 AI 輔助家庭專案。
- **授權條款：** GNU AGPL v3（或更新版本）
- **儲存庫：** https://github.com/marceld23/BlocksBeyondTheStars
- **平台：** Windows、Linux、macOS（實驗性）、WebGL（瀏覽器）
- **狀態：** **活躍開發中** — 36 星標，1,687+ 提交。可完整遊玩的測試版。[^blocks-github]

### 5. Wrecking Wheels (Vehicle-Building-Game)

- **描述：** 網頁版 2D 車輛建造遊戲。玩家組合零件來建造機械裝置以解決物理謎題和對抗敵方裝置。解鎖零件、建造車輛、完成關卡。偏解謎/機械建造風格而非開放世界沙盒。
- **授權條款：** MIT
- **儲存庫：** https://github.com/BSchoolland/Vehicle-Building-Game
- **平台：** 網頁瀏覽器（Node.js 伺服器）
- **狀態：** **活躍開發中** — 2 星標，421 提交。可在 wreckingwheels.com 遊玩。[^wheels-github]

### 6. OpenSandbox

- **描述：** 受 Garry's Mod 啟發的開源沙盒遊戲，基於 ThreeCore/SourceTech 引擎（Quake 3 分支）。可生成道具、NPC、載具、Nextbots，使用 Physgun 和 Toolgun 工具，載入 Quake 3 地圖，執行模組，支援多人連線。約 20MB。另有獨立續作專案 OpenSandbox³。
- **授權條款：** GNU GPL v2
- **儲存庫：** https://github.com/noire-dev/opensandbox-SDK（遊戲程式碼）/ https://github.com/noire-dev/threecore（引擎）
- **平台：** Windows、Linux、SteamDeck
- **狀態：** **活躍開發中** — SDK 儲存庫 16+ 星標。ModDB 有多個版本（最新：v2025.07.03）。[^opensandbox-github]

### 7. OpenBlock

- **描述：** 基於 Torque3D 遊戲引擎（MIT 授權）的開源積木建造遊戲。為 Torque3D 的分支，目標是打造類似 Blockland 的積木建造體驗。繼承 Torque3D 的車輛和物理能力。
- **授權條款：** MIT（繼承自 Torque3D）
- **儲存庫：** https://github.com/DShiznit/OpenBlock
- **平台：** Windows、Linux（Torque3D 目標）
- **狀態：** **休眠中** — 0 星標，無活躍開發。基本上是 Torque3D 的更名分支。[^openblock-github]

### 8. Land of Dran

- **描述：** 受 Blockland 啟發的開源積木建造沙盒遊戲。強調 Lua 腳本易於修改。具備多人連線、具有物理屬性的磚塊材料，以及建造/駕駛機制。
- **授權條款：** 未明確指定
- **平台：** Windows
- **狀態：** **開發中（資訊不明確）** — 最後提及於 2023 年底/2024 年初。原始碼可取得性不明。[^dran-forum]

## 榮譽提及

以下專案雖非純粹的車輛建造遊戲，但相關且值得注意：

- **Minetest/Luanti**（https://github.com/luanti-org/luanti）— 開源體素遊戲引擎。有車輛模組（汽車、船、飛機等），但非開箱即用的車輛建造遊戲。[^minetest]
- **Torcs**（https://torcs.sourceforge.net/）— 開源賽車模擬器。可建立 AI 駕駛員和實驗車輛動力學，但非建造沙盒。[^torcs]
- **CARLA**（https://carla.org/）— 開源自動駕駛模擬器。學術研究平台而非遊戲。[^carla]
- **Chunk Stories**（https://github.com/Hugobros3/chunkstories）— 以 Kotlin/Java 寫成的體素遊戲引擎，可透過模組 API 建立載具。[^chunkstories]

## 摘要比較表

| 遊戲名稱 | 授權條款 | 星標數 | 平台 | 建構技術 | 載具類型 | 狀態 |
|---|---|---|---|---|---|---|
| Rigs of Rods | GPLv3 | 1,200+ | Win/Lin/Mac | C++ (OGRE) | 汽車、卡車、飛機、船、火車、直升機 | 活躍 |
| Principia | BSD 3-Clause | 414+ | Win/Lin/Android | C++ (SDL) | 遙控車、卡車、機械裝置 | 活躍 |
| Tinybox | AGPLv3 | 54 | Win/Lin/Mac | Godot 4 | 通用物理載具/機械 | 活躍 |
| Blocks Beyond the Stars | AGPLv3 | 36 | Win/Lin/Mac/Web | Unity 6 + .NET 10 | 太空船、太空站 | 活躍 |
| Wrecking Wheels | MIT | 2 | Web | Node.js/JS | 2D 機械裝置 | 活躍 |
| OpenSandbox | GPLv2 | 16+ | Win/Lin/SteamDeck | ThreeCore (C) | 可生成載具 | 活躍 |
| OpenBlock | MIT | 0 | Win/Lin | Torque3D (C++) | 引擎內建載具 | 休眠 |
| Land of Dran | 不明 | — | Windows | C++ | 積木車輛 | 不明 |

## 結論與建議

目前 FOSS 車輛建造遊戲生態系中，**Rigs of Rods** 是最成熟且功能最完整的選擇，特別適合追求真實物理模擬的玩家。**Principia** 則提供最多樣的機械建造可能性，且跨平台支援最佳。對於偏好太空主題的玩家，**Blocks Beyond the Stars** 是唯一專注太空船建造的 FOSS 選項。若偏好輕量級網頁遊戲，**Wrecking Wheels** 可直接在瀏覽器中遊玩。

值得注意的是，與商業作品（如 From the Depths、Stormworks）相比，FOSS 車輛建造遊戲在精緻度和內容量上仍有差距，但社群驅動的開發模式正在逐步縮小這個差距。

---

[^ror-github]: Rigs of Rods. (n.d.). *Rigs of Rods — Soft-body physics vehicle simulation*. GitHub. Retrieved 2026-09-20, from https://github.com/RigsOfRods/rigs-of-rods

[^principia-github]: Bithack. (n.d.). *Principia — Physics sandbox game*. GitHub. Retrieved 2026-09-20, from https://github.com/Bithack/principia

[^tinybox-github]: Douglas, C. (n.d.). *Tinybox — Godot 4 physics sandbox*. GitHub. Retrieved 2026-09-20, from https://github.com/caelan-douglas/tinybox

[^blocks-github]: MarcelD23. (n.d.). *Blocks Beyond the Stars — Space block building sandbox*. GitHub. Retrieved 2026-09-20, from https://github.com/marceld23/BlocksBeyondTheStars

[^wheels-github]: Schoolland, B. (n.d.). *Vehicle-Building-Game — 2D contraption builder*. GitHub. Retrieved 2026-09-20, from https://github.com/BSchoolland/Vehicle-Building-Game

[^opensandbox-github]: noire-dev. (n.d.). *OpenSandbox SDK — Open source sandbox game*. GitHub. Retrieved 2026-09-20, from https://github.com/noire-dev/opensandbox-SDK

[^openblock-github]: DShiznit. (n.d.). *OpenBlock — Open source brick building game*. GitHub. Retrieved 2026-09-20, from https://github.com/DShiznit/OpenBlock

[^dran-forum]: GameDev.net. (2024). *Land of Dran — Open source block building game* [Forum post]. Retrieved 2026-09-20, from https://gamedev.net/forums/topic/715337-land-of-dran-open-source-block-building-game/

[^minetest]: Luanti Project. (n.d.). *Luanti — Open source voxel game engine*. GitHub. Retrieved 2026-09-20, from https://github.com/luanti-org/luanti

[^torcs]: TORCS. (n.d.). *TORCS — The Open Racing Car Simulator*. Retrieved 2026-09-20, from https://torcs.sourceforge.net/

[^carla]: CARLA. (n.d.). *CARLA — Open source simulator for autonomous driving research*. Retrieved 2026-09-20, from https://carla.org/

[^chunkstories]: Hugobros3. (n.d.). *Chunk Stories — Voxel game engine*. GitHub. Retrieved 2026-09-20, from https://github.com/Hugobros3/chunkstories