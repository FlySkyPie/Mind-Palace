# CityGML 與 CityJSON 的差異

## 概述

CityGML 與 CityJSON 都是用於儲存和交換三維城市模型的開放格式。兩者共享相同的概念資料模型（三維城市物件、語意表面、細節層級等），但在**編碼方式**上有根本差異。簡而言之，CityJSON 是 CityGML 資料模型的 **JSON 編碼實作**，而非競爭對手。[^cityjson-about]

---

## 背景與起源

### CityGML

- 由德國 **SIG 3D**（Special Interest Group 3D）開發
- 2008 年成為 **OGC（Open Geospatial Consortium）正式標準**（v1.0），2012 年 v2.0，2021 年 v3.0
- 定義了**概念資料模型（UML）** + **GML/XML 編碼**
- 支援完整的城市物件分類：建築物、道路、橋樑、植被、水體、地形、隧道、城市傢俱等
- 支援 **LoD 0~4** 五種細節層級[^ogc-citygml]

### CityJSON

- 由 **TU Delft**（荷蘭代爾夫特理工大學）Hugo Ledoux 等人於 2017 年開始開發
- v1.0 於 2019 年發佈，v1.1 於 2022 年，**v2.0 於 2023 年成為 OGC Community Standard**
- 目標：解決 CityGML 的 GML/XML 編碼「冗長、複雜、不適合網路」的問題
- 提供**輕量、開發者友善、Web 原生**的替代編碼[^cityjson-paper]

---

## 核心差異比較

| 面向 | CityGML | CityJSON |
|---|---|---|
| **檔案格式** | XML (GML Application Schema) | JSON |
| **副檔名** | `.gml` 或 `.xml` | `.city.json` |
| **標準地位** | OGC 正式標準（Adopted Standard） | OGC 社群標準（Community Standard） |
| **可讀性** | 差 — 深層巢狀、命名空間、XLink | 佳 — 扁平結構、簡潔的鍵值對 |
| **解析難度** | 高 — 需 XML + GML + CityGML 專屬邏輯 | 低 — 任何語言的標準 JSON 程式庫即可 |
| **驗證方式** | XML Schema (XSD) | JSON Schema |
| **串流支援** | 無 | CityJSONSeq（2024） |
| **二進位變體** | 無 | FlatCityBuf（2025） |

### 檔案大小

CityJSON 平均比 CityGML-XML 小 **6~10 倍**，來自三個機制：

1. **無 XML 標籤開銷** — JSON 本質上比 XML 精簡
2. **共享頂點索引** — 所有座標集中儲存於檔案根層級的 `vertices` 陣列，幾何透過索引參照
3. **整數座標 + 轉換矩陣** — 座標儲存為整數，搭配 `transform`（scale + translate）還原，避免冗長浮點數字串[^cityjson-filesize]

**實際數據範例**：

| 資料集 | CityJSON | CityGML-XML | 壓縮比 |
|---|---|---|---|
| Den Haag（單一圖磚） | 2.7 MB | 19 MB | **7.0×** |
| Ingolstadt（LoD3） | 4.8 MB | 40 MB | **8.3×** |
| 蒙特婁（單一圖磚） | 5.6 MB | 53 MB | **9.5×** |
| 紐約（單一圖磚） | 110 MB | 682 MB | **6.0×** |
| 蘇黎世 | 293 MB | 2100 MB | **7.0×** |

### 資料模型差異

兩者共享相同的 CityGML 概念資料模型（Building、Bridge、Tunnel、Transportation、Vegetation、WaterBody、CityFurniture、LandUse、Relief 等模組、語意表面類型、LoD），但實作方式不同：

| 面向 | CityGML | CityJSON |
|---|---|---|
| **物件階層** | 深層 XML 巢狀結構（建築 → 部件 → 牆 → 窗 → 幾何） | 扁平化 — 所有物件在頂層以 ID 為鍵，透過 `parent`/`children` 連結 |
| **幾何儲存** | 座標內嵌於每個幾何元素，經常重複 | 共享 `vertices` 陣列，幾體透過索引參照 |
| **座標精度** | XML 文字中的浮點數 | 整數 + `transform`（scale + translate），確保精確往返 |
| **語意表面** | 多種實作方式，依賴 XLink 交叉參照 | 獨立的 `semantics.surfaces` 陣列 + `values` 索引 |
| **CRS 處理** | 理論上每個物件可自有 CRS | 每個檔案單一 CRS（`metadata.referenceSystem`） |
| **擴展機制** | ADE（Application Domain Extensions）— 新的 XML Schema | Extensions — 簡單的 JSON 檔案定義新型別/屬性 |
| **完整度** | 完整支援 CityGML 概念模型 | **子集** — 省略少用或過於複雜的功能 |

---

## 工具生態系

### CityGML 工具（成熟、企業導向）

- **資料庫**：3DCityDB（PostgreSQL/Oracle）
- **GIS 整合**：ArcGIS Pro（需 Data Interoperability 擴充）、QGIS（有限支援）
- **ETL**：FME（Safe Software）— 最全面的 CityGML 支援
- **程式庫**：citygml4j（Java）
- **驗證**：val3dity（3D GML 幾何驗證）

### CityJSON 工具（快速成長、開發者導向）

- **檢視器**：ninja（網頁版）、azul（macOS 原生）、QGIS 外掛、Blender 外掛
- **CLI**：**cjio**（Python 瑞士刀）、cjval（驗證器）、cjseq（串流處理）
- **轉換工具**：**citygml-tools**（雙向 CityJSON ↔ CityGML）、tyler（CityJSON → 3D Tiles）
- **資料庫**：cjdb（PostgreSQL）
- **程式庫**：Python（標準 `json`）、JavaScript、Java、C# 等皆支援
- **生成工具**：3dfier（2D GIS → 3D）、FME 2020+ 內建讀寫

**關鍵**：透過 citygml-tools 可雙向轉換 CityGML 與 CityJSON，選擇 CityJSON 不等於放棄 CityGML 生態系。[^cityjson-software]

---

## 優勢與劣勢

### CityGML

**優勢**：
- 最完整的三維城市語意模型
- OGC 正式標準，最高層級的官方認可
- 成熟的政府/企業生態系
- 強大的 GIS 平台整合（ArcGIS、FME）
- 3DCityDB 提供成熟的資料庫儲存方案

**劣勢**：
- XML/GML 檔案比 CityJSON 大 6~10 倍
- 需要專用程式庫才能解析（無完整的 JavaScript 解析器）
- 同一幾何有 **25 種以上** GML 編碼變體，造成實作碎片化
- XLink 交叉參照增加解析複雜度
- 不適合網頁與行動應用
- 無串流支援

### CityJSON

**優勢**：
- 檔案小 6~10 倍，節省儲存與傳輸成本
- 任何語言的標準 JSON 程式庫即可解析
- Web 原生（瀏覽器與 JavaScript 原生支援）
- 扁平結構，易於理解與除錯
- 解析速度比 XML 快數個數量級
- 雙向轉換 CityGML，不鎖定生態系
- 串流（CityJSONSeq）與二進位（FlatCityBuf）創新
- 活躍的開放 GitHub 社群

**劣勢**：
- **CityGML 概念模型的子集**（但涵蓋絕大多數常用功能）
- OGC Community Standard，層級低於 CityGML 的正式標準
- 生態系較新，政府/企業採用尚在成長中
- 無原生的 3DCityDB 支援（但有 cjdb 替代方案）
- 新手容易忘記套用 `transform` 導致座標異常

---

## 採用趨勢

**CityGML** 仍是多國政府（荷蘭、德國、瑞士、法國、日本等）制定國家三維城市模型計畫的標準格式，在學術研究與企業 GIS 領域根深蒂固。

**CityJSON** 正快速成長，許多城市已開放 CityJSON 資料（海牙、鹿特丹、阿姆斯特丹、蘇黎世、紐約、蒙特婁、柏林等），荷蘭 3DBAG（約一千萬棟建築）以 CityJSON 為主要下載格式。OGC Community Standard 地位（2023）進一步提升可信度。

**結論趨勢**：兩者**互補而非競爭**。CityGML 作為**權威概念模型**與官方規格語言，而實際資料處理中越來越多從業者**轉換為 CityJSON** 進行操作。CityJSON 可理解為「CityGML 資料模型的**更好用的編碼實作**」。[^spatialworkflow][^serdarozden]

---

## 參考資料

[^cityjson-about]: CityJSON. (n.d.). About CityJSON. Retrieved 2026-09-25, from https://www.cityjson.org/about/
[^ogc-citygml]: Open Geospatial Consortium. (n.d.). CityGML. Retrieved 2026-09-25, from https://www.ogc.org/standards/citygml/
[^cityjson-paper]: Ledoux, H., Arroyo Ohori, K., & Peters, R. (2019). CityJSON: a compact and easy-to-use encoding of the CityGML data model. *Open Geospatial Data, Software and Standards*, 4, 4. Retrieved 2026-09-25, from https://link.springer.com/article/10.1186/s40965-019-0064-0
[^cityjson-filesize]: CityJSON. (n.d.). File size comparison. Retrieved 2026-09-25, from https://www.cityjson.org/filesize/
[^cityjson-software]: CityJSON. (n.d.). Software. Retrieved 2026-09-25, from https://www.cityjson.org/software/
[^spatialworkflow]: Spatial Workflow. (n.d.). CityJSON and CityGML Explained. Retrieved 2026-09-25, from https://www.spatialworkflow.io/cityjson-and-citygml-explained/
[^serdarozden]: Özden, S. (n.d.). CityJSON vs CityGML — Web GIS 3D Buildings Guide. Retrieved 2026-09-25, from https://www.serdarozden.com/cityjson-vs-citygml-web-gis-3d-buildings-guide/
[^ogc-cityjson]: Open Geospatial Consortium. (2023). OGC CityJSON Community Standard (20-072r5). Retrieved 2026-09-25, from https://docs.ogc.org/cs/20-072r5/20-072r5.html