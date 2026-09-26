# 相容於 BIM 但適合公民（非專業人士）使用的室內 3D 資料格式研究

## 摘要

本研究探討有哪些資料格式既與 BIM（Building Information Modeling）相容或概念相通，又比完整 IFC/XMILE 更簡潔易懂，適合非專業公民進行室內 3D 建模使用。排除純藝術資產格式（如 glTF），聚焦於具備語意（semantic）與幾何（geometry）的室內空間建模場景。研究發現 **CityJSON** 為最平衡的候選格式，其次為 **IndoorGML**（空間圖論）、**GeoJSON-3D**（極簡方案）與 **ifcJSON**（保留 IFC 完整語意但改用 JSON 編碼）。

---

## 1. 背景與問題定義

BIM 的核心標準 **Industry Foundation Classes (IFC)**（ISO 16739）雖然是完整的開放標準，但其資料模型使用 EXPRESS 語言定義，序列化格式以 STEP Physical File (.ifc) 為主，對非專業公民而言學習門檻極高[^ifc]。同時，純視覺化格式如 glTF 雖在 Web 3D 渲染表現優異，但缺乏建築語意（牆、門、窗、房間），不適合需要查詢與操作的室內建模場景。

因此需要尋找一類「BIM-lite」格式，滿足以下條件：

- 具備建築語意（牆、門、窗、房間、樓層）
- 支援室內 3D 幾何
- 比 IFC-SPF 更易讀、易寫、易學
- 為開放標準或開放原始碼
- 非純藝術資產格式（排除 glTF）

---

## 2. 主要格式分析

### 2.1 CityJSON — 最平衡的全能候選

CityJSON 是 OGC 官方標準（文件 20-072r5），以 JSON 編碼 CityGML 資料模型的子集，專為「簡潔且對開發者友善」而設計[^cityjson-spec]。

**核心優勢：**

- **JSON 格式**：對 Web 開發者零學習成本，比 XML 友善許多
- **體積約為 CityGML-XML 的 1/7**：傳輸與儲存效率高[^cityjson-about]
- **扁平化層級結構**：CityJSON 設計原則明確指出「盡可能扁平化 CityGML 的深層嵌套」
- **完整的建築語意**：支援 `BuildingRoom`、`BuildingStorey`、`BuildingConstructiveElement`、`BuildingFurniture` 等室內物件[^cityjson-building]
- **語意表面（Semantic Surfaces）**：每個幾何面可標記為 `InteriorWallSurface`、`CeilingSurface`、`FloorSurface`、`Door`、`Window` 等[^cityjson-semantics]
- **開源生態系**：官方提供網頁檢視器（ninja.cityjson.org）、命令列工具（cjio）、驗證器（cityjson-validator）
- **雙向轉換**：可與 CityGML-XML 互轉

**室內建模細節：**

其 `Building` 模組定義了 8 種城市物件，其中直接用於室內的有：

| 物件類型 | 說明 |
|---|---|
| `BuildingRoom` | 房間，可用 Solid / CompositeSolid / MultiSurface 表示 |
| `BuildingStorey` | 樓層 |
| `BuildingUnit` | 單元（如公寓） |
| `BuildingConstructiveElement` | 結構元件（牆、柱、樑） |
| `BuildingFurniture` | 室內家具 |

`BuildingRoom` 的父層可以是 `Building` 或 `BuildingPart`；而 `BuildingFurniture` 與 `BuildingConstructiveElement` 的父層可以是 `BuildingRoom`，形成直覺的樹狀結構[^cityjson-building]。

**限制：**

- 主要設計目標為城市級建模，並非為了建築施工細節
- 尚未被主流 BIM 工具（Revit、ArchiCAD）原生支援（需轉換中介）

---

### 2.2 IndoorGML — 室內導航與空間拓樸的最佳選擇

IndoorGML 是 OGC 標準，專為室內空間導航用途設計的資料模型與 XML Schema[^indoorgml]。

**核心優勢：**

- **細胞空間模型（Cellular Space Model）**：將室內空間視為細胞集合（房間、走廊），非常直覺
- **可選幾何**：可透過三種選項處理幾何 — 選項一（外部引用 CityGML/IFC）、選項二（內嵌簡易幾何）、選項三（無幾何），極具彈性
- **圖論基礎**：房間為節點、門為邊，對非專業人士直觀易懂
- **多層空間模型**：可同時表達同一空間的不同詮釋（如建築物+感測器覆蓋範圍）
- **v2.0 開發中**：特別強調簡化資料模型，並計劃在 Part 2 支援 JSON 編碼[^indoorgml-v2]

**限制：**

- 不適合存放詳細 3D 幾何 — 幾何本身是選用且次要的
- 目前仍以 GML/XML 為主，JSON 支援尚未正式發布
- 最佳使用方式為與其他幾何格式搭配，而非獨立使用

---

### 2.3 ifcJSON — 保留 IFC 完整語意的 JSON 編碼

ifcJSON 是 buildingSMART 的專案，提供 IFC4/IFC4.3 資料模型的 JSON 編碼，目標是讓開發者無需理解 EXPRESS 語言即可存取 IFC 資料[^ifcjson]。

**核心優勢：**

- **JSON 格式**：消除理解 EXPRESS 和 STEP 檔案格式的障礙
- **與 IFC 完全相容**：可透過官方 Python 轉換器與標準 IFC-SPF 檔案雙向轉換
- **開放原始碼（MIT 授權）**
- **保留完整 IFC 語意**：牆、門、窗、樓板、空間等所有建築元素

**限制：**

- **僅編碼層面簡化，語意模型仍然複雜**：雖然從 STEP 改為 JSON 降低了技術門檻，但 IFC 本身的資料模型規模龐大，非專業人士仍難以掌握
- **官方承認可讀性並非首要設計目標**[^ifcjson]

---

### 2.4 GeoJSON-3D — 最簡單的幾何方案

GeoJSON 是 RFC 7946 定義的開放標準，用 JSON 表示地理特徵，可透過 `[x, y, z]` 座標支援 3D[^geojson]。

**核心優勢：**

- **極度簡單**：只要了解基本座標幾何即可使用
- **無所不在**：幾乎所有 GIS 與地圖函式庫都支援
- **人類可讀**：適合手動建立或程式生成
- **可自行擴展 properties**：在 `properties` 中加入建築語意資訊

**室內 3D 應用：**

- `Point` → 房間中心點或設備位置
- `Polygon` → 房間輪廓、樓層平面
- `MultiPolygon` → 建築足跡或多房間輪廓
- `properties` → 自訂房間名稱、用途、樓層等

**限制：**

- 無原生建築語意（牆、門、窗）
- 無拓樸概念
- 不支援實體幾何（Solid）或複雜 3D 結構
- 非 BIM 格式，僅為通用幾何格式

---

### 2.5 其他值得注意的格式

#### gbXML（Green Building XML）

專為能源分析開發的開放 XML Schema，以空間為核心的簡化建築幾何模型，具備 Campus → Building → Floor → Space 的層級結構。XML 格式使其比 IFC-SPF 易讀，但比 JSON 格式冗長[^gbxml]。

#### COBie（Construction Operations Building Information Exchange）

以試算表（XLSX/CSV）為主要格式的 BIM 資料交換標準，專注於設施移交階段的資產管理。對非專業人士極度友善（僅需試算表技能），但不含任何 3D 幾何 — 是純資料格式[^cobie]。

#### BCF（BIM Collaboration Format）

JSON 為基礎的議題追蹤格式，允許非專業人士在 BIM 模型上標註評論、截圖與視角，無需編輯模型本身。適用於公民參與回饋場景，但不含幾何資料[^bcf]。

#### 3D Tiles

OGC 社群標準，用於串流大型 3D 地理空間內容。其底層使用 glTF 作為容器，雖支援 BIM/CAD 內容及每特徵中繼資料，但生產端需要處理管線，且與 glTF 的緊密耦合可能不符「非藝術資產」的前提[^3dtiles]。

---

## 3. 綜合比較

| 格式 | 幾何 | 建築語意 | 公民友善度 | 最佳使用場景 |
|---|---|---|---|---|
| **CityJSON** | ✅ 完整 3D（Solid/MultiSurface） | ✅ 房間、牆、門、窗、家具 | ⭐⭐⭐⭐ JSON、生態系完整 | **室內 3D 建模首選** |
| **IndoorGML** | ⚠️ 選用（外部引用為佳） | ✅ 細胞空間、導航圖論 | ⭐⭐⭐ 圖論直覺、彈性高 | 室內導航、空間規劃 |
| **ifcJSON** | ✅ 完整 IFC 幾何 | ✅ 完整 IFC 語意 | ⭐⭐ JSON 友善但模型複雜 | 需與專業 BIM 工作流程相容 |
| **GeoJSON-3D** | ✅ 簡易幾何（點/線/面） | ❌ 僅自訂 properties | ⭐⭐⭐⭐⭐ 極簡 | 快速草稿、樓層平面 |
| **gbXML** | ✅ 簡化空間/表面 | ✅ 空間、開口 | ⭐⭐⭐ 層級結構易懂 | 能源分析、空間配置 |
| **COBie** | ❌ 無 | ✅ 設備/資產/空間 | ⭐⭐⭐⭐⭐ 試算表 | 非幾何的建築資料交換 |
| **BCF** | ❌ 僅視角 | ✅ 議題追蹤 | ⭐⭐⭐⭐⭐ 簡潔 JSON | 公民參與回饋 |
| **3D Tiles** | ✅ 高品質 3D | ✅ 每特徵中繼資料 | ⭐⭐ 需工具鏈 | Web 大場景視覺化 |

---

## 4. 結論與建議

對於「非專業公民進行室內 3D 建模」的需求，本研究推薦以下策略：

### 主要推薦：CityJSON

**CityJSON 是最能平衡「建築語意完整度」與「公民可及性」的格式**。它提供了完整的房間、牆面、門窗、樓層語意，同時使用現今最普及的 JSON 格式，具備開源工具生態系與 OGC 標準地位。對於需要建立可查詢、可操作的室內 3D 模型的公民專案，CityJSON 應作為首要考量。

若專案需要與專業 BIM 工作流程（Revit、ArchiCAD）互通，則應考慮 **ifcJSON** 作為橋接格式，但需注意其語意複雜度對公民的學習負擔。

### 特定場景建議

- **純空間規劃（無需詳細幾何）** → IndoorGML
- **極簡快速原型** → GeoJSON-3D 擴展 properties
- **公民參與回饋** → BCF（標註）+ CityJSON（模型）
- **非專業者只需檢視（不需編輯）** → BIMx（Graphisoft 免費工具）

### 關鍵發現

目前學術界與產業界已意識到「公民參與 BIM」的需求，Graphisoft 的 BIMx 工具[^bimx] 與多篇論文[^rosu2023][^osello2016] 均顯示此趨勢。然而在**資料格式層面**，目前仍未有專為「公民室內建模」設計的格式 — 所有現有方案均為專業格式的簡化改編或子集。若此領域需求持續增長，未來可能催生如 **CitizenBIM** 或 **IndoorJSON** 這類專為非專業人士設計的新格式。

---

## 參考資料

[^ifc]: buildingSMART. (n.d.). Industry Foundation Classes (IFC). Retrieved 2026-09-25, from https://www.buildingsmart.org/standards/bsi-standards/industry-foundation-classes/

[^cityjson-spec]: CityJSON. (n.d.). CityJSON Specification 2.0.2. Retrieved 2026-09-25, from https://www.cityjson.org/specs/2.0.2/

[^cityjson-about]: CityJSON. (n.d.). About CityJSON. Retrieved 2026-09-25, from https://www.cityjson.org/about/

[^cityjson-building]: CityJSON. (n.d.). CityJSON Specification — Building (§2.3). Retrieved 2026-09-25, from https://www.cityjson.org/specs/2.0.2/

[^cityjson-semantics]: CityJSON. (n.d.). CityJSON Specification — Semantics of Geometric Primitives (§3.3). Retrieved 2026-09-25, from https://www.cityjson.org/specs/2.0.2/

[^indoorgml]: Open Geospatial Consortium. (n.d.). IndoorGML Standard. Retrieved 2026-09-25, from https://www.ogc.org/standard/indoorgml/

[^indoorgml-v2]: IndoorGML. (n.d.). IndoorGML v2.0. Retrieved 2026-09-25, from https://www.indoorgml.net/

[^ifcjson]: buildingSMART. (n.d.). ifcJSON. Retrieved 2026-09-25, from https://github.com/buildingSMART/ifcJSON

[^geojson]: IETF. (2016). The GeoJSON Format (RFC 7946). Retrieved 2026-09-25, from https://en.wikipedia.org/wiki/GeoJSON

[^gbxml]: Wikipedia. (n.d.). Green Building XML (gbXML). Retrieved 2026-09-25, from https://en.wikipedia.org/wiki/Green_Building_XML

[^cobie]: Wikipedia. (n.d.). COBie. Retrieved 2026-09-25, from https://en.wikipedia.org/wiki/COBie

[^bcf]: Wikipedia. (n.d.). BIM Collaboration Format. Retrieved 2026-09-25, from https://en.wikipedia.org/wiki/BIM_Collaboration_Format

[^3dtiles]: CesiumGS. (n.d.). 3D Tiles. Retrieved 2026-09-25, from https://github.com/CesiumGS/3d-tiles

[^bimx]: Graphisoft. (n.d.). BIMx. Retrieved 2026-09-25, from https://www.graphisoft.com/en-tw/plans-and-products/bimx/

[^rosu2023]: Rosu, S. (2023). Smart Citizen in a Smart Building in Society 5.0. IEEE. Retrieved 2026-09-25, from https://ieeexplore.ieee.org/abstract/document/10214838/

[^osello2016]: Osello, A., et al. (2016). District information models. The DIMMER project: BIM tools for the urban scale. Retrieved 2026-09-25, from https://www.torrossa.com/gs/resourceProxy?an=3445288&publisher=FM0520