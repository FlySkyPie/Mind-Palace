# Minecraft NPC 建造房屋類 MOD 的建築儲存檔案格式

## 概述

許多 Minecraft MOD 加入了 NPC（非玩家角色）自動建造房屋的功能。這些 MOD 需要將建築藍圖以某種檔案格式儲存，以便 NPC 讀取並逐塊建造。本研究調查了四個代表性的 MOD，分析其建築儲存檔案格式。

## 主要 MOD 及其檔案格式

### 1. MineColonies（透過 Structurize 函式庫）

**檔案格式：`.nbt`（標準 Minecraft NBT 結構檔案）**

MineColonies 使用 **Structurize** 函式庫儲存建築藍圖[^mc1]。所有建築皆以 Minecraft 原版的 NBT 結構檔案格式（`.nbt`）儲存，命名慣例如下：

```
{StyleName}/{HutName}{HutLevel}.nbt
```

例如：`wooden/Builder1.nbt`、`wooden/Builder5.nbt`、`wooden/TownHall1.nbt`。

玩家使用 **掃描工具（Scan Tool / Scepter of Steel）** 在遊戲中掃描建築，掃描結果會儲存至 `*/structurize/scans/new/` 目錄下為 NBT 檔案[^mc2]。該格式包含所有方塊資料、方塊實體（tile entity）和實體資料。這些檔案可放入 `*/structurize/schematics/` 作為風格包，由建築 NPC 讀取並逐塊建造。

此 `.nbt` 格式即 Minecraft 原版結構方塊（Structure Block）所使用的標準 NBT 結構模板格式，包含方塊陣列、基於調色盤（palette）的方塊狀態資料、以及方塊實體/實體資料。

### 2. Millénaire

**檔案格式：自訂多層 PNG + 純文字設定檔（`.txt`）**

Millénaire 採用完全自訂且極具創意的格式，將**內容（視覺佈局）**與**引擎（建造邏輯）**分離[^ml1]。

**A) 建築計畫設定檔（`.txt`）**：
每個建築有對應的純文字設定檔，如 `armoury_A.txt`、`farm_B.txt` 等。檔案內容為分號分隔的 `key:value` 配對，指定建築的寬度、長度、優先順序、居民類型、商店類型、標籤等屬性。

**B) 多層 PNG 示意圖（`.png`）**：
建築的每個等級對應一個 PNG 檔案，如 `armoury_A0.png`、`armoury_A1.png`（表示升級版本）。此 PNG 以**單張二維影像編碼完整的三維結構**：

- PNG 的高度 = 建築的 Z 軸深度（長度）
- PNG 的寬度 = Y 軸樓層**水平並排**，以 1 畫素間隔分隔
- 每個畫素的 RGB 色彩對應至特定 Minecraft 方塊，透過 `blocklist.txt` 色彩對照表查詢
- 白色畫素 = 空氣（不放置）
- 透明畫素（alpha < 0xFF）= 升級時「不覆蓋」
- 特殊色彩編碼功能性標記：睡眠位置、儲物箱、工作台、農田、動物生成點、道路連接等

**C) `blocklist.txt`**：
以分號分隔的對照檔案，將 RGB 色彩值對應至 Minecraft 方塊 ID/附加值（語法：`name;blockRef;meta;secondStep;R/G/B`）。

**核心設計理念**：內容與引擎分離——美術人員可透過繪製二維影像來建立或修改建築，無需撰寫程式碼。

### 3. Ancient Warfare 2

**檔案格式：`.aws`（自訂 Ancient Warfare 結構檔案，包裝於 `.zip` 壓縮檔中）**

Ancient Warfare 2 使用模板式結構系統，玩家以**結構掃描器（Structure Scanner）**工具掃描建築後儲存為 `.aws` 檔案[^aw1]。

- **副檔名**：`.aws`（Ancient Warfare Structure）
- **散佈方式**：結構包以 `.zip` 壓縮檔散佈，內含多個 `.aws` 檔案的目錄
- **組織方式**：依派系/部落分類（例如 `AWNationElf/ElfNoldorFortress.aws`、`AWTribalBarbarian/...`）
- **內部架構**：根據 Encyclopedia 專案的分析，`.aws` 檔案採用壓縮/二進位格式，其內部結構近似 JSON 風格的資料架構，包含方塊資料、實體資料、驗證設定、生態域/維度白名單及世界生成設定[^aw2]
- **掃描流程**：玩家定義邊界框、設定「建造鍵」（錨點/方向），並透過圖形介面設定驗證選項，最後匯出為模板
- **使用方式**：結構可在創造模式中即時放置（透過 Structure Builder）、在生存模式中由工匠 NPC（Craftsman）經由 Drafting Station 建造，或作為世界生成模板使用

### 4. Recurrent Complex

**檔案格式：`.rcst`（ZIP 壓縮檔，包含 `structure.json` + `worldData.nbt`）**

Recurrent Complex 的結構以 `.rcst` 檔案儲存，本質上是 **ZIP 壓縮檔**[^rc1]，內部包含：

- **`structure.json`**——結構的後設資料（生成設定、標記、腳本、戰利品表、子結構參考）
- **`worldData.nbt`**——實際的方塊與實體資料，以 NBT 格式儲存

該 MOD 亦可匯入/轉換 `.schem`、`.schematic` 及原版 `.nbt` 檔案作為交換來源，但 `.rcst` 為其原生格式。

## 比較總表

| MOD | 檔案格式 | 編碼方式 | 標準或自訂？ |
|---|---|---|---|
| **MineColonies** (Structurize) | `.nbt` | 標準 NBT（Minecraft 結構方塊格式） | **標準** |
| **Millénaire** | `.txt` 設定 + 多層 `.png` | 自訂（RGB 畫素對應方塊，透過色彩對照表） | **自訂** |
| **Ancient Warfare 2** | `.aws`（包裝於 `.zip`） | 自訂二進位/壓縮格式 | **自訂** |
| **Recurrent Complex** | `.rcst`（ZIP 內含 `structure.json` + `worldData.nbt`） | NBT + JSON 於 ZIP 內 | **混合** |

## 結論

綜合來看，Minecraft NPC 建造房屋類 MOD 的建築儲存格式呈現多樣化局面：

1. **標準 NBT 格式（`.nbt`）**最為常見——MineColonies 直接使用 Minecraft 原版的結構方塊格式，相容性最高，也最適合與其他工具交換。
2. **完全自訂格式**如 Millénaire 的 PNG 編碼和 Ancient Warfare 2 的 `.aws`，體現了各 MOD 對內容創作流程的特殊考量（如 Millénaire 讓美術人員可用繪圖軟體編輯建築）。
3. **混合格式**如 Recurrent Complex 的 `.rcst`，結合了 NBT 的結構資料儲存能力和 JSON 的後設資料靈活性，並以 ZIP 壓縮減少檔案大小。

這些格式的選擇反映了各 MOD 在**開發便利性**、**內容創作流程**、**與原版相容性**之間的不同取捨。

---

[^mc1]: MineColonies. (n.d.). *Schematic Tutorials*. Retrieved 2026-09-22, from https://minecolonies.com/wiki/tutorials/schematics/

[^mc2]: Jorch72. (n.d.). *Minecolonies Wiki — Schematics*. Retrieved 2026-09-22, from https://github.com/Jorch72/MinecoloniesWiki/blob/master/source/tutorials/schematics.md

[^mc3]: harleypig. (n.d.). *dump-minecolonies-resources*. Retrieved 2026-09-22, from https://github.com/harleypig/dump-minecolonies-resources

[^ml1]: jason920612. (n.d.). *Millénaire Fabric — Building Schematic Documentation*. Retrieved 2026-09-22, from https://github.com/jason920612/millenaire-fabric/blob/main/docs/intent/03-building-schematic.md

[^aw1]: Maxashen. (n.d.). *Ancient Warfare — Structures Overview*. Retrieved 2026-09-22, from https://maxashen.github.io/AncientWarfare/Structures%20Overview.html

[^aw2]: P3pp3rF1y. (n.d.). *Ancient Warfare 2 Wiki — Structures*. Retrieved 2026-09-22, from https://github-wiki-see.page/m/P3pp3rF1y/AncientWarfare2/wiki/Structures

[^aw3]: TheOddlySeagull. (n.d.). *Ancient Warfare Encyclopedia Website*. Retrieved 2026-09-22, from https://github.com/TheOddlySeagull/ancient-warfare-encyclopedia-website

[^rc1]: Akiak7. (n.d.). *Recurrent Complex Volts — Structure Authoring*. Retrieved 2026-09-22, from https://github.com/Akiak7/RecurrentComplexVolts/blob/main/docs/user/structure-authoring.md

[^rc2]: Minecraft Recurrent Complex Wiki. (n.d.). *Structure File*. Retrieved 2026-09-22, from https://minecraft-recurrent-complex.fandom.com/wiki/Structure_File