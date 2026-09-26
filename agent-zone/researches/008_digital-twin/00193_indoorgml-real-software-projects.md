# IndoorGML 在真實軟體專案中的應用調查

> **研究問題**：IndoorGML 除了學術論文與 OGC（Open Geospatial Consortium）官方展示之外，是否曾被實際用於真實的軟體專案？

## 摘要

IndoorGML 是由 OGC 制定的室內空間資料標準，但自 2010 年代初期發展至今，其在商業軟體產品中的採用程度相當有限。本研究透過網路搜尋與 GitHub 分析，發現目前唯一內建原生 IndoorGML 讀寫支援的大型商業軟體為 **FME（Safe Software）**。其餘多數實作皆源自學術單位（尤其是韓國釜山大學的 STEMLab 與荷蘭台夫特理工大學），且多數工具屬於研究等級，缺乏大規模商業部署的證據。不過，仍有少數工具已達「實際可用」的階段，如 azul（App Store 上架）、VIM（實際部署於韓國地鐵站）及 Intratech 公司的 Revit/AutoCAD 商用外掛。

## 1. 背景

IndoorGML 是 OGC 於 2014 年發布的室內空間資料標準，用於描述建築物內部的空間結構、連通性與導航資訊[^ogc-standard]。截至 2026 年，最新版本為 IndoorGML 1.1（2020 年發布），IndoorGML 2.0 仍在開發中。

本研究旨在釐清：IndoorGML 是否已脫離學術研究與標準展示階段，進入了實際軟體產品的採用範疇。

## 2. 研究方法

- 使用 GitHub 關鍵字搜尋 `IndoorGML`，篩選具實質程式碼貢獻且非僅為學術論文的儲存庫
- 搜尋各大 GIS/BIM/導航軟體的官方文件，確認是否提供 IndoorGML 支援
- 交叉比對相關工具的 GitHub 星數、授權條款與開發活躍度
- 排除純學術論文與 OGC 官方展示專案

## 3. 主要發現

### 3.1 商業軟體支援

| 軟體 | IndoorGML 支援 | 說明 |
|------|---------------|------|
| **FME（Safe Software）** | ✅ 原生內建 | FME 是唯一完整支援 IndoorGML 讀寫的主流商業 GIS 資料整合平台，適用於 FME Form（桌面版）、FME Flow（伺服器版）與 FME Flow Hosted（雲端版），支援 Windows、Linux、Mac 全平台[^fme-doc] |
| **ArcGIS Indoors（Esri）** | ❌ 無 | Esri 採用自有封閉的 ArcGIS Indoors Information Model（地理資料庫格式），可匯入 Revit、IFC、CAD 與 PDF，但完全不支援 IndoorGML[^esri-arcgis] |
| **QGIS** | ❌ 無 | QGIS 外掛商店中「indoorgml」標籤下無任何已發布的外掛[^qgis-plugins] |
| **Autodesk Revit** | ❌ 無 | Revit 原生不支援 IndoorGML；其開放交換格式為 IFC[^revit-ifc] |
| **其他 BIM 軟體（ArchiCAD 等）** | ❌ 無 | 未發現任何 BIM 軟體內建 IndoorGML 支援 |

**結論**：在主流 GIS 與 BIM 軟體領域，IndoorGML 的商業採用極度有限，僅 **FME** 一家提供原生支援。

### 3.2 商用級外掛與工具

**Intratech Corp（韓國）** 是唯一已知開發 IndoorGML 商業產品的公司。該公司成立超過 30 年，專注於工程資料互通與 BIM 工作流程，旗下產品包括[^intratech]：

- **IndoorGML.Exporter.Revit**：Autodesk Revit 外掛，可將 Revit 2019 中的建築空間資料（邊界、空間、門-房間連通）匯出為 IndoorGML 格式
- **IndoorGML.Exporter.AutoCAD**：AutoCAD 專用匯出外掛
- **IndoorGML.Core**：C#/.NET 核心程式庫，支援 IndoorGML XML 的讀取、寫入與合併
- **IndoorGML.Adapter**：接收空間與連通資訊並匯出為 IndoorGML 格式

上述產品以 Apache 2.0 授權發布於 GitHub，附有完整的使用手冊（英文與韓文），文件日期為 2022 年 2 月。

### 3.3 開放原始碼工具生態系

#### 3.3.1 STEMLab（韓國釜山大學）

STEMLab 是 IndoorGML 開源工具的最大貢獻者，旗下工具皆以 MIT 或 LGPL 授權發布[^stemlab]：

| 工具 | 語言 | 星數 | 說明 |
|------|------|------|------|
| **InEditor** | JavaScript（Node.js） | ⭐33 | 基於 Web 的 IndoorGML 繪圖編輯器，支援導航、不可通行空間、樓層與 POI 擴充模組 |
| **InViewer** | JavaScript（Three.js/React） | ⭐26 | 基於 Three.js 的 3D IndoorGML 網頁檢視器 |
| **InViewer-Desktop** | C#（Unity3D） | ⭐16 | 基於 Unity3D 的桌面 IndoorGML 3D 檢視器 |
| **3DINV** | JavaScript（Cesium.js） | ⭐28 | 3D 室內導航檢視器，使用 IndoorGML 約束使用者在室內空間中的移動 |
| **InFactory** | Java（Spring） | ⭐18 | IndoorGML 1.0.3 文件的 RESTful 後端伺服器與程式庫，搭配 H2GIS 空間資料庫 |
| **API_IndoorFeatures** | Python（pygeoapi） | ⭐2 | IndoorGML 2.0 的 OGC API - Features 實作，使用 PostgreSQL/PostGIS、pgRouting，支援 IndoorJSON 編碼 |
| **VIM（Voice Indoor Maps）** | Android（Java） | ⭐5 | 視障者語音室內導航 Android 應用程式，使用 IndoorGML 1.0 Core 格式，曾於 **韓國東大邱站（동대구역）** 進行實地展示[^vim] |
| **indoorgml-to-imdf** | JavaScript | ⭐3 | IndoorGML 與 Apple IMDF（Indoor Mapping Data Format）之間的轉換器，以 npm 套件發布 |
| **InCOVID** | Python | ⭐5 | 使用 IndoorGML 資料進行室內空間病毒傳播模擬 |

#### 3.3.2 台夫特理工大學（TU Delft）

- **azul**：3D 城市模型檢視器，適用於 macOS 與 iOS，已於 **Apple App Store 上架**，原生支援 IndoorGML、CityGML、CityJSON 等格式，以 C++17/Swift 撰寫，使用 Metal 渲染引擎，GPLv3 授權[^azul]
- **val3dity**（⭐109）：根據 ISO19107 標準驗證 3D 基本幾何，包含 IndoorGML 支援[^val3dity]
- **IndoorJSON**：IndoorGML 2.0 概念的 JSON 編碼，仍為實驗性質，尚未建議用於正式產品[^indoorjson]

#### 3.3.3 其他專案

- **navground_ri**[^navground]：隸屬於歐盟 REXASI-PRO 地平線計畫（資助編號 101070028），是 Python/ROS 導航套件，可讀取 IndoorGML 地圖進行**自主輪椅路徑規劃**，支援最短路徑與最平穩路徑兩種模式
- **ifc2indoorgml**[^ifc2indoorgml]：UNSW（澳洲新南威爾斯大學）開發的 IFC 轉 IndoorGML 轉換工具（C++，⭐12）
- **IFC2BCM**[^ifc2bcm]：從 IFC/BIM 模型產生 IndoorGML 與 BCM（Building Configuration Model）的工具，應用於醫院等複雜建築的空間佈局有效性評估
- **IndoorGMLEditor**[^gmleditor]：Java/JavaFX 開發的桌面應用程式，提供 IndoorGML 檔案的圖形化編輯功能，支援拖曳編輯 CellSpace 座標
- **indoorjson-cpp**[^indoorjsoncpp]：C++14 的 IndoorJSON 序列化/反序列化程式庫，使用 GEOS 處理幾何，明確標示為實驗性質，**不建議用於正式產品**

### 3.4 實際部署案例

| 專案 | 部署場域 | 說明 |
|------|---------|------|
| **VIM** | 韓國東大邱站（동대구역） | 視障者語音導航 Android 應用，實際於韓國高鐵站進行展示 |
| **azul** | Apple App Store | 可於 Mac 與 iOS 裝置下載的 3D 城市模型檢視器，支援 IndoorGML 渲染 |
| **navground_ri** | 歐盟 REXASI-PRO 計畫 | 自主輪椅導航，使用 IndoorGML 作為室內地圖輸入格式 |
| **API_IndoorFeatures** | FOSS4G 2026 發表 | 基於 pygeoapi 的室內定位服務平台，提供 OGC API 標準介面 |
| **Lotte World Mall（韓國）** | 資料樣本 | 公開可用的 IndoorGML 室內地圖資料樣本（45.1 MB） |
| **Pusan National University 建築物** | 資料樣本 | 多棟建築物的 IndoorGML 資料集可用於測試 |

## 4. 分析與討論

### 4.1 採用瓶頸

IndoorGML 在商業軟體中的採用之所以有限，可能原因如下：

1. **標準複雜度**：IndoorGML 基於 GML（Geography Markup Language）的 XML 語法，較為複雜，缺乏 JSON 等輕量替代編碼直到 IndoorGML 2.0 才開始推廣
2. **前有強勢競爭**：Apple 主導的 IMDF（Indoor Mapping Data Format）在室內導航領域更具實務優勢，Google 則採用自有封閉格式
3. **缺乏大廠背書**：Esri、Autodesk 等 GIS/BIM 龍頭皆未內建支援，導致生態系難以擴大
4. **學術色彩過重**：多數工具由學術實驗室開發，維護能量有限，缺乏長期商業支援承諾

### 4.2 潛在利基領域

儘管整體採用有限，IndoorGML 在以下領域仍具潛在應用價值：

- **BIM 與 GIS 互通**：透過 Intratech 外掛與 ifc2indoorgml 等工具，IndoorGML 可作為 BIM 模型與 GIS 之間的橋樑
- **室內導航無障礙**：VIM 專案證明了 IndoorGML 在視障者導航上的實用性
- **自主機器人導航**：navground_ri 顯示 IndoorGML 適用於輪椅與機器人室內路徑規劃
- **災害應變**：透過 IFC2BCM 等工具，IndoorGML 可協助醫院等設施的空間管理

## 5. 結論

IndoorGML 在真實軟體專案中的採用情況可歸納為：

- **已進入商業領域**：FME（Safe Software）內建原生支援，Intratech Corp 提供 Revit/AutoCAD 商用外掛
- **已實際部署**：azul（App Store 上架）、VIM（韓國地鐵站展示）、navground_ri（歐盟機器人計畫）
- **多數仍屬研究等級**：STEMLab 生態系的 InEditor/InFactory/InViewer 等工具雖具實用性，但缺乏長期商業維護承諾
- **主流軟體未支援**：ArcGIS Indoors、QGIS、Autodesk Revit 等皆未內建 IndoorGML 讀寫能力

**總體而言，IndoorGML 尚未達到廣泛商業採用的階段，但其在特定利基領域（BIM 互通、無障礙導航、機器人導航）已展現實際應用價值。**

---

[^ogc-standard]: Open Geospatial Consortium. (n.d.). OGC IndoorGML Standard. Retrieved 2026-09-25, from https://www.ogc.org/standard/indoorgml/

[^fme-doc]: Safe Software. (n.d.). FME IndoorGML Reader/Writer Documentation. Retrieved 2026-09-25, from https://docs.safe.com/fme/html/FME-Form-Documentation/FME-ReadersWriters/indoorgml/indoorgml.htm

[^esri-arcgis]: Esri. (n.d.). ArcGIS Indoors Information Model. Retrieved 2026-09-25, from https://pro.arcgis.com/en/pro-app/latest/help/data/indoors/arcgis-indoors-information-model.htm

[^qgis-plugins]: QGIS. (n.d.). QGIS Plugins tagged "indoorgml". Retrieved 2026-09-25, from https://plugins.qgis.org/plugins/tags/indoorgml/

[^revit-ifc]: Autodesk. (n.d.). Revit IFC Export. Retrieved 2026-09-25, from https://help.autodesk.com/view/RVT/2026/ENU/?guid=IFC_Export_html

[^intratech]: Intratech Corp. (2022). KICT-Autocad-RevitToIndoorGML: Export Revit/AutoCAD Spatial to IndoorGML. Retrieved 2026-09-25, from https://github.com/intratech/KICT-Autocad-RevitToIndoorGML

[^stemlab]: STEMLab (Pusan National University). (n.d.). GitHub repositories for IndoorGML tools. Retrieved 2026-09-25, from https://github.com/STEMLab

[^vim]: STEMLab. (n.d.). VIM (Voice Indoor Maps) Application for Android. Retrieved 2026-09-25, from https://github.com/STEMLab/VIM

[^azul]: TU Delft 3D Geoinformation. (n.d.). azul: 3D city model viewer for macOS and iOS. Retrieved 2026-09-25, from https://github.com/tudelft3d/azul

[^val3dity]: TU Delft 3D Geoinformation. (n.d.). val3dity: validation of 3D primitives according to ISO19107. Retrieved 2026-09-25, from https://github.com/tudelft3d/val3dity

[^indoorjson]: TU Delft 3D Geoinformation. (n.d.). IndoorJSON: IndoorGML concept in JSON. Retrieved 2026-09-25, from https://github.com/tudelft3d/indoorjson

[^navground]: IDSIA Robotics. (n.d.). navground_ri: Navground interface to the robot indoor path planner. Retrieved 2026-09-25, from https://github.com/idsia-robotics/navground_ri

[^ifc2indoorgml]: UNSW Grid. (n.d.). ifc2indoorgml: IFC to IndoorGML converter. Retrieved 2026-09-25, from https://github.com/grid-unsw/ifc2indoorgml

[^ifc2bcm]: ScienceDirect. (2024). IFC2BCM: A software tool for generating IndoorGML and BCM from IFC. Retrieved 2026-09-25, from https://www.sciencedirect.com/science/article/pii/S2352711024003455

[^gmleditor]: fox-1942. (n.d.). IndoorGMLEditor: IndoorGML Editor for Java/JavaFX. Retrieved 2026-09-25, from https://github.com/fox-1942/IndoorGMLEditor

[^indoorjsoncpp]: IndoorJson. (n.d.). indoorjson-cpp: C++14 serialization/deserialization library for IndoorJSON. Retrieved 2026-09-25, from https://github.com/IndoorJson/indoorjson-cpp