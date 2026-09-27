# FOSS CityGML 視覺化軟體調查報告

## 概述

CityGML 是 OGC（Open Geospatial Consortium）制定的 3D 城市模型開放資料標準，支援 LOD（Level of Detail）0 到 4。本報告調查可視覺化 CityGML 檔案的**自由開源（FOSS）** 軟體，涵蓋桌面應用、網頁應用及程式庫。

---

## 主要工具詳細分析

### 1. KITModelViewer（FZKViewer 後繼者）

| 項目 | 內容 |
|---|---|
| **開發者** | Karlsruhe Institute of Technology (KIT), IAI |
| **授權** | Freeware（外掛 SDK 為 Apache 2.0），**原始碼未完全公開** |
| **平台** | **僅 Windows** |
| **CityGML 支援** | 0.4.0、1.0.0、**2.0.0**、**3.0.0**（v7.1 起）、ADE（EnergyADE, Noise 等） |
| **其他格式** | IFC、gbXML、GML（XPlanGML、INSPIRE、NAS、ALKIS）、LandXML、CityJSON、EnergyPlus（IDF/epJSON）、點雲（e57、las、laz）、GeoJSON、Shapefile、GeoPackage、DXF、FBX、OBJ、glTF、Gaussian Splatting 等 |
| **重點功能** | • OpenGL 3D 渲染（新版引擎，支援 3D 眼鏡）<br>• 物件屬性檢視<br>• 依屬性顏色編碼<br>• BIM + GIS 模型合併<br>• 地理參考（多種 CRS）<br>• VR 支援（SteamVR、HTC Vive、HP 頭戴裝置）<br>• Python API + C++ Plugin SDK<br>• OGC Web Services（WFS、WMS、WMTS、WCS、W3DS）<br>• OpenStreetMap 整合<br>• 3D Tiles 外掛（v7.5 起） |
| **維護狀態** | **活躍維護中**，最新版 v7.5（2025-12-19） |
| **備註** | FZKViewer 已停止維護，由 KITModelViewer 取代。**僅二進位檔免費，原始碼未完全開放。** |

**連結**：<https://www.iai.kit.edu/english/4561.php> | GitHub：[KIT-IAI/SDM_KITModelViewer](https://github.com/KIT-IAI/SDM_KITModelViewer) [^kit-main]

### 2. azul

| 項目 | 內容 |
|---|---|
| **開發者** | TU Delft（3D Geoinformation 研究群） |
| **授權** | **GPLv3** — 完全開源 |
| **平台** | **macOS 13+**（Apple Silicon & Intel）和 **iOS 15+**（iPhone & iPad） |
| **CityGML 支援** | **1.0**、**2.0**（透過 GML 編碼） |
| **CityJSON 支援** | 1.0、1.1、2.0 |
| **其他格式** | IndoorGML、OBJ、OFF、POLY |
| **重點功能** | • 原生 Apple Metal + C++17 + Swift 5<br>• 快速 SIMD 向量/矩陣運算<br>• 同時載入多個檔案<br>• 3D 視圖或側邊欄點選選取物件<br>• 依 LOD 過濾<br>• 瀏覽物件屬性<br>• 深色模式<br>• 材質與紋理支援（v1.5+）<br>• 圖片匯出<br>• 搜尋物件 |
| **維護狀態** | **非常活躍**，最新版 v1.5.2（2025-08-14） |
| **備註** | 僅支援 macOS/iOS，**不支援 CityGML 3.0**。 |

**連結**：<https://github.com/tudelft3d/azul> [^azul]

### 3. CityGMLViewer（桌面版 + 瀏覽器版）

| 項目 | 內容 |
|---|---|
| **開發者** | HFT Stuttgart（Matthias Betz） |
| **授權** | **桌面版**：Apache 2.0 — **瀏覽器版（JS）**：AGPLv3 |
| **平台** | **桌面版**：Windows、Linux（需 Java 17） — **瀏覽器版**：任何瀏覽器 |
| **CityGML 支援** | **2.0** 和 **3.0** |
| **重點功能** | • **桌面版**：OpenGL 加速，處理大型檔案較快<br>• **瀏覽器版**：純 JavaScript，不上傳伺服器，本地處理 |
| **維護狀態** | **低度活躍**，桌面版 v0.0.2，瀏覽器版偶有更新 |
| **備註** | 瀏覽器版無需安裝、不上傳資料，隱私友善。 |

**連結**：<https://transfer.hft-stuttgart.de/gitlab/citygml/citygmlviewer> [^hft-viewer]
**瀏覽器版**：<https://transfer.hft-stuttgart.de/pages/citydoctor/citygml-viewer/>

### 4. 3DCityDB Web Map Client

| 項目 | 內容 |
|---|---|
| **開發者** | TU Munich（Chair of Geoinformatics）+ Virtual City Systems |
| **授權** | **Apache 2.0** — 完全開源 |
| **平台** | **網頁版**（任何支援 WebGL 的瀏覽器） |
| **CityGML 支援** | **1.0**、**2.0**（需先匯入 3DCityDB 再匯出為可視化格式） |
| **其他格式** | KML、COLLADA、glTF、CZML、GeoJSON、Cesium 3D Tiles、I3S |
| **重點功能** | • 基於 Cesium Virtual Globe（WebGL，無需外掛）<br>• 處理**任意大規模** 3D 城市模型（如柏林 55 萬棟建築、紐約 100 萬+）<br>• 分塊 KML/glTF 動態載入/卸載<br>• 圖層管理<br>• 連結主題資料（Google Sheets、PostgreSQL/PostgREST、OGC Features）<br>• 物件高亮、隱藏、顯示<br>• 陰影視覺化<br>• 行動端支援（GPS、指南針、第一人稱視角）<br>• 場景連結分享（URL 保存所有設定）<br>• Docker 部署 |
| **維護狀態** | **活躍維護中**，最新版 v2.0.0，GitHub 426 stars |
| **備註** | 無法直接開啟 .gml 檔案，需先透過 Importer/Exporter 將 CityGML 匯入 PostgreSQL/PostGIS 或 Oracle 資料庫，再匯出為可視化格式。是最適合大規模模型的方案。 |

**連結**：<https://github.com/3dcitydb/3dcitydb-web-map> [^web-map]

### 5. CityGML Viewer JS（Radiate Berlin）

| 項目 | 內容 |
|---|---|
| **開發者** | Radiate Berlin |
| **授權** | 開源（基於 Three.js） |
| **平台** | **網頁版**（任何瀏覽器） |
| **CityGML 支援** | **2.0**（客戶端處理） |
| **維護狀態** | 資訊有限 |
| **備註** | 基於 Three.js 的 CityGML 客戶端檢視器，資料不上傳伺服器。 |

**連結**：<https://radiate.berlin/en/citygml-viewer/> [^radiate]

### 6. citygl（JavaScript 程式庫）

| 項目 | 內容 |
|---|---|
| **開發者** | citygl team |
| **授權** | **MIT** |
| **平台** | **網頁版**（JavaScript 程式庫） |
| **CityGML 支援** | 一般 GML 解析 |
| **維護狀態** | **已不活躍**，僅 6 次提交、9 stars，最後提交為多年前 |
| **備註** | 可視為學習資源或原型專案，不適合生產環境。 |

**連結**：<https://github.com/citygl/citygl> [^citygl]

### 7. FZKViewer（已停止維護）

| 項目 | 內容 |
|---|---|
| **開發者** | KIT（Karlsruhe Institute of Technology） |
| **授權** | Apache 2.0 |
| **平台** | **僅 Windows** |
| **CityGML 支援** | 0.4.0、1.0.0、**2.0.0**、ADE |
| **維護狀態** | **已停止維護**，由 KITModelViewer 取代 |
| **備註** | 仍可下載但不再開發。 |

**連結**：<https://www.iai.kit.edu/english/1648.php> [^fzk]

### 8. Aristoteles（University of Bonn）

歷史專案，原始下載連結已無法存取，推測已停止維護。[^wiki-open]

---

## 輔助工具（非檢視器，但為 CityGML 生態系重要工具）

| 工具 | 用途 | 授權 | 狀態 |
|---|---|---|---|
| **citygml4j** | Java CityGML 處理程式庫 | Apache 2.0 | 活躍 |
| **citygml-tools** | CLI CityGML 處理（驗證、轉換） | Apache 2.0 | 活躍 |
| **libcitygml** | C++ CityGML 解析程式庫 | LGPL | 活躍 |
| **val3dity** | 3D 圖元 ISO 19107 幾何驗證 | GPLv3 | 活躍 |
| **CityGML2OBJs** | CityGML 轉 OBJ 格式 | MIT | 活躍 |

[^citygml-tools]: citygml-tools GitHub. Retrieved 2026-09-25, from <https://github.com/citygml4j/citygml-tools>
[^val3dity]: val3dity GitHub. Retrieved 2026-09-25, from <https://github.com/tudelft3d/val3dity>
[^citygml2objs]: CityGML2OBJs GitHub. Retrieved 2026-09-25, from <https://github.com/tudelft3d/CityGML2OBJs>

---

## 比較總表

| 工具 | CG 2.0 | CG 3.0 | Windows | macOS | Linux | 網頁 | 授權 | 維護狀態 |
|---|---|---|---|---|---|---|---|---|
| **KITModelViewer** | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | Freeware | ✅ **活躍** |
| **azul** | ✅ | ❌ | ❌ | ✅ | ❌ | ❌ | GPLv3 | ✅ **活躍** |
| **CityGMLViewer 桌面** | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ | Apache 2.0 | ⚠️ 低 |
| **CityGMLViewer 瀏覽器** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | AGPLv3 | ⚠️ 低 |
| **3DCityDB Web Map** | ✅ (需 DB) | ❌ | ✅ | ✅ | ✅ | ✅ | Apache 2.0 | ✅ **活躍** |
| **Radiate Viewer** | ✅ | ❌ | ✅ | ✅ | ✅ | ✅ | 開源 | 未知 |
| **citygl** | ✅ | ❌ | ✅ | ✅ | ✅ | ✅ | MIT | ❌ 停滯 |
| **FZKViewer** | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ | Apache 2.0 | ❌ **停止** |
| **Aristoteles** | ✅ | ❌ | ✅ | ✅ | ✅ | ❌ | 開源 | ❌ **停止** |

---

## 使用情境推薦

| 需求 | 最佳選擇 |
|---|---|
| **Windows 最佳桌面檢視器** | **KITModelViewer**（支援 CG 3.0、ADE，功能最豐富） |
| **macOS 桌面檢視器** | **azul**（原生 Metal 效能，活躍開發） |
| **快速免安裝視覺化** | **CityGMLViewer JS 瀏覽器版**（支援 CG 2.0 & 3.0） |
| **大規模城市模型** | **3DCityDB Web Map Client**（Cesium 分塊載入） |
| **跨平台桌面（Windows + Linux）** | **CityGMLViewer 桌面版**（Java，支援 CG 2.0 & 3.0） |
| **完全開源方案** | **azul**（GPLv3）或 **3DCityDB Web Map**（Apache 2.0） |

---

## 結論

CityGML 視覺化有豐富的 FOSS 選擇：

- 若使用 **Windows 且需要 CityGML 3.0 支援**，**KITModelViewer** 是最全面的選擇（但原始碼未完全開放）。
- 若使用 **macOS 且不需要 CG 3.0**，**azul** 是最佳的完全開源選擇，效能優異且活躍開發中。
- 若需要 **跨平台或免安裝方案**，**CityGMLViewer 瀏覽器版** 支援 CG 3.0 且無需上傳資料至伺服器。
- 若處理 **超大規模城市模型**，**3DCityDB Web Map Client** 搭配 3D City Database 是最可擴展的 FOSS 方案。

---

## 參考資料

[^kit-main]: Karlsruhe Institute of Technology. (n.d.). KITModelViewer. Retrieved 2026-09-25, from <https://www.iai.kit.edu/english/4561.php>
[^kit-release]: Karlsruhe Institute of Technology. (n.d.). KITModelViewer Release Notes. Retrieved 2026-09-25, from <https://www.iai.kit.edu/english/4560.php>
[^kit-github]: KIT-IAI. (n.d.). SDM_KITModelViewer. GitHub. Retrieved 2026-09-25, from <https://github.com/KIT-IAI/SDM_KITModelViewer>
[^azul]: TU Delft 3D Geoinformation. (n.d.). azul. GitHub. Retrieved 2026-09-25, from <https://github.com/tudelft3d/azul>
[^hft-viewer]: Betz, M. (n.d.). CityGMLViewer. HFT Stuttgart GitLab. Retrieved 2026-09-25, from <https://transfer.hft-stuttgart.de/gitlab/citygml/citygmlviewer>
[^web-map]: 3DCityDB. (n.d.). 3DCityDB Web Map Client. GitHub. Retrieved 2026-09-25, from <https://github.com/3dcitydb/3dcitydb-web-map>
[^radiate]: Radiate Berlin. (n.d.). CityGML Viewer. Retrieved 2026-09-25, from <https://radiate.berlin/en/citygml-viewer/>
[^citygl]: citygl. (n.d.). citygl. GitHub. Retrieved 2026-09-25, from <https://github.com/citygl/citygl>
[^fzk]: Karlsruhe Institute of Technology. (n.d.). FZKViewer. Retrieved 2026-09-25, from <https://www.iai.kit.edu/english/1648.php>
[^wiki-open]: CityGML Wiki. (n.d.). Open Source. Retrieved 2026-09-25, from <https://www.citygmlwiki.org/index.php?title=Open_Source>
[^citygml4j]: citygml4j. (n.d.). citygml4j. GitHub. Retrieved 2026-09-25, from <https://github.com/citygml4j/citygml4j>
[^awesome-citygml]: awesome-citygml. (n.d.). GitHub. Retrieved 2026-09-25, from <https://github.com/OloOcki/awesome-citygml>