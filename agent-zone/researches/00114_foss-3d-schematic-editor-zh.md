# FOSS 3D 編輯器：Minecraft `.schematic` 檔案調查

## 概述

本報告調查可用於**三維體素編輯** `.schematic` 檔案的**自由且開源（FOSS）**編輯器。`.schematic` 是 Minecraft 社群建立的建築藍圖格式，以 NBT（Named Binary Tag）格式儲存方塊 ID 與資料值，最初由 MCEdit 與 WorldEdit 等工具定義[^schematic-format]。

**排除標準：**
- 純二進位/十六進位 NBT 編輯器（如 NBTExplorer、NBT Studio、jdNBTExplorer）
- 僅檢視器、不具編輯能力的工具
- 僅命令列、無 3D 視覺介面的工具
- 遊戲內模組（如 WorldEdit、Litematica——這些不是獨立編輯器）
- 閉源或付費軟體

## 調查結果

### 1. MCEdit-Unified

| 項目 | 內容 |
|---|---|
| **說明** | 經典 MCEdit 1.0 的社群維護分支，為最成熟的 FOSS 3D 世界/結構編輯器。完整 OpenGL 3D 視埠、飛行攝影機、方塊層級編輯。原生支援 `.schematic` 讀寫。 |
| **儲存庫** | https://github.com/Podshot/MCEdit-Unified |
| **授權** | **ISC（等同 MIT）** ✅ 完全自由開源 |
| **平台** | Windows、macOS、Linux（Python 2.7 + PyOpenGL + PyGame） |
| **維護狀態** | ⚠️ 低維護——最後提交已距今數年。需 Python 2.7（不支援 Python 3）。 |
| **3D 編輯** | ✅ 完整 3D 視埠（飛行/軌道/縮放）|
| **讀取 .schematic** | ✅ 原生支援 |
| **寫入 .schematic** | ✅ 原生支援 |
| **限制** | 僅支援 Minecraft 1.12 之前的數字方塊 ID，無法正確處理 1.13+ 的方塊狀態（平整化後的新版方塊）|

[^mcedit-unified]: Podshot. (n.d.). *MCEdit-Unified*. Retrieved 2026-09-22, from https://github.com/Podshot/MCEdit-Unified

---

### 2. MCEdit 2.0

| 項目 | 內容 |
|---|---|
| **說明** | MCEdit 系列的最新重構版本，以 Qt（PySide）+ OpenGL 打造現代化 UI。完整 3D 視埠，支援 `.schematic` 匯入/匯出。 |
| **儲存庫** | https://github.com/mcedit/mcedit2 |
| **授權** | **BSD 3-Clause** ✅ 完全自由開源 |
| **平台** | Windows、macOS、Linux（Python 2.7 + PySide + PyOpenGL） |
| **維護狀態** | ❌ Alpha 階段，功能不完整，開發活動極低 |
| **3D 編輯** | ✅ Qt + OpenGL 3D 視埠 |
| **讀取 .schematic** | ✅ 原生支援 |
| **寫入 .schematic** | ✅ 原生支援 |
| **限制** | Alpha 品質，許多功能不穩定或缺失，仍依賴 Python 2.7 |

[^mcedit2]: MCEdit Team. (2016). *MCEdit 2.0*. Retrieved 2026-09-22, from https://github.com/mcedit/mcedit2

---

### 3. MinecraftSchematicEditor（Faraway24）

| 項目 | 內容 |
|---|---|
| **說明** | 最現代且功能最豐富的瀏覽器型 FOSS 3D 結構編輯器。以 Three.js 打造，完全客戶端運作（不上傳任何檔案）。支援多種格式、完整編輯工具組、真實 Minecraft 紋理渲染、方塊狀態感知（如階梯方向、柵欄連接）。 |
| **儲存庫** | https://github.com/Faraway24/MinecraftSchematicEditor |
| **授權** | **MIT** ✅ 完全自由開源 |
| **平台** | 任何瀏覽器（WebGL2），亦可透過 `npm run dev` 在本機執行 |
| **維護狀態** | ✅ 積極維護 |
| **3D 編輯** | ✅ 完整 3D（three.js），軌道控制 + WASD 步行模式 |
| **讀取 .schematic** | ✅ 可讀取舊版 MCEdit 格式（將數字方塊 ID 對應至現代方塊狀態）|
| **寫入 .schematic** | ❌ 僅匯出至現代格式（`.litematic`、`.schem`、`.nbt`），不寫入舊版 `.schematic` |
| **編輯工具** | 放置/擦除/筆刷/繪畫/選取/複製/鏡像/旋轉/階梯生成器/地板工具/推拉工具/對稱模式/復原重做 |
| **額外功能** | 真實 Minecraft 紋理（從本機 `.jar` 或資源包載入）、模組方塊支援、方塊狀態感知渲染、多分頁、自動儲存 |

[^faraway24]: Faraway24. (2022). *Minecraft Schematic Editor*. Retrieved 2026-09-22, from https://github.com/Faraway24/MinecraftSchematicEditor

---

### 4. MCWebEdit（atomdellow）

| 項目 | 內容 |
|---|---|
| **說明** | 以 Three.js + Vue 3 + Socket.io 打造的 3D 結構編輯器，具備即時多人協作功能。支援 `.schem` 與 `.schematic` 的匯入/匯出。 |
| **儲存庫** | https://github.com/atomdellow/MCWebEdit |
| **授權** | **MIT** ✅ 完全自由開源 |
| **平台** | Web（需要 Node.js 後端 + MongoDB） |
| **維護狀態** | ⚠️ 新專案，開發活動有限 |
| **3D 編輯** | ✅ Three.js 3D 編輯器，RTS 風格鏡頭控制 |
| **讀取 .schematic** | ✅ 支援 |
| **寫入 .schematic** | ✅ 支援 |
| **限制** | 需自行架設後端（Node.js + MongoDB），非純前端獨立應用 |

[^mcwebedit]: atomdellow. (2023). *MCWebEdit*. Retrieved 2026-09-22, from https://github.com/atomdellow/MCWebEdit

---

### 5. Build Planner / schematic-editor（actualpugbot）

| 項目 | 內容 |
|---|---|
| **說明** | 輕量級瀏覽器型 3D 結構編輯器/檢視器，支援舊版 `.schematic` 寫出。具備分層檢視、材料清單（含界伏盒購物檢視與合成配方）。 |
| **儲存庫** | https://github.com/actualpugbot/schematic-editor |
| **授權** | **MIT** ✅ 完全自由開源 |
| **平台** | 任何瀏覽器（WebGL2） |
| **維護狀態** | ⚠️ 新專案，編輯工具較陽春 |
| **3D 編輯** | ✅ 基本 3D 視埠（軌道控制、自動旋轉）|
| **讀取 .schematic** | ✅ 支援（舊版 MCEdit 格式）|
| **寫入 .schematic** | ✅ 支援舊版 `.schematic`（難得可寫回舊格式的瀏覽器工具）|
| **編輯工具** | 基本放置/刪除/填充，功能較 Faraway24 編輯器少 |

[^buildplanner]: actualpugbot. (2023). *Schematic Editor*. Retrieved 2026-09-22, from https://github.com/actualpugbot/schematic-editor

---

### 6. MCEdit 1.0（原始版）

| 項目 | 內容 |
|---|---|
| **說明** | 定義 `.schematic` 格式的原始 Minecraft 世界編輯器。完整 3D OpenGL 視埠、方塊編輯、CraftScript 濾鏡系統。 |
| **儲存庫** | https://github.com/mcedit/mcedit |
| **授權** | **BSD 3-Clause** ✅ 完全自由開源 |
| **平台** | Windows、macOS、Linux（Python 2.7 + PyOpenGL + PyGame） |
| **維護狀態** | ❌ 已封存，不再開發 |
| **3D 編輯** | ✅ OpenGL 3D 視埠 |
| **讀取 .schematic** | ✅ 原生支援（格式定義者）|
| **寫入 .schematic** | ✅ 原生支援 |
| **限制** | 極度老舊，僅支援 ~1.12 之前的方塊 ID，需 Python 2.7 |

[^mcedit1]: MCEdit Team. (2011). *MCEdit*. Retrieved 2026-09-22, from https://github.com/mcedit/mcedit

---

### 7. Vengi（voxedit）

| 項目 | 內容 |
|---|---|
| **說明** | 專業級通用體素編輯器，支援 40+ 格式。具備圖層、動畫、腳本化、程序生成、場景圖等進階功能。 |
| **儲存庫** | https://github.com/vengi-voxel/vengi |
| **授權** | **MIT** ✅ 完全自由開源 |
| **平台** | Windows、macOS、Linux（原生桌面）+ WebAssembly 瀏覽器版本 |
| **維護狀態** | ✅ 積極維護 |
| **3D 編輯** | ✅ 專業 3D 體素編輯 |
| **讀取 .schematic** | ✅ 支援 |
| **寫入 .schematic** | ✅ 支援 |
| **限制** | 非 Minecraft 專門工具——不具備 Minecraft 方塊感知（如柵欄連接、階梯方向），無真實 Minecraft 紋理 |

[^vengi]: Vengi Voxel. (2023). *Vengi*. Retrieved 2026-09-22, from https://github.com/vengi-voxel/vengi

---

## 比較總表

| 工具 | 授權 | 平台 | 讀取 .schematic | 寫入 .schematic | 3D 編輯 | 方塊感知 | 積極維護 |
|---|---|---|---|---|---|---|---|
| **MCEdit-Unified** | ISC | Win/Mac/Linux | ✅ | ✅ | ✅ OpenGL | ⚠️ 有限 | ⚠️ 低 |
| **MCEdit 2.0** | BSD-3 | Win/Mac/Linux | ✅ | ✅ | ✅ Qt+OpenGL | ⚠️ 有限 | ❌ Alpha |
| **MCEdit 1.0** | BSD-3 | Win/Mac/Linux | ✅ | ✅ | ✅ OpenGL | ⚠️ 有限 | ❌ 封存 |
| **MinecraftSchematicEditor** | MIT | 瀏覽器 | ✅ | ❌（僅現代格式）| ✅ three.js | ✅ 完整 | ✅ 是 |
| **MCWebEdit** | MIT | Web（+後端）| ✅ | ✅ | ✅ Three.js | ⚠️ 基本 | ⚠️ 低 |
| **Build Planner** | MIT | 瀏覽器 | ✅ | ✅（舊版）| ✅ WebGL | ⚠️ 基本 | ⚠️ 低 |
| **Vengi (voxedit)** | MIT | Win/Mac/Linux/Web | ✅ | ✅ | ✅ 專業 | ❌ 一般體素 | ✅ 是 |

## 結論與推薦

### 若需要完整的 `.schematic` 讀寫 + 3D 編輯

**MCEdit-Unified** 是最成熟的選擇——它出身自定義 `.schematic` 格式的專案，具備完整的 3D 編輯工具組，且可直接讀寫 `.schematic` 檔案。缺點是需 Python 2.7、僅支援 1.12 之前的舊版 Minecraft 方塊。

### 若需要現代、跨平台、免安裝

**MinecraftSchematicEditor（Faraway24）** 是最佳 FOSS 選擇——MIT 授權、瀏覽器運行、支援 `.schematic` 讀取（但僅匯出至現代格式 `.schem`/`.litematic`/`.nbt`）、完整方塊狀態感知、真實 Minecraft 紋理。若不需要向後相容舊版 `.schematic`，這是最強大且活躍的選擇。

### 若必須寫回舊版 `.schematic` 格式

**Build Planner（actualpugbot）** 或 **MCWebEdit** 可在瀏覽器中寫回 `.schematic`，但功能較少。或使用 **MCEdit-Unified** 桌面應用。

### 若需要專業級體素編輯（不限 Minecraft）

**Vengi (voxedit)** 支援 40+ 格式、動畫、腳本、程序生成，為 MIT 授權的跨平台專業工具。但無 Minecraft 方塊感知渲染。

## 參考資料

[^schematic-format]: Minecraft Wiki. (n.d.). *Schematic file format*. Retrieved 2026-09-22, from https://minecraft.fandom.com/wiki/Schematic_file_format

[^mcedit-unified]: Podshot. (n.d.). *MCEdit-Unified*. Retrieved 2026-09-22, from https://github.com/Podshot/MCEdit-Unified

[^mcedit2]: MCEdit Team. (2016). *MCEdit 2.0*. Retrieved 2026-09-22, from https://github.com/mcedit/mcedit2

[^faraway24]: Faraway24. (2022). *Minecraft Schematic Editor*. Retrieved 2026-09-22, from https://github.com/Faraway24/MinecraftSchematicEditor

[^mcwebedit]: atomdellow. (2023). *MCWebEdit*. Retrieved 2026-09-22, from https://github.com/atomdellow/MCWebEdit

[^buildplanner]: actualpugbot. (2023). *Schematic Editor*. Retrieved 2026-09-22, from https://github.com/actualpugbot/schematic-editor

[^mcedit1]: MCEdit Team. (2011). *MCEdit*. Retrieved 2026-09-22, from https://github.com/mcedit/mcedit

[^vengi]: Vengi Voxel. (2023). *Vengi*. Retrieved 2026-09-22, from https://github.com/vengi-voxel/vengi