# FOSS 軟體編輯 CityGML 檔案調查報告

## 摘要

CityGML 是 OGC (Open Geospatial Consortium) 制定的 3D 都市模型資料交換標準格式[^ogc-citygml]，廣泛用於智慧城市、都市規劃、能源模擬等領域。本報告系統性調查現有自由開源軟體 (FOSS) 中能夠編輯、檢視、轉換或處理 CityGML 檔案的解決方案，並依使用場景分類整理。

## 編輯與建模工具

### Blender + VCS 3DCityDB Importer/Exporter 附加元件

Blender 是知名的開源 3D 建模軟體[^blender]，而 `virtualcitySYSTEMS` 開發的 CityGML 匯入/匯出附加元件使其成為目前最強大的 FOSS CityGML 編輯方案[^vcs-addon]。此工具支援：

- CityGML 2.0 與 3.0 格式的雙向匯入匯出
- **ModelTyper**：根據幾何形狀自動分類語義表面（如 WallSurface、RoofSurface、GroundSurface 等）
- **Openings Cutter**：在牆面上建立門窗開口（CityGML Opening）
- **Assign Object Part**：建立巢狀建築結構（BuildingPart、BuildingInstallation）
- 紋理材質與 UV 座標保留
- 基於邊界框、GML-ID、LOD（0–4）、特徵類型的篩選
- XSD 驗證、大檔案串流讀取、CLI 無頭模式
- 直接對接 3DCityDB 資料庫（PostgreSQL/PostGIS）

授權條款：MIT。支援 Windows、Linux、macOS。

### io_cityGML_basic（Blender 附加元件，簡化版）

較簡易的 Blender 附加元件，僅匯入 CityGML 幾何（無語義編輯），所有幾何會合併為單一物件[^io-citygml-basic]。適合純視覺化用途，不適合需要保留語義資訊的編輯工作。

授權條款：GPL。

### GEORES（SketchUp 附加元件）

免費開源的 SketchUp 附加元件，支援 CityGML 匯入/匯出與註解功能[^geores]。授權條款：MIT。

## 資料庫管理方案

### 3D City Database (3DCityDB)

3DCityDB 是目前最完整的 CityGML 開源地理資料庫解決方案[^3dcitydb]，建構於 PostgreSQL/PostGIS 或 Oracle Spatial 之上。支援 CityGML 1.0、2.0、3.0，包含：

- **3DCityDB Importer/Exporter**：Java 圖形化工具，匯入/匯出 CityGML 資料
- **citydb-tool**：v5 新增的 CLI 命令列工具
- 完整實現 CityGML 3.0 概念模型
- 匯出為 glTF、KML/COLLADA、CityJSON
- 全球超過 21 個國家、65 個地區使用，管理超過 2.1 億棟建築模型

授權條款：Apache 2.0。

### GeoRocket

高效能雲端地理空間資料儲存系統[^georocket]，支援 Elasticsearch、MongoDB、Amazon S3 後端。Schema 無關且保持原始格式，可原樣儲存 CityGML 檔案。適合大規模 CityGML 資料集的雲端管理。

## 命令列批次處理工具

### citygml-tools

基於 citygml4j 的強大命令列工具組，支援多種批次處理操作[^citygml-tools]：

- `validate`：依據 CityGML XML Schema 驗證
- `stats`：產生檔案統計資訊
- `apply-xslt`：以 XSLT 轉換都市物件
- `change-height`：調整高程偏移
- `remove-apps` / `to-local-apps`：操作外觀紋理
- `clip-textures`：將紋理裁切至表面範圍
- `merge`：合併多個檔案
- `subset`：建立子集合
- `filter-lods`：篩選 LOD 層級
- `reproject`：重新投影座標參考系統
- `from-cityjson` / `to-cityjson`：CityGML/CityJSON 互轉
- `upgrade`：CityGML 1.0/2.0 升級至 3.0

支援 CityGML 1.0、2.0、3.0 以及 CityJSON 1.0、1.1、2.0。需求 Java 17+。

授權條款：Apache 2.0。

### citygml4j（函式庫）

CityGML 應用的 Java 核心函式庫，提供讀取、寫入、處理 CityGML 資料的 API[^citygml4j]。citygml-tools 及其他許多工具皆以此為基礎。

### libcitygml（C++ 函式庫）

輕量級 C++ CityGML 解析函式庫，搭配 XercesC XML 解析器[^libcitygml]，解析時同時進行幾何三角化。附帶 OpenSceneGraph 外掛及 `citygml2vrml` 轉換工具。適合 3D 渲染應用。

授權條款：LGPL-2.1。

## 檢視工具

### FZKViewer / KITModelViewer

由德國卡爾斯魯厄理工學院 (KIT) 開發的 CityGML、IFC、BIM/GIS 資料檢視器[^kitviewer]，支援：

- 3D 模型顯示、導航、互動
- 詳細屬性/關係顯示
- Python API 與外掛 SDK
- 基於屬性的彩色編碼檢視
- BIM 與 GIS 整合
- OpenStreetMap 與 OGC Web Services 支援

### CityGMLViewer（HFT Stuttgart）

以 OpenGL 為基礎的 CityGML 檢視器，可快速顯示大型檔案[^hft-viewer]。提供桌面版（Java 17）與瀏覽器版（JavaScript，不需上傳資料）。

授權條款：AGPLv3。

### azul（macOS/iOS 檢視器）

由荷蘭代爾夫特理工大學 (TU Delft) 開發的 3D 都市模型檢視器[^azul]，支援多檔案載入、物件選取、屬性瀏覽、可見性切換。支援 CityGML 1.0/2.0、CityJSON、IndoorGML。

授權條款：GPLv3。支援 macOS 13+（Apple Silicon 與 Intel）及 iOS。

### 3DCityDB Web Map Client

基於 Cesium 的 3D 網頁檢視器與 JavaScript API[^webmap-client]，用於視覺化儲存在 3DCityDB 中的 CityGML 都市模型。授權條款：Apache 2.0。

## 驗證與品質保證工具

### val3dity

依據 ISO19107 標準驗證 3D 幾何圖元的正確性，被稱為「3D 版本的 ST_IsValid」[^val3dity]。支援 MultiSurface、CompositeSurface、Solid、MultiSolid、CompositeSolid。也驗證 CityJSON 與 IndoorGML 特定規則。提供網頁版上傳驗證。

授權條款：GPLv3。

### CityDoctor (CityDoctor2)

3D 都市模型的驗證與自動修復工具[^citydoctor]，可檢查並修復語法、幾何、語義問題，支援自訂驗證方案以滿足特定模擬需求。由 HFT Stuttgart 開發。

## GIS 整合方案

### 3DCityDB Tools for QGIS

QGIS 外掛，連接 3DCityDB 資料庫[^qgis-plugin]，將 CityGML 資料載入為 GIS 圖層進行 2D/3D 視覺化。支援屬性編輯並直接回存資料庫。針對 3DCityDB v4.x。

## 生成與轉換工具

### 3dfier

將 2D GIS 資料集（如地形圖）以點雲（LAS/LAZ）高程資訊「3D 化」為語義 3D 模型[^3dfier]，輸出 CityGML（建築物轉為 LOD1 方塊、水域轉為水平多邊形、道路等）。

### Random3Dcity

Python 程序化建模引擎，生成合成建築物及其他都市特徵的 CityGML（LOD 0–4）[^random3dcity]，適用於測試資料集產生。TU Delft 開發。

### osm2citygml

從 OpenStreetMap（經由 Overpass API）提取建築資料，轉為 CityGML 格式[^osm2citygml]。

### CityGML2OBJs

語義感知的 CityGML 轉 OBJ 工具[^citygml2objs]，支援物件分離與屬性轉換為顏色。TU Delft 開發。

### CityGML2X / CityGML2XCLI

Java 函式庫與命令列工具，將 CityGML 轉換為 X3D 格式[^citygml2x]。

### BIMserver

開源 BIM 伺服器，支援 IFC 轉 CityGML 匯出（含 GeoBIM ADE）[^bimserver]。

## 總結與建議

| 使用場景 | 推薦 FOSS 工具 |
|---------|--------------|
| 完整語義 3D 編輯 | Blender + VCS 3DCityDB Importer/Exporter |
| 資料庫管理 | 3DCityDB |
| 批次處理 / CLI | citygml-tools |
| 驗證 | val3dity + CityDoctor |
| GIS 分析 | QGIS + 3DCityDB Tools 外掛 |
| 快速檢視 (跨平台) | KITModelViewer、CityGMLViewer |
| 快速檢視 (macOS) | azul |
| 格式轉換 | citygml-tools、CityGML2OBJs、CityGML2X |
| 2D 生成 3D | 3dfier |
| 程序化生成 | Random3Dcity |
| 雲端儲存 | GeoRocket |
| 開發函式庫 | citygml4j (Java)、libcitygml (C++) |

---

[^ogc-citygml]: Open Geospatial Consortium. (n.d.). CityGML. Retrieved 2026-09-25, from https://www.ogc.org/standard/citygml/
[^blender]: Blender Foundation. (n.d.). Blender. Retrieved 2026-09-25, from https://www.blender.org/
[^vcs-addon]: virtualcitySYSTEMS. (n.d.). Blender CityGML Importer/Exporter. Retrieved 2026-09-25, from https://github.com/virtualcitySYSTEMS/blender-citygml-importer-exporter
[^io-citygml-basic]: Blender Add-ons. (n.d.). io_cityGML_basic. Retrieved 2026-09-25, from https://blender-addons.org/io_citygml_basic/
[^geores]: GeoplexGIS. (n.d.). GEORES. Retrieved 2026-09-25, from https://github.com/GeoplexGIS/geores
[^3dcitydb]: 3DCityDB. (n.d.). 3D City Database. Retrieved 2026-09-25, from https://github.com/3dcitydb
[^georocket]: GeoRocket. (n.d.). GeoRocket. Retrieved 2026-09-25, from https://georocket.io/
[^citygml-tools]: citygml4j. (n.d.). citygml-tools. Retrieved 2026-09-25, from https://github.com/citygml4j/citygml-tools
[^citygml4j]: citygml4j. (n.d.). citygml4j. Retrieved 2026-09-25, from https://github.com/citygml4j/citygml4j
[^libcitygml]: Klimke, J. (n.d.). libcitygml. Retrieved 2026-09-25, from https://github.com/jklimke/libcitygml
[^kitviewer]: Karlsruhe Institute of Technology, Institute for Automation and Applied Informatics. (n.d.). KITModelViewer. Retrieved 2026-09-25, from https://github.com/KIT-IAI/SDM_KITModelViewer
[^hft-viewer]: HFT Stuttgart. (n.d.). CityGML Viewer. Retrieved 2026-09-25, from http://simstadt.hft-stuttgart.de/related-softwares/city-gml-viewer/
[^azul]: TU Delft 3D Geoinformation Group. (n.d.). azul. Retrieved 2026-09-25, from https://github.com/tudelft3d/azul
[^webmap-client]: 3DCityDB. (n.d.). 3DCityDB Web Map Client. Retrieved 2026-09-25, from https://github.com/3dcitydb/3dcitydb-web-map
[^val3dity]: TU Delft 3D Geoinformation Group. (n.d.). val3dity. Retrieved 2026-09-25, from https://github.com/tudelft3d/val3dity
[^citydoctor]: HFT Stuttgart. (n.d.). CityDoctor. Retrieved 2026-09-25, from https://transfer.hft-stuttgart.de/pages/citydoctor/citydoctorhomepage/en/
[^qgis-plugin]: TU Delft 3D Geoinformation Group. (n.d.). 3DCityDB Tools for QGIS. Retrieved 2026-09-25, from https://plugins.qgis.org/plugins/citydb-tools/
[^3dfier]: TU Delft 3D Geoinformation Group. (n.d.). 3dfier. Retrieved 2026-09-25, from https://github.com/tudelft3d/3dfier
[^random3dcity]: TU Delft 3D Geoinformation Group. (n.d.). Random3Dcity. Retrieved 2026-09-25, from https://github.com/tudelft3d/Random3Dcity
[^osm2citygml]: Lee, C. (n.d.). osm2citygml. Retrieved 2026-09-25, from https://github.com/cuulee/osm2citygml
[^citygml2objs]: TU Delft 3D Geoinformation Group. (n.d.). CityGML2OBJs. Retrieved 2026-09-25, from https://github.com/tudelft3d/CityGML2OBJs
[^citygml2x]: 900k. (n.d.). CityGML2X. Retrieved 2026-09-25, from https://github.com/900k/CityGML2X
[^bimserver]: opensourceBIM. (n.d.). BIMserver. Retrieved 2026-09-25, from https://github.com/opensourceBIM/BIMserver