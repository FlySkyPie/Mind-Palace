# CityJSON 檔案樣本調查

CityJSON 是一種以 JSON 為基礎的 3D 城市模型編碼格式，實作了 OGC CityGML 資料模型（v3.0）的子集[^cityjson]。相較於 CityGML-XML，CityJSON 檔案體積約小 7 倍，且結構簡單、易於程式解析，是 OGC 官方社群標準（文件編號 20-072r5）[^ogc]。

## 官方樣本資料來源

CityJSON 官方網站提供了多個分類的樣本檔案[^datasets]：

### 簡單幾何測試檔（適合初學）

這些檔案體積極小，適合用於學習與測試：

| 檔案 | 說明 |
|---|---|
| `cube.city.json` | 單一立方體 |
| `tetra.city.json` | 四面體 |
| `torus.city.json` | 環面 |
| `msol.city.json` | 多重立體（Multi-Solid）範例 |
| `csol.city.json` | 複合立體（Composite-Solid）範例 |
| `twocube.city.json` | 雙立方體 |
| `geomtemplate.city.json` | 幾何模板（Geometry Template）用法示範 |

### 快速入門檔案

`twobuildings.city.json` 包含 2 棟建築物，是官方教學使用的入門範例[^tutorial]。

### 擴充（Extension）示範

`noise_data.city.json` 展示 CityJSON 擴充機制的用法（噪音資料）[^tutorial]。

### 真實城市資料集

以下為轉換為 CityJSON v2.0 的真實城市開放資料：

| 城市 | 檔案大小 | 備註 |
|---|---|---|
| 海牙（Den Haag） | 2.7 MB | 建築物 + 地形，LoD2 |
| 因戈爾施塔特（Ingolstadt） | 4.8 MB | LoD3 建築物 |
| 蒙特婁（Montréal） | 5.6 MB | LoD2 建築物 |
| 紐約（NYC） | 110 MB | LoD2，DA13 圖幅 |
| 鹿特丹（Rotterdam） | 2.7 MB | LoD2，Delfshaven 街區 |
| 維也納（Vienna） | 5.6 MB | LoD2 建築物 |
| 蘇黎世（Zürich） | 293 MB | 大型 LoD2 資料集 |
| 鐵路示範（Railway） | 4.5 MB | 建築物、鐵路、地形、植被、水域、隧道 |

## 大規模資料集

- **3DBAG（荷蘭）**：涵蓋荷蘭全境約 1,000 萬棟建築物，以 CityJSON 格式提供[^3dbag]。
- **PDOK 3D 地形**：荷蘭全國 3D 地形資料[^pdok]。
- **新加坡 HDB 資料**：新加坡組屋（公共住宅）的 CityJSON 資料[^hdb]。

## 測試與基準測試（Benchmark）

**cityjson-corpus** 是一個專為 CityJSON 設計的共享測試語料庫，包含[^corpus]：
- 一致性測試（Conformance tests）
- 合成基準測試（Synthetic benchmarks）
- 真實世界資料

## 生態系工具

| 工具 | 說明 |
|---|---|
| **cjio** | Python CLI 工具，用於處理與操作 CityJSON 檔案[^cjio] |
| **CityJSON Ninja** | 線上 3D 檢視器，支援拖放上傳[^ninja] |
| **CityJSON Validator** | 線上驗證工具[^validator] |
| **QGIS Plugin** | QGIS 外掛 |
| **Three.js Loader** | Three.js 載入器 |
| **Blender Add-on** | Blender 外掛 |
| **DuckDB Extension** | DuckDB 擴充 |

## 直接下載連結整理

### 簡單幾何測試檔

```
https://3d.bk.tudelft.nl/opendata/cityjson/simplegeom/v2.0/cube.city.json
https://3d.bk.tudelft.nl/opendata/cityjson/simplegeom/v2.0/tetra.city.json
https://3d.bk.tudelft.nl/opendata/cityjson/simplegeom/v2.0/torus.city.json
https://3d.bk.tudelft.nl/opendata/cityjson/simplegeom/v2.0/msol.city.json
https://3d.bk.tudelft.nl/opendata/cityjson/simplegeom/v2.0/csol.city.json
https://3d.bk.tudelft.nl/opendata/cityjson/simplegeom/v2.0/twocube.city.json
https://3d.bk.tudelft.nl/opendata/cityjson/simplegeom/v2.0/geomtemplate.city.json
```

### 入門範例

```
https://www.cityjson.org/tutorials/files/twobuildings.city.json
```

### 擴充示範

```
https://www.cityjson.org/tutorials/files/noise_data.city.json
```

### 真實城市資料（v2.0）

```
https://3d.bk.tudelft.nl/opendata/cityjson/3dcities/v2.0/DenHaag_01.city.json
https://3d.bk.tudelft.nl/opendata/cityjson/3dcities/v2.0/Ingolstadt.city.json
https://3d.bk.tudelft.nl/opendata/cityjson/3dcities/v2.0/VM05_2009.city.json
https://3d.bk.tudelft.nl/opendata/cityjson/3dcities/v2.0/DA13_3D_Buildings_Merged.city.json
https://3d.bk.tudelft.nl/opendata/cityjson/3dcities/v2.0/LoD3_Railway.city.json
https://3d.bk.tudelft.nl/opendata/cityjson/3dcities/v2.0/3-20-DELFSHAVEN.city.json
https://3d.bk.tudelft.nl/opendata/cityjson/3dcities/v2.0/Vienna_102081.city.json
https://3d.bk.tudelft.nl/opendata/cityjson/3dcities/v2.0/Zurich_Building_LoD2_V10.city.json
```

## 參考資源

| 資源 | 網址 |
|---|---|
| CityJSON 官方網站 | https://www.cityjson.org/ |
| CityJSON 樣本資料集總頁面 | https://www.cityjson.org/datasets/ |
| CityJSON GitHub 組織 | https://github.com/cityjson |
| cityjson-corpus | https://3dgi.github.io/cityjson-corpus/ |
| 3DBAG（荷蘭） | https://3dbag.nl/ |
| 開放城市資料目錄 | https://3d.bk.tudelft.nl/opendata/opencities/ |
| OGC CityJSON 標準 | https://www.ogc.org/standards/cityjson/ |

[^cityjson]: CityJSON. (n.d.). *CityJSON — A compact JSON format for 3D city models*. Retrieved 2026-09-25, from https://www.cityjson.org/

[^ogc]: Open Geospatial Consortium. (n.d.). *CityJSON Community Standard (20-072r5)*. Retrieved 2026-09-25, from https://www.ogc.org/standards/cityjson/

[^datasets]: CityJSON. (n.d.). *Datasets*. Retrieved 2026-09-25, from https://www.cityjson.org/datasets/

[^tutorial]: CityJSON. (n.d.). *Getting Started with CityJSON*. Retrieved 2026-09-25, from https://www.cityjson.org/tutorials/getting-started/

[^3dbag]: 3D BAG. (n.d.). *3D BAG — 3D Building data of the Netherlands*. Retrieved 2026-09-25, from https://3dbag.nl/

[^pdok]: PDOK. (n.d.). *3D Basisvoorziening*. Retrieved 2026-09-25, from https://www.pdok.nl/3d-basisvoorziening

[^hdb]: Urban Analytics Lab, Singapore. (n.d.). *hdb3d-data*. Retrieved 2026-09-25, from https://github.com/ualsg/hdb3d-data

[^corpus]: 3D Geoinformation Group. (n.d.). *cityjson-corpus*. Retrieved 2026-09-25, from https://3dgi.github.io/cityjson-corpus/

[^cjio]: CityJSON. (n.d.). *cjio — CityJSON Input/Output*. Retrieved 2026-09-25, from https://github.com/cityjson/cjio

[^ninja]: CityJSON. (n.d.). *CityJSON Ninja — Online 3D Viewer*. Retrieved 2026-09-25, from https://ninja.cityjson.org/

[^validator]: CityJSON. (n.d.). *CityJSON Validator*. Retrieved 2026-09-25, from https://validator.cityjson.org/