# 時間變化 GPS 資料在 Grafana Geomap 的資料格式

## 問題

若要將「隨時間改變的 GPS 資料，例如貨車」的資訊在 Grafana 的 Geomap 顯示，IoT 端應該輸出什麼樣的資料？

---

## 結論

IoT 端每筆 GPS 定位資料應至少包含三個欄位：**時間戳記 (time)、緯度 (lat)、經度 (lon)**。用標準表格表示即為：

| time (epoch ms 或 datetime) | lat (float) | lon (float) | vehicle_id (string) | speed (float) |
|-----------------------------|-------------|-------------|----------------------|---------------|
| 1700000000000               | 46.0        | 6.0         | truck-1              | 55.2          |
| 1700000001000               | 46.1        | 6.1         | truck-1              | 57.0          |
| 1700000002000               | 46.2        | 6.2         | truck-1              | 58.4          |

此格式直接對應 Grafana 內部使用的 **DataFrame** 結構，也是 Geomap 面板唯一能消費的資料形式[^official-geomap-docs]。

## Geomap 面板支援的資料格式

Geomap 面板不直接吃原始 IoT 協定（MQTT、HTTP JSON 等）的 Payload，而是消費**資料源查詢回傳的 DataFrame**。DataFrame 中的欄位名稱和型別決定地圖如何繪製位置。支援的 Location 模式有三種[^official-geomap-docs]：

1. **Coords（經緯度座標）** — 兩個數值欄位（latitude + longitude），手動對應
2. **Auto（自動偵測）** — 根據欄位名稱自動匹配，支援 `lat`/`latitude`、`lon`/`lng`/`longitude`、`geohash`、`lookup`、`h3`、`wkt`（忽略大小寫）[^location-ts-source]
3. **Geohash** — 一個字串欄位（geohash 編碼）
4. **Lookup** — 一個文字欄位搭配地名辭典（國家、美國州名、機場代碼）

GeoJSON 僅用於靜態底圖圖層（GeoJSON Layer / Dynamic GeoJSON Layer），不是動態車輛軌跡的主要格式[^official-geomap-docs]。

## 時間變化 GPS 的顯示方式

Geomap 面板本身**沒有時間軸播放動畫功能**。所有資料點在查詢結果中一併渲染，動態效果透過以下機制實現：

### A. Route 圖層 — 軌跡線

- 吃一個 DataFrame，將所有 Point 依時間順序串成一個 `LineString` 繪製為路線[^route-layer-source]
- **每個 Route 圖層只能處理一個 DataFrame（一台車輛）**；多台車輛需分別查詢、分別建立圖層[^multi-route-thread]

### B. Icon at last point（Alpha 功能）— 車輛即時位置圖標

- Alpha 階段功能，需啟用 `GF_PANELS_ENABLE_ALPHA=true` 環境變數或 `[panels] enable_alpha = true` 設定
- 每次資料更新時取 DataFrame 的**最後一筆（最新時間戳）**作為圖標位置[^last-point-tracker-source]
- 搭配儀表板 **Auto Refresh**（如 5-10 秒）即可達到「車輛移動」效果

### C. Markers 圖層 — 散佈所有定位點

- 每個資料列渲染為一個地圖標記，適合顯示查詢時間範圍內的全部歷史定位

### D. Route + Hover Crosshair — 時間連動

- Route 圖層可監聽其他面板（如 Time Series）的 `DataHoverEvent`，在時間軸上移動滑鼠時於地圖顯示對應位置的十字準星[^route-layer-source]

## 多台車輛的實作策略

由於 Route 和 Icon at last point 圖層目前都只處理「第一個 DataFrame」[^route-layer-source]，多車追蹤的建議做法：

1. **每台車輛一個資料源查詢**，各自獨立產生 DataFrame
2. **每個查詢對應一個 Geomap 圖層**，圖層疊加顯示
3. 社群有提出「Group by field」的 Route 圖層 PR（#102259），但尚未合入穩定版[^multi-route-thread]

## IoT 端資料輸出規範

綜合以上，IoT 端傳送 GPS 資料時應遵循：

### 必要欄位

| 欄位 | 格式 | 說明 |
|------|------|------|
| `time` | Unix epoch ms（整數）或 ISO 8601 字串 | 定位時間戳，Grafana 自動解析 |
| `lat` / `latitude` | 浮點數度數（WGS84） | 緯度，-90 至 90 |
| `lon` / `lng` / `longitude` | 浮點數度數（WGS84） | 經度，-180 至 180 |

### 建議額外欄位

| 欄位 | 類型 | 用途 |
|------|------|------|
| `vehicle_id` | 字串 | 車輛識別碼，用於查詢過濾或多車區分 |
| `speed` | 浮點數 | 可作為 Marker 大小/顏色對應的度量值 |
| `status` | 字串 | 狀態標籤，可作為圖標顏色分類 |

### 注意事項

- **lat/lon 必須在同一筆資料列**，不可分開發送到不同 Series（這是 InfluxDB 等時間序列資料庫的常見陷阱，需用 `pivot()` 或 SQL JOIN 合併）[^gis-community-post]
- 欄位名稱使用 `lat`/`lon` 可觸發 Auto 模式自動對應，節省手動設定

## 資料流總結

```mermaid
flowchart LR
    IoT["IoT 裝置<br/>GPS 定位器"] -->|"MQTT / HTTP<br/>{time, lat, lon, id}"| DB["時間序列資料庫<br/>InfluxDB / TimescaleDB / SQLite"]
    DB -->|"查詢回傳 DataFrame<br/>time | lat | lon | vehicle_id"| Grafana["Grafana Geomap<br/>Route / Markers / Icon-at-last-point"]
    Grafana -->|"Auto Refresh 5-10s"| Map["地圖即時更新"]
```

## 參考文獻

[^official-geomap-docs]: Grafana Labs. (n.d.). Geomap panel. Retrieved 2026-09-20, from https://grafana.com/docs/grafana/latest/panels-visualizations/visualizations/geomap/
[^location-ts-source]: Grafana. (n.d.). public/app/features/geo/utils/location.ts. Retrieved 2026-09-20, from https://github.com/grafana/grafana/blob/main/public/app/features/geo/utils/location.ts
[^route-layer-source]: Grafana. (n.d.). public/app/plugins/panel/geomap/layers/data/routeLayer.tsx. Retrieved 2026-09-20, from https://github.com/grafana/grafana/blob/main/public/app/plugins/panel/geomap/layers/data/routeLayer.tsx
[^last-point-tracker-source]: Grafana. (n.d.). public/app/plugins/panel/geomap/layers/data/lastPointTracker.ts. Retrieved 2026-09-20, from https://github.com/grafana/grafana/blob/main/public/app/plugins/panel/geomap/layers/data/lastPointTracker.ts
[^multi-route-thread]: Grafana Community. (2025). Show multiple routes in a single Geomap panel. Retrieved 2026-09-20, from https://community.grafana.com/t/show-multiple-routes-in-a-single-geomap-panel/112045
[^gis-community-post]: Grafana Community. (2024). Grafana Geomap Route. Retrieved 2026-09-20, from https://community.grafana.com/t/grafana-geomap-route/106009