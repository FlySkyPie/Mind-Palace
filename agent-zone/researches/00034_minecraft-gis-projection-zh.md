# 使用正規 GIS 投影（特別是 EPSG:3857）顯示 Minecraft 地圖的研究

## 概述

本報告調查是否有任何專案或個人使用正規 GIS（地理資訊系統）技術，尤其是 **EPSG:3857（Web Mercator）** 投影來顯示 Minecraft 地圖。調查結果顯示，現有的 Minecraft 地圖渲染生態系大致可分為兩個方向：一是「將 Minecraft 世界渲染為 Web 地圖」，二是「將真實 GIS 資料匯入 Minecraft 世界」。在後者中有明確使用 EPSG:3857 的專案，而前者則多採用自訂投影。

---

## 方向一：Minecraft 世界 → 瀏覽器地圖（多為自訂投影）

這類工具解決的問題是：將 Minecraft 世界（由區塊/chunk 構成的無限網格）轉換為可在瀏覽器中縮放瀏覽的瓦片地圖。這類工具通常不使用 EPSG:3857。

### 1. Minecraft Overviewer（自訂等軸投影，不使用 EPSG:3857）

Minecraft Overviewer 是目前最知名的 Minecraft 世界 Web 地圖渲染工具[^overviewer-github]，使用 Leaflet.js 作為前端，但其**不使用 EPSG:3857**。根據其官方設計文件：

> Minecraft 世界以近似的等軸投影（approximately isometric projection）從斜角渲染，投影作用如同眼睛從無限遠方以 45 度角從東南方向俯瞰世界。[^overviewer-design]

Overviewer 的瓦片系統使用**四元樹（quadtree）**定址方案，而非標準的 Web Mercator 瓦片網格。瓦片大小為 384×384 像素（等於一個 chunk section 的大小），並提供自訂的 Minecraft 世界座標與 Leaflet 地圖座標之間的轉換函式[^overviewer-tiles]。

該專案目前已**不再維護**，官方推薦的替代方案為 BlueMap。[^overviewer-github]

### 2. Dynmap 與 BlueMap

Dynmap 和 BlueMap 是另一個常見的 Minecraft 伺服器 Web 地圖解決方案：

- **Dynmap**：將 Minecraft 世界渲染為 HTML/JavaScript 地圖，使用自訂的 2D 正投影（top-down view），並非標準 GIS 投影。[^dynmap]
- **BlueMap**：使用 3D 渲染技術，支援自由旋轉視角，同樣不使用 EPSG:3857 標準投影。[^bluemap]

---

## 方向二：真實 GIS 資料 → Minecraft 世界（明確使用 EPSG:3857）

這類工具解決的問題相反：將地球上的真實地理資料（經緯度、高程、OSM 向量資料）轉換為 Minecraft 世界中的方塊。

### 3. Arnis — 最明確使用 Web Mercator 投影的專案（重點專案）

**Arnis**[^arnis-github]（17.9k stars，以 Rust + Tauri 開發）是一個將 OpenStreetMap 及高程資料生成 Minecraft 世界的工具。該專案**明確實作了 Web Mercator 投影**（即 EPSG:3857 所採用的數學模型）。

根據其原始碼[^arnis-webmercator]：

- 定義了 `ProjectionMode::WebMercator` 作為主要投影模式。
- 使用 `EARTH_RADIUS = 6,371,000.0` 公尺（WGS84 球面近似）。
- 提供 `forward()`（經緯度 → 投影座標）和 `inverse()`（投影座標 → 經緯度）雙向轉換。
- 支援以原點經緯度為中心的偏移與縮放（blocks-per-meter）。
- 輸出的 XZ 直接對應 Minecraft 世界中的方塊座標：X 向東增加而 Z 向北減少（即正北方為負 Z）。

Arnis 的 `WebMercatorProjection` 實作[^arnis-github-code]公開於 `src/projection/web_mercator.rs`，包含完整的單位測試（原點映射驗證、雙向往返驗證、尺度因數測試等）。

### 4. BuildTheEarth / Terra121（簡化比例投影）

BuildTheEarth 計畫[^bte]旨在以 1:1 比例（1 方塊 ≈ 1 公尺）在 Minecraft 中重建地球。其座標轉換方式[^terra121]為：

- X（Minecraft）= 經度 × 10⁵
- Z（Minecraft）= 緯度 × 10⁵
- Y（Minecraft）= 海拔（公尺）

這是一種簡化的**等距圓柱投影（equirectangular projection）**，並非正規的 Web Mercator（EPSG:3857），在全球高緯度地區會產生顯著的形狀變形。相關工具包含 `terraplusplus`（高效能分支）和 `terrapyconvert`（Python 套件）[^terraplusplus]。

### 5. GIS-2-MC / NYPL Historical Minecraft

一個使用 QGIS 結合 GRASS 與輪廓線工具的流程[^gis2mc]，將真實地圖的 TIFF 高程資料轉換為 Minecraft 世界。此流程使用 QGIS 處理地理參考（geo-rectified）地圖，匯出為點陣圖後以 Python 逐方塊生成 Minecraft 世界。屬於 GIS → Minecraft 方向，但並非直接使用 EPSG:3857。

### 6. Minecraft Earth Map（QGIS + WorldPainter）

此專案[^minecraft-earth-map]使用 QGIS 搭配 Python 腳本匯出 OpenStreetMap 瓦片，再透過 WorldPainter 匯入 Minecraft。提供座標計算器轉換真實經緯度與 Minecraft 遊戲內 X/Z 座標，但主要使用經緯度平面分瓦而非正規 Web Mercator。

---

## 方向三：逆向轉換的近似方法

### 7. Delaunay 三角剖分近似法

對於使用不明投影參數的 Minecraft 地球地圖，jkm.dev 提出了一種基於計算幾何的方法[^jkm]：

1. 在地圖上放置大量錨點（已知真實經緯度與 Minecraft X/Z 的對應關係）
2. 以錨點建立 Delaunay 三角網
3. 當需要轉換時，找出包含該點的三角形
4. 使用重心插值（barycentric interpolation）計算經緯度

這是一種無法得知投影參數時的實用替代方案。

---

## 結論

| 專案 | 方向 | 使用的 GIS 技術 | 使用 EPSG:3857？ | 主要用途 |
|---|---|---|---|---|
| **Minecraft Overviewer** | MC → Web 地圖 | Leaflet（自訂等軸 CRS） | ❌ 自訂投影 | 瀏覽器瀏覽 MC 世界 |
| **Dynmap / BlueMap** | MC → Web 地圖 | 自訂渲染引擎 | ❌ 自訂投影 | 伺服器 Web 地圖 |
| **Arnis** | GIS 資料 → MC | **Web Mercator（明確實作）**、OSM、高程 | ✅ **明確使用** | 生成真實世界 MC 地圖 |
| **BuildTheEarth / Terra121** | GIS 資料 → MC | 經度×10⁵ / 緯度×10⁵ | ❌ 簡化等距投影 | 1:1 地球 MC |
| **GIS-2-MC / NYPL** | GIS 資料 → MC | QGIS、GRASS | ❌ 使用地理參考座標 | 歷史地圖轉 MC |
| **Minecraft Earth Map** | GIS 資料 → MC | QGIS Python、WorldPainter | ❌ 經緯度平面分瓦 | 生成 MC 地球地圖 |
| **jkm.dev Dalaunay 法** | MC ↔ 經緯度 | Delaunay 三角剖分、重心插值 | ❌ 近似法 | 未知投影的座標查詢 |

**關鍵發現：**

1. **目前沒有專案使用正規 GIS（如 QGIS、PostGIS、GeoServer）搭配 EPSG:3857 來顯示或服務 Minecraft 地圖的瓦片。** Minecraft 地圖渲染工具（Overviewer、Dynmap、BlueMap）均使用自訂投影，而非標準 GIS 投影。

2. **Arnis 是唯一明確實作 Web Mercator（EPSG:3857 數學模型）的開源專案**，但其方向是將真實地理資料匯入 Minecraft（生成世界），而非將 Minecraft 世界輸出到 GIS 系統。

3. 這兩個方向的根本差異在於：Minecraft 世界的網格座標系統與真實地球的曲率/投影系統在本質上不同，將 Minecraft 世界視為 GIS 圖層需要解決座標原點、變形、以及 Minecraft 世界並非球面等根本問題。

[^arnis-github]: Erbkamm, L. (2022-2026). Arnis: Generate any location from the real world in Minecraft. Retrieved 2026-09-13, from https://github.com/louis-e/arnis
[^arnis-github-code]: Erbkamm, L. (n.d.). Arnis — web_mercator.rs. Retrieved 2026-09-13, from https://github.com/louis-e/arnis/blob/master/src/projection/web_mercator.rs
[^bluemap]: BlueColored. (n.d.). BlueMap: A Minecraft mapping tool. Retrieved 2026-09-13, from https://bluemap.bluecolored.de/
[^bte]: BuildTheEarth Team. (n.d.). BuildTheEarth: 1:1 scale Earth in Minecraft. Retrieved 2026-09-13, from https://buildtheearth.net/
[^dynmap]: dynmap. (n.d.). Dynmap: A dynamic, web-based map for Minecraft servers. Retrieved 2026-09-13, from https://github.com/webbukkit/dynmap
[^gis2mc]: tzerk. (n.d.). GIS-2-MC: Geographic Information System to Minecraft. Retrieved 2026-09-13, from https://github.com/tzerk/GIS-2-MC
[^jkm]: jkm.dev. (2023, updated 2026). Geographical Coordinates from Map Projection. Retrieved 2026-09-13, from https://www.jkm.dev/posts/geographical-coordinates-from-map-projection/
[^minecraft-earth-map]: Minecraft Earth Map. (n.d.). QGIS Tile Export. Retrieved 2026-09-13, from https://earth.motfe.net/tiles-qgis/
[^overviewer-design]: The Overviewer Team. (n.d.). Design Documentation — Overviewer Docs. Retrieved 2026-09-13, from https://docs.overviewer.org/en/latest/design/designdoc/
[^overviewer-github]: The Overviewer Team. (n.d.). Minecraft-Overviewer. Retrieved 2026-09-13, from https://github.com/overviewer/Minecraft-Overviewer
[^overviewer-tiles]: The Overviewer Team. (n.d.). Tile Rendering — Design Documentation. Retrieved 2026-09-13, from https://docs.overviewer.org/en/latest/design/designdoc/#tile-layout
[^terra121]: orangeadam3. (n.d.). Terra121 — Calculate Coordinates. Retrieved 2026-09-13, from https://github.com/orangeadam3/terra121/blob/master/CALCULATECOORDS.md
[^terraplusplus]: BuildTheEarth. (n.d.). terraplusplus. Retrieved 2026-09-13, from https://github.com/BuildTheEarth/terraplusplus