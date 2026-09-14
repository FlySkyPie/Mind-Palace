# Grafana GIS 時間序列支援調查

## 概述

本報告調查 Grafana 是否支援 GIS（地理資訊系統）結合時間維度的資料視覺化，包括原生地圖面板、時間序列能力、動畫播放、即時追蹤等面向。

## 原生 GIS / 地圖面板

Grafana 提供兩種地圖面板選項：

- **Geomap Panel（核心內建）**：自 Grafana v8.1 起提供的原生地圖視覺化元件，完全取代舊版 Worldmap 外掛。支援 **Markers（標記）**、**Heatmap（熱力圖）**、**GeoJSON**、**Route（路線）**、**Network（網路）**、**Photos（照片）**、**Night/Day（日夜）** 等資料圖層，以及多種底圖選項（OpenStreetMap、CARTO、ArcGIS、XYZ tiles、MapLibre）。[^geomap-doc]

- **Worldmap Panel（社群外掛 / 已棄用）**：`grafana-worldmap-panel` 社群外掛現已棄用，建議改用 Geomap。[^worldmap-dep]

支援的位置資料格式：**經緯度（Latitude/Longitude）**、**Geohash**、以及**查閱碼（Lookup codes）**（國家、美國州名、機場代碼）。[^geomap-location]

## 時間序列資料於地圖上的顯示

可以。Geomap 面板與資料源無關，只要查詢回傳的欄位包含位置資訊，即可搭配任一時序資料源（Prometheus、InfluxDB、PostgreSQL、Graphite 等）。[^geomap-doc]

- **標準時間範圍控制器**：整個儀表板的時間範圍選擇器會影響所有面板（含 Geomap），調整時間範圍時 Geomap 會重新查詢並更新地圖資料。[^time-range]
- **自動重新整理（auto-refresh）**：搭配定時自動更新，可在近即時場景下呈現資料變化。官方文件明確指出：「當您有即時變化的位置資料，並想視覺化物件如何移動時，Geomap 搭配 auto-refresh 非常有用。」[^geomap-realtime]

**然而，Geomap 面板內建並無時間軸播放按鈕或時間滑桿（time-slider）能在地圖上逐格瀏覽時間變化。**[^geomap-limits]

## 動畫 / 時間滑桿功能

Grafana **無內建**地圖時間軸動畫功能，但有社群或外部方案：

### 1. GeoLoop Panel 外掛（社群）

此為 Grafana 面板外掛，專用於將 GeoJSON 與時間序列資料結合，並以迴圈方式動畫地理特徵：[^geoloop]

- 可將 Polygon、Point、Line 屬性連結至 Grafana 取得的動態資料
- 應用案例：視覺化氣象站每小時溫度變化、降雨隨時間演變
- **注意**：該外掛的「動畫速度/迴圈選項」仍標示為「尚未實作」（Not yet implemented）[^geoloop-limits]
- 需 GeoJSON 檔案以 JSONP callback 提供、MapBox API Key、以及 InfluxDB

### 2. grafanimate（外部 Python 工具）

此為**非 Grafana 外掛**，而是一套 Python CLI 工具：[^grafanimate]

- 透過操作 Grafana 儀表板的時間範圍控制器，逐步移動時間並截圖
- 輸出為 PNG 序列、動態 GIF 或影片
- **可作用於任何儀表板與面板**（含 Geomap）
- 使用案例：Luftdaten.info 感測器涵蓋範圍隨時間變化、跨年夜細懸浮微粒擴散動畫
- 底層使用 Firefox（透過 Marionette）自動化瀏覽器操作

### 3. 社群功能請求

Grafana 社群論壇有活躍的功能請求討論，期望能像氣象雷達圖般在地圖上顯示隨時間變化的標記，但該功能尚未原生實作。[^community-request]

## 即時移動物體追蹤

**可以。** Grafana 支援在地圖上追蹤移動物體（如車隊），並為此提供多項功能：[^questdb-tutorial]

### 即時追蹤（搭配 auto-refresh）
- 查詢資料源取得車輛最新位置（如 `SELECT * FROM buses LATEST BY plate`）
- 啟用 Grafana 的 auto-refresh 定期輪詢新位置
- Markers 圖層可設定：
  - **旋轉角度（rotation）**：對應行進方位角（bearing/heading）
  - **顏色（color）**：對應速度或其他度量
  - **標籤（labels）**：顯示車輛 ID
  - **自訂符號**：SVG 圖示區分不同車型

### 路線/歷史軌跡（Beta）
- **Route 圖層**（Beta 功能）可將資料點渲染為路線/路徑
- 可依速度著色，並顯示行進方向箭頭
- **限制**：目前 Grafana 無法在單一圖層區分多條路線，每輛車的路線需各自獨立的圖層或面板。可使用「repeat by」面板選項搭配 Grafana 變數，為每輛車建立獨立面板。[^route-bug]

### 進階標記功能
- **「Icon at last point」（Alpha）**：僅在最後一個資料點顯示圖示，適合表示移動物體當前位置
- **「Dynamic GeoJSON」（Alpha）**：根據查詢結果動態設定 GeoJSON 樣式
- **「Fit to data → Last value」**：地圖視角自動對齊最新資料點

## 可用外掛一覽

| 外掛名稱 | 說明 | 狀態 |
|---|---|---|
| **Geomap（核心）** | 內建地圖，支援標記、熱力圖、GeoJSON、路線、網路、照片圖層 | ✅ 現行推薦 |
| **Worldmap Panel** | 舊版圖塊地圖，圓形覆蓋圖 | ❌ 已棄用 |
| **GeoLoop Panel** | 將 GeoJSON 與時間序列結合的動畫地圖 | ⚠️ 社群 / 動畫功能未完整 |
| **Geomap WMS Panel** | 新增 OGC Web Map Service（WMS 1.3.0）底圖支援 + 空間查詢過濾 | ✅ 社群 |
| **Mapgl** | 在地圖上呈現網路圖（node-graph），WebGL 渲染、下鑽 | ✅ 社群 |
| **Orchestra Cities Map** | 擴充 Geomap：GeoJSON 形狀、FontAwesome 圖示、彈出視覺化、叢集 | ✅ 社群 |
| **grafanimate**（外部） | Python CLI 工具：操作時間範圍製作動畫 GIF/影片 | 🔧 外部工具 |

## 結論

| 能力 | 支援狀態 |
|---|---|
| 原生地圖面板（Geomap） | ✅ 內建 |
| 地圖顯示時間序列資料 | ✅ 透過儀表板時間選擇器 + auto-refresh |
| 地圖動畫 / 時間滑桿 | ❌ 無原生支援 — 需 GeoLoop 外掛（部分）或 grafanimate 外部工具 |
| 車輛即時追蹤 | ✅ 透過 Markers + rotation + color + auto-refresh 完整支援 |
| 路線/歷史軌跡視覺化 | ⚠️ Beta（Route 圖層）；單一圖層僅支援單一實體 |
| 多車輛同時追蹤 | ⚠️ 需透過「repeat by」面板選項或分層處理 |
| GIS 外掛生態系 | ✅ WMS、Mapgl、GeoJSON、動畫地圖等社群外掛 |

Grafana 具備基礎的 GIS 地圖能力與時間序列整合，可支援即時位置追蹤與歷史路線繪製。然而，若需要**在地圖上播放時間動畫**（如氣象雷達逐格播放、軌跡歷程動畫），則需依賴 GeoLoop 外掛或外部工具 grafanimate，此功能目前非 Grafana 原生支援。

[^geomap-doc]: Grafana Labs. (n.d.). Geomap visualization. Retrieved 2026-09-13, from https://grafana.com/docs/grafana/latest/visualizations/panels-visualizations/visualizations/geomap/
[^geomap-location]: Grafana Labs. (n.d.). Geomap — Location data. Retrieved 2026-09-13, from https://grafana.com/docs/grafana/latest/visualizations/panels-visualizations/visualizations/geomap/
[^worldmap-dep]: Grafana Labs. (n.d.). Worldmap panel (deprecated). Retrieved 2026-09-13, from https://github.com/grafana/worldmap-panel
[^time-range]: Grafana Labs. (n.d.). Dashboard time range controls. Retrieved 2026-09-13, from https://grafana.com/docs/grafana/latest/visualizations/dashboards/use-dashboards/
[^geomap-realtime]: Grafana Labs. (n.d.). Geomap — Real-time tracking. Retrieved 2026-09-13, from https://grafana.com/docs/grafana/latest/visualizations/panels-visualizations/visualizations/geomap/#real-time-tracking
[^geomap-limits]: Grafana Community. (n.d.). Is it possible to show markers along the time? Retrieved 2026-09-13, from https://community.grafana.com/t/is-it-possibile-to-show-markers-along-the-time-asking-for-hints/128031
[^geoloop]: CitiLogics. (n.d.). GeoLoop Panel — Grafana plugin for animating geo features over time. Retrieved 2026-09-13, from https://github.com/CitiLogics/citilogics-geoloop-panel
[^geoloop-limits]: CitiLogics. (n.d.). GeoLoop Panel — Animation Speed/Looping Options (Not yet implemented). Retrieved 2026-09-13, from https://github.com/CitiLogics/citilogics-geoloop-panel
[^grafanimate]: grafana-toolbox. (n.d.). grafanimate — Python CLI to animate Grafana dashboards via time range manipulation. Retrieved 2026-09-13, from https://github.com/grafana-toolbox/grafanimate
[^community-request]: Grafana Community. (n.d.). Marker Animations in Grafana Geomap Dashboard Route Layer. Retrieved 2026-09-13, from https://community.grafana.com/t/marker-animations-in-grafana-geomap-dashboard-route-layer/112844
[^questdb-tutorial]: QuestDB. (2023). Working with Grafana Maps Markers — Tracking Sydney Buses. Retrieved 2026-09-13, from https://questdb.com/blog/working-with-grafana-maps-markers/
[^route-bug]: Grafana. (n.d.). GitHub Issue #72878 — Route layer connecting first/last data points of grouped data. Retrieved 2026-09-13, from https://github.com/grafana/grafana/issues/72878