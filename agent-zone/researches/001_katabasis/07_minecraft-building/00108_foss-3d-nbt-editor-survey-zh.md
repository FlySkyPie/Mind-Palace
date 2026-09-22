# FOSS Minecraft 3D NBT 結構編輯器調查：聚焦 Web UI 方案

## 調查背景

不同於純粹的二進位 NBT 標籤編輯器，本篇調查鎖定在**能在 3D 視圖中擺放方塊、編輯 Minecraft 結構，並匯出為 `.nbt` 格式**的自由開源編輯器，優先考慮 Web UI（瀏覽器）方案。

## 什麼是 3D NBT 編輯器

此類工具類似遊戲內的結構方塊（Structure Block）或 WorldEdit 的視覺化版本——使用者在 3D 場景中以滑鼠點擊擺放方塊，編輯完成後儲存為 Minecraft 結構檔案（`.nbt`），可在遊戲中透過結構方塊或 `/place` 指令讀取。

## Web UI 方案（FOSS）

### 1. MCWebEdit（atomdellow）— 唯一 FOSS Web 3D 編輯器

- **原始碼**：https://github.com/atomdellow/MCWebEdit
- **授權**：MIT[^mcwebedit]
- **技術**：Node.js + Vue 3 + Three.js + Socket.io + MongoDB
- **3D 編輯**：✅ 完整 3D 立體方塊編輯，即時多人協作
- **FOSS**：✅ MIT 授權
- **可自行代管**：✅ 需 Node.js 伺服器 + MongoDB
- **.nbt 匯出**：❌ 僅匯出 `.schem` / `.schematic`（WorldEdit 格式）
- **.nbt 匯入**：✅ 使用 prismarine-nbt 解析 `.schem` / `.schematic`
- **活躍度**：2 次提交、0 星——極初期專案
- **潛力**：底層已使用 prismarine-nbt 處理 NBT，加入 `.nbt` 匯出支援並不困難

### 2. Blockbench（JannisX11）— Web 版存在但不適用

- **網址**：https://web.blockbench.net/
- **原始碼**：https://github.com/JannisX11/blockbench
- **授權**：GPL v3（6k+ 星）[^blockbench]
- **3D 編輯**：⚠️ 編輯的是**物品/實體/方塊模型**（3D mesh），不是世界中的方塊結構
- **.nbt 匯出**：❌ 無結構 `.nbt` 匯出
- **結論**：雖然有 Web 版且開源，但用途完全不同——它是模型編輯器，不是結構編輯器

## Web UI 方案（非開源，僅供對照）

以下工具功能最完整，但原始碼未公開：

| 工具 | 網址 | 3D 編輯 | .nbt 匯出 | 授權 |
|------|------|:-------:|:---------:|:----:|
| cubical.xyz | https://cubical.xyz | ✅ 20+ 工具、WASD 控制 | ✅ .nbt / .schem / BO2 | 免費封閉[^cubical] |
| Shulkr Editor | https://www.shulkr.com/en/editor | ✅ 方塊放置/選取/填滿 | ✅ .nbt / .schem / .litematic | 免費封閉[^shulkr] |
| Structmatic Studio | https://structmatic.com/studio | ✅ 3D 編輯 + AI 生成 | ✅ .nbt / .schem / .litematic | AI 生成付費[^structmatic] |
| MCBE Essentials Structure Editor | https://mcbe-essentials.github.io/structure-editor | ✅ 方塊繪製 + 實體編輯 | ✅ .nbt（Java版） | 免費封閉[^mcbe-essentials] |

## 桌面端 FOSS 方案

### 1. Amulet Editor ⭐ 首選（桌面）

- **網址**：https://www.amuletmc.com
- **原始碼**：https://github.com/Amulet-Team/Amulet-Map-Editor
- **授權**：開源（2.2k+ 星，2,103 次提交）[^amulet]
- **語言**：Python
- **平台**：Windows / macOS / Linux
- **3D 編輯**：✅ 完整世界編輯器，支援 3D 視圖
- **.nbt 匯出/匯入**：✅ 原生支援 NBT 結構處理
- **版本支援**：Java 1.12+ 至最新、Bedrock 1.7+
- **外掛系統**：✅ 有
- **活躍度**：✅ 持續維護中（v0.10）

### 2. MCEdit 2.0（經典，已停滯）

- **原始碼**：https://github.com/mcedit/mcedit2
- **授權**：BSD（開源）[^mcedit]
- **3D 編輯**：✅ 經典世界編輯器
- **.nbt 匯出**：✅ 支援 `.schematic`（NBT 格式）
- **活躍度**：❌ 最後測試版 2016，僅支援 Java 1.11 以前
- **結論**：已過時，Amulet 為其精神繼承者

### 3. Blockwright（檢視器 + AI 生成）

- **原始碼**：https://github.com/matheussartori/blockwright
- **授權**：開源（Electron + React + Three.js）[^blockwright]
- **3D 編輯**：❌ 檢視器與 AI 生成器，非方塊編輯
- **.nbt 匯出/匯入**：✅ 可開啟與輸出 `.nbt`

### 4. Mineways（3D 匯出工具）

- **原始碼**：https://github.com/erich666/Mineways
- **授權**：開源（C++，530+ 星）[^mineways]
- **3D 編輯**：❌ 僅匯出世界為 3D 模型供渲染/列印，非編輯器
- **.schematic 匯出**：✅ 可匯出 OBJ/STL/Schematic

## 情境對照表

| 工具 | 類型 | FOSS | 3D 方塊編輯 | .nbt 匯出 | 可自行代管 | 維護狀態 |
|------|:----:|:----:|:----------:|:---------:|:---------:|:--------:|
| **MCWebEdit** | Web | ✅ MIT | ✅ | ❌ .schem | ✅ Node.js | 初期 |
| Blockbench | Web | ✅ GPL3 | ❌ 模型編輯 | ❌ | ✅ | ✅ 活躍 |
| **Amulet Editor** | 桌面 | ✅ | ✅ | ✅ | N/A | ✅ 活躍 |
| MCEdit 2.0 | 桌面 | ✅ BSD | ✅ | ✅ | N/A | ❌ 停滯 |
| cubical.xyz | Web | ❌ | ✅ | ✅ | ❌ | ✅ 活躍 |
| Shulkr | Web | ❌ | ✅ | ✅ | ❌ | ✅ 活躍 |
| Structmatic | Web | ❌ | ✅ | ✅ | ❌ | ✅ 活躍 |
| MCBE Essentials | Web | ❌ | ✅ | ✅ | ❌ | ✅ 活躍 |

## 綜合建議

### 若堅持 Web UI + FOSS

目前**不存在成熟可用的 FOSS Web 3D 結構編輯器**。最接近的是 **MCWebEdit**（MIT 授權，Three.js 3D 編輯），但它：

- 極初期專案（2 次提交）
- 僅匯出 `.schem` 非 `.nbt`
- 需 Node.js + MongoDB 自行代管

若要自行開發，MCWebEdit 或 Blockbench 的程式碼庫可作為基礎，搭配 prismarine-nbt 處理 NBT 格式。

### 最佳實用解（放寬開源要求）

**cubical.xyz** 是最強大的瀏覽器 3D 結構編輯器——20+ 編輯工具、WASD 操縱、支援 `.nbt` 匯入匯出、完全免費。缺點是封閉原始碼。

### 最佳桌面 FOSS 解

**Amulet Editor** 是目前功能最完整、持續維護的桌面開源 Minecraft 世界/結構編輯器，原生支援 NBT，支援所有現代 Minecraft 版本。

---

[^mcwebedit]: atomdellow. (n.d.). MCWebEdit - Minecraft Web Editor. Retrieved 2026-09-22, from https://github.com/atomdellow/MCWebEdit
[^blockbench]: JannisX11. (n.d.). Blockbench - A low-poly 3D model editor. Retrieved 2026-09-22, from https://github.com/JannisX11/blockbench
[^cubical]: cubical.xyz. (n.d.). cubical - Online Minecraft schematic editor. Retrieved 2026-09-22, from https://cubical.xyz/
[^shulkr]: Shulkr. (n.d.). Minecraft Schematic Editor. Retrieved 2026-09-22, from https://www.shulkr.com/en/minecraft-schematic-editor
[^structmatic]: Structmatic. (n.d.). Structmatic Studio. Retrieved 2026-09-22, from https://structmatic.com/
[^mcbe-essentials]: MCBE Essentials. (n.d.). Structure Editor. Retrieved 2026-09-22, from https://mcbe-essentials.github.io/structure-editor/
[^amulet]: Amulet Team. (n.d.). Amulet Map Editor. Retrieved 2026-09-22, from https://github.com/Amulet-Team/Amulet-Map-Editor
[^mcedit]: MCEdit Team. (n.d.). MCEdit 2.0. Retrieved 2026-09-22, from https://github.com/mcedit/mcedit2
[^blockwright]: matheussartori. (n.d.). Blockwright - 3D NBT structure viewer and AI generator. Retrieved 2026-09-22, from https://github.com/matheussartori/blockwright
[^mineways]: erich666. (n.d.). Mineways. Retrieved 2026-09-22, from https://github.com/erich666/Mineways