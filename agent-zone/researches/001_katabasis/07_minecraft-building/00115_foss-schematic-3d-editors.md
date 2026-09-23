# FOSS `.schematic` 三維編輯器調查報告

## 概述

本文調查能編輯 Minecraft `.schematic`（含 `.schem`、`.litematic`）結構檔案的**自由且開源 (FOSS) 三維編輯器**。排除二維編輯器、純粹二進位編輯器，以及僅能檢視無法編輯的工具。優先考慮 Web UI 解決方案。

## 術語說明

- **`.schematic`**：Minecraft 建築/結構儲存格式，最初由 MCEdit 定義，現已衍生出多種格式。`struct`（或稱 `.schem`）是 Sponge 專案推出的新一代 NBT 結構格式。`.litematic` 是 Litematica Mod 使用的格式。[^format]
- **FOSS**：Free and Open Source Software，自由且開源軟體。
- **三維編輯器**：能以 3D 視角瀏覽、放置、刪除或修改方塊的編輯器，非純文字或表格操作。

## Web UI 方案

### 1. Faraway24 MinecraftSchematicEditor — 唯一 FOSS Web 三維編輯器

- **授權**：MIT（開源）
- **倉庫**：<https://github.com/Faraway24/MinecraftSchematicEditor>
- **支援格式**：`.litematic`、`.schem`、`.schematic`、`.nbt`
- **功能**：完全在瀏覽器中運行，支援方塊放置/擦除/筆刷、鏡像、克隆、階梯工具等 3D 編輯操作，無須後端伺服器。

這是目前唯一**同時滿足「Web UI」「三維編輯」「開源」**三個條件的工具。[^fae24]

### 2. MCWebEdit — 簡易型 FOSS Web 編輯器

- **授權**：MIT（開源）
- **倉庫**：<https://github.com/atomdellow/MCWebEdit>
- **支援格式**：`.schem`、`.schematic`
- **功能**：支援基本的 3D 檢視與編輯，功能較精簡，屬於原型階段。[^mcwe]

## 非 FOSS 但功能完整的 Web 方案

這些工具是免費的 Web 三維編輯器，但**非開源**，僅供參考：

| 工具 | 網址 | 支援格式 |
|------|------|----------|
| Cubical.xyz | <https://cubical.xyz/> | `.schem`、`.nbt`、`.bo2` |
| Shulkr Editor | <https://www.shulkr.com/en/minecraft-schematic-editor> | `.litematic`、`.schem`、`.nbt`、`.mcstructure` |
| Structmatic Studio | <https://structmatic.com/studio> | `.schem`、`.litematic`、`.nbt`、`.mcstructure` |
| Bloxelizer Voxelizer | <https://bloxelizer.com/editor/voxelizer> | `.schem`、`.litematic`、`.nbt`、`.mcstructure` |

## Desktop FOSS 方案

### 1. Amulet Map Editor（推薦桌面方案）

- **授權**：開源
- **倉庫**：<https://github.com/Amulet-Team/Amulet-Map-Editor>
- **官方網站**：<https://www.amuletmc.com/>
- **支援格式**：`.schem`、`.schematic`、`.litematic`、`.nbt`、`.mcstructure`、`.mcworld`
- **語言**：Python（PySide/PyOpenGL）
- **平台**：Windows、macOS、Linux
- **功能**：全功能 3D 地圖編輯器，支援格式轉換、方塊編輯、結構匯入匯出。是目前最活躍維護的現代桌面方案。[^amulet]

### 2. MCEdit 2.0（經典工具，Alpha 階段）

- **授權**：BSD（開源）
- **倉庫**：<https://github.com/mcedit/mcedit2>
- **支援格式**：`.schematic`、`.schem`
- **語言**：Python（PySide/PyOpenGL）
- **狀態**：Alpha 階段，開發較不活躍。支援 3D 編輯與結構匯入匯出。[^mcedit]

## 格式轉換工具

若僅需在不同結構格式之間轉換而非編輯，可參考以下 FOSS 工具：

| 工具 | 類型 | 授權 | 說明 |
|------|------|------|------|
| SchemConvert | CLI（Java） | 開源 | <https://github.com/PiTheGuy/SchemConvert> — 支援 `.schem`、`.litematic`、`.nbt`、`.bp`、`.schematic` 互轉[^schemconv] |

## 結論與建議

| 需求情境 | 推薦工具 | 說明 |
|----------|----------|------|
| **Web + FOSS + 三維編輯** | Faraway24 MinecraftSchematicEditor | 唯一滿足全部條件的方案，MIT 授權，瀏覽器內可直接運行 |
| **Web + 功能最完整（非 FOSS 可接受）** | Cubical.xyz | 遊戲級操控、腳本支援、筆刷/生成器工具 |
| **桌面 FOSS + 現代維護** | Amulet Map Editor | 跨平台、活躍開發、格式支援最廣 |
| **純格式轉換** | SchemConvert | CLI 工具，支援格式多 |

## 參考資料

[^format]: Minecraft Wiki. (n.d.). Structure file format. Retrieved 2026-09-22, from https://minecraft.fandom.com/wiki/Structure_file_format
[^fae24]: Faraway24. (n.d.). MinecraftSchematicEditor. Retrieved 2026-09-22, from https://github.com/Faraway24/MinecraftSchematicEditor
[^mcwe]: atomdellow. (n.d.). MCWebEdit. Retrieved 2026-09-22, from https://github.com/atomdellow/MCWebEdit
[^amulet]: Amulet Team. (n.d.). Amulet Map Editor. Retrieved 2026-09-22, from https://github.com/Amulet-Team/Amulet-Map-Editor
[^mcedit]: MCEdit Team. (n.d.). MCEdit 2.0. Retrieved 2026-09-22, from https://github.com/mcedit/mcedit2
[^schemconv]: PiTheGuy. (n.d.). SchemConvert. Retrieved 2026-09-22, from https://github.com/PiTheGuy/SchemConvert
