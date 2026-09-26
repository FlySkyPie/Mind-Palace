# IndoorGML 的競爭標準與替代格式及其生態系成熟度調查

## 概要

IndoorGML 是由 Open Geospatial Consortium（OGC）制定的室內空間導航開放標準，自 2014 年發布以來，在室內導航的語意與拓樸模型領域有其獨特定位。然而，其生態系規模與業界採用程度遠不及多個競爭或替代標準。本報告系統性調查 IndoorGML 在室內空間資料領域的競爭者與替代方案，並評估各自生態系的規模與成熟度。

---

## 1. 研究範圍與定義

### 1.1 所謂「競爭者」

本報告將 IndoorGML 的競爭者與替代方案分為兩大類：

- **直接競爭者**：同樣專注於室內空間導航、定位或地圖的標準／格式
- **間接替代者**：非專為室內導航設計，但能承載室內空間資料且擁有更大生態系的標準／格式

### 1.2 生態系成熟度評估維度

評估標準包括：標準發布年份、業界採用程度、商業軟體支援數量、政府或大型機構強制採用、工具鏈完整度、開發者社群活躍度。

---

## 2. 直接競爭者：室內導航／室內地圖專用標準與格式

### 2.1 Apple Indoor Mapping Data Format（IMDF）

Apple 於 2021 年 10 月發布的室內地圖資料格式，以 GeoJSON（RFC 7946）為基礎，定義了 Venue、Building、Level、Unit、Opening、Amenity、Anchor 等室內空間要素。[^apple-imdf]

| 面向 | 說明 |
|------|------|
| 標準體 | Apple（專屬規格，但規格文件對外公開） |
| 格式 | GeoJSON（輕量 JSON） |
| 導航模型 | 定位 + 地圖展示，無正式拓樸圖 |
| 室內語意 | 豐富（空間類型、設施、出入口） |
| 生態系成熟度 | **中等且成長中** |
| 真實部署 | 全球各大機場、購物中心、體育場（Apple Maps） |

**生態系評估**：IMDF 是唯一直接與 IndoorGML 在室內導航領域競爭的**已量產**標準。它比 IndoorGML 簡單許多（JSON vs GML/XML），且已實際部署於數百個大型場域。但缺點是鎖定 Apple 生態系，規格雖公開但由 Apple 單方控制，非開放標準。[^apple-maps-venues]

### 2.2 Google Indoor Maps（專屬格式）

Google 自 2011 年啟動室內地圖計畫，涵蓋機場、博物館、購物中心、大型零售店、大學與交通站。場域業主透過 Google Maps Connect 提交平面圖，Google 內部處理後於 Google Maps 上顯示。[^google-indoor]

| 面向 | 說明 |
|------|------|
| 標準體 | Google（完全專屬） |
| 格式 | 內部封閉格式 |
| 導航模型 | 定位 + 地圖展示 |
| 生態系成熟度 | **高（涵蓋數千個場域）** |
| 真實部署 | 全球 Google Maps 使用者 |

**生態系評估**：Google Indoor Maps 擁有極高的消費者觸及面，但完全封閉，無公開規格、無匯出格式、無第三方工具鏈。對開放標準而言是競爭者而非合作對象。

### 2.3 HERE Indoor Mapping

HERE Technologies（原 Nokia/Navteq）提供室內定位與地圖平台，主要鎖定企業與汽車領域。[^here-indoor]

| 面向 | 說明 |
|------|------|
| 標準體 | HERE（專屬） |
| 生態系成熟度 | **中低**（企業領域） |
| 真實部署 | 部分機場與企業場域 |

**生態系評估**：HERE 的室內方案以定位 SDK 為核心，非開放資料格式，影響範圍有限。

### 2.4 Mapbox Indoor

Mapbox 提供室內地圖功能，使用 Mapbox Streets／vector tiles 搭配自訂屬性，無公開的室內專屬資料模型。[^mapbox-indoor]

**生態系評估**：Mapbox 的開發者工具（GL JS、Mobile SDK）品質優良，但室內地圖僅為其平台功能之一，非獨立標準，影響力有限。

### 2.5 OpenStreetMap Simple Indoor Tagging

OSM 社群自 2014 年建立的室內空間標記慣例，使用 `indoor=room`、`indoor=corridor`、`level=*`、`door=*` 等標籤描述室內空間，與 Simple 3D Buildings（S3DB）相容。[^osm-indoor-tagging]

| 面向 | 說明 |
|------|------|
| 標準體 | OSM 社群（共識驅動） |
| 格式 | OSM 標籤（key=value） |
| 導航模型 | 無原生導航圖，可自訂路由 |
| 生態系成熟度 | **中高**（OSM 生態系內） |
| 真實部署 | 全球各地志願者繪製的室內地圖 |

**生態系評估**：OSM Simple Indoor Tagging 是最開放的室內資料方案，所有資料開源、可自由下載，但導航模型需要自行實作。室內資料覆蓋不均，品質依賴志願者貢獻。

---

## 3. 間接替代者：可承載室內空間資料的大型開放標準

### 3.1 Industry Foundation Classes（IFC，ISO 16739）

buildingSMART 開發的建築資訊模型（BIM）開放標準，1994 年起發展，現為 ISO 16739-1:2024。IFC 定義了數百種實體（IfcWall、IfcSpace、IfcDoor 等），涵蓋建築全生命週期資料。[^ifc-standard]

| 面向 | 說明 |
|------|------|
| 標準體 | buildingSMART → ISO |
| 格式 | IFC-SPF、IFC-XML、ifcJSON、ifcOWL 等 |
| 導航模型 | 無內建導航圖 |
| 室內空間 | 透過 IfcSpace、IfcBuildingStorey 表達 |
| 生態系成熟度 | **極高（全球 BIM 產業標準）** |
| 商業軟體支援 | Revit、Archicad、Tekla、Bentley、Vectorworks、FreeCAD 等 |
| 政府強制採用 | 丹麥（2010）、芬蘭（2017）、英國（2016）、新加坡、挪威、韓國、日本（2023）等約 15-20 國[^ifc-mandates] |
| 認證機制 | buildingSMART IFC 認證計畫（匯出 €2,500／匯入 €1,500） |
| 發展時程 | 30+ 年 |

**生態系評估**：IFC 是所有競爭標準中生態系最龐大、最成熟者。政府強制採用遍及歐洲與亞洲，建築產業幾乎無可迴避。然而 IFC 的設計目標是建築構造與營運，而非室內導航——它缺乏 IndoorGML 的 Node-Relation Graph（NRG）與 Multi-Layered Space Model（MLSM）。實務上 IFC 常作為 IndoorGML 的幾何資料來源。[^ifc-certification]

### 3.2 CityGML（OGC 標準）

OGC 的 3D 城市模型標準，v3.0（2019+）已全面支援室內空間。CityGML 3.0 放棄了舊版的 LoD4（專屬室內細節層級），改為讓所有 LoD0-LoD3 的建築物特徵類型都能包含室內元素。[^citygml-3]

| 面向 | 說明 |
|------|------|
| 標準體 | OGC |
| 格式 | GML/XML、CityJSON |
| 導航模型 | 無內建導航圖 |
| 室內空間 | LoD0-LoD3 皆可包含室內元素 |
| 生態系成熟度 | **高** |
| 主要工具 | 3DCityDB（PostgreSQL）、CesiumJS、QGIS、FME、azul |
| 真實部署 | 德國巴伐利亞州（約 800 萬棟建物模型）、柏林、倫敦、紐約、鹿特丹、新加坡等數百個城市[^citygml-adoption] |

**生態系評估**：CityGML 擁有成熟的工具鏈與廣泛的城市採用，LoD4 雖已取消但室內元素可分布於各 LoD。與 IndoorGML 的關係是互補而非取代——IndoorGML 本身設計上即可透過外部參考（External Reference）連結 CityGML 的幾何資料。

### 3.3 GeoJSON（IETF RFC 7946）

IETF 標準的地理特徵幾何格式，2016 年發布。支援 Point、LineString、Polygon 等幾何型別，是最通用的網頁地理資料格式。[^geojson]

| 面向 | 說明 |
|------|------|
| 標準體 | IETF |
| 格式 | JSON |
| 室內語意 | 無（僅幾何） |
| 生態系成熟度 | **極高（網頁 GIS 的事實標準）** |
| 採用 | Leaflet、Mapbox GL、OpenLayers、PostGIS、MongoDB 等全面支援 |

**生態系評估**：GeoJSON 本身非室內專用，但其極高的採用率與簡單的 JSON 格式，使其成為許多室內地圖實作的底層格式。Apple IMDF 即建構在 GeoJSON 之上。

### 3.4 3D Tiles（OGC 社群標準）

Cesium 創建、現由 Bentley 維護的大型 3D 地理空間資料串流標準，基於 glTF。2018 年成為 OGC 社群標準，v1.1 已發布，v2.0 開發中。[^3d-tiles]

| 面向 | 說明 |
|------|------|
| 標準體 | OGC 社群標準 |
| 格式 | glTF + tile 集 |
| 室內導航 | 僅視覺呈現 |
| 生態系成熟度 | **高且快速成長** |
| GitHub | 2,600+ stars、502 forks、1,719 commits[^3d-tiles-github] |
| 主要工具 | CesiumJS、Unreal Engine、Three.js、QGIS |

**生態系評估**：3D Tiles 專注於大規模 3D 視覺化與串流，非室內導航模型，但在數位雙胞胎（Digital Twin）領域快速成長，許多室內場景透過 3D Tiles 展示。

### 3.5 KML（OGC 標準，原 Google）

原 Keyhole/Google Earth 的 3D 地理視覺化格式，2008 年成為 OGC 標準。[^kml]

**生態系評估**：極高採用率（Google Earth 效應），但僅限於視覺化註記，無室內語意模型。

### 3.6 Green Building XML（gbXML）

開放式 BIM 轉能源分析資料交換格式，由 Autodesk 發起，約 55-60 個軟體產品整合支援，主要用於能源模擬。[^gbxml]

**生態系評估**：在建築能源分析領域有穩固地位，但用途與 IndoorGML 無重疊。

### 3.7 COBie（Construction Operations Building Information Exchange）

美國與英國政府強制使用的設施管理交接資料規範，以試算表格式為核心。[^cobie]

**生態系評估**：有政府強制採用（美、英），正被 buildingSMART ISO 級 FM Handover 標準取代。與室內導航無關。

---

## 4. 生態系比較矩陣

| 標準／格式 | 類型 | 室內專用 | 導航模型 | 生態系規模 | 商業軟體支援 | 政府強制 | 真實量產 |
|-----------|------|---------|---------|-----------|------------|---------|---------|
| **IndoorGML** | OGC | ✅ 是 | ✅ NRG+MLSM | ⭐⭐ | FME（唯一） | 無 | 極少 |
| **Apple IMDF** | 專屬（規格公開） | ✅ 是 | ⚠️ 簡易 | ⭐⭐⭐ | Apple 生態系 | 無 | ✅ 全球場域 |
| **Google Indoor Maps** | 專屬封閉 | ✅ 是 | ⚠️ 簡易 | ⭐⭐⭐⭐ | Google Maps | 無 | ✅ 全球場域 |
| **OSM Indoor Tagging** | 社群 | ✅ 是 | 可自訂 | ⭐⭐⭐ | OSM 工具鏈 | 無 | ⚠️ 志願者貢獻 |
| **IFC** | ISO/buildingSMART | ❌ 非專用 | ❌ 無 | ⭐⭐⭐⭐⭐ | Revit 等數十套 | 15-20 國 | ✅ 全球 BIM |
| **CityGML** | OGC | ❌ 非專用 | ❌ 無 | ⭐⭐⭐⭐ | 3DCityDB、FME、Cesium | 部分國家 | ✅ 數百城市 |
| **GeoJSON** | IETF | ❌ 非專用 | ❌ 無 | ⭐⭐⭐⭐⭐ | 全面支援 | 無 | ✅ 通用格式 |
| **3D Tiles** | OGC 社群 | ❌ 非專用 | ❌ 無 | ⭐⭐⭐⭐ | Cesium、Unreal、QGIS | 無 | ✅ 數位雙胞胎 |
| **gbXML** | 開放（Autodesk） | ❌ 非專用 | ❌ 無 | ⭐⭐⭐ | 55+ 工具 | 無 | ✅ 能源分析 |
| **COBie** | buildingSMART | ❌ 非專用 | ❌ 無 | ⭐⭐⭐ | BIM 工具 + 試算表 | 英、美 | ✅ 設施管理 |
| **KML** | OGC | ❌ 非專用 | ❌ 無 | ⭐⭐⭐⭐⭐ | Google Earth、Cesium | 無 | ✅ 視覺化 |

---

## 5. 關鍵發現

### 5.1 無標準與 IndoorGML 完全重疊

IndoorGML 的核心獨特價值——**Node-Relation Graph（NRG，基於龐加萊對偶的拓樸圖）** 與 **Multi-Layered Space Model（MLSM，多層空間模型，可疊加 Wi-Fi、RFID、感測器不同語意層）**——沒有任何其他標準提供等效功能。這既是它的護城河，也是它生態系小的原因：學習曲線陡峭、實作複雜、需求僅限於室內導航且有感測器整合需求的進階應用。

### 5.2 生態系最大者：IFC

IFC 是整體生態系最大、最成熟的標準。關鍵指標：
- 發展 30+ 年，ISO 國際標準
- 全球 BIM 軟體全面支援（數十套商業軟體）
- 約 15-20 國政府強制或建議採用
- 專屬認證計畫

但它不提供 IndoorGML 級的導航模型。實務中常見的 pipeline 是 IFC → IndoorGML 轉換（學術界大量研究）。

### 5.3 已量產的直接競爭者：Apple IMDF

IMDF 是目前唯一在真實場域大規模部署的室內專用開放規格。相較 IndoorGML 的優勢：
- **格式輕量**：JSON vs GML/XML
- **實作簡單**：對開發者友善
- **已量產**：Apple Maps 中數百場域
- **延伸性**：自訂特徵擴充模型

劣勢：鎖定 Apple 生態系，非開放標準（Apple 單方控制）。

### 5.4 室內導航的「事實標準」：無

室內導航領域目前處於**標準碎片化**狀態：
- 消費者端：Google Maps／Apple Maps 壟斷，但格式完全封閉
- 專業 BIM/GIS 端：IFC 與 CityGML 主導室內幾何資料，但導航模型各自為政
- 學術研究：IndoorGML 是首選，但商業轉換困難
- 開源社群：OSM Simple Indoor Tagging 最有潛力，但導航路由需自行實作

### 5.5 IndoorGML 2.0 的簡化方向正確

IndoorGML 2.0 Part 1 概念模型已在 2024 年核准，主要方向為「**簡化資料模型**」，並將 Part 2 實作綱要擴充為 GML、SQL、JSON 三種格式。[^indoorgml-v2] 加入 JSON 編碼將大幅降低採用門檻，是縮小與 IMDF 差距的關鍵。

---

## 6. 結論

| 競爭者 | 與 IndoorGML 的關係 | 生態系比較 |
|--------|-------------------|-----------|
| **IFC** | 互補（IFC 供應室內幾何） | **遠大於** IndoorGML |
| **CityGML** | 互補（External Reference 設計） | **大於** IndoorGML |
| **Apple IMDF** | 直接競爭 | **大於** IndoorGML（已量產） |
| **Google Indoor Maps** | 直接競爭（封閉） | **大於** IndoorGML（消費者觸及） |
| **OSM Indoor Tagging** | 直接競爭（開源） | **約等於或略大於** IndoorGML |
| **GeoJSON** | 底層格式 | **遠大於** IndoorGML |
| **3D Tiles** | 間接替代（視覺化） | **大於** IndoorGML |
| **gbXML／COBie** | 領域不同 | 無法直接比較 |

IndoorGML 擁有無可取代的導航拓樸與多層空間模型，但生態系明顯小於多數競爭者。其突圍路徑取決於 IndoorGML 2.0 的 JSON/SQL 編碼是否能吸引更多開發者與商業工具採用，以及 OGC 是否能推動更多政府或產業聯盟強制採用。

---

[^apple-imdf]: Apple Inc. (2021). Indoor Mapping Data Format (IMDF) — Conforms to GeoJSON (RFC 7946). Retrieved 2026-09-25, from https://register.apple.com/resources/imdf/

[^apple-maps-venues]: Apple Inc. (n.d.). iOS Feature Availability — Maps Indoor Maps Airports. Retrieved 2026-09-25, from https://www.apple.com/ios/feature-availability/#maps-indoor-maps-airports

[^google-indoor]: Google LLC. (n.d.). Google Maps Partner Program — Indoor Maps. Retrieved 2026-09-25, from https://www.google.com/maps/about/partners/indoormaps/

[^here-indoor]: HERE Technologies. (n.d.). HERE Indoor Mapping. Retrieved 2026-09-25, from https://www.here.com/products/location-services/indoor-mapping

[^mapbox-indoor]: Mapbox Inc. (n.d.). Mapbox Indoor Mapping. Retrieved 2026-09-25, from https://www.mapbox.com/indoor

[^osm-indoor-tagging]: OpenStreetMap Wiki. (2026). Simple Indoor Tagging. Retrieved 2026-09-25, from https://wiki.openstreetmap.org/wiki/Simple_Indoor_Tagging

[^ifc-standard]: buildingSMART International. (n.d.). Industry Foundation Classes (IFC). Retrieved 2026-09-25, from https://www.buildingsmart.org/standards/bsi-standards/industry-foundation-classes/

[^ifc-mandates]: Wikipedia. (n.d.). Building Information Modeling — Government BIM Requirements. Retrieved 2026-09-25, from https://en.wikipedia.org/wiki/Building_information_modeling

[^ifc-certification]: buildingSMART International. (n.d.). IFC Software Certification. Retrieved 2026-09-25, from https://www.buildingsmart.org/compliance/software-certification/ifc/

[^citygml-3]: OGC. (2019). CityGML 3.0 Conceptual Model User Guide. Retrieved 2026-09-25, from https://docs.ogc.org/guides/20-066.html

[^citygml-adoption]: Wikipedia. (n.d.). 3D City Model. Retrieved 2026-09-25, from https://en.wikipedia.org/wiki/3D_city_model

[^geojson]: IETF. (2016). RFC 7946 — The GeoJSON Format. Retrieved 2026-09-25, from https://tools.ietf.org/html/rfc7946

[^3d-tiles]: OGC. (2022). OGC 3D Tiles v1.1 (Doc No. 22-025r4). Retrieved 2026-09-25, from https://www.ogc.org/standard/3dtiles/

[^3d-tiles-github]: CesiumGS. (n.d.). 3D Tiles Specification — GitHub Repository. Retrieved 2026-09-25, from https://github.com/CesiumGS/3d-tiles

[^kml]: OGC. (2015). OGC KML v2.3. Retrieved 2026-09-25, from https://www.ogc.org/standard/kml/

[^gbxml]: gbXML.org. (n.d.). Software Tools that Support gbXML. Retrieved 2026-09-25, from https://www.gbxml.org/Software_Tools_that_Support_GreenBuildingXML_gbXML

[^cobie]: buildingSMART International. (n.d.). COBie. Retrieved 2026-09-25, from https://www.buildingsmart.org/standards/bsi-standards/cobie/

[^indoorgml-v2]: OGC. (2024). IndoorGML 2.0 Part 1 — Conceptual Model (Doc No. 22-045r5). Retrieved 2026-09-25, from https://docs.ogc.org/is/22-045r5/22-045r5.html