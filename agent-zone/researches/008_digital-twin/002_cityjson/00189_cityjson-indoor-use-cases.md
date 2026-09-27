# CityJSON 是否能處理室內（Indoor）應用場景？

## 1. 概述

CityJSON 是一種以 **JSON** 為基礎的 **3D 城市模型** 編碼格式，實作了 OGC CityGML v3.0 資料模型的主要部分，並已成為 **OGC 官方標準**（文件 20-072r5）[^cityjson-spec]。其核心設計目標包括：比 CityGML-XML 平均 **小 7 倍的檔案體積**、對開發者友善的語法、以及與 CityGML-XML 的雙向轉換。

本文探討 CityJSON 是否能滿足室內（indoor）應用場景的需求，包括房間建模、室內語義表面、室內導航、以及與 BIM/IFC 的互通性。

## 2. CityJSON 對室內空間的內建支援

### 2.1 室內相關的城市物件類型

CityJSON v2.0.2 規範明確定義了以下可用於室內建模的物件類型[^cityjson-spec]：

| 物件類型 | 說明 | 室內相關性 |
|---|---|---|
| `BuildingRoom` | 建築物內的個別房間 | **直接建模房間**，可使用 Solid 或 CompositeSolid 幾何 |
| `BuildingStorey` | 建築樓層 | 建模建築物的垂直分層 |
| `BuildingUnit` | 建築單元（如公寓） | 建模建築內的空間分割 |
| `BuildingFurniture` | 建築內家具 | 建模室內物體 |
| `BuildingConstructiveElement` | 結構元素（牆、柱） | 室內結構組件 |
| `TunnelHollowSpace` | 隧道內部中空空間 | 建模隧道內部 |
| `TunnelFurniture` | 隧道內家具 | 隧道內部物體 |
| `BridgeRoom` | 橋梁結構內的房間 | 建模橋梁內部 |
| `BridgeFurniture` | 橋梁內家具 | 橋梁內部物體 |

### 2.2 室內語義表面

CityJSON 規範明確定義了以下適用於室內的語義表面類型（Semantic Surface）[^cityjson-spec]：

- `InteriorWallSurface` — 室內牆面
- `CeilingSurface` — 天花板
- `FloorSurface` — 地板
- `ClosureSurface` — 封閉表面
- `OuterCeilingSurface` — 外部天花板
- `OuterFloorSurface` — 外部地板

這些表面類型可附加於 `BuildingRoom` 的 Solid 幾何上，對每個面進行語義標註。

### 2.3 室內空間的具體範例

以下為官方規範中建模一個房間的範例[^cityjson-spec]：

```json
"myroom": {
  "type": "BuildingRoom",
  "attributes": {
    "usage": "living room"
  },
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

## 3. 相關擴充功能（Extensions）

CityJSON 的擴充功能機制允許新增自訂屬性與物件類型。目前官方擴充功能註冊表中[^extensions] 與室內相關的主要擴充功能為：

- **energy-space-heating**（v1.1.1）：基於 TU Delft Özge Tufan 碩士論文開發，使用 `BuildingRoom` 進行空間供暖需求計算，明確以室內空間及其屬性（房間幾何、表面、熱性質）作為運算基礎[^energy-ext]。

其他可應用於室內場景的擴充功能：
- **dynamizer**（v2.0.0）：表示動態資料（如室內感測器資料）
- **lcc**（v0.3.0）：線性細胞複合體（Linear Cell Complex），可用於表示室內連通性拓撲
- **quality**（v1.0.1）：資料品質標註

**目前尚無專用的「Indoor」擴充功能註冊**。若有室內導航網路或感測器整合等進階需求，需透過 [Extension Builder](https://www.cityjson.org/extensions/builder/) 建立自訂擴充功能。

## 4. CityJSON 與 CityGML 的室內建模比較

| 面向 | CityJSON | CityGML |
|---|---|---|
| 建築模組支援 | 100%（含 `BuildingRoom`、`BuildingStorey`、`BuildingUnit`） | 完整概念模型 |
| 隧道室內部 | 部分支援（有 `TunnelHollowSpace`） | 完整概念模型 |
| 橋梁室內部 | 部分支援（有 `BridgeRoom`） | 完整概念模型 |
| 室內語義 | 支援 `InteriorWallSurface`、`CeilingSurface`、`FloorSurface`、`ClosureSurface` | 相同語義（繼承自 CityGML） |
| 室內 LoD | 使用 TU Delft 精煉 LoD（如 2.2、3.1），**無明確「LoD4」** | CityGML 有專用 LoD4 |
| 拓樸關係 | **不支援** — 無法表達 `relativeToTerrain`、`relativeToWater` 或 XLinks | 完整支援 |
| 房間連通性 | **無內建支援** | 有限支援 |
| IndoorGML 整合 | 非原生；val3dity 可分別驗證 IndoorGML | 可與 IndoorGML 結合 |
| BIM/IFC 轉換 | 透過 **IFCCityJSON** 工具雙向轉換 | 透過標準流程轉換 |
| 檔案體積 | 約小 7 倍 | 較冗長 |

## 5. 已知應用與學術研究

1. **Ledoux et al. (2019)** — 《CityJSON: A compact and easy-to-use encoding of the CityGML data model》（Open Geospatial Data, Software and Standards, 4:4）。奠基論文，指出 CityJSON「可儲存建築物的外部與可能的內部」[^ledoux2019]。

2. **Özge Tufan — TU Delft 碩士論文** — 《CityJSON Energy Extension for Space Heating Demand Calculation》，明確使用 `BuildingRoom` 物件進行室內能源模擬，成果即為 **energy-space-heating** 擴充功能。

3. **IFCCityJSON 工具**（IfcOpenShell 專案的一部分）— 實現 CityJSON 與 IFC（Industry Foundation Classes）之間的雙向轉換，橋接城市尺度與建築尺度 BIM 模型[^software]。

4. **val3dity** — 3D 驗證工具，同時支援 CityJSON **與 IndoorGML** 作為輸入格式，顯示 CityJSON 生態系統可與專用室內格式互通。

5. **3D City DB** — 可在空間資料庫（PostGIS/Oracle）中儲存與管理包含建築內部的 CityJSON 資料。

6. **官方應用列表** — 列出與室內建模相關的使用案例：設施管理（Facility management）、緊急應變（Emergency response）、導航視覺化（Visualisation for navigation）、虛擬導覽（Virtual tours）、路徑規劃（Routing）[^applications]。

## 6. 限制與不足

1. **無原生房間連通性/鄰接關係** — CityJSON 沒有內建機制表達房間之間的連通關係（門、走廊、相鄰）。需透過自訂擴充功能或從幾何計算取得。

2. **無室內導航網路** — 不像 IndoorGML 專為室內導航設計了細胞空間與連通圖，CityJSON 缺乏等效的導航模組。

3. **無正式 LoD4** — 雖然 TU Delft 精煉 LoD 更靈活，但缺乏標準化的「LoD4」可能與期待 CityGML LoD 分類的工具產生互通性問題。

4. **拓樸關係不支援** — 無法表達兩個房間共享牆面（XLink 語義），或建築物的 `relativeToTerrain` 關係。

5. **無專用走廊/通道類型** — 可用 `BuildingRoom` 搭配屬性表示走廊，但無專屬物件類型。

6. **僅具幾何室內空間** — 可良好表示房間的立體幾何，但缺乏高階概念如房間連通性、可及性或功能分區。

7. **複雜屬性簡化** — CityGML 的 `gml:Measure` 屬性（附單位）在 CityJSON 中被簡化，若無擴充功能則遺失明確度量單位。

8. **無 Versioning 模組** — CityGML v3.0 的版本管理模組不支援，追蹤室內空間隨時間的變化需要基於 Git 的變通方案。

9. **語義粒度有限** — 雖有室內牆面、天花板、地板表面類型，但缺乏 BIM/IFC 等級的細節（如材料層、熱性質），需自訂擴充功能。

10. **室內工具覆蓋率不足** — 多數 CityJSON 工具聚焦於外部城市建模（視覺化、日照分析、噪音模擬），少數工具專門針對室內應用。

## 7. 結論

| 能力 | CityJSON 支援程度 |
|---|---|
| 以 3D 立體表示房間 | ✅ 支援（`BuildingRoom` + Solid 幾何） |
| 表示樓層 | ✅ 支援（`BuildingStorey`） |
| 語義表面分類（室內牆、地板、天花板） | ✅ 支援（`InteriorWallSurface`、`FloorSurface`、`CeilingSurface`、`ClosureSurface`） |
| 表示室內家具 | ✅ 支援（`BuildingFurniture`） |
| 隧道內部 | ✅ 支援（`TunnelHollowSpace`） |
| 橋梁內部房間 | ✅ 支援（`BridgeRoom`） |
| 房間連通性/鄰接關係 | ❌ 無原生支援 |
| 室內導航網路 | ❌ 不支援（需使用 IndoorGML） |
| BIM/IFC 互通性 | ✅ 透過 IFCCityJSON 工具 |
| 專用室內擴充功能 | ❌ 註冊表中無（可自建） |
| 室內能源模擬 | ✅ 透過 energy-space-heating 擴充功能 |

**總結：CityJSON 能夠處理基本的室內應用場景**——以帶有語義的 3D 幾何表示房間、樓層、室內表面與家具。在設施管理、能源模擬、室內視覺化等領域已具實用性。

然而，對於**室內導航、路徑規劃、連通圖分析、網路分析**等進階室內應用，CityJSON 單獨使用並不足夠，需結合 IndoorGML 或建立自訂擴充功能來補足缺失的功能。

---

## 參考文獻

[^cityjson-spec]: CityJSON. (n.d.). CityJSON Specification v2.0.2. Retrieved 2026-09-25, from https://www.cityjson.org/specs/2.0.2/
[^cityjson-about]: CityJSON. (n.d.). About CityJSON. Retrieved 2026-09-25, from https://www.cityjson.org/about/
[^citygml-v30]: CityJSON. (n.d.). CityGML 3.0 Implementation. Retrieved 2026-09-25, from https://www.cityjson.org/citygml/v30/
[^extensions]: CityJSON. (n.d.). Extensions. Retrieved 2026-09-25, from https://www.cityjson.org/extensions/
[^extensions-repo]: CityJSON. (n.d.). CityJSON Extensions Registry. Retrieved 2026-09-25, from https://github.com/cityjson/extensions
[^energy-ext]: CityJSON. (2023). energy-space-heating Extension v1.1.1. Retrieved 2026-09-25, from https://github.com/cityjson/extensions/tree/main/extensions/energy-space-heating/1.1.1
[^ledoux2019]: Ledoux, H., Arroyo Ohori, K., & Kumar, K. (2019). CityJSON: A compact and easy-to-use encoding of the CityGML data model. *Open Geospatial Data, Software and Standards*, 4:4. Retrieved 2026-09-25, from http://dx.doi.org/10.1186/s40965-019-0064-0
[^software]: CityJSON. (n.d.). Software. Retrieved 2026-09-25, from https://www.cityjson.org/software/
[^applications]: CityJSON. (n.d.). Applications. Retrieved 2026-09-25, from https://www.cityjson.org/applications/
[^val3dity]: tudelft3d. (n.d.). val3dity. Retrieved 2026-09-25, from https://github.com/tudelft3d/val3dity