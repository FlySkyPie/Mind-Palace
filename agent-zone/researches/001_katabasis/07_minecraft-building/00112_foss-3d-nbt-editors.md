# FOSS 3D 編輯 Minecraft .nbt 檔案工具調查

## 概述

本報告調查可用於編輯 Minecraft Java Edition `.nbt` 結構檔案的**自由且開源（FOSS）**三維體素編輯器。排除以下類別：
- 純二進位/十六進位編輯器（如 NBTExplorer、NBT Studio）
- 僅轉換而不具備 3D 互動編輯視野的工具
- 閉源或付費軟體

## 調查結果

### 1. Schematic Editor（Faraway24）

| 項目 | 內容 |
|---|---|
| **說明** | 一款基於瀏覽器的全功能 3D Minecraft 結構編輯器，支援 `.litematic`、`.schem`、`.schematic` 以及 **.nbt** 檔案格式。具備互動式 3D 視野、繪畫、擦除、筆刷、鏡像、選取、剪貼簿、連接桿、尺規、推拉等工具，並支援完整的 Blockstate 編輯（如階梯方向、柵欄連接等）。可使用本地 Minecraft 紋理或色彩平均回退渲染。所有操作完全在本地端執行。 |
| **儲存庫** | https://github.com/Faraway24/MinecraftSchematicEditor |
| **授權** | **MIT** |
| **平台** | 瀏覽器（WebGL2），亦可透過 `npm run dev` 在本機執行 |

[^schematic-editor]: Faraway24. (2022). *Minecraft Schematic Editor*. Retrieved 2026-09-22, from https://github.com/Faraway24/MinecraftSchematicEditor

---

### 2. MCEdit 2.0

| 項目 | 內容 |
|---|---|
| **說明** | Minecraft Java Edition 的經典世界編輯器，提供完整 3D 視野。支援 `.schematic`（舊版格式）、`.schem`（Sponge v1/v2/v3）及 `.litematic`。**不支援** `.nbt` 結構方塊格式（因 MCEdit 先於結構方塊格式出現），但仍是舊版結構生態系的強大體素編輯器。目前處於 alpha 階段。 |
| **儲存庫** | https://github.com/mcedit/mcedit2 |
| **授權** | **BSD 3-Clause** |
| **平台** | Windows、macOS、Linux（Python 2.7 / PySide / OpenGL） |

[^mcedit]: MCEdit Team. (2016). *MCEdit 2.0*. Retrieved 2026-09-22, from https://github.com/mcedit/mcedit2

---

### 3. ObjToSchematic 1.0（LucasDower）

| 項目 | 內容 |
|---|---|
| **說明** | 將 3D `.obj` 模型轉換為 Minecraft 結構檔案的視覺化工具。支援匯出 **.nbt**、`.schematic`、`.litematic`、`.schem` 格式，具備體素化結果的互動 3D 預覽。支援色彩對應方塊、抖色、紋理圖集及多種體素化演算法。v2.0 為閉源，v1.0 仍為開源。 |
| **儲存庫** | https://github.com/LucasDower/ObjToSchematic |
| **授權** | **BSD 3-Clause** |
| **平台** | Windows、macOS、Linux（Electron/Node.js，桌面應用）；亦提供瀏覽器版本 |

[^objtoschematic]: LucasDower. (2022). *ObjToSchematic*. Retrieved 2026-09-22, from https://github.com/LucasDower/ObjToSchematic

---

### 4. BedrockMap

| 項目 | 內容 |
|---|---|
| **說明** | 針對 Minecraft Bedrock Edition 的地圖編輯器，以 Qt6 與 C++17 開發，提供**3D 體素預覽**。支援瀏覽與編輯 `.mcstructure`（Bedrock 的結構格式，使用 little-endian NBT）。功能包括 NBT 編輯、區塊編輯、生態域視覺化、GLB 模型匯出。 |
| **儲存庫** | https://github.com/bedrock-dev/BedrockMap |
| **授權** | **AGPL-3.0** |
| **平台** | Windows 10+ 限定 |

[^bedrockmap]: bedrock-dev. (2021). *BedrockMap*. Retrieved 2026-09-22, from https://github.com/bedrock-dev/BedrockMap

---

### 5. WorldPainter

| 項目 | 內容 |
|---|---|
| **說明** | 互動式 Minecraft 地形產生器，以繪畫方式雕塑地形、鋪設方塊、放置樹木等。可匯入/匯出 `.schematic` 作為工作流程的一部分。**主要為地形編輯器**，非專用 NBT 結構方塊編輯器。 |
| **儲存庫** | https://github.com/Captain-Chaos/WorldPainter |
| **授權** | **GPL-3.0** |
| **平台** | Windows、macOS、Linux（Java） |

[^worldpainter]: Captain-Chaos. (2012). *WorldPainter*. Retrieved 2026-09-22, from https://github.com/Captain-Chaos/WorldPainter

---

## 總結比較表

| 工具 | 編輯 .nbt（Java） | 3D 體素視野 | FOSS 授權 | 平台 |
|---|---|---|---|---|
| Schematic Editor | ✅ 讀寫支援 | ✅ 完整 3D 編輯 | MIT | 瀏覽器（跨平台） |
| MCEdit 2.0 | ❌ 僅 .schematic | ✅ 完整 3D 編輯 | BSD 3-Clause | Win/Mac/Linux |
| ObjToSchematic 1.0 | ✅ 僅匯出 | ✅ 3D 預覽 | BSD 3-Clause | Win/Mac/Linux + Web |
| BedrockMap | ✅ .mcstructure | ✅ 3D 體素預覽 | AGPL-3.0 | Windows 限定 |
| WorldPainter | ❌ 地形為主 | ✅ 3D 地形編輯 | GPL-3.0 | Win/Mac/Linux |

## 結論與推薦

如需**在 3D 體素編輯器中編輯 Minecraft Java Edition 的 .nbt 結構檔案**，最佳 FOSS 選項為：

**Schematic Editor**（Faraway24）—— MIT 授權，基於瀏覽器（無需安裝），支援 .nbt 讀寫、完整 3D 編輯工具、Blockstate 感知（如柵欄連接、階梯方向）、真實 Minecraft 紋理渲染，且所有操作完全在本地端執行。

若需將 3D 模型轉換為 .nbt 結構，**ObjToSchematic 1.0**（BSD 3-Clause）為最佳 FOSS 選擇。

---

## References

[^schematic-editor]: Faraway24. (2022). *Minecraft Schematic Editor*. Retrieved 2026-09-22, from https://github.com/Faraway24/MinecraftSchematicEditor

[^mcedit]: MCEdit Team. (2016). *MCEdit 2.0*. Retrieved 2026-09-22, from https://github.com/mcedit/mcedit2

[^objtoschematic]: LucasDower. (2022). *ObjToSchematic*. Retrieved 2026-09-22, from https://github.com/LucasDower/ObjToSchematic

[^bedrockmap]: bedrock-dev. (2021). *BedrockMap*. Retrieved 2026-09-22, from https://github.com/bedrock-dev/BedrockMap

[^worldpainter]: Captain-Chaos. (2012). *WorldPainter*. Retrieved 2026-09-22, from https://github.com/Captain-Chaos/WorldPainter