# FOSS 資源：Minecraft 格式相容之原創美術資產

> 調查日期：2026-09-12
> 範圍說明：本報告尋找以開放授權（CC0、CC-BY、MIT、Unlicense 等）發布的原創美術資產，支援 Minecraft 的 JSON 格式（blockstates、block models）或 PNG 紋理格式。**嚴格排除**從 Mojang/Microsoft 官方 JAR 提取的資產，以及任何要求使用者下載官方 client.jar 的方案。

---

## 目錄

1. [問題與範圍](#1-問題與範圍)
2. [開放授權原創紋理（Texture PNG）](#2-開放授權原創紋理texture-png)
3. [開放授權原創 Block Model / Blockstate JSON](#3-開放授權原創-block-model--blockstate-json)
4. [標竿開放原始碼 Mod 資產庫](#4-標竿開放原始碼-mod-資產庫)
5. [Faithful 系列：從 Vanilla 重建的資產](#5-faithful-系列從-vanilla-重建的資產)
6. [Luanti（Minetest）生態系：非 JSON 但相容的紋理](#6-luantiminetest-生態系非-json-但相容的紋理)
7. [陣列與搜尋索引方案](#7-陣列與搜尋索引方案)
8. [綜合對照表與建議](#8-綜合對照表與建議)
9. [潛在目標與注意事項](#9-潛在目標與注意事項)

---

## 1. 問題與範圍

Botcraft 所需的 Minecraft 格式資產可分為兩大類：

1. **方塊屬性資料（`custom/Blocks.json`、`Items.json`、`Biomes.json`）** — 結構化資料，不含 Mozilla/Microsoft 版權
2. **美術資產（`minecraft/blockstates/*.json`、`models/block/*.json`、`textures/block/*.png`）** — 此為版權敏感區域

Minecraft EULA 已清楚規範不可重新分發從 Mojang JAR 提取的資產。因此若要以 FOSS 方式發布完整資產集，必須使用社群創作的「原創」美術作品（original art），其視覺風格可能與 Vanilla 相似，但為獨立創作且以開放授權釋出。

本報告聚焦於**不需要下載或提取 Minecraft JAR**、直接以開放授權取得的資產來源。

## 2. 開放授權原創紋理（Texture PNG）

### 2.1 CC0（公眾領域）— 完全免標註

#### Auseawesome/Minecraft-Textures

位址：https://github.com/Auseawesome/Minecraft-Textures [^auseawesome]

CC0 授權的 16×16 Minecraft 風格方塊與物品紋理集。包含 `blocks/` 目錄內有 `bronze_block.png`、`chiseled_gold_block.png`、`tin_block.png` 等。README 明確標示「These textures are all licensed under CC0 meaning you can use them for anything.」

#### Rearth/Oritech

位址：https://github.com/Rearth/Oritech [^oritech]

CC0 授權的科技模組，含原創方塊紋理與模型。

#### Flomik 系列

- https://github.com/Flomik10002/RespiteCreatorsFabric [^respite] — CC0
- https://github.com/Flomik10002/CulturalCreatorsFabric [^cultural] — CC0
- https://github.com/SWUTM/More-Nugget-Fabric [^nugget] — CC0

### 2.2 CC-BY-4.0（須標示來源出處）

#### malcolmriley/unused-textures ⭐527

位址：https://github.com/malcolmriley/unused-textures [^unused]

七年累積的大型原創 16×16 紋理收藏，約數百張方塊紋理（礦石、金屬、磚、木材、籃子、水泥圖案、乳酪等）。包含 `.mcmeta` 動畫紋理定義。授權明確標示 Creative Commons Attribution 4.0 International。

**這是最豐富的獨立紋理來源之一**。

#### Foreck1/foreck-textures ⭐41

位址：https://github.com/Foreck1/foreck-textures [^foreck]

來自「Rebirth of the Night」模組包的紋理與模型集，包含 Blockbench 產生的自訂模型、武器、物品與方塊。也附帶 Optifine CTM 功能。「All of these textures and models are available under a CC BY 4.0 license.」

#### NightML / Open Assets Lib

位址：https://modrinth.com/resourcepack/nightml-open-assets-lib [^oal]

Modrinth 上的資源包函式庫，提供自訂方塊紋理。支援 1.14+。CC-BY-4.0。

### 2.3 CC-BY-SA-4.0（須標示出處、相同方式分享）

#### FreneticScribbler/OpenTextures ⭐24

位址：https://github.com/FreneticScribbler/OpenTextures [^opentex]

免費原創紋理集，組織於 Blocks/ 與 Items/ 目錄中。

#### Futureazoo/TextureRepository ⭐80

位址：https://github.com/Futureazoo/TextureRepository [^texturerepo]

多人貢獻的紋理倉庫，依使用者名稱組織。含原創方塊/物品紋理。

### 2.4 MIT

#### cleannrooster/forg-cleannrooster-assets ⭐29

位址：https://github.com/cleannrooster/forg-cleannrooster-assets [^cleann]

五年累積的黑暗 RPG 風格資產，含 Blockbench 模型、盔甲、武器紋理、法術圖示、生物紋理。

## 3. 開放授權原創 Block Model / Blockstate JSON

純 JSON 格式 Block Model 與 Blockstate 檔案獨立於倉庫的情況罕見，通常與 Minecraft Mod 專案一起發布。下列專案均包含完整的 `models/block/*.json` 與 `blockstates/*.json`，且以開放授權釋出。

### 3.1 Create Mod（MIT）⭐4.5k

位址：https://github.com/Creators-of-Create/Create [^create]

Minecraft 最大規模的原創機械模組。資產路徑為 `src/main/resources/assets/create/models/block/` 與 `blockstates/`。包含數百個原創 Block Model JSON。**最成熟的原創 Block Model / Blockstate 來源之一**。

### 3.2 EnderIO（Unlicense — 公眾領域）

位址：https://github.com/Team-EnderIO/EnderIO [^enderio]

完整模組資產以 Unlicense 釋出，近乎公眾領域。含 Block Model 與 Blockstate JSON。

### 3.3 FarmersDelight（MIT）

位址：https://github.com/vectorwing/FarmersDelight [^farmers]

烹飪相關方塊模型，路徑 `assets/farmersdelight/models/block/`。包含灶台、砧板、櫥櫃等。

### 3.4 The Undergarden（MIT）

位址：https://github.com/quek04/undergarden [^undergarden]

完整的地下維度模組資產，包含自訂生態域方塊的完整模型與 blockstate 集。

### 3.5 TechReborn（MIT）

位址：https://github.com/TechReborn/TechReborn [^techreborn]

工業模組，含大量方塊模型 JSON。

### 3.6 Adorn（MIT）

位址：https://github.com/Juuxel/Adorn [^adorn]

傢俱模組，含裝飾性方塊模型 JSON。

### 3.7 Croptopia（MIT）

位址：https://github.com/ExcessiveAmountsOfZombies/Croptopia [^croptopia]

農作物模組，含作物與食物方塊模型。

### 3.8 Functional Storage（MIT）

位址：https://github.com/Buuz135/FunctionalStorage [^funstorage]

儲存模組，含方塊模型與 blockstate JSON。

## 4. 標竿開放原始碼 Mod 資產庫

下列模組以開放授權釋出，包含大量可直接使用的 Minecraft 格式美術資產。

| 模組名稱 | 授權 | Block Model JSON | Blockstate JSON | Texture PNG | 方塊數量級 |
|---------|------|----------------|---------------|-------------|-----------|
| Create | MIT | ✅ 數百 | ✅ 完整 | ✅ 數百 | 大型 |
| EnderIO | Unlicense | ✅ 大量 | ✅ 完整 | ✅ 大量 | 大型 |
| FarmersDelight | MIT | ✅ 原創 | ✅ 完整 | ✅ 原創 | 中型 |
| The Undergarden | MIT | ✅ 完整維度 | ✅ 完整 | ✅ 完整維度 | 中型 |
| TechReborn | MIT | ✅ 大量 | ✅ 完整 | ✅ 大量 | 大型 |
| Adorn | MIT | ✅ 傢俱系列 | ✅ 完整 | ✅ 系列 | 中型 |
| Croptopia | MIT | ✅ 作物系列 | ✅ 完整 | ✅ 系列 | 中型 |
| Functional Storage | MIT | ✅ 儲存系列 | ✅ 完整 | ✅ 系列 | 中型 |

## 5. Faithful 系列：從 Vanilla 重建的資產

位址：https://github.com/Faithful-Resource-Pack [^faithful]
子組織：https://github.com/ClassicFaithful [^classicfaithful]

Faithful 系列是社群維護的 Vanilla 紋理重建專案，從零開始以更高解析度（32×、64×）重建每一張 Vanilla 紋理。授權為 **Faithful License v4**（自訂但開放），允許使用、修改、重新分發，條件為標示來源與連結至 faithfulpack.net。

**特點**：
- 覆蓋所有 Vanilla 方塊紋理（`textures/block/*.png`）
- 包含完整的 `models/block/*.json` 與 `blockstates/*.json`
- 雖非 CC 授權，但 Faithful License 已充分開放

**注意**：Faithful 雖為「重建」（recreation），視覺設計與 Vanilla 極度相似，嚴格來說設計並非「原創」。然而其程式碼（JSON 定義）為獨立創作，紋理從空白畫布繪製。

## 6. Luanti（Minetest）生態系：非 JSON 但相容的紋理

雖然 Luanti 不使用 Minecraft 的 JSON block model / blockstate 格式，但生態系中存在大量以開放授權釋出的紋理包，其 PNG 可直接用於 Botcraft，僅需將檔名對應至 Botcraft 的紋理命名系統。

### 6.1 CC0 紋理包

| 名稱 | 作者 | 解析度 | 紋理數量 | 位址 |
|------|------|--------|---------|------|
| Dungeon Soup | sirrobzeroone | 32× | ~1400 | [ContentDB](https://content.luanti.org/packages/sirrobzeroone/dungeonsoup/) [^dsoup] |
| Hand Painted Pack | drummyfish | 128× | 大量 | [ContentDB](https://content.luanti.org/packages/drummyfish/drummyfish/) [^hpp] |

### 6.2 CC-BY-SA-4.0 紋理包

| 名稱 | 作者 | 解析度 | 說明 | 位址 |
|------|------|--------|------|------|
| REFI Textures | MysticTempest | 16× | 從草稿重建的 Minecraft 風格紋理，活躍開發中 | [GitHub](https://github.com/MysticTempest/REFI_Textures) [^refi] |
| PixelPerfection | XSSheep / sofar | 16× | MineClone2 預設紋理包，風格精緻一致 | [ContentDB](https://content.luanti.org/packages/sofar/pixelperfection/) [^pp] |
| Baunilha | Mirtilo | 16× | 活躍開發中（2026-07 更新） | [ContentDB](https://content.luanti.org/packages/Mirtilo/baunilha/) [^baunilha] |
| Soothing 32 | Zughy | 32× | 僅 32 色調色板，輕量卡通風格 | [ContentDB](https://content.luanti.org/packages/Zughy/soothing32/) [^soothing] |

## 7. 陣列與搜尋索引方案

若要將上述散落資產組合成一套完整、可由 `AssetsManager` 載入的資產目錄，可參考以下方案。

### 7.1 組合策略

```
graph TD
    A[Botcraft AssetsManager] --> B[assets/minecraft/blockstates/]
    A --> C[assets/minecraft/models/block/]
    A --> D[assets/minecraft/textures/block/]
    A --> E[assets/custom/]

    B --> F[Create Mod (MIT) blockstates]
    B --> G[EnderIO (Unlicense) blockstates]
    C --> H[Create Mod (MIT) models]
    C --> I[EnderIO (Unlicense) models]
    D --> J[unused-textures (CC-BY-4.0)]
    D --> K[Auseawesome (CC0)]
    D --> L[Luanti 紋理包 (CC0/CC-BY-SA)]
    E --> M[PrismarineJS/minecraft-data 自行轉換]
```

### 7.2 過程中注意事項

- **版權彙整**：不同授權（CC0、CC-BY-4.0、CC-BY-SA-4.0、MIT、Unlicense）的資產混用時需遵守各自授權條件，CC-BY 系列須保留標示。
- **風格一致性**：上述資產來自不同創作者，視覺風格可能不一致。需手動過濾或建立統一調色板。
- **路徑對映**：Mod 中的資產路徑（如 `create:block/brass_block`）與 Botcraft 期望的 `minecraft:block/brass_block` 不同。需要路徑重新對映或修改 AssetsManager 的搜尋順序。
- **結構化資料生成**：尚未有直接從以上資產自動產生 `custom/Blocks.json`、`Blocks_info.json` 的工具。可考慮搭配 `minecraft-data` 的結構化 JSON 來輔助生成（但 `minecraft-data` 本身亦為 Mojang 資料的反向工程，存在灰色地帶）。

## 8. 綜合對照表與建議

### Botcraft 資產類別 × 最佳 FOSS 原創來源

| Botcraft 資產類別 | 最佳原創 FOSS 來源 | 授權 | 局限性 |
|------------------|------------------|------|--------|
| `textures/block/*.png` | **malcolmriley/unused-textures** | CC-BY-4.0 | 僅紋理，無模型，數百張而非完整 Vanilla 覆蓋 |
| `textures/block/*.png` | **Auseawesome/Minecraft-Textures** | CC0 | 數量較少，可用於補缺 |
| `textures/block/*.png` | **Dungeon Soup**（Luanti）| CC0 | ~1400 張，但非 Minecraft 命名規則 |
| `models/block/*.json` | **Create Mod** | MIT | 專用於機械方塊，非 Vanilla 通用 |
| `models/block/*.json` | **EnderIO** | Unlicense | 工業風格方塊 |
| `models/block/*.json` + `blockstates/*.json` | **Faithful 32×/64×** | Faithful License | 設計重建自 Vanilla，非 100% 原創 |
| `custom/Blocks.json` + `Items.json` | **PrismarineJS/minecraft-data** | MIT | 不含美術資產，僅結構化資料 |

### 分階段採用建議

**第一階段：最小可行性資產集**
- 紋理 → `malcolmriley/unused-textures`（CC-BY-4.0）+ `Auseawesome/Minecraft-Textures`（CC0）補充
- Block Model → `EnderIO`（Unlicense）或 `Create`（MIT）
- Blockstate → 同 Block Model 來源
- 結構化資料 → 從 `minecraft-data` 轉換

**第二階段：擴充資產集**
- 引入 `Faithful 32×` 的完整紋理與模型
- 從 Luanti 生態系篩選 CC0 紋理補缺

**第三階段：自訂生成**
- 使用 `FabricModelProvider` 或 `Blockbench` 生成自訂的 block model/blockstate JSON
- 建立授權管理標籤系統

## 9. 潛在目標與注意事項

- **無單一 FOSS 資產可完整取代 Vanilla**：目前不存在任何單一 FOSS 專案能提供與 minecraft `client.jar` 同等級覆蓋率的原創資產。必須從多個來源組合。
- **CC-BY-SA 的兼容性**：CC-BY-SA 要求衍生作品以相同授權發布，若 Botcraft 本身使用 MIT 或其他授權，須注意授權鏈兼容性。CC0 與 MIT 則無此限制。
- **信賴來源索引**：PresentKim 整理了一份大型 Treasure Trove 索引（[GitHub Gist](https://gist.github.com/PresentKim/a0a4e73285bcfd55cee250843ba5fce7) [^gist]），列出數十個 MIT/CC0/開放授權的 Minecraft Mod 資產來源，值得定期參考。Charles 在開發 Botcraft 時可能需要從該索引中篩選符合格式的資產。
- **格式精確性**：Mod 中的 Block Model JSON 可能使用 Mod 專屬的 `loader`（如 `forge:obj`、`fabric:composite`），這些非標準格式無法被 Botcraft 的解析器處理。在選用時需確認僅使用標準 Vanilla Model 格式。

---

## 參考資料

[^auseawesome]: Auseawesome. (n.d.). *Minecraft-Textures*. GitHub. Retrieved 2026-09-12, from https://github.com/Auseawesome/Minecraft-Textures

[^oritech]: Rearth. (n.d.). *Oritech*. GitHub. Retrieved 2026-09-12, from https://github.com/Rearth/Oritech

[^respite]: Flomik10002. (n.d.). *RespiteCreatorsFabric*. GitHub. Retrieved 2026-09-12, from https://github.com/Flomik10002/RespiteCreatorsFabric

[^cultural]: Flomik10002. (n.d.). *CulturalCreatorsFabric*. GitHub. Retrieved 2026-09-12, from https://github.com/Flomik10002/CulturalCreatorsFabric

[^nugget]: SWUTM. (n.d.). *More-Nugget-Fabric*. GitHub. Retrieved 2026-09-12, from https://github.com/SWUTM/More-Nugget-Fabric

[^unused]: malcolmriley. (n.d.). *unused-textures*. GitHub. Retrieved 2026-09-12, from https://github.com/malcolmriley/unused-textures

[^foreck]: Foreck1. (n.d.). *foreck-textures*. GitHub. Retrieved 2026-09-12, from https://github.com/Foreck1/foreck-textures

[^oal]: NightML. (n.d.). *NightML Open Assets Lib*. Modrinth. Retrieved 2026-09-12, from https://modrinth.com/resourcepack/nightml-open-assets-lib

[^opentex]: FreneticScribbler. (n.d.). *OpenTextures*. GitHub. Retrieved 2026-09-12, from https://github.com/FreneticScribbler/OpenTextures

[^texturerepo]: Futureazoo. (n.d.). *TextureRepository*. GitHub. Retrieved 2026-09-12, from https://github.com/Futureazoo/TextureRepository

[^cleann]: cleannrooster. (n.d.). *forg-cleannrooster-assets*. GitHub. Retrieved 2026-09-12, from https://github.com/cleannrooster/forg-cleannrooster-assets

[^create]: Creators-of-Create. (n.d.). *Create*. GitHub. Retrieved 2026-09-12, from https://github.com/Creators-of-Create/Create

[^enderio]: Team-EnderIO. (n.d.). *EnderIO*. GitHub. Retrieved 2026-09-12, from https://github.com/Team-EnderIO/EnderIO

[^farmers]: vectorwing. (n.d.). *FarmersDelight*. GitHub. Retrieved 2026-09-12, from https://github.com/vectorwing/FarmersDelight

[^undergarden]: quek04. (n.d.). *undergarden*. GitHub. Retrieved 2026-09-12, from https://github.com/quek04/undergarden

[^techreborn]: TechReborn. (n.d.). *TechReborn*. GitHub. Retrieved 2026-09-12, from https://github.com/TechReborn/TechReborn

[^adorn]: Juuxel. (n.d.). *Adorn*. GitHub. Retrieved 2026-09-12, from https://github.com/Juuxel/Adorn

[^croptopia]: ExcessiveAmountsOfZombies. (n.d.). *Croptopia*. GitHub. Retrieved 2026-09-12, from https://github.com/ExcessiveAmountsOfZombies/Croptopia

[^funstorage]: Buuz135. (n.d.). *FunctionalStorage*. GitHub. Retrieved 2026-09-12, from https://github.com/Buuz135/FunctionalStorage

[^faithful]: Faithful-Resource-Pack. (n.d.). GitHub. Retrieved 2026-09-12, from https://github.com/Faithful-Resource-Pack

[^classicfaithful]: ClassicFaithful. (n.d.). GitHub. Retrieved 2026-09-12, from https://github.com/ClassicFaithful

[^dsoup]: sirrobzeroone. (n.d.). *Dungeon Soup*. Luanti ContentDB. Retrieved 2026-09-12, from https://content.luanti.org/packages/sirrobzeroone/dungeonsoup/

[^hpp]: drummyfish. (n.d.). *Hand Painted Pack*. Luanti ContentDB. Retrieved 2026-09-12, from https://content.luanti.org/packages/drummyfish/drummyfish/

[^refi]: MysticTempest. (n.d.). *REFI Textures*. GitHub. Retrieved 2026-09-12, from https://github.com/MysticTempest/REFI_Textures

[^pp]: sofar. (n.d.). *PixelPerfection*. Luanti ContentDB. Retrieved 2026-09-12, from https://content.luanti.org/packages/sofar/pixelperfection/

[^baunilha]: Mirtilo. (n.d.). *Baunilha*. Luanti ContentDB. Retrieved 2026-09-12, from https://content.luanti.org/packages/Mirtilo/baunilha/

[^soothing]: Zughy. (n.d.). *Soothing 32*. Luanti ContentDB. Retrieved 2026-09-12, from https://content.luanti.org/packages/Zughy/soothing32/

[^gist]: PresentKim. (n.d.). *Treasure Trove of Minecraft Resources*. GitHub Gist. Retrieved 2026-09-12, from https://gist.github.com/PresentKim/a0a4e73285bcfd55cee250843ba5fce7