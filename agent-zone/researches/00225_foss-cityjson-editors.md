# FOSS CityJSON 編輯器調查報告

## 摘要

本報告調查目前市面上可用的自由開源（FOSS）CityJSON 編輯與處理工具。CityJSON 是以 JSON 格式儲存 3D 城市模型的標準格式，基於 CityGML 資料模型。調查結果顯示，目前已有成熟且活躍維護的 FOSS 生態系，其中最值得推薦的圖形化編輯器為 **Up3date**（Blender 外掛），命令列批次處理則推薦 **cjio**。

---

## 1. 圖形化編輯器（GUI Editor）

### 1.1 Up3date — Blender 外掛（強力推薦）

- **儲存庫**：https://github.com/cityjson/Up3date
- **授權**：MIT
- **活躍度**：81 Stars，持續維護中（最後更新 2026 年 9 月）[^up3date]
- **功能**：官方 CityJSON Blender 外掛，支援 CityJSON 2.0 完整讀取、編輯、匯出。功能涵蓋：
  - 城市物件屬性、親子關係、擴充類型
  - 所有幾何基元：MultiPoint、MultiLineString、MultiSurface、CompositeSurface、Solid、CompositeSolid、MultiSolid
  - LoD 0–3（含小數 LoD，如 2.2）
  - 語義表面（透過 Blender 材質）
  - 座標轉換、CRS/後設資料、外觀、幾何模板
  - 完整往返：匯入 → 編輯 → 匯出，保留 CityJSON 結構
- **需求**：Blender 5.0+
- **備註**：由官方 `cityjson` GitHub 組織維護，是目前功能最完整的 FOSS 圖形化編輯器。

### 1.2 ninja — 網頁版檢視與編輯器

- **網站**：https://ninja.cityjson.org
- **儲存庫**：https://github.com/cityjson/ninja
- **授權**：Apache-2.0
- **活躍度**：57 Stars，持續維護中[^ninja]
- **功能**：官方 CityJSON 網頁檢視器，支援**編輯**（篩選、子集選取、轉換）。使用 Vue.js + Three.js 建構，無需安裝，開啟網頁上傳 `.json` 或 `.jsonl` 檔案即可使用。
- **備註**：適合快速檢視與簡單編輯，不需要安裝任何軟體。

### 1.3 CityJSON QGIS 外掛

- **儲存庫**：https://github.com/cityjson/cityjson-qgis-plugin
- **授權**：Apache-2.0
- **活躍度**：49 Stars，432 commits，持續維護（CI 測試支援 QGIS 3.40、3.44、4.x）[^qgis-plugin]
- **功能**：在 QGIS 中以圖層方式載入 CityJSON 資料集，支援 3D 檢視、依物件類型分割圖層、CityJSONSeq 支援。
- **備註**：適合 GIS 專業使用者，在 QGIS 工作流程中整合 CityJSON。

### 1.4 RhinoCityJSON — Rhino/Grasshopper 外掛

- **儲存庫**：https://github.com/cityjson/RhinoCityJSON
- **授權**：MIT
- **活躍度**：19 Stars，2026 年 9 月持續更新[^rhino]
- **功能**：在 Rhino 3D 與 Grasshopper 中讀取 CityJSON，支援語義資訊。
- **備註**：適合建築與工業設計領域使用者。

---

## 2. 命令列工具（CLI / Batch Processing）

### 2.1 cjio — Python CLI（強力推薦）

- **儲存庫**：https://github.com/cityjson/cjio
- **授權**：MIT
- **活躍度**：149 Stars，1,034 commits，極度活躍[^cjio]
- **功能**：Python CLI 與函式庫，用於批次處理 CityJSON 檔案。關鍵編輯操作：
  - `subset` — 依 ID、bbox、類型、半徑或隨機選取物件
  - `lod_filter` — 保留特定 LoD
  - `attribute_rename`、`attribute_remove` — 修改屬性
  - `merge` — 合併多個 CityJSON 檔案
  - `crs_assign`、`crs_reproject`、`crs_translate` — 座標操作
  - `triangulate`、`vertices_clean` — 幾何清理
  - `export` 至 GLB/OBJ/STL/B3DM/JSONL
  - `validate` — 結構驗證
  - `upgrade` — 從 v1.0/v1.1 升級至 v2.0
- **安裝**：`pip install cjio`
- **備註**：適合自動化腳本與批次處理。

---

## 3. AI 輔助編輯

### 3.1 CityJSON MCP / Datum

- **儲存庫**：https://github.com/Yarroudh/cityjson-mcp
- **授權**：MIT
- **活躍度**：3 Stars，45 commits，活躍開發中（2026 年專案）[^mcp]
- **功能**：37 個 MCP 工具，可用於檢視、驗證、轉換、查詢 CityJSON 模型。底層使用 cjio、cjval、val3dity、citygml-tools、cjdb。包含 **Datum** 網頁聊天應用，讓 LLM Agent 執行 CityJSON 操作。可與 Claude Desktop、Cursor、VS Code 搭配使用。支援 Docker。
- **備註**：適合希望以自然語言操作 CityJSON 的使用者。

---

## 4. 驗證工具

| 工具 | 連結 | 授權 | 備註 |
|------|------|------|------|
| **cjval** | https://github.com/cityjson/cjval | MIT | 官方結構驗證器（Rust），網頁版 https://validator.cityjson.org[^cjval] |
| **val3dity** | https://github.com/tudelft3d/val3dity | GPL-3.0 | 3D 幾何驗證（ISO19107），支援 CityJSON、IndoorGML、OBJ、OFF、JSON-FG[^val3dity] |

---

## 5. 轉換工具

| 工具 | 連結 | 功能 |
|------|------|------|
| **citygml-tools** | https://github.com/citygml4j/citygml-tools | CityGML ↔ CityJSON 雙向轉換 |
| **3dfier** | https://github.com/tudelft3d/3dfier | 2D GIS + LiDAR → 3D 城市模型（635 Stars，極活躍）|
| **tyler** | https://github.com/3DGI/tyler | CityJSON → 3D Tiles（含 glTF）|
| **cityjson2jsonfg** | https://github.com/3DGI/cityjson2jsonfg | CityJSON → OGC JSON-FG |
| **IFCCityJSON** | https://github.com/IfcOpenShell/IfcOpenShell | CityJSON ↔ IFC |
| **cityparquet** | https://github.com/cityjson/cityparquet | CityJSON → Parquet 欄位編碼 |

---

## 6. 資料庫 / 儲存

| 工具 | 連結 | 授權 | 功能 |
|------|------|------|------|
| **cjdb** | https://github.com/cityjson/cjdb | MIT | CityJSONL ↔ PostgreSQL+PostGIS 匯入匯出 |
| **3D City DB** | http://www.3dcitydb.org | FOSS | PostGIS/Oracle 上的 3D 城市模型地理資料庫 |
| **duckdb-cityjson** | https://github.com/cityjson/duckdb-cityjson | MIT | （實驗性）DuckDB 擴充，支援 CityJSON |

---

## 7. 建議與使用情境

| 使用情境 | 最佳 FOSS 工具 |
|----------|---------------|
| 編輯幾何與屬性（GUI） | **Up3date**（Blender）— 功能最完整的圖形化編輯器，支援完整往返 |
| 快速檢視與輕量編輯（免安裝） | **ninja** — 瀏覽器即可執行 |
| 批次處理與腳本自動化 | **cjio** — Python CLI，鏈式操作 |
| GIS 環境中檢視 | **CityJSON QGIS 外掛** |
| AI 輔助處理 | **CityJSON MCP / Datum** |
| 結構驗證 | **cjval** |
| 3D 幾何驗證 | **val3dity** |
| CityGML 轉換 | **citygml-tools** |
| 2D GIS → 3D 模型 | **3dfier** |
| 資料庫儲存 | **cjdb** |
| 3D Tiles 建立 | **tyler** |

---

## 8. 結論

CityJSON 擁有完整且活躍的 FOSS 生態系。對於編輯需求，**Up3date**（Blender 外掛）是目前唯一提供完整圖形化編輯與匯出功能且支援 CityJSON 2.0 的工具；**ninja** 提供免安裝的網頁編輯體驗；**cjio** 則是最強大的命令列處理工具。所有工具皆持續維護中。[^cityjson-org]

---

## 參考資料

[^up3date]: cityjson. (n.d.). *Up3date — Blender add-on for CityJSON*. GitHub. Retrieved 2026-09-25, from https://github.com/cityjson/Up3date

[^ninja]: cityjson. (n.d.). *ninja — CityJSON web viewer/editor*. GitHub. Retrieved 2026-09-25, from https://github.com/cityjson/ninja

[^qgis-plugin]: cityjson. (n.d.). *cityjson-qgis-plugin*. GitHub. Retrieved 2026-09-25, from https://github.com/cityjson/cityjson-qgis-plugin

[^rhino]: cityjson. (n.d.). *RhinoCityJSON*. GitHub. Retrieved 2026-09-25, from https://github.com/cityjson/RhinoCityJSON

[^cjio]: cityjson. (n.d.). *cjio — CityJSON CLI and Python library*. GitHub. Retrieved 2026-09-25, from https://github.com/cityjson/cjio

[^mcp]: Yarroudh, A. (2026). *cityjson-mcp — MCP server for CityJSON*. GitHub. Retrieved 2026-09-25, from https://github.com/Yarroudh/cityjson-mcp

[^cjval]: cityjson. (n.d.). *cjval — CityJSON validator*. GitHub. Retrieved 2026-09-25, from https://github.com/cityjson/cjval

[^val3dity]: tudelft3d. (n.d.). *val3dity — 3D primitive validator*. GitHub. Retrieved 2026-09-25, from https://github.com/tudelft3d/val3dity

[^cityjson-org]: cityjson. (n.d.). *CityJSON Software*. Retrieved 2026-09-25, from https://www.cityjson.org/software/