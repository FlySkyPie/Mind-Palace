# Grafana GIS 地圖圖磚與圖層支援調查

## 概述

Grafana 從 v8.1 開始引入 **Geomap** 面板，提供完整的地理空間資料視覺化能力。支援多種底圖（Basemap）圖磚來源、自訂圖層疊加，以及多種地理資料格式。

---

## 底圖圖磚（Basemap Tiles）支援

Grafana Geomap 面板內建五種底圖圖磚類型[^geomap-doc]：

| 底圖類型 | 說明 |
|---|---|
| **OpenStreetMap** | 免費協作式全球地理資料庫圖磚 |
| **CARTO basemap** | CARTO Raster 底圖服務 |
| **ArcGIS MapServer** | ESRI ArcGIS MapServer 圖層 |
| **XYZ Tile layer** | **通用的自訂圖磚圖層**，可指向任何遵循 XYZ 慣例的圖磚伺服器 |
| **MapLibre Style layer** | 支援 MapLibre/Mapbox 樣式 URL，可自訂地圖樣式與資料來源 |

其中 **XYZ Tile layer** 與 **MapLibre Style layer** 是關鍵功能，允許使用者接入自訂圖磚伺服器（例如自架 OpenStreetMap 圖磚、Mapbox、Google Maps 風格或其他 GIS 圖磚服務）。

---

## 資料圖層（Data Layers）支援

Geomap 支援九種資料圖層類型，用來疊加在地圖上展示資料[^geomap-doc]：

| 圖層類型 | 說明 |
|---|---|
| **Markers** | 在每個資料點顯示標記（圓形、方形、星形等） |
| **Heatmap** | 資料點密度熱力圖 |
| **GeoJSON** | 從 GeoJSON 檔案載入靜態地理資料 |
| **Night / Day** | 渲染日夜區域 |
| **Route** | 將資料點渲染為路線 |
| **Photos** | 在資料點顯示照片 |
| **Network** | 從資料繪製網路圖 |
| **Icon at last point (Alpha)** | 僅在最後資料點顯示圖示（實驗性） |
| **Dynamic GeoJSON (Alpha)** | 根據查詢結果動態設定 GeoJSON 樣式（實驗性） |

---

## 地理資料格式支援

Geomap 面板支援三種地理資料格式[^geomap-doc]：

1. **經緯度（Latitude / Longitude）** — 欄位命名為 `latitude`/`lat` 與 `longitude`/`lon`/`lng`
2. **Geohash** — 欄位命名為 `geohash`
3. **查詢碼（Lookup）** — 國家代碼、美國州碼、機場代碼

支援自動偵測欄位名稱，亦可手動指定。

---

## 多圖層疊加與控制

- 支援在同一個地圖上疊加**多個圖層**
- 可調整圖層順序（拖曳排序）
- 每個圖層可獨立命名、刪除、重新設定
- 支援「Share view」選項，讓多個地圖面板的平移與縮放同步
- 支援將地圖視角範圍同步至儀表板變數，用於動態查詢

---

## 結論

Grafana **完全支援 GIS 地圖圖磚與圖層**：

- 可透過 **XYZ Tile layer** 接入任何標準圖磚伺服器（自訂圖磚來源）
- 可透過 **MapLibre Style layer** 使用 MapLibre/Mapbox 樣式 URL
- 內建 OpenStreetMap、CARTO、ArcGIS 等底圖選項
- 支援多層資料疊加（標記、熱力圖、GeoJSON、路線、網路等）
- 支援經緯度、Geohash、國家/地區查詢碼等地理資料格式

---

[^geomap-doc]: Grafana Labs. (n.d.). Geomap visualization. Grafana Documentation. Retrieved 2026-09-13, from https://grafana.com/docs/grafana/latest/visualizations/panels-visualizations/visualizations/geomap/