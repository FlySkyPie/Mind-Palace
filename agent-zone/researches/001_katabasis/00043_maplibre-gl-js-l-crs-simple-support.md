# MapLibre GL JS 是否支援類似 Leaflet 的 L.CRS.Simple 功能？

## 問題

Leaflet 提供 `L.CRS.Simple`，讓開發者能在任意座標系統（如像素座標、平面圖、遊戲地圖）上渲染地圖，而不受限於真實的地理座標（經緯度）。本研究探討 MapLibre GL JS 是否有內建或類似的功能。

## 結論

**MapLibre GL JS 沒有內建支援類似 L.CRS.Simple 的自訂/非地理座標系統。** 它從底層建立在 Web Mercator (EPSG:3857) 與 WGS84 地理座標之上，目前不存在任何可切換到任意像素座標系統的內建機制。[^issue168]

## 詳細現狀

### 官方回覆

- **Issue #168**（2021 年 6 月提出）請求「支援多種 CRS 渲染」，至今仍處於**開啟**狀態，標註為「PR is more than welcomed」，但無實作計畫。[^issue168]
- **Discussion #163**（專案層級討論）提議「自訂座標系統 / EPSG / 非 Mercator 圖磚」，列於 [MapLibre GL JS 官方路線圖](https://maplibre.org/roadmap/maplibre-gl-js/non-mercator-projection/) 中作為**目標**（計畫包括「以非 Mercator 投影初始化地圖視圖」、「載入並顯示自訂座標系統的向量圖磚」、「指定 EPSG 座標系統與圖磚矩陣集」），但沒有時程表。[^disc163]
- **Discussion #4658**（2024 年 9 月）：維護者 HarelM 明確回覆：「**Currently it doesn't. You'll need to convert your data to WGS84 for maplibre to show it properly.**」[^disc4658]
- 2025 年 8 月，HarelM 再次表示：「I don't think this will land soon... but you should be able to convert the coordinate system you have to WGS84 in order to work with MapLibre.」[^disc163]

### 已知的權宜方案

1. **「任意畫布」手法**：將地理邊界框 `[-180, -90, 180, 90]` 視為抽象畫布，自行將自訂/像素座標換算為這個經緯度空間內的相對位置。這是最接近 L.CRS.Simple 的做法，但需要自行處理所有座標換算。[^issue168]

2. **將所有資料轉換為 WGS84**：預先處理座標（平面圖、像素網格、遊戲地圖）成經緯度數值，再使用 MapLibre 正常渲染。這是維護者推薦的做法。[^disc4658]

3. **自訂 WebGL 圖層**：使用 `CustomLayerInterface` 撰寫自己的 WebGL 渲染程式碼，繞過 MapLibre 的座標系統，但也會失去 MapLibre 內建的圖磚載入、樣式與互動功能。[^customlayer]

4. **改用其他函式庫**：對於平面圖、遊戲地圖或任意座標系統，**Leaflet 搭配 L.CRS.Simple** 與 [`leaflet-rastercoords`](https://github.com/commenthol/leaflet-rastercoords) 等外掛仍然是最佳選擇。Discussion #163 中的評論也指出這點。[^disc163]

5. **maplibre-gl-indoor 外掛**：支援多樓層室內地圖，但內部仍基於 WGS84 座標——它僅加入了「樓層」維度，並非任意座標系統。[^indoor]

## 關鍵結論

MapLibre GL JS 是一個基於 Web Mercator/WGS84 的地理地圖渲染引擎，**並非為非地理/像素座標地圖所設計**。若主要用途為平面圖、遊戲地圖或任意像素座標地圖，Leaflet 搭配 L.CRS.Simple 仍是正確的工具。若必須使用 MapLibre，則需要預先將資料轉換為 WGS84，或使用「任意畫布」的取巧手法。

---

[^issue168]: MapLibre GL JS Issue #168. *Support rendering in multiple CRS*. Retrieved 2026-09-13, from https://github.com/maplibre/maplibre-gl-js/issues/168

[^disc163]: MapLibre Discussion #163. *Custom coordinate system / EPSG / non-Mercator tiles*. Retrieved 2026-09-13, from https://github.com/maplibre/maplibre/discussions/163

[^disc4658]: MapLibre GL JS Discussion #4658. *Loading Vector Tile Layers with custom projections*. Retrieved 2026-09-13, from https://github.com/maplibre/maplibre-gl-js/discussions/4658

[^customlayer]: MapLibre GL JS API. *CustomLayerProjectionData*. Retrieved 2026-09-13, from https://maplibre.org/maplibre-gl-js/docs/API/type-aliases/CustomLayerProjectionData/

[^indoor]: TUM-Dev. *maplibre-gl-indoor: Multi-floor MapLibre plugin*. Retrieved 2026-09-13, from https://github.com/TUM-Dev/maplibre-gl-indoor