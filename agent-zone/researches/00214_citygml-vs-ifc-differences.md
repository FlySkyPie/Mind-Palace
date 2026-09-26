# CityGML 與 IFC 之差異分析

## 概述

CityGML（City Geography Markup Language）與 IFC（Industry Foundation Classes）分別是 **3D 城市模型**與**建築資訊模型（BIM）**領域中最重要且最廣泛使用的開放式國際標準。兩者雖然都處理三維空間中的建築與環境物件，但其**目的、資料模型、幾何表示方式、語義粒度與應用場景**有本質上的差異。

本文旨在為不熟悉任一標準的讀者，系統性地比較這兩種標準，說明其各自的設計哲學與適用場景，並探討在**城市數位孿生（Urban Digital Twin）**中如何協同使用。

---

## 各論

### CityGML

CityGML 是由 **Open Geospatial Consortium（OGC）** 制定的國際標準（2008 年首次發布，目前最新版本為 3.0），用於儲存與交換虛擬 **3D 城市模型**與景觀模型。它基於 GML3（Geography Markup Language）實作，描述**城市作為一組地理參考特徵**——建築、道路、植被、水體、地形、城市家具等[^citygml-wiki]。

關鍵特徵：
- **語義與幾何協同模型**：不僅描述物體的幾何形狀，還包含其語義（用途、分類、彼此關係）
- **五層細節層次（LOD0~LOD4）**：從 2.5D 地形（LOD0）到包含室內結構（LOD4），允許同一物體以多種細節層次表示
- **模組化設計**：核心模組搭配 Building、Transportation、Vegetation、WaterBody、LandUse、Bridge、Tunnel 等擴展主題模組
- **ADE（Application Domain Extensions）**：允許使用者自訂擴展以滿足特定領域需求
- **地理座標系統**：所有物件位於真實世界地理座標中
- **主要檔案格式**：GML/XML（.gml/.xml）、CityJSON（.city.json）[^citygml-ogc]

### IFC

IFC 是由 **buildingSMART International**（前身為 IAI）開發與維護的開放式資料交換格式，正式註冊為 **ISO 16739-1:2024** 國際標準。它的設計目的是讓建築、工程、營造（AEC）產業中不同的 BIM 軟體能夠交換與共享建築資料，涵蓋建築從設計、施工、營運到拆除的**全生命週期**[^ifc-wiki]。

關鍵特徵：
- **物件導向的實體-關係模型**：使用 EXPRESS 語言（ISO 10303 系列）定義，IFC4.3 版本包含約 800 個實體物件
- **5W1H 分類體系**：參與者（Who）、控制項（Why）、群組（What）、產品（Where）、流程（When）、資源（How）
- **工程構件級語義**：牆、梁、柱、板、門、窗、樓梯、管線、閥門、設備等
- **多種幾何表示**：B-rep、NURBS、CSG、掃掠體（Extrusion）等實體模型
- **局部工程座標**（通常為毫米），需透過 `IfcMapConversion` 對接到世界座標
- **多種編碼格式**：IFC-SPF（.ifc）、IFC-XML（.ifcXML）、IFC-ZIP（.ifcZIP）、ifcJSON（.json）等[^ifc-buildingsmart]

---

## 比較分析

### 標準組織與領域歸屬

| 維度 | IFC | CityGML |
|------|-----|---------|
| 標準組織 | buildingSMART（ISO 16739） | OGC（Open Geospatial Consortium） |
| 領域 | BIM（建築資訊模型） | 3D GIS（三維地理資訊系統） |
| 核心使命 | 描述建築作為一個營造專案 | 描述城市作為一組地理參考特徵 |
| 資料模型語言 | EXPRESS（ISO 10303-11） | GML3 / UML 概念模型（XML Schema） |

一句話概括：**IFC 描述「建築怎麼蓋」，CityGML 描述「城市裡有什麼」**[^gisyxs]。

### Level of Detail vs Level of Development

這是最常被混淆的一組概念。兩者名稱相似但含義完全不同：

| 概念 | CityGML 的 LOD | IFC 的 LOD（LoD） |
|------|----------------|-------------------|
| 正式名稱 | Level of Detail（細節層級） | Level of Development（發展成熟度） |
| 意義 | **幾何抽象程度**——外觀細節的多寡 | **資訊可靠度**——模型元素被確信的程度 |
| 層級範例 | LOD0（地形）→ LOD1（體塊）→ LOD2（屋頂）→ LOD3（立面）→ LOD4（室內） | LOD 100（概念）→ 200（大致）→ 300（精確）→ 400（製造）→ 500（竣工） |
| 能否互轉？ | **不能直接對應** | — |

正如 3D Geospatial 網站指出：「IFC 沒有 LOD；其細節跟隨專業模型與專案階段，所以兩者之間進行『LOD 比較』是類別錯誤（category error）。」[^3dgeospatial]

### 幾何表示方式

| 面向 | IFC | CityGML |
|------|-----|---------|
| 幾何類型 | 實體模型（Solid Model）：拉伸體、布林運算、B-Rep、NURBS | 邊界表面（B-Rep / Boundary Surface）：三角網格、多邊形面 |
| 精細度 | 構件級——每面牆、每根梁都有獨立幾何與屬性 | 表面級——屋頂面、牆面、地面等語義表面 |
| 座標系統 | 局部工程座標（預設毫米） | 地理座標系統（CRS，單位為米） |

IFC 的幾何可以回答「這面牆是什麼材料、厚度多少」，CityGML 的幾何可以回答「這棟建築在哪個位置、高度多少」[^3dgeospatial]。

### 語義豐富度

| 面向 | IFC | CityGML |
|------|-----|---------|
| 語義粒度 | 工程構件級——牆、梁、柱、板、門、窗、管線、設備、閥門 | 城市實體級——建築、道路、橋樑、隧道、植被、水體、城市家具 |
| 可表達屬性 | 材料、防火等級、製造商、成本、所屬樓層、熱工性能 | 建築用途、樓層數、地址、地籍編號、屋頂類型、LOD 層級 |
| 無法表達 | 城市上下文（道路、周邊建築、地籍關係） | 工程構件級的材料、設備系統等工程細節 |

**兩者互不為子集**：IFC 有 CityGML 無法表達的構件，CityGML 有 IFC 缺乏的城市上下文與地理參考[^3dgeospatial]。

### 空間範圍

| 面向 | IFC | CityGML |
|------|-----|---------|
| 聚焦範圍 | 單體建築內部——構件級、樓層級、空間級 | 城市級——建築、街區、行政區、整座城市 |
| 覆蓋類型 | 建築、結構、MEP（機電）、管線 | 建築、道路、橋樑、隧道、鐵路、植被、水體、地形、城市家具 |
| 室內覆蓋 | 原生支援——房間、樓層、空間為核心概念 | 僅 LOD4 支援室內結構（層次較粗） |

值得注意的是，IFC4X3 已開始納入基礎設施（道路、鐵路、橋樑）的擴充；CityGML 3.0 也強化了對 BIM 室內空間及動態感測資料的支援，兩者正逐步收斂[^mdpi-2413]。

### 典型使用者

| IFC | CityGML |
|-----|---------|
| 建築師、結構工程師、機電工程師 | 都市規劃師、GIS 分析師 |
| 營造廠、施工管理團隊 | 智慧城市平台開發者 |
| 設施管理（FM）團隊 | 地籍管理、測量與國土調查單位 |
| BIM 協調員、工程顧問公司 | 政府城鄉發展部門 |

### 檔案格式

| 面向 | IFC | CityGML |
|------|-----|---------|
| 主要編碼 | IFC-SPF（.ifc 文字格式）、IFC-XML、IFC-ZIP | GML/XML（.gml/.xml）、CityJSON（.city.json） |
| 工具生態 | Revit、ArchiCAD、Tekla、IfcOpenShell、xBIM Toolkit | 3DCityDB、FME、citygml-tools、Cesium ion、Azul CityJSON |
| 標準版本 | IFC4X3（基礎設施擴充，2024） | CityGML 3.0（感測器支援、BIM 整合強化，2021） |

### 在數位孿生中的協同使用

在真實的**城市數位孿生（Urban Digital Twin）**專案中，IFC 與 CityGML 並非二選一的抉擇，而是**兩者並存、各司其職**：

```
BIM 軟體 (Revit / ArchiCAD)
   │
   ▼
IFC ──────► 保留建築構件原始資訊（主數據存檔）
   │
   ▼（轉換 / 語義映射）
CityGML ──► 城市級語義模型（GIS 管理、空間分析）
   │
   ▼（輕量化發布）
3D Tiles / glTF ──► Web 端三維渲染（Cesium / Unreal Engine）
```

實務建議[^abri][^mdpi-sensors]：

1. **IFC 作為源數據**：保留完整的 BIM 構件資訊，適合施工、維護、營運管理等需要精細構件屬性的場景
2. **CityGML 作為 GIS 層**：從 IFC 透過幾何概括（generalization）與語義映射（semantic mapping）轉換而來，適合城市級空間分析、規劃管理
3. **兩者轉換必然有損**：IFC → CityGML 會遺失構件材料等工程資訊；CityGML → IFC 則只能產出不含工程語義的表面外殼
4. **保留兩份數據**：IFC 作為交換與存檔格式，CityGML 作為 GIS 資料收存與管理格式

關鍵整合技術包括 IFC2CityGML 轉換工具（如 ifc2citygml GitHub 專案）、ifcOWL/CityGML 本體論搭配 BOT（Building Topology Ontology）進行語義對接，以及 3DCityDB 作為空間資料庫管理層[^ifc2citygml]。

---

## 結論

CityGML 與 IFC 反映了兩個不同領域（GIS 與 BIM）各自的資料抽象傳統：

- **IFC** 從**建築工程**出發，追求對單一建築物內每一個構件、材料與系統的精準描述，使用局部座標與實體幾何模型，覆蓋建築全生命週期
- **CityGML** 從**地理空間**出發，追求對城市層級各類物件的語義分類與地理定位，使用地理座標與邊界表面幾何，覆蓋多種城市地物類型

在數位孿生的脈絡下，兩者非競爭關係而是互補關係——**IFC 提供建築級的資料深度，CityGML 提供城市級的空間廣度**。將兩者透過語義映射技術整合，才能真正實現從建築內部到城市尺度的無縫數位孿生。

---

## 參考文獻

[^citygml-wiki]: Wikipedia. (n.d.). CityGML. Retrieved 2026-09-25, from https://en.wikipedia.org/wiki/CityGML
[^citygml-ogc]: Open Geospatial Consortium. (n.d.). CityGML Standard. Retrieved 2026-09-25, from https://www.ogc.org/standards/citygml/
[^ifc-wiki]: Wikipedia. (n.d.). Industry Foundation Classes. Retrieved 2026-09-25, from https://en.wikipedia.org/wiki/Industry_Foundation_Classes
[^ifc-buildingsmart]: buildingSMART International. (n.d.). Industry Foundation Classes (IFC). Retrieved 2026-09-25, from https://technical.buildingsmart.org/standards/ifc/
[^3dgeospatial]: 3D Geospatial. (n.d.). IFC vs CityGML for Building Twins. Retrieved 2026-09-25, from https://www.3d-geospatial.com/3d-geospatial-fundamentals-for-digital-twins/3d-format-standards-comparison/ifc-vs-citygml-for-building-twins/
[^gisyxs]: GIS 研習社. (n.d.). CityGML 與 IFC 的區別. Retrieved 2026-09-25, from https://www.gisyxs.com/p/1364.html
[^mdpi-2413]: MDPI. (2017). Path to an Integrated Modelling between IFC and CityGML. *ISPRS International Journal of Geo-Information*, 1(3), 25. Retrieved 2026-09-25, from https://www.mdpi.com/2413-8851/1/3/25
[^mdpi-sensors]: MDPI. (2024). Digital Twin Smart City: Integrating IFC and CityGML. *Sensors*, 24(12), 3761. Retrieved 2026-09-25, from https://www.mdpi.com/1424-8220/24/12/3761
[^abri]: 內政部建築研究所（台灣）. (n.d.). IFC 與 CityGML 資訊交換研究. Retrieved 2026-09-25, from https://www.abri.gov.tw/News_Content_Table.aspx?n=807&s=39536
[^ifc2citygml]: idibau. (n.d.). IFC2CityGML — GitHub Repository. Retrieved 2026-09-25, from https://github.com/idibau/ifc2citygml