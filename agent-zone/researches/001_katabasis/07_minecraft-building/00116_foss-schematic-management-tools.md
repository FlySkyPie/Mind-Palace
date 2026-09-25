# FOSS 用於管理 Minecraft `.schematic` 檔案的工具

## 概述

本文調查適用於 Minecraft `.schematic`（地圖／建築結構）檔案的**自由開源軟體 (FOSS)**，涵蓋桌面應用程式、遊戲內模組、網頁檢視器、命令列工具及程式庫。`.schematic` 為 Minecraft 社群廣泛使用的結構儲存格式，最初由 MCEdit 及 WorldEdit 定義。

---

## 桌面應用程式

### 1. MCEdit 2.0

完整的地圖編輯器，支援匯入/匯出及編輯 `.schematic` 檔案，提供 3D 檢視、選取工具與複製貼上功能。

- **授權：** BSD
- **語言：** Python
- **GitHub：** [github.com/mcedit/mcedit2](https://github.com/mcedit/mcedit2)

### 2. Amulet Map Editor

現代化 Minecraft 地圖編輯器，支援 Java 1.12+ 及 Bedrock 1.7+。可在多種格式間轉換（`.schematic`、`.schem`、`.mcstructure`、`.construction`）。

- **授權：** 開放原始碼
- **語言：** Python
- **GitHub：** [github.com/Amulet-Team/Amulet-Map-Editor](https://github.com/Amulet-Team/Amulet-Map-Editor)
- **網站：** [amuletmc.com](https://www.amuletmc.com/)

### 3. NBTExplorer

圖形化 NBT 資料編輯器，可低階檢視與編輯 `.schematic` 檔案的原始 NBT 結構、區塊 Palette 與中繼資料。

- **授權：** MIT
- **語言：** C# (.NET / Mono)
- **GitHub：** [github.com/jaquadro/NBTExplorer](https://github.com/jaquadro/NBTExplorer)

### 4. Schemy

跨平台（Windows、macOS、Linux、Android）3D 檢視器，支援互動式軌道操作、實例化渲染、紋理支援及 Windows Shell 整合（縮圖與預覽窗格）。

- **授權：** MIT
- **語言：** TypeScript/Electron
- **格式：** `.schematic`、`.schem` (v1–v3)、`.nbt`、`.litematic`
- **GitHub：** [github.com/Sablednah/Schemy](https://github.com/Sablednah/Schemy)

### 5. LitematicaPreview（Windows 限定）

基於 Tauri（Rust + TypeScript）的本機 3D 預覽工具，使用 Nucleation 解析管線，支援多種格式，渲染效能極佳。

- **授權：** AGPL-3.0
- **GitHub：** [github.com/Arcadi4/LitematicaPreview](https://github.com/Arcadi4/LitematicaPreview)

---

## 遊戲內模組

### 6. WorldEdit（EngineHub）

Minecraft 最經典的地圖編輯模組，可建立、儲存、載入 `.schem`（Sponge v2/v3）及 `.schematic`（舊版），具備複製/貼上、筆刷、腳本化及剪貼簿管理功能。

- **授權：** GPL-3.0
- **語言：** Java
- **GitHub：** [github.com/EngineHub/WorldEdit](https://github.com/EngineHub/WorldEdit)
- **網站：** [enginehub.org/worldedit](https://enginehub.org/worldedit/)

### 7. Litematica

客戶端模組（Fabric、LiteLoader），以疊層/幽靈模式顯示結構，提供放置引導、貼上、區域克隆、移動、填充等功能，使用 `.litematic` 格式。

- **授權：** LGPL-3.0
- **語言：** Java
- **GitHub：** [github.com/maruohon/litematica](https://github.com/maruohon/litematica)

### 8. Schematica

Litematica 的前身（Forge 模組），將結構以幽靈疊層顯示，方便逐塊建造。

- **授權：** MIT
- **語言：** Java
- **GitHub：** [github.com/Lunatrius/Schematica](https://github.com/Lunatrius/Schematica)

---

## 網頁應用程式（線上檢視器）

### 9. @craftfrom/schema-viewer

npm 套件與網頁應用，使用 Three.js 在 3D 中載入與比較 `.litematic`/`.schematic` 檔案，具備圖庫側欄與拖放功能。

- **授權：** MIT
- **GitHub：** [github.com/arpitjp/craft-from-schema-viewer](https://github.com/arpitjp/craft-from-schema-viewer)
- **線上展示：** [arpitjp.github.io/craft-from-schema-viewer](https://arpitjp.github.io/craft-from-schema-viewer/)

### 10. minecraft-schematic-viewer（Jopgood）

支援大型結構最佳化渲染的 3D 檢視器，具備圖層檢視、紋理圖集、實例化渲染、線框模式及 Litematica 專用解析器。

- **授權：** MIT
- **GitHub：** [github.com/Jopgood/minecraft-schematic-viewer](https://github.com/Jopgood/minecraft-schematic-viewer)

### 11. schematic.viewer（Davide0995）

瀏覽器端檢視器，所有處理在本機進行（不上傳）。支援隱藏面剔除、線框疊層、Y 軸切片逐層檢視、告示牌文字顯示及自訂資源包。

- **授權：** MIT
- **GitHub：** [github.com/Davide0995/schematic.viewer](https://github.com/Davide0995/schematic.viewer)
- **線上展示：** [davide0995.github.io/schematic.viewer](https://davide0995.github.io/schematic.viewer/)

### 12. Minecraft Schematic Visualizer for Builder

提供俯視 2D 逐層藍圖與完整紋理 3D 檢視，具有「斷層掃描照明」（僅亮目前圖層）、材料清單（含堆疊感知庫存計算）及 GPU 實例化渲染。

- **GitHub：** [github.com/rombri02/MinecraftSchematicVisualizer_forBuilder](https://github.com/rombri02/MinecraftSchematicVisualizer_forBuilder)

### 13. Litematic Viewer（endingcredits）

輕量網頁檢視器，支援拖放、URL 載入與環繞/軌道操作。

- **GitHub：** [github.com/endingcredits/litematic-viewer](https://github.com/endingcredits/litematic-viewer)
- **線上瀏覽：** [endingcredits.github.io/litematic-viewer](https://endingcredits.github.io/litematic-viewer/)

---

## 命令列工具 (CLI)

### 14. schem — minecraft-schematic-manager

自稱為「Minecraft 結構的 ImageMagick」，可在所有主流格式間轉換，提供 CSG 語言進行程式化結構生成，可將 3D 模型（OBJ/STL）立體像素化為結構，並支援裁切、合併、取代、旋轉與鏡射。

- **授權：** MIT
- **語言：** Python
- **GitHub：** [github.com/vakermit/minecraft-schematic-manager](https://github.com/vakermit/minecraft-schematic-manager)

### 15. craftmatic

Node.js 結構工具組，可解析、產生、渲染與轉換 `.schem` 檔案。支援 10 種結構類型、9 種風格預設、20 種房間類型、紋理化 2D PNG 渲染（平面圖、剖面圖、外觀圖）、互動式 3D 檢視及獨立 HTML 檢視器匯出。

- **授權：** MIT
- **GitHub：** [github.com/tribixbite/craftmatic](https://github.com/tribixbite/craftmatic)
- **線上：** [craftmatic.click](https://craftmatic.click/)

---

## 程式庫與引擎

### 16. Nucleation（Schem-at）

高效能 Rust 結構引擎，提供 **7 種語言繫結**（Rust、JavaScript/TypeScript、Python、Kotlin/JVM、PHP、C、C++）。支援載入/建造/模擬/網格化/渲染結構、SDF 地形生成、3D 模型立體像素化（GLB/OBJ）、紅石模擬、精確刻模擬、世界分割、區塊資料庫（1196 種方塊）、可插拔儲存（記憶體、檔案系統、SSH、S3、Redis、Postgres），以及 Verilog → `.schem` 紅石 EDA 編譯器。

- **授權：** MIT
- **GitHub：** [github.com/Schem-at/Nucleation](https://github.com/Schem-at/Nucleation)
- **PyPI：** `pip install nucleation`
- **npm：** `npm install nucleation`

### 17. litemapy（SmyerMC）

Python 版 `.litematic` 檔案讀寫程式庫，提供直覺的類別操作結構、區域、方塊狀態、實體與方塊實體。

- **GitHub：** [github.com/SmyerMC/litemapy](https://github.com/SmyerMC/litemapy)

### 18. mcschematic

Python 套件，用於程式化建立 Minecraft `.schem` 結構檔案。

- **PyPI：** [pypi.org/project/mcschematic](https://pypi.org/project/mcschematic/)

---

## 資產管理平台

### 19. mc-schematic-manager-web（NikolaMilinkovic）

基於 Vite/React/Express/MongoDB 的網頁結構資產管理系統，處理結構的儲存、顯示與存取管理，並可透過 FAWE 轉換後上傳至伺服器。

- **GitHub：** [github.com/NikolaMilinkovic/mc-schematic-manager](https://github.com/NikolaMilinkovic/mc-schematic-manager)

### 20. Schemat.io

結構分享、檢視與管理平台，提供瀏覽器內 3D 檢視、格式轉換、Discord 整合（附開源 Discord 機器人）。

- **網站：** [schemat.io](https://schemat.io/)
- **開源機器人：** [github.com/Schem-at/schemati-bot](https://github.com/Schem-at/schemati-bot)

---

## 總結對照表

| 工具 | 類型 | 支援格式 | 授權 |
|------|------|---------|------|
| MCEdit 2 | 桌面編輯器 | `.schematic` | BSD |
| Amulet Editor | 桌面編輯器 | 多格式 | 開源 |
| NBTExplorer | 桌面編輯器 | `.schematic` + NBT | MIT |
| Schemy | 桌面 3D 檢視器 | `.schematic`、`.schem`、`.litematic`、`.nbt` | MIT |
| LitematicaPreview | 桌面 3D 檢視器 | `.litematic`、`.schem`、`.schematic`、`.nbt`、`.mcstructure` | AGPL-3.0 |
| WorldEdit | 遊戲內模組 | `.schem`、`.schematic` | GPL-3.0 |
| Litematica | 遊戲內模組 | `.litematic` | LGPL-3.0 |
| Schematica | 遊戲內模組 | `.schematic` | MIT |
| @craftfrom/schema-viewer | 網頁檢視器 | `.litematic`、`.schematic` | MIT |
| schematic.viewer | 網頁檢視器 | `.litematic`、`.schematic`、`.nbt` | MIT |
| schem (CLI) | 命令列 + CSG | 6+ 格式 | MIT |
| craftmatic | CLI + 程式庫 + 網頁 | `.schem` | MIT |
| Nucleation | 引擎/程式庫 | 所有主流格式 | MIT |
| mc-schematic-manager | 網頁資產管理 | 多格式 | 開源 |

---

## 參考來源

- MCEdit 2 (n.d.). Retrieved 2026-09-22, from [https://github.com/mcedit/mcedit2](https://github.com/mcedit/mcedit2)
- Amulet-Team (n.d.). Amulet Map Editor. Retrieved 2026-09-22, from [https://github.com/Amulet-Team/Amulet-Map-Editor](https://github.com/Amulet-Team/Amulet-Map-Editor)
- jaquadro (n.d.). NBTExplorer. Retrieved 2026-09-22, from [https://github.com/jaquadro/NBTExplorer](https://github.com/jaquadro/NBTExplorer)
- Sablednah (n.d.). Schemy. Retrieved 2026-09-22, from [https://github.com/Sablednah/Schemy](https://github.com/Sablednah/Schemy)
- Arcadi4 (n.d.). LitematicaPreview. Retrieved 2026-09-22, from [https://github.com/Arcadi4/LitematicaPreview](https://github.com/Arcadi4/LitematicaPreview)
- EngineHub (n.d.). WorldEdit. Retrieved 2026-09-22, from [https://github.com/EngineHub/WorldEdit](https://github.com/EngineHub/WorldEdit)
- maruohon (n.d.). Litematica. Retrieved 2026-09-22, from [https://github.com/maruohon/litematica](https://github.com/maruohon/litematica)
- Lunatrius (n.d.). Schematica. Retrieved 2026-09-22, from [https://github.com/Lunatrius/Schematica](https://github.com/Lunatrius/Schematica)
- arpitjp (n.d.). @craftfrom/schema-viewer. Retrieved 2026-09-22, from [https://github.com/arpitjp/craft-from-schema-viewer](https://github.com/arpitjp/craft-from-schema-viewer)
- Jopgood (n.d.). minecraft-schematic-viewer. Retrieved 2026-09-22, from [https://github.com/Jopgood/minecraft-schematic-viewer](https://github.com/Jopgood/minecraft-schematic-viewer)
- Davide0995 (n.d.). schematic.viewer. Retrieved 2026-09-22, from [https://github.com/Davide0995/schematic.viewer](https://github.com/Davide0995/schematic.viewer)
- rombri02 (n.d.). Minecraft Schematic Visualizer for Builder. Retrieved 2026-09-22, from [https://github.com/rombri02/MinecraftSchematicVisualizer_forBuilder](https://github.com/rombri02/MinecraftSchematicVisualizer_forBuilder)
- endingcredits (n.d.). Litematic Viewer. Retrieved 2026-09-22, from [https://github.com/endingcredits/litematic-viewer](https://github.com/endingcredits/litematic-viewer)
- vakermit (n.d.). minecraft-schematic-manager. Retrieved 2026-09-22, from [https://github.com/vakermit/minecraft-schematic-manager](https://github.com/vakermit/minecraft-schematic-manager)
- tribixbite (n.d.). craftmatic. Retrieved 2026-09-22, from [https://github.com/tribixbite/craftmatic](https://github.com/tribixbite/craftmatic)
- Schem-at (n.d.). Nucleation. Retrieved 2026-09-22, from [https://github.com/Schem-at/Nucleation](https://github.com/Schem-at/Nucleation)
- NikolaMilinkovic (n.d.). mc-schematic-manager. Retrieved 2026-09-22, from [https://github.com/NikolaMilinkovic/mc-schematic-manager](https://github.com/NikolaMilinkovic/mc-schematic-manager)
- Schemat.io (n.d.). Retrieved 2026-09-22, from [https://schemat.io/](https://schemat.io/)
