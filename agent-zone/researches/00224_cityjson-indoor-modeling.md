# CityJSON 用於室內建模之可行性分析

## 摘要

CityJSON 為 CityGML 之輕量化 JSON 編碼格式，由荷蘭臺夫特理工大學（TU Delft）3D 地理資訊團隊主導開發。本報告探討 CityJSON 是否能夠用於描述室內空間，結論為：**CityJSON 2.0 原生完整支援室內建模**，涵蓋房間、樓層、室內語意表面等核心功能，並在部分面向（如延伸 LoD）超越 CityGML 3.0。[^cityjson-spec]

---

## 1. 室內專屬 City Object 類型

CityJSON 2.0 定義了多種專屬室內物件類型，歸屬於 Building Module[^cityjson-spec-obj]：

| City Object | 說明 |
|---|---|
| `BuildingRoom` | 建築物內部房間／空間 |
| `BuildingStorey` | 建築物樓層 |
| `BuildingUnit` | 功能單元（如公寓） |
| `BuildingFurniture` | 室內家具 |
| `BuildingConstructiveElement` | 結構性室內元素（如柱、樑） |
| `BridgeRoom` | 橋梁內部空間 |
| `TunnelHollowSpace` | 隧道內部中空空間 |
| `TunnelFurniture` | 隧道內家具 |

這些物件皆為一等市民（first-class city object），可擁有獨立的幾何、語意及屬性。例如 `BuildingRoom` 的 JSON 表示：

```json
"myroom": {
  "type": "BuildingRoom",
  "attributes": { "usage": "living room" },
  "parents": ["id-1"],
  "geometry": [{
    "type": "Solid",
    "lod": "2",
    "boundaries": [
      [ [[0, 3, 2, 1]], [[4, 5, 6, 7]], [[0, 1, 5, 4]], ... ]
    ]
  }]
}
```

---

## 2. 室內語意表面類型

CityJSON 支援下列室內專屬語意表面（Semantic Surface）[^cityjson-geom]：

| 語意類型 | 說明 |
|---|---|
| `InteriorWallSurface` | 室內牆面 |
| `CeilingSurface` | 天花板 |
| `FloorSurface` | 地板 |

加上既有的室外表面如 `RoofSurface`、`WallSurface`、`GroundSurface`、`Window`、`Door` 等，可完整描述建築物內外所有表面的語意分類。

這些室內表面自 v1.1.1 起加入核心規範（對應 GitHub issue [#98]）[^issue-98]。

---

## 3. 細節層級（LoD）支援

### 3.1 CityGML 2.0 之 LoD4

CityGML 2.0 中，LoD4 為最高細節層級，涵蓋完整的室內模型：房間、內牆、地板、天花板、家具、建築設施。[^ogc-citygml]

### 3.2 CityGML 3.0 之變革

CityGML 3.0 廢除了單一 LoD4 概念，改為**在各 LoD 層級中分別支援室內表示**。OGC 官方說明：「相比之前版本，與 BIM 的整合更佳，且能在不同 LoD 層級中表示室內空間」。[^ogc-citygml3]

### 3.3 TU Delft 精細化 LoD

CityJSON 原生支援 TU Delft 提出的精細化 LoD 框架（Biljecki et al., 2016）[^tud-lod]，其中包含多種室內層級：

- **LoD0.2、LoD0.3** — 樓層平面圖
- **LoD1.2、LoD2.2、LoD3.2** — 不同細節程度的室內空間

---

## 4. 與 CityGML 之比較

| 功能 | CityGML 3.0 | CityJSON 2.0 |
|---|---|---|
| 建築房間 (`BuildingRoom`) | ✅ | ✅ |
| 建築樓層 (`BuildingStorey`) | ✅ | ✅ |
| 建築單元 (`BuildingUnit`) | ✅ | ✅ |
| 建築家具 (`BuildingFurniture`) | ✅ | ✅ |
| 室內牆面語意 | ✅ | ✅ |
| 天花板語意 | ✅ | ✅ |
| 地板語意 | ✅ | ✅ |
| TU Delft 精細化 LoD | ❌ 非核心 | ✅ 原生支援 |
| XLink 拓樸關係 | ✅ | ❌ 不支援 |
| 版控模組 | ✅ Versioning | ❌ 採 Git 替代方案 |

CityJSON v2.0 對 CityGML 3.0 Building Module 的實作率為 **100%**。[^cityjson-gml-impl]

---

## 5. 實際應用案例

### IFC_BuildingEnvExtractor（TU Delft）

最重要的實際專案，可將 BIM（IFC）模型轉換為 CityJSON 並包含完整室內資訊[^tud-ifc]：

> 「本軟體可建立含有懸挑結構（LoD3/3.2）**以及室內空間與／或樓層（LoD0.2、0.3、1.2、2.2、3.2）** 的 CityJSON 模型。」

轉換結果包含：
- **外殼（Outer Shell）** — 各 LoD 的建築輪廓
- **內殼（Inner Shell）** — 按樓層 → 空間／房間階層組織的室內空間

此專案由 TU Delft、CHEK 專案、Geonovum、VNG 及 Eindhoven 市政府共同資助（執行至 2026 年 8 月）。

### Inclusive TU Delft Map

可載入自訂幾何（建築外殼與房間）並匯出為 CityJSON 格式，支援自訂屬性[^tud-inclusive]。

---

## 6. 限制與不足

### 6.1 缺乏 XLink 拓樸關係

CityJSON 不支援 XLink，無法原生表達相鄰房間之間的拓樸連通關係。對於室內導航等需要空間鄰接資訊的應用，需於使用端自行計算或透過 Extension 機制擴充。[^cityjson-gml-impl]

### 6.2 無版控模組

CityGML 的 Versioning 模組（用於追蹤室內模型隨時間的變更）未實作，改以 Git 為替代方案。

### 6.3 無多 CRS 支援

單一 CityJSON 檔案內所有幾何必須使用同一坐標參考系統，對跨坐標系統的大型室內模型可能構成限制。

### 6.4 洞口語意繼承

規範指出：「孔洞、內部表面無法擁有獨立語意，而是繼承所屬表面的語意資訊」——即室內表面上的開口（如門窗洞）繼承父表面的語意。

### 6.5 IndoorGML 整合

相較 CityGML 可與 IndoorGML 搭配進行室內導航分析，CityJSON 目前尚無原生的 IndoorGML 整合路徑。

---

## 7. 結論

CityJSON **完全可以**用於描述室內空間。其原生支援：

- 房間、樓層、單元等室內物件類型
- `InteriorWallSurface`、`CeilingSurface`、`FloorSurface` 等室內語意表面
- TU Delft 精細化 LoD（含多種室內細節層級）
- 完整的 BIM（IFC） → CityJSON 轉換管道
- 可透過 Extension 機制擴充自訂室內屬性

主要限制為缺乏 XLink 拓樸關係與版控模組。對於絕大多數室內建模使用場景——包含房間層級表示、樓層組織、語意表面分類——CityJSON 提供與 CityGML 3.0 同等甚至更優越的能力。

---

[^cityjson-spec]: CityJSON. (n.d.). CityJSON Specification 2.0.2. Retrieved 2026-09-25, from https://www.cityjson.org/specs/2.0.2/
[^cityjson-spec-obj]: CityJSON. (n.d.). CityJSON Specification 2.0.2 — City Objects. Retrieved 2026-09-25, from https://www.cityjson.org/specs/2.0.2/
[^cityjson-geom]: CityJSON. (n.d.). CityJSON Specification 2.0.2 — Geometry System. Retrieved 2026-09-25, from https://deepwiki.com/cityjson/specs/2.4-geometry-system
[^issue-98]: CityJSON Specs. (n.d.). GitHub Issue #98 — Add semantic types for interior surfaces. Retrieved 2026-09-25, from https://github.com/cityjson/specs/issues/98
[^ogc-citygml]: Open Geospatial Consortium. (n.d.). CityGML Standard. Retrieved 2026-09-25, from https://www.ogc.org/standards/citygml/
[^ogc-citygml3]: CityJSON. (n.d.). CityGML v3.0 Implementation Details. Retrieved 2026-09-25, from https://www.cityjson.org/citygml/v30/
[^tud-lod]: Biljecki, F., Ledoux, H., & Stoter, J. (2016). An improved LOD specification for 3D building models. Retrieved 2026-09-25, from https://3d.bk.tudelft.nl/hledoux/pdfs/16_ceus_lod_specs.pdf
[^tud-ifc]: TU Delft 3D Geoinformation. (n.d.). IFC_BuildingEnvExtractor. Retrieved 2026-09-25, from https://github.com/tudelft3d/IFC_BuildingEnvExtractor
[^tud-inclusive]: TU Delft 3D Geoinformation. (n.d.). Inclusive TU Delft Map. Retrieved 2026-09-25, from https://tudelft3d.github.io/Inclusive-TU-Delft-Map/
[^cityjson-gml-impl]: CityJSON. (n.d.). CityJSON Implementation of CityGML v3.0. Retrieved 2026-09-25, from https://www.cityjson.org/citygml/v30/