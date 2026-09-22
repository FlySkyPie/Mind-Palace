# Minecraft Schematic 檔案格式

## 什麼是 Schematic 檔案？

Minecraft 的 **schematic** 檔案是一種社群建立的檔案格式（非官方 Mojang），用來儲存 Minecraft 世界中的某一區塊結構——本質上就是一份「建築藍圖」——以便在另一個世界中貼入使用。副檔名 `.schematic` 格式最初由社群為 MCEdit、WorldEdit 等第三方工具所建立[^mcwiki-schematic]。

Schematic 檔案的核心特徵：

- 以 **NBT（Named Binary Tag）** 格式儲存
- 大致基於 **Indev 等級格式**
- 使用 **YZX** 方塊順序（X 座標變化最快）
- 無法區分「應覆蓋既有方塊的空氣」與「應忽略的空氣」

## Schematic 的用途

Schematic 檔案主要用於 **儲存與貼上結構**：

- **WorldEdit / FAWE**：使用 `//copy` 複製區域，`//schem save <名稱>` 存檔；`//schem load <名稱>` 後 `//paste` 貼上。檔案存放於 `plugins/WorldEdit/schematics/` 或 `plugins/FastAsyncWorldEdit/schematics/`。
- **MCEdit**：選取區域後匯出為 `.schematic`。
- **Litematica**：用戶端模組，以全息投影顯示建築供手動搭建。使用 `.litematic` 格式，但可匯出為 `.schem`。
- **原版結構方塊（Structure Block）**：使用 `.nbt` 格式（非 schematic），限制 **48×48×48** 方塊。
- **Axiom 模組**：使用 `.bp`（blueprint）格式。

WorldEdit/FAWE 的常見貼上旗標[^worldedit-flags]：

| 旗標 | 效果 |
|------|------|
| `-a` | 跳過空氣方塊（保留既有地形） |
| `-o` | 貼在原座標位置 |
| `-e` | 貼上實體（物品框、盔甲架等） |
| `-b` | 貼上生態域 |
| `-s` | 貼上後選取該區域 |
| `-n` | 僅預覽（不實際貼上） |

## 技術細節：.schematic（舊版 MCEdit 格式）

### NBT 結構

根標籤為名為 `"Schematic"` 的 Compound，包含以下欄位[^mcwiki-schematic][^bloxelizer-schematic]：

| 欄位名稱 | 型態 | 說明 |
|---------|------|------|
| **Width** | `Short` | X 軸大小 |
| **Height** | `Short` | Y 軸大小 |
| **Length** | `Short` | Z 軸大小 |
| **Materials** | `String` | `"Classic"`（Classic 等級）、`"Pocket"`（PE）、`"Alpha"`（Alpha 及更新） |
| **Blocks** | `Byte Array` | 方塊 ID，每方塊 8 位元。YZX 排序 |
| **AddBlocks** | `Byte Array` | 選擇性，方塊 ID > 255 時儲存額外位元（半位元組） |
| **Data** | `Byte Array` | 方塊資料值（僅低 4 位元使用，但每個方塊儲存一整個位元組） |
| **Entities** | `List of Compound` | 實體 |
| **TileEntities** | `List of Compound` | 方塊實體（如箱子、告示牌） |
| **Icon** | `Compound` | Schematica 的圖示物品 |
| **SchematicaMapping** | `Compound` | 將方塊名稱對應至儲存時的數字 ID |
| **WEOriginX/Y/Z** | `Int` | WorldEdit 原點座標 |
| **WEOffsetX/Y/Z** | `Int` | WorldEdit 偏移座標 |

### 方塊索引計算

在座標 (X, Y, Z) 處的方塊於 **Blocks** 與 **Data** 陣列中的索引為：

```
index = (Y × Length + Z) × Width + X
```

此為 **YZX 順序**——X 變化最快。

### 主要限制

Minecraft 1.13「平整化（The Flattening）」之後，方塊 ID 從數字改為文字識別碼。`.schematic` 格式使用數字方塊 ID（搭配選擇性的 AddBlocks 處理 > 255 的值），因此 **無法正確呈現新版方塊**[^donschematics-comparison]。新版方塊在舊系統中沒有數字 ID，可能遺失或被轉換為錯誤方塊。

## Sponge Schematic（.schem）vs 舊版 .schematic

為了解決舊版格式的侷限性，SpongePowered 團隊制定了 **Sponge Schematic 規範**，副檔名刻意選用 `.schem` 以避免與舊版衝突[^sponge-spec][^sponge-v3]。

| 特性 | .schematic（舊版） | .schem（Sponge） |
|------|-------------------|------------------|
| 建立者 | MCEdit 社群 | SpongePowered 團隊 |
| 副檔名 | `.schematic` | `.schem` |
| 時代 | 1.13 之前 | 1.13 之後（現代） |
| 方塊 ID | 數字（位元組陣列） | 文字格式資源定位（如 `minecraft:oak_planks`） |
| 方塊狀態 | 有限（經由 Data 位元組陣列） | 完整方塊狀態支援（如 `minecraft:oak_fence[waterlogged=true]`） |
| 調色盤 | 無（重複完整 ID） | 有——每種方塊狀態僅儲存一次 |
| 檔案大小 | 隨體積成長 | 隨方塊種類成長（調色盤壓縮） |
| 資料版本 | 無 | 有——記錄建立時的 Minecraft 版本 |
| 3D 生態域 | 無 | 有（v3+） |
| 實體 | 有限 | 完整支援 |
| 壓縮 | GZip | GZip |
| 回溯相容 | FAWE/WorldEdit 仍可讀取舊檔 | 無法存為 .schematic（會遺失資料） |

## Sponge Schematic（.schem）Version 3 技術細節

### 根結構

根 Compound 名稱為 `"Schematic"`（巢狀於 NBT 根 Compound 內）[^sponge-v3]：

```
{
    "": {
        "Schematic": {
            "Version": 3,
            "DataVersion": <整數>,
            "Metadata": { ... },
            "Width": <無號短整數>,
            "Height": <無號短整數>,
            "Length": <無號短整數>,
            "Offset": [x, y, z],
            "Blocks": {
                "Palette": { "minecraft:air": 0, ... },
                "Data": <varint[]>
            },
            "Biomes": {
                "Palette": { "minecraft:plains": 0, ... },
                "Data": <varint[]>
            },
            "Entities": [ ... ],
            "BlockEntities": [ ... ]
        }
    }
}
```

### 重要欄位

| 欄位 | 型態 | 說明 |
|------|------|------|
| **Version** | `integer` | 格式版本（目前為 **3**） |
| **DataVersion** | `integer` | Minecraft 資料版本號（如 1.12.2 = 1343） |
| **Metadata** | `Object` | 名稱、作者、日期（epoch 毫秒）、所需模組 |
| **Width/Height/Length** | `unsigned short` | 區域尺寸（X、Y、Z 軸） |
| **Offset** | `integer[3]` | 相對於貼入位置的偏移，預設 `[0, 0, 0]` |
| **Blocks.Palette** | `Object` | 方塊狀態字串 → 調色盤索引 |
| **Blocks.Data** | `varint[]` | `Width × Height × Length` 筆資料，索引方式為 `x + z×Width + y×Width×Length` |
| **Blocks.BlockEntities** | `Object[]` | 指定位置的方塊實體 |
| **Biomes.Palette** | `Object` | 生態域字串 → 調色盤索引 |
| **Biomes.Data** | `varint[]` | 與 Blocks.Data 同尺寸；每項為調色盤索引 |
| **Entities** | `Object[]` | 含 Pos、Id 及選擇性 Data 的實體 |

### 方塊狀態調色盤格式

方塊狀態使用 **資源定位（Resource Location）** 字串，選擇性以方括號附加屬性：

- `minecraft:air` — 無屬性
- `minecraft:oak_planks` — 無屬性
- `minecraft:furnace[facing=north,lit=false]` — 含屬性
- `minecraft:wheat[age=3]` — 整數屬性
- `a_mod:custom_block[power=15]` — 模組方塊

### 索引計算（Sponge .schem）

Data 陣列（方塊與生態域皆同）的索引方式：

```
index = x + z × Width + y × Width × Length
```

此為 **xzy 順序**（Z 次之、Y 再次之）。

### 版本歷史

| 版本 | 日期 | 主要變更 |
|------|------|---------|
| **1** | 2016-08-23 | 初始版本 |
| **2** | 2019-05-08 | 加入 Entities、Biomes、DataVersion；TileEntities 更名為 BlockEntities；移除 ContentVersion |
| **3** | 2021-05-04 | 3D 生態域支援；Palette 更名為 BlockPalette；釐清 varint/palette 用法 |

## 調色盤（Palette）概念

調色盤是 `.schem` 格式的核心創新。它不為每個方塊位置儲存 ID（檔案大小隨體積成長），而是將每種 **獨特的方塊狀態** 以命名條目儲存一次，然後方塊資料陣列僅存指向調色盤的小整數索引[^donschematics-comparison][^structmatic-formats]。

**範例調色盤：**

```
"Palette": {
    "minecraft:air": 0,
    "minecraft:stone": 1,
    "minecraft:oak_planks": 2
}
```

**優點：**
- 檔案大小隨 **方塊種類** 成長，而非方塊數量
- 一個僅含 3 種方塊的大型建築檔案會很小
- 一個含數百種方塊的小型建築檔案則較大
- 方塊狀態資料完全保留
- 跨版本相容（方塊名稱而非任意數字識別）

## 常用工具對照表

| 工具 | 可讀格式 | 可寫格式 | 說明 |
|------|---------|---------|------|
| **WorldEdit** | `.schem`, `.schematic` | `.schem` | 伺服端方塊編輯工具 |
| **FastAsyncWorldEdit (FAWE)** | `.schem`, `.schematic` | `.schem` | WorldEdit 效能分支；非同步操作 |
| **MCEdit** | `.schematic` | `.schematic` | 舊版世界編輯器（1.13 之前） |
| **Litematica** | `.litematic`, `.schem` | `.litematic` | 用戶端 Fabric/Forge 模組；全息投影 + 材料表 |
| **Schematica** | `.schematic` | `.schematic` | 舊版用戶端模組 |
| **原版結構方塊** | `.nbt` | `.nbt` | 原版；硬限制 48×48×48 |
| **Axiom** | `.schem`, `.bp` | `.bp`, `.schem` | 現代建築模組 |
| **Minecraft Note Block Studio** | `.schematic` | `.schematic` | 音樂創作工具 |
| **Bloxelizer** | 所有格式 | 所有格式 | 線上格式轉換器 |
| **Structmatic** | `.schem`, `.litematic` | 所有格式 | 線上建築編輯器 |

## 檔案大小限制

### .schematic（舊版）

- 無硬性格式限制（Width/Height/Length 為 NBT Short，最大值各 32,767）
- 實務上 **Blocks** 位元組陣列為 `Width × Height × Length` 位元組——256×256×256 的 schematic 約有 1,680 萬方塊，僅方塊 ID 就約 16.8 MB
- **AddBlocks** 陣列（選擇性）為 ID > 255 的方塊增加額外半位元組

### .schem（Sponge）

- 尺寸使用 **unsigned short** = 每軸最大值 **65,535**——實務上無上限
- 規範中無硬性大小限制，但**伺服端限制**可設定
- **WorldEdit 設定**可限制最大檔案大小（如 Create Mod 預設 256 KB，可在設定檔調整）
- FAWE 有每玩家 schematic 儲存限制（可在設定中以 KB 和檔案數量調整）
- 調色盤格式使大小取決於方塊種類而非體積[^structmatic-formats]

### .nbt（原版結構方塊）

- **硬限制**：48×48×48 方塊（由 Minecraft 本身強制執行）
- 更大結構必須拆分為多個檔案

## 其他技術細節

### GZip 壓縮

`.schematic` 與 `.schem` 檔案皆為 **GZip 壓縮的 NBT 資料**。NBT 資料先寫入記憶體，再以 GZip 壓縮後寫入磁碟[^mcwiki-schematic]。

### Varint 編碼（Sponge .schem v2+）

Sponge schematic 的方塊資料使用 **varint** 編碼——一種可變長度整數編碼，較小的數字使用較少位元組。第一個位元組決定長度（32 位元整數最多 5 個位元組），後續位元組各貢獻 7 位元資料。此機制在調色盤索引值小時維持資料緊湊[^sponge-v3]。

### BlockEntities vs Entities

- **BlockEntities**（v1 稱為 TileEntities）：帶有額外資料的方塊——箱子（內容物）、告示牌（文字）、熔爐（烹飪狀態）、旗幟（圖案）、生怪磚等。位置以整數表示。
- **Entities**：自由移動物體——物品框、盔甲架、畫作、生物。位置以倍精度浮點數表示。

### UUID 問題

Schematic **不能保證** 提供實體的 UUID。多數實作會在從 schematic 生成實體時產生新的 UUID。

### 回溯相容性

現代的 WorldEdit 與 FAWE 仍可經由相容層**讀取**舊版 `.schematic` 檔案，但**無法以舊格式儲存**（該方向會遺失資料）。從數字 ID 到現代方塊的轉換「並非總是完美」，但大致合理[^donschematics-comparison]。

### 平整化（Minecraft 1.13）

Minecraft 1.13 的「The Flattening」將方塊系統從數字 ID（如 stone = ID 1, data value 0）改為文字資源定位（如 `minecraft:stone`）。這是 `.schematic` 格式過時的主要原因——它建基於不再存在的數字 ID[^donschematics-comparison]。

### 舊版格式的侷限性

- 無法表達「應覆蓋的空氣」與「應跳過的空氣」之差別
- 無資料版本追蹤（工具無從得知哪個 Minecraft 版本產生該檔案）
- 方塊資料僅限 4 位元值（0–15）
- 方塊 ID 若無 AddBlocks 則限於 255
- 原始規格無生態域儲存

## 參考資料

[^mcwiki-schematic]: Minecraft Wiki. (n.d.). *Schematic file format*. Retrieved 2026-09-22, from https://minecraft.wiki/w/Schematic_file_format
[^sponge-spec]: SpongePowered. (n.d.). *Schematic Specification*. Retrieved 2026-09-22, from https://github.com/SpongePowered/Schematic-Specification
[^sponge-v3]: SpongePowered. (2021-05-04). *Schematic Specification Version 3*. Retrieved 2026-09-22, from https://github.com/SpongePowered/Schematic-Specification/blob/master/versions/schematic-3.md
[^bloxelizer-schematic]: Bloxelizer. (n.d.). *.schematic Format Guide*. Retrieved 2026-09-22, from https://bloxelizer.com/formats/schematic/guide
[^bloxelizer-schem]: Bloxelizer. (n.d.). *.schem Format Guide*. Retrieved 2026-09-22, from https://bloxelizer.com/formats/schem/guide
[^donschematics-comparison]: Donut Schematics. (n.d.). *Minecraft Schematic File Formats Explained*. Retrieved 2026-09-22, from https://donutschematics.com/blog/minecraft-schematic-file-formats-explained
[^structmatic-formats]: Structmatic. (n.d.). *Schematic File Formats*. Retrieved 2026-09-22, from https://structmatic.com/guides/schematic-file-formats
[^worldedit-flags]: EngineHub. (n.d.). *WorldEdit Configuration*. Retrieved 2026-09-22, from https://worldedit.enginehub.org/en/latest/config/