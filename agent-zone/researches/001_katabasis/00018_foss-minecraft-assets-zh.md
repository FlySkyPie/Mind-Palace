# FOSS 資源：符合「Minecraft 相容資產」需求的開源解決方案調查

> 調查日期：2026-09-12
> 目標用途：Botcraft 專案中取代 Minecraft 官方 client.jar 的資產來源，涵蓋 blocks、blockstates、block models、textures、items、biomes 等 JSON/PNG 格式。

---

## 目錄

1. [背景與問題](#1-背景與問題)
2. [原始資料集倉庫](#2-原始資料集倉庫)
3. [JAR 提取工具](#3-jar-提取工具)
4. [程式化資產解析與生成](#4-程式化資產解析與生成)
5. [線上資產瀏覽器](#5-線上資產瀏覽器)
6. [Mod 框架的資料生成 API](#6-mod-框架的資料生成-api)
7. [語言包裝與查詢層](#7-語言包裝與查詢層)
8. [資產編輯與驗證工具](#8-資產編輯與驗證工具)
9. [綜合對照表](#9-綜合對照表)

---

## 1. 背景與問題

Minecraft 相容資產（assets）意指能夠被 Minecraft 用戶端原生讀取的資源格式，或能夠被第三方 Bot 程式（如 Botcraft）理解的結構化資料。這些資產包括：

- **Block JSON**（`blockstates/<name>.json` — 方塊狀態變體）
- **Block Model JSON**（`models/block/<name>.json` — 方塊幾何模型）
- **Item JSON**（選擇性，Botcraft 不使用物品模型）
- **Texture PNG**（`textures/block/<name>.png` — 方塊紋理）
- **自訂定義 JSON**（`Blocks.json`、`Items.json`、`Biomes.json` 等方塊屬性資料）

商業條款上，Mojang 的 EULA（最終用戶許可協議）禁止直接重新分發從 `client.jar` 提取的資產，因此開源專案必須透過（a）自動化工具，讓使用者自行從官方 JAR 提取；（b）以程式化生成的 JSON 資料取代官方程式的資產。本文調查的資源均遵循 MIT、LGPL、GPL v3 等開放授權。

## 2. 原始資料集倉庫

### 2.1 PrismarineJS/minecraft-data（MIT）

位址：https://github.com/PrismarineJS/minecraft-data [^mcdata]

最成熟的語言中立 Minecraft 資料集，每版本一份 JSON 目錄結構。支援 Minecraft PC 版 0.30c ~ 1.21.10 / 26.1，完全涵蓋 Botcraft 目標版本範圍（1.12.2 ~ 1.21.11）。提供的資料類型包含：

| 資料類型 | 說明 |
|---------|------|
| Blocks | 方塊 ID、名稱、metadata、硬度、掉落物、工具倍率 |
| Items | 物品 ID、名稱、stack size |
| Biomes | 生態域 ID、名稱、降雨量、溫度 |
| Recipes | 合成配方 |
| Block collision shapes | 方塊碰撞箱形狀 |
| Materials | 不同材質的工具速度倍率 |
| Enchantments | 附魔效果 |
| Entities | 實體定義 |
| Foods | 食物列表（飽食度、食物點數） |
| Protocol | 協定描述 |

所有資料以結構化 JSON 檔案存放在 `data/<version>/` 目錄，可直接被標準 JSON 剖析器讀取。Botcraft 的自訂定義檔（`Blocks.json`、`Items.json`、`Biomes.json`）可直接由此資料集轉換產生。

此外，專案內含 JSON Schema 定義用於驗證資料正確性。

### 2.2 PrismarineJS/minecraft-assets（MIT）

位址：https://github.com/PrismarineJS/minecraft-assets [^mcassets]

提供與 `minecraft-data` 互補的紋理資產資料。直接儲存了實際的 `textures/block/*.png` 檔案，以及路徑映射 JSON 如 `pre_flattening_texturepack_mappings.json` 和 `legacy_texturepack_mappings.json`，用於將舊版物品名稱解析為新版紋理路徑。

支援版本：1.8.8、1.9、1.10、1.11.2、1.12、1.13、1.13.2、1.14.4、1.15.2、1.16.1、1.16.4、1.17.1、1.18.1、1.19.1、1.20.2、1.21.1、1.21.4、1.21.5、1.21.6、1.21.7、1.21.8。

由 `minecraft-jar-extractor` 的 `image_names.js` 自動生成，支援透過 GitHub Actions 自動更新。

### 2.3 InventivetalentDev/minecraft-assets（未明確授權）

位址：https://github.com/InventivetalentDev/minecraft-assets [^invassets]

另一份按版本組織的原始資產提取倉庫，包含 block models、textures、sounds 等。此倉庫同時也作為 **mcasset.cloud** 的資料來源，但不清楚授權條款，使用前須注意 EULA 合規性。

## 3. JAR 提取工具

這類工具允許使用者自行從官方 `client.jar` 或 `server.jar` 提取資產，繞過重新分發的版權問題。

### 3.1 mcextract-py（MIT）

位址：https://github.com/legopitstop/mcextract-py，PyPI: https://pypi.org/project/mcextract/ [^mcextract]

Python CLI 與函式庫，可從 Minecraft JAR 提取 `assets/` 和 `data/` 目錄。主要功能：

- 提取 `assets/` 或 `data/` 資料夾
- 透過 asset index 映射物件，取得聲音、語言等隱藏資產
- 執行內建 Minecraft data generator，產生 reports、registries、vanilla world generation 檔案

安裝方式：`pip3 install mcextract`

### 3.2 Burger（MIT）

位址：https://github.com/TkTech/Burger [^burger]

Python 框架，以「topping」模組化方式從 Minecraft JAR 提取資料。可運行特定提取器（語言、stats、blocks、items 等）或一次全部執行，輸出結果為單一 JSON dictionary。

使用範例：`python munch.py -D --output output.json`

已被 `PrismarineJS/minecraft-data` 的早期版本用作資料來源。

### 3.3 PrismarineJS/minecraft-jar-extractor（MIT）

位址：https://github.com/PrismarineJS/minecraft-jar-extractor [^mjextractor]

Node.js 提取工具，直接從 Minecraft server JAR 提取結構化資料。能力包含：

- **image_names.js** — 產生紋理名稱→路徑映射（即 `minecraft-assets` 的來源）
- **lang.js** — 提取 `en_us.lang` 與 `en_us.json`
- **protocol_extractor.js** — 提取協定封包 ID 映射
- **extract_lootTables.js** — 提取戰利品表（1.14+）
- **extract_datafolder.js** — 提取 `data/` 目錄

### 3.4 McAssetExtractor / McAssetExtractor（Java / Python GUI，MIT）

- **Java 版**：https://github.com/rmheuer/McAssetExtractor [^javama] — 從 Mojang API 下載 assets，以 GSON 解析 JSON。
- **Python GUI 版**：https://github.com/umittadelen/MCAssetExtractor [^pyma] — Windows GUI 工具，讀取 `~/.minecraft/assets/objects` 目錄。

## 4. 程式化資產解析與生成

### 4.1 zardoy/mc-assets（MIT）

位址：https://github.com/zardoy/mc-assets，npm: `mc-assets` [^zardoy]

被描述為「NEXT-GEN Minecraft Assets Library」的 TypeScript 套件，提供最完整的資產解析能力。特點：

- **自動更新** — 新版本自動發布至 npm
- **全面類型支援** — TypeScript 類型定義
- **版本精確** — 涵蓋 1.7.10 起所有版本（models 從 1.13 flattening 後）
- **方塊實體模型** — 社群維護的方塊實體模型
- **新版物品模型格式** — 支援 condition-based 物品模型

核心 API：

```typescript
import { AssetsParser, getLoadedModelsStore, getLoadedBlockstatesStore } from 'mc-assets'
const blockstatesStore = getLoadedBlockstatesStore(blockstatesModels)
const modelsStore = getLoadedModelsStore(blockstatesModels)

// 解析指定方塊的所有變體模型
const resolvedModel = assetsParser.getAllResolvedModels(
  { name: 'stone', properties: {} }, false
)
// 返回已解析 parent chain、紋理、elements 的完整模型
```

可同時在 Node.js 與瀏覽器環境執行。

### 4.2 Fabric API — FabricModelProvider（Apache 2.0）

文件：https://docs.fabricmc.net/develop/data-generation/block-models [^fabric]

Java 資料生成 API，以程式碼生成 `blockstates/*.json`、`models/block/*.json`、`models/item/*.json`。主要方法：

- `blockStateModelGenerator.createTrivialCube(block)` — 簡單立方體全方向同紋理
- `blockStateModelGenerator.registerSingleton(block, TexturedModel.COLUMN_ALT)` — 定義頂底與側面不同紋理
- `blockStateModelGenerator.family(stairs/slabs/fences)` — 階梯/半磚/柵欄系列
- `blockStateModelGenerator.createDoor()` / `createTrapdoor()` — 門/活板門

輸出格式與 Minecraft 官方資源包 100% 相容。適合在不需要實際 client.jar 的情況下，完整生成 Botcraft 所需的 blockstate 與 block model JSON。

### 4.3 Minecraft Forge — BlockStateProvider（LGPL）

文件：https://docs.minecraftforge.net/en/latest/datagen/client/modelproviders/ [^forge]

Java 資料生成 API，提供 `BlockStateProvider` 一次產生 blockstate、block model（經由 `BlockModelProvider`）與 item model（經由 `ItemModelProvider`）。支援：

- Variant-based 與 multipart blockstate
- 自訂 model loader（OBJ、composite、fluid containers）
- `ModelBuilder` 逐 element 建構模型（face、UV、rotation、cullface、tint index）

### 4.4 NeoForge — Model Datagen（LGPL）

文件：https://docs.neoforged.net/docs/1.20.4/resources/client/models/datagen/ [^neoforge]

與 Forge 類似的資料生成 API，提供 `BlockStateProvider` 與 `registerStatesAndModels()`。

### 4.5 Blockbench（GPL v3）

位址：https://blockbench.net，GitHub: https://github.com/JannisX11/blockbench [^blockbench]

開源 3D 模型編輯器，可匯出為 Minecraft Java 版 block model JSON 格式。支援所有 Minecraft 模型功能（elements、faces、UV mapping、display transforms、cullface、tint index）。內建 JavaScript 外掛 API 可進行腳本化模型生成。

## 5. 線上資產瀏覽器

### 5.1 mcasset.cloud

位址：https://mcasset.cloud/latest [^mccloud]

Web 資產瀏覽器，可按版本瀏覽並取得原始 block models、textures、sounds、fonts、shaders。URL 可直接在程式碼中使用，例如：

```
GET https://mcasset.cloud/1.21/assets/minecraft/models/block/stone.json
```

每個版本的 `blockstates/`、`models/block/`、`textures/block/` 均可經由 HTTP GET 直接取得。與 Botcraft 所需的 `minecraft/` 目錄結構完全一致。

### 5.2 Misode's Blockstate Generator（MIT）

位址：https://misode.github.io/assets/blockstate/
原始碼：https://github.com/misode/misode.github.io [^misode]

Web GUI 工具，用於生成 Minecraft blockstate JSON。支援 variant 與 multipart 兩種模式。生成的 JSON 可直接用於資源包。

## 6. Mod 框架的資料生成 API

以下資料生成 API 是 Java 生態系中最成熟的資產生成方案。不直接提供資產資料，但提供程式化建構 `blockstates/*.json`、`models/block/*.json`、`models/item/*.json` 的能力。

| 框架 | 授權 | 核心類別 | 特點 |
|------|------|---------|------|
| Fabric API | Apache 2.0 | `FabricModelProvider`, `BlockStateModelGenerator` | 多功能內建方法、自動 parent 模型、texture mapping |
| Forge | LGPL | `BlockStateProvider`, `ModelBuilder` | Variant/multipart、自訂 loader、逐元素建構 |
| NeoForge | LGPL | `BlockStateProvider` | 與 Forge 類似，持續維護 |

## 7. 語言包裝與查詢層

以下包裝套件將 `minecraft-data` 的 JSON 對映為特定語言的類型化 API：

| 名稱 | 語言 | npm/PyPI/crates.io | 說明 |
|------|------|-------------------|------|
| node-minecraft-data | Node.js | `minecraft-data` | 最完整的查詢 API[^nmd] |
| python-minecraft-data | Python | `minecraft_data` | 完整資料查詢[^pmd] |
| minebase | Python | `minebase` | 現代 Python 包裝，支援 PC/Bedrock[^mb] |
| minecraft-data-rs | Rust | `minecraft-data-rs` | 編譯期嵌入，強型別 API[^rs] |
| mcdata | Go | GitHub | 程式碼生成 Go structs[^go] |

## 8. 資產編輯與驗證工具

### 8.1 minecraft-schemas（MIT）

PyPI: https://pypi.org/project/minecraft-schemas/ [^schemas]
原始碼：https://codeberg.org/IzanagiTokiyuki/minecraft-schemas

Python 套件，可結構化剖析 blockstate JSON 與 model JSON 等 Minecraft 專用格式。

### 8.2 Levertion/minecraft-json-schemas（CC BY 4.0）

位址：https://github.com/Levertion/minecraft-json-schemas [^lvjs]

JSON Schema 定義集，涵蓋 block model、advancements、loot tables 等格式。用於驗證生成的 JSON 是否符合預期格式。更新至 1.14。

### 8.3 SpyglassMC/vanilla-mcdoc（MIT）

位址：https://github.com/SpyglassMC/vanilla-mcdoc [^mcdoc]

正式 `.mcdoc` schema 檔案，描述所有 Minecraft JSON/NBT 格式的資料結構，包括 blockstate 與 model。被 Misode 的生成器用作型別驗證。

## 9. 綜合對照表

### Botcraft 所需 vs. FOSS 方案

| Botcraft 資源 | 最推薦 FOSS 來源 | 備註 |
|--------------|----------------|------|
| `custom/Blocks.json` | PrismarineJS/minecraft-data `data/<version>/blocks.json` | 直接對應方塊 ID/name/metadata |
| `custom/Blocks_info.json` | PrismarineJS/minecraft-data `blocks.json` + `blockCollisionShapes.json` | 需自行轉換 hardness、colliders、tools |
| `custom/Items.json` | PrismarineJS/minecraft-data `data/<version>/items.json` | 直接對應 ID/name/stack_size |
| `custom/Biomes.json` | PrismarineJS/minecraft-data `data/<version>/biomes.json` | 直接對應 |
| `minecraft/blockstates/*.json` | mcasset.cloud 直接下載 或 mcextract 從 client.jar 提取 | 官方格式完全一致 |
| `minecraft/models/block/*.json` | mcasset.cloud 直接下載 或 mcextract 提取 | 同上 |
| `minecraft/textures/block/*.png` | PrismarineJS/minecraft-assets (依版本) 或 mcasset.cloud | 僅在 BOTCRAFT_USE_OPENGL_GUI=ON 時需要 |
| 自訂 blockstate/model 生成 | Fabric API FabricModelProvider / Forge BlockStateProvider | Java 生態最成熟 |
| 資產自動更新管線 | minecraft-jar-extractor + GitHub Actions | 可用於維護最新版本對應 |

### 按授權分類之推薦方案

```
graph TD
    A[需求：Minecraft 相容資產] --> B{資料類型}
    B --> C[結構化資料 blocks/items/biomes]
    B --> D[JSON 資產 blockstate/model]
    B --> E[紋理 PNG]
    B --> F[自訂生成]

    C --> G[PrismarineJS/minecraft-data (MIT)]
    D --> H[mcasset.cloud (HTTP 直接存取)]
    D --> I[mcextract-py (MIT, 自行提取)]
    E --> J[PrismarineJS/minecraft-assets (MIT)]
    E --> K[mcasset.cloud]
    F --> L[Fabric API datagen (Apache 2.0)]
    F --> M[Forge BlockStateProvider (LGPL)]
    F --> N[zardoy/mc-assets (MIT, 解析)]
    F --> O[Blockbench (GPL v3, 編輯)]
```

### 考量要點

1. **資料時效性**：PrismarineJS 生態支援版本最廣且持續維護。`minecraft-assets` 最後更新至 1.21.8，而 Botcraft 需要到 1.21.11 / 26.2。如需最新版本的紋理與 blockstate，須搭配 `mcextract` 或 `minecraft-jar-extractor` 自行從 client.jar 提取。

2. **版權合規**：PrismarineJS/minecraft-data 僅含結構化資料（無實際紋理或模型 JSON），無 EULA 風險。`minecraft-assets` 含實際紋理 PNG，是否重新分發須遵守 Mojang EULA 的個人使用規定。提取工具（mcextract、Burger、minecraft-jar-extractor）由使用者自行從官方 JAR 提取，無分發問題。

3. **與 Botcraft AssetsManager 的整合**：
   - `minecraft-data` 的 `blocks.json` 直接對應 `custom/Blocks.json`
   - 需自行撰寫轉換工具，將 minecraft-data 的欄位映射至 Botcraft 的 `Blocks_info.json`
   - `minecraft-assets` 的紋理可直接放入 `minecraft/textures/block/`
   - Blockstate / model JSON 無需轉換，可直接使用

4. **格式化生成**：若需要不依賴 Mojang JAR 而建構自訂方塊，Fabric API 的 `FabricModelProvider` 是最成熟的方案，生成的 JSON 可直接被 Botcraft 的惰性載入機制讀取。

---

## 參考資料

[^mcdata]: PrismarineJS. (n.d.). *minecraft-data: Language independent module providing minecraft data for minecraft clients, servers and libraries*. GitHub. Retrieved 2026-09-12, from https://github.com/PrismarineJS/minecraft-data

[^mcassets]: PrismarineJS. (n.d.). *minecraft-assets: Provide minecraft assets along with json files that help to use them*. GitHub. Retrieved 2026-09-12, from https://github.com/PrismarineJS/minecraft-assets

[^invassets]: InventivetalentDev. (n.d.). *minecraft-assets*. GitHub. Retrieved 2026-09-12, from https://github.com/InventivetalentDev/minecraft-assets

[^mcextract]: legopitstop. (n.d.). *mcextract-py: Extract assets and data from the Minecraft jar*. GitHub. Retrieved 2026-09-12, from https://github.com/legopitstop/mcextract-py

[^burger]: TkTech. (n.d.). *Burger: A simple tool for picking out information from the minecraft JARs*. GitHub. Retrieved 2026-09-12, from https://github.com/TkTech/Burger

[^mjextractor]: PrismarineJS. (n.d.). *minecraft-jar-extractor: Extract structured data from the minecraft jar*. GitHub. Retrieved 2026-09-12, from https://github.com/PrismarineJS/minecraft-jar-extractor

[^javama]: rmheuer. (n.d.). *McAssetExtractor*. GitHub. Retrieved 2026-09-12, from https://github.com/rmheuer/McAssetExtractor

[^pyma]: umittadelen. (n.d.). *MCAssetExtractor*. GitHub. Retrieved 2026-09-12, from https://github.com/umittadelen/MCAssetExtractor

[^zardoy]: zardoy. (n.d.). *mc-assets: The most complete solution for Minecraft assets*. GitHub. Retrieved 2026-09-12, from https://github.com/zardoy/mc-assets

[^fabric]: FabricMC. (n.d.). *Block Model Generation*. Fabric Documentation. Retrieved 2026-09-12, from https://docs.fabricmc.net/develop/data-generation/block-models

[^forge]: MinecraftForge. (n.d.). *Model Providers*. Forge Documentation. Retrieved 2026-09-12, from https://docs.minecraftforge.net/en/latest/datagen/client/modelproviders/

[^neoforge]: NeoForge. (n.d.). *Model Datagen*. NeoForge Documentation. Retrieved 2026-09-12, from https://docs.neoforged.net/docs/1.20.4/resources/client/models/datagen/

[^blockbench]: JannisX11. (n.d.). *Blockbench*. GitHub. Retrieved 2026-09-12, from https://github.com/JannisX11/blockbench

[^mccloud]: inventivetalent. (n.d.). *MC Assets - Browser for Minecraft Asset Files*. mcasset.cloud. Retrieved 2026-09-12, from https://mcasset.cloud/latest

[^misode]: misode. (n.d.). *misode.github.io — Blockstate Generator*. GitHub. Retrieved 2026-09-12, from https://github.com/misode/misode.github.io

[^nmd]: PrismarineJS. (n.d.). *node-minecraft-data*. npm. Retrieved 2026-09-12, from https://github.com/PrismarineJS/node-minecraft-data

[^pmd]: SpockBotMC. (n.d.). *python-minecraft-data*. GitHub. Retrieved 2026-09-12, from https://github.com/SpockBotMC/python-minecraft-data

[^mb]: py-mine. (n.d.). *minebase*. PyPI. Retrieved 2026-09-12, from https://github.com/py-mine/minebase

[^rs]: Trivernis. (n.d.). *minecraft-data-rs*. GitHub. Retrieved 2026-09-12, from https://github.com/Trivernis/minecraft-data-rs

[^go]: wlwanpan. (n.d.). *mcdata*. GitHub. Retrieved 2026-09-12, from https://github.com/wlwanpan/mcdata

[^schemas]: IzanagiTokiyuki. (n.d.). *minecraft-schemas*. Codeberg. Retrieved 2026-09-12, from https://codeberg.org/IzanagiTokiyuki/minecraft-schemas

[^lvjs]: Levertion. (n.d.). *minecraft-json-schemas*. GitHub. Retrieved 2026-09-12, from https://github.com/Levertion/minecraft-json-schemas

[^mcdoc]: SpyglassMC. (n.d.). *vanilla-mcdoc*. GitHub. Retrieved 2026-09-12, from https://github.com/SpyglassMC/vanilla-mcdoc