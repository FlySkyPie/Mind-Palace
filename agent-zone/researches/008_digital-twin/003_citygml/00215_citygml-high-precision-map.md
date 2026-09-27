# CityGML 是高精度地圖嗎？

## 摘要

CityGML（City Geography Markup Language）是由 Open Geospatial Consortium（OGC）制定的開放標準，用於儲存和交換具備語意內涵的 3D 城市模型。**CityGML 本身並非「高精度地圖」（High-Definition Map, HD Map）**，而是一種語意豐富的 3D 城市資料模型 / 交換格式。它與高精度地圖在用途、精度等級、更新頻率與核心受眾上存在顯著差異，但兩者在智慧城市與自動駕駛領域可互補使用。

## 1. 什麼是 CityGML？

CityGML 是一種基於 XML 的開放資料標準，用於表示、儲存和交換虛擬 3D 城市與景觀模型。不同於純視覺化的 3D 格式（如 OBJ、Collada），CityGML 不僅描述物體的外觀（幾何形狀＋紋理），更著重於物體**「是什麼」** — 建築物、道路、橋樑、植被、水體、城市家具等，以及它們之間的拓撲關係。每個物件可攜帶主題屬性（如建造年份、功能用途、建材、樓層數）[^ocg-standard]。

CityGML 最初於 2002 年在德國由 Special Interest Group 3D（SIG3D）開發，2008 年正式成為 OGC 國際標準[^wikipedia]。目前最新版本為 3.0（概念模型於 2021 年 9 月通過，GML 編碼於 2023 年 7 月通過）[^gisuser]。

## 2. 核心用途 vs. 高精度地圖

| 面向 | CityGML | 高精度地圖（HD Map） |
|---|---|---|
| **核心目的** | 城市規劃、環境模擬、數位雙生、資料交換 | 即時車輛定位、路徑規劃、感知輔助 |
| **精度範圍** | LOD 相依，≤0.2m～≤5m | 公分級（通常 1–10 cm 絕對精度） |
| **道路細節** | 交通空間、車道（v3.0 起） | 車道線幾何、道路標誌、路緣、號誌 |
| **語意範疇** | 建築、植被、地形、水體、城市家具、管線、橋隧 | 以道路網路及駕駛相關靜態基礎設施為主 |
| **更新頻率** | 週期性／靜態（v3.0 的 Dynamizer 模組可接入感測器資料） | 近即時更新（交通、施工、道路變更） |
| **主要受眾** | 都市規劃師、模擬工程師、GIS 專業人員 | 自動駕駛系統、ADAS 開發者 |

CityGML 提供的是**城市尺度的語意化 3D 脈絡**，而高精度地圖提供的是**自駕車導航所需的精確道路幾何**。兩者之間是互補關係，而非競爭關係[^ocg-standard][^gisuser]。

## 3. LOD（Levels of Detail）精度等級

CityGML 透過 LOD 機制來控制幾何與語意的細緻程度：

| LOD | 描述 | 典型精度 | 應用場景 |
|---|---|---|---|
| **LOD 0** | 2.5D 足跡／屋頂面，僅數值地形模型加上建築物輪廓 | ≤ **5 公尺** | 區域規劃、大範圍總覽 |
| **LOD 1** | 方塊模型 — 將建築物足跡垂直拉伸，無屋頂結構或立面細節 | ≤ **5 公尺** | 全市模擬（太陽能潛力、噪音地圖） |
| **LOD 2** | 具備一般化屋頂結構與立面附屬物（陽台等），區分邊界面（屋頂、牆、地面） | ≤ **2 公尺** | 街區分析、可視性研究 |
| **LOD 3** | 詳細建築外觀 — 門窗、完整立面幾何與紋理 | ≤ **0.5 公尺** | 都市設計、詳細日照分析、街景視覺化 |
| **LOD 4** *(v2.0 限定)* | 室內結構 — 房間、樓梯、家具、室內設施 | ≤ **0.2 公尺** | 室內導航、設施管理 |

> 在 CityGML 3.0 中，LOD 概念已與內外部分離，室內特徵可在任何 LOD 層級呈現，不再僅限於 LOD 4[^ocg-guide]。

最精細的 LOD 3（≤0.5m）與 LOD 4（≤0.2m）在精度上仍顯著低於高精度地圖的公分級要求。這反映出兩者設計目標的根本差異。

## 4. CityGML 與 GIS 及 3D 城市建模的關係

CityGML 是 GIS 領域中「語意化 3D 城市模型」最主流的標準[^tudelft]。它：

- 以 GML（Geography Markup Language）應用綱要的形式實作，直接連結 OGC/ISO 地理空間標準體系
- 將 3D 城市資料組織為主題模組：建築、橋樑、隧道、交通、植被、水體、土地使用、地形、城市家具等
- 支援與 BIM（Building Information Modeling）互通（CityGML 3.0 改善了與 IFC 的對齊）
- 為都市數位雙生（Urban Digital Twins）與智慧城市倡議提供骨幹架構[^tum-news]

## 5. CityGML 3.0 交通模組與自動駕駛

CityGML 3.0 的交通（Transportation）模組是值得關注的進展。它顯著強化了對道路網路的建模能力：

- 支援將交通空間細分至**個別行車道（driving lane）**等級
- 涵蓋多模式運輸（道路、鐵路、水路）
- OGC 官方文件明確提到其適用於「自動駕駛與駕駛輔助系統」[^ocg-standard]

然而，這並不代表 CityGML 因此變成高精度地圖。它在這方面的角色更接近於：
1. 提供**高精度地圖的資料來源**之一（貢獻建築物脈絡、交通空間拓撲）
2. 作為**自動駕駛模擬環境**的基礎城市模型資料
3. 支援**交通規劃模擬**，而非即時車輛定位[^grokipedia]

## 6. 結論

CityGML **不是**高精度地圖。它是一種具備語意內涵的 3D 城市資料模型與交換標準，精度範圍從 5 公尺到 0.2 公尺不等，取決於 LOD 等級。高精度地圖則專注於公分級的道路幾何與即時定位，兩者在精度要求、更新頻率與應用場景上有本質差異。

但它們的關係是**互補而非互相取代**：CityGML 提供城市尺度的語意化 3D 脈絡，可作為高精度地圖的資料來源與模擬基礎；高精度地圖則提供自動駕駛所需的精密道路幾何。隨著 CityGML 3.0 強化交通模組，這層互補關係將更加緊密。

## 參考資料

[^ocg-standard]: Open Geospatial Consortium. (n.d.). *CityGML Standard*. Retrieved 2026-09-25, from https://www.ogc.org/standards/citygml/
[^wikipedia]: Wikipedia. (n.d.). *CityGML*. Retrieved 2026-09-25, from https://en.wikipedia.org/wiki/CityGML
[^gisuser]: GISuser. (2021-09-14). *OGC Membership Approves the CityGML v3.0 Conceptual Model as Official OGC Standard*. Retrieved 2026-09-25, from https://gisuser.com/2021/09/ogc-membership-approves-the-citygml-v3-0-conceptual-model-as-official-ogc-standard/
[^tudelft]: TU Delft. (n.d.). *3D GeoBIM Benchmark — CityGML*. Retrieved 2026-09-25, from https://3d.bk.tudelft.nl/projects/geobim-benchmark/citygml.html
[^grokipedia]: Grokipedia. (n.d.). *CityGML*. Retrieved 2026-09-25, from https://grokipedia.com/page/CityGML
[^tum-news]: Chair of Geoinformatics, Technical University of Munich. (2021-09-13). *Publication of the new CityGML 3.0 standard with the participation of the Chair of Geoinformatics of TUM*. Retrieved 2026-09-25, from https://www.asg.ed.tum.de/en/gis/news/article/veroeffentlichung-des-neuen-citygml-30-standards-unter-mitarbeit-des-lehrstuhls-fuer-geoinformatik-der-tum/
[^ocg-guide]: Open Geospatial Consortium. (n.d.). *OGC CityGML 3.0 Conceptual Model Users Guide*. Retrieved 2026-09-25, from https://docs.ogc.org/guides/20-066.html