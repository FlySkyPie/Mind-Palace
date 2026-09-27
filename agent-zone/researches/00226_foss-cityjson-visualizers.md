# FOSS CityJSON 視覺化工具調查

CityJSON 是一種基於 JSON 的 3D 城市模型格式，相較於 CityGML 更輕量且容易處理。本文調查現有的自由開源（FOSS）CityJSON 視覺化工具，涵蓋桌面應用程式、網頁工具與程式庫。

## 桌面工具

### azul — 原生 3D 城市模型檢視器

azul 是由荷蘭代爾夫特理工大學（TU Delft）3D 地理資訊團隊開發的原生 3D 城市模型檢視器，支援 macOS 與 iOS。[^azul]

- **語言/平台**：C++17、Swift 5，原生 macOS 13+（Apple Silicon 與 Intel）、iOS 15+
- **支援格式**：CityJSON 1.0、1.1、2.0（含 CityJSONSeq）、CityGML、IndoorGML、OBJ、OFF、POLY
- **關鍵功能**：同時載入多個檔案、點選選取物件、側邊欄瀏覽、LoD 篩選、物件屬性檢視
- **授權條款**：GPLv3
- **儲存庫**：https://github.com/tudelft3d/azul

### Up3date — Blender 外掛

Up3date 是 Blender 的 CityJSON 外掛，支援匯入、檢視、編輯與匯出 CityJSON 2.0 3D 城市模型。[^up3date]

- **語言/平台**：Python，執行於 Blender 5.0+（跨平台：Windows、macOS、Linux）
- **支援幾何型別**：MultiPoint、MultiLineString、MultiSurface、CompositeSurface、Solid、CompositeSolid、MultiSolid
- **關鍵功能**：完整雙向匯入/匯出、支援 LoD 0–3（含小數 LoD 如 2.2）、保留語義表面與屬性、處理幾何模板與外觀
- **授權條款**：MIT
- **儲存庫**：https://github.com/cityjson/Up3date

### CityJSON Loader for QGIS — QGIS 外掛

此 QGIS 外掛讓使用者能在 QGIS 中載入並以 3D 視覺化 CityJSON 資料集（含 CityJSONSeq）。[^qgis-plugin]

- **語言/平台**：Python，QGIS 外掛（跨平台）
- **關鍵功能**：載入 CityJSON/CityJSONSeq 為圖層、可依 CityObject 類型分割圖層、自動啟用 QGIS 3D 地圖檢視
- **授權條款**：Apache-2.0
- **儲存庫**：https://github.com/cityjson/cityjson-qgis-plugin

### CityJSON Reader Plugin for ParaView — ParaView 外掛

此外掛讓 ParaView 能直接讀取 CityJSON 檔案，並利用 ParaView 強大的科學視覺化管線進行分析。[^paraview-plugin]

- **語言/平台**：C++，ParaView 外掛（跨平台，提供 Linux 二進位檔）
- **授權條款**：MIT
- **儲存庫**：https://github.com/cityjson/cityjson-paraview-plugin

## 網頁工具

### CityJSON Ninja

CityJSON Ninja 是官方提供的網頁版 CityJSON 檢視器與編輯器，部署於 ninja.cityjson.org。[^ninja]

- **語言/平台**：JavaScript（瀏覽器端）
- **關鍵功能**：拖放上傳檔案、3D 視覺化與軌道控制、支援幾何挖洞、瀏覽器內編輯、支援 CityJSON 與 CityJSONSeq
- **授權條款**：FOSS，屬於官方 CityJSON 生態系
- **網址**：https://ninja.cityjson.org/
- **原始碼**：屬於 cityjson GitHub 組織

### CityJSON-viewer (fhb1990)

以 Three.js 打造的簡易網頁 CityJSON 檢視器，提供拖放介面快速檢視。[^fhb-viewer]

- **語言/平台**：JavaScript（Three.js、WebGL），瀏覽器端
- **關鍵功能**：拖放或檔案選取器、3D 軌道控制（平移、縮放、旋轉）、自動顯示三角化幾何、可直接以 HTTP Server 離線執行
- **限制**：僅支援三角化模型，不支援挖洞
- **儲存庫**：https://github.com/fhb1990/CityJSON-viewer

### CityView — 多平台 CityJSON 渲染器

CityView 支援 Three.js、React-three-fiber 與 Jupyter Notebook/JupyterLab，同時提供 CityJSON 與 CityJSONSeq 支援。[^cityview]

- **語言/平台**：Python 套件（PyPI）與 JavaScript/TypeScript 套件（npm）
- **關鍵功能**：
  - `cityview`（Python）：在 Jupyter Notebook/Lab 中渲染，提供 `VirtualView`（純 3D）與 `MapView`（含地圖背景）
  - `three-cityjson`（npm）：Three.js / React-three-fiber 的 CityJSON 載入器
  - 支援點擊事件處理、明/暗主題
- **授權條款**：MIT
- **儲存庫**：https://github.com/ozekik/cityview
- **PyPI**：https://pypi.org/project/cityview/

### xeokit SDK — 高效能 WebGL 渲染引擎

xeokit 是一套開源的 WebGL 3D 網頁圖形 SDK，主要服務 BIM/AEC 領域。可將 CityJSON 轉換為其原生 XKT 格式後於瀏覽器中高效渲染。[^xeokit]

- **語言/平台**：JavaScript/TypeScript，瀏覽器端 + Node.js CLI
- **關鍵功能**：高壓縮比 XKT 格式（約 5:1）、`convert2xkt` CLI 轉換工具、雙精度全域座標、物件選取、可見性切換、攝影機控制
- **授權條款**：AGPLv3（商業授權可用雙重授權）
- **儲存庫**：https://github.com/xeokit/xeokit-sdk
- **網站**：https://xeokit.io/

### Measur3D — 全端 CityJSON 管理應用

Measur3D 是一套以 MERN（MongoDB、Express、React、Node.js）架構開發的全端網頁應用，可作為資料庫後端的 CityJSON 城市模型管理與檢視工具。[^measur3d]

- **語言/平台**：JavaScript（MongoDB、Express.js、React、Node.js）
- **關鍵功能**：MongoDB 儲存 CityJSON 模型（Mongoose 驗證 CityJSON 1.0.1 規範）、Three.js 3D 視覺化、物件選取與高亮、支援幾例與幾何模板、OGC API Features 相容、Swagger API 文件
- **授權條款**：Apache-2.0
- **儲存庫**：https://github.com/GANys/Measur3D

## 轉換工具間接視覺化

### tyler — CityJSON 轉 3D Tiles 轉換器

tyler 以 Rust 撰寫，可將 CityJSON/CityJSONSeq 轉換為 3D Tiles v1.1 等瓦片化格式，再透過 CesiumJS 等相容檢視器呈現。[^tyler]

- **語言/平台**：Rust，跨平台 CLI 工具（亦提供 Docker 映像）
- **輸出格式**：3D Tiles v1.1（含 glTF 二進位與特徵中繼資料）、瓦片化 OBJ、CityJSON、CityJSONSeq、TSV、GeoPackage
- **關鍵功能**：隱式瓦片化、LoD 選取、屬性篩選、依 CityObject 類型著色、CRS 重投影（PROJ）
- **授權條款**：Apache-2.0
- **儲存庫**：https://github.com/3DGI/tyler

## 程式庫與 API

| 工具 | 語言 | 功能 | 儲存庫 |
|------|------|------|--------|
| C# library (bertt) | C# (.NET) | 讀寫 CityJSON | https://github.com/bertt/cityjson |
| citygml4j | Java | 讀寫 CityGML 與 CityJSON | https://github.com/citygml4j |
| cjio | Python | CityJSON 處理 CLI 與程式庫，官方驗證器 | https://github.com/cityjson/cjio |

## 總結對照表

| 工具 | 類型 | 平台 | 直接視覺化 | 授權 |
|------|------|------|-----------|------|
| azul | 桌面 | macOS/iOS | ✅ | GPLv3 |
| Up3date | 桌面 (Blender) | 跨平台 (Blender) | ✅ | MIT |
| QGIS Plugin | 桌面 (QGIS) | 跨平台 (QGIS) | ✅ | Apache-2.0 |
| ParaView Plugin | 桌面 (ParaView) | 跨平台 (ParaView) | ✅ | MIT |
| CityJSON Ninja | 網頁 | 瀏覽器 | ✅ | FOSS |
| CityJSON-viewer | 網頁 | 瀏覽器 (Three.js) | ✅ | FOSS |
| CityView | 網頁 + Jupyter | 瀏覽器/Notebook | ✅ | MIT |
| xeokit SDK | 網頁 SDK | 瀏覽器 | ⚠️ 需轉換 | AGPLv3 |
| Measur3D | 網頁全端 | 瀏覽器 (MERN) | ✅ | Apache-2.0 |
| tyler | CLI 轉換 | 跨平台 (Rust) | ❌ 輸出 3D Tiles | Apache-2.0 |

## 建議

1. **快速檢視單一檔案**：CityJSON Ninja（網頁，無需安裝）或 azul（macOS 原生）
2. **整合 GIS 工作流程**：QGIS Plugin 最適合已使用 QGIS 的使用者
3. **進階編輯與建模**：Up3date（Blender 外掛）提供完整的編輯能力
4. **Jupyter 資料分析**：CityView 可直接在 Notebook 中渲染
5. **自訂網頁應用**：xeokit SDK 或 CityView（three-cityjson npm 套件）

[^azul]: TU Delft 3D Geoinformation. (n.d.). azul — Native 3D city model viewer for macOS/iOS. Retrieved 2026-09-26, from https://github.com/tudelft3d/azul
[^up3date]: cityjson. (n.d.). Up3date — Blender add-on for CityJSON 2.0. Retrieved 2026-09-26, from https://github.com/cityjson/Up3date
[^qgis-plugin]: cityjson. (n.d.). cityjson-qgis-plugin. Retrieved 2026-09-26, from https://github.com/cityjson/cityjson-qgis-plugin
[^paraview-plugin]: cityjson. (n.d.). cityjson-paraview-plugin. Retrieved 2026-09-26, from https://github.com/cityjson/cityjson-paraview-plugin
[^ninja]: cityjson. (n.d.). CityJSON Ninja. Retrieved 2026-09-26, from https://ninja.cityjson.org/
[^fhb-viewer]: fhb1990. (n.d.). CityJSON-viewer. Retrieved 2026-09-26, from https://github.com/fhb1990/CityJSON-viewer
[^cityview]: Ozeki, K. (n.d.). cityview — CityJSON loader and renderer for Three.js and Jupyter. Retrieved 2026-09-26, from https://github.com/ozekik/cityview
[^xeokit]: xeokit. (n.d.). xeokit-sdk. Retrieved 2026-09-26, from https://github.com/xeokit/xeokit-sdk
[^measur3d]: Nys, G. A. (n.d.). Measur3D — CityJSON city model web application. Retrieved 2026-09-26, from https://github.com/GANys/Measur3D
[^tyler]: 3DGI. (n.d.). tyler — CityJSON to 3D Tiles converter. Retrieved 2026-09-26, from https://github.com/3DGI/tyler
[^cityjson-software]: cityjson. (n.d.). CityJSON Software. Retrieved 2026-09-26, from https://www.cityjson.org/software/