# IndoorGML 能否處理傢俱（Furniture）？——研究報告

## 1. 前言

IndoorGML 是 OGC（Open Geospatial Consortium）制定的室內空間資料標準，主要用於室內導航、定位服務及路徑規劃。本報告旨在回答一個具體問題：**IndoorGML 是否能處理傢俱的表示？** 如果不能，有哪些替代標準？

## 2. IndoorGML 的核心定位：導航導向，非物件導向

IndoorGML 的核心目標是**空間中心（space-centered）**而非**物件中心（object-centered）**。它不關注建築元件本身（如牆壁、天花板、屋頂），而是關注由這些元件所定義的**空間**（如房間、走廊、樓梯），用於描述物體可在其中定位與導航的空腔[^ogc22]。

IndoorGML 2.0 規範第 7.1.1 條明確指出：

> *「Components irrelevant to describe the spaces, such as furniture, are not within the scope of IndoorGML.」*（與描述空間無關的元件，例如傢俱，不屬於 IndoorGML 的範疇。）[^ogc22]

規範進一步將室內空間資訊分為兩類：

- **類別 1**：建築元件與內部設施（包括傢俱）——用於建築/設施管理
- **類別 2**：空腔（房間、走廊）或虛擬劃分——用於室內 LBS、導航

IndoorGML 僅涵蓋**類別 2**。[^ogc22]

## 3. IndoorGML 如何表示室內特徵

IndoorGML 採用**細胞空間模型（Cellular Space Model）**：

- 室內空間被分解為**細胞（CellSpace）**——最小的組織單元
- 每個細胞有唯一 ID、名稱，及可選的幾何資訊
- 同一主題圖層（SpaceLayer）內的細胞**不可重疊**[^ogc22]

細胞的幾何表示有三種方式：

1. **明確幾何**：使用 GM\_Solid（3D）或 GM\_Surface（2D）
2. **外部參照**：連結到 CityGML、IFC 等外部資料集中的物件
3. **無幾何**：僅以識別碼定義細胞[^ogc22]

此外，IndoorGML 採用**多重圖層空間模型（MLSM）**，允許同一空間在不同主題圖層（如地形圖層、Wi-Fi 覆蓋圖層）中有不同解釋，並透過 InterLayerConnection 連結。

## 4. 傢俱在 IndoorGML 中的處理方式

**結論：IndoorGML 沒有傢俱的一級特徵類型（first-class feature type）。**

IndoorGML 的 UML 模型中不存在任何 `Furniture` 或 `BuildingFurniture` 的類型。規範中僅在提及導航障礙物時提到傢俱，例如：

> 「室內有大量障礙物，如傢俱、柱子、圍欄、裝飾品。」[^ogc20]

規範圖例 Fig. 7 展示了一個「有傢俱的室內空間」，但傢俱本身並未被建模——而是將**傢俱佔據的空間排除**在可導航區域之外，以定義「自由空間」供行走[^ogc22]。

## 5. 理論上的非標準變通做法

雖然 IndoorGML 沒有官方支援，但理論上可在**多重圖層空間模型（MLSM）**中建立一個自訂的主題圖層（例如「傢俱佔用圖層」），將傢俱所佔據的區域表示為 CellSpace。然而：

- 傢褀會被表示為**「空間」**而非**「傢俱物件」**
- 這屬於**應用端自訂擴充**，不屬於標準的一部分
- 目前沒有任何官方擴充模組處理此問題[^ogc22]

## 6. 替代標準比較

| 標準 | 傢俱支援 | 最佳用途 |
|---|---|---|
| **IFC**（Industry Foundation Classes） | **優異** — 以 `IfcFurnishingElement` 作為一級實體。IFC 是 BIM（建築資訊模型）的國際標準 ISO 16739-1:2024，涵蓋所有建築元素包括傢俱，具備完整幾何、材質與屬性。 | 詳細 BIM/設施管理、建築全生命週期資料 |
| **CityGML 3.0**（OGC 標準） | **支援** — CityGML 可在多種 LoD 中建模室內空間，包含 `BuildingFurniture` 特徵類型。 | 含室內細節的 3D 城市模型 |
| **IMDF**（Apple Indoor Mapping Data Format） | 有限 — 聚焦於場地、單元、設施，可包含固定裝置/便利設施資料，但非傢俱專用。 | 室內導航 App、Apple Maps 室內定位 |
| **IndoorGML** | **不支援** — 空間中心、導航導向，明確排除傢俱 | 室內導航與路徑規劃 |

IndoorGML 規範自身也定位為**CityGML、IFC、LADM、IMDF 的互補標準**，透過**外部參照**機制（Option 2 幾何）指向這些標準中的詳細幾何模型[^ogc22]。

## 7. IndoorGML 模組結構現況

IndoorGML 2.0 Part 1 僅包含兩個標準化模組：

- **Core 模組**：定義 CellSpace、SpaceLayer、State、Transition、MultiLayeredGraph
- **Navigation 擴充模組**：加入導航語意（可導航/不可導航細胞、連接限制、移動類型）

**目前尚未標準化的模組**包括傢俱模組或室內物件擴充。規範中指出「包括室內設施管理的其他需求將由下一版 IndoorGML 處理」，暗示未來版本可能擴張到類別 1 領域，但截至 v2.0 尚未實現[^ogc22]。

## 8. 結論

IndoorGML **無法**直接處理傢俱表示。若需要傢俱層級的室內建模，建議使用 **IFC**（含 `IfcFurnishingElement`）或 **CityGML 3.0**（含 `BuildingFurniture`）。IndoorGML 可透過外部參照與這些標準協作，但本身不具備傢俱的語意或幾何表示能力。

---

[^ogc22]: Open Geospatial Consortium. (2026). *IndoorGML 2.0 Part 1 – Conceptual Model* (OGC 22-045r5). Retrieved 2026-09-25, from https://docs.ogc.org/is/22-045r5/22-045r5.html

[^ogc20]: Open Geospatial Consortium. (2020). *IndoorGML 1.1 Specification* (OGC 19-011r4). Retrieved 2026-09-25, from https://docs.ogc.org/is/19-011r4/19-011r4.html