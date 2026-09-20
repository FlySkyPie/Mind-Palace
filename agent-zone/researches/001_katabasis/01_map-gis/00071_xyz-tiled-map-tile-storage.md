# XYZ 圖磚地圖的儲存方式

## 概述

XYZ 圖磚地圖（也稱為 Tiled Web Map 或 Slippy Map），是一種將地圖切分為多張小型圖片（稱為「圖磚」，tile）的技術，使用者瀏覽地圖時客戶端只會載入當前視野範圍內所需的圖磚，而不需要一次載入整張大地圖[^wiki]。這種技術最早由 Google Maps 推廣，後來成為 OpenStreetMap 等服務的標準做法。

## 圖磚的命名慣例（XYZ / Slippy Map）

在 OpenStreetMap 的 Slippy Map 標準（也稱為 XYZ 標準）中，每張圖磚的 URL 遵循以下格式[^osm]：

```
http://.../Z/X/Y.png
```

- **Z**：縮放等級（Zoom level），整數，從 0（最遠、最縮小）到 19 以上（最近、最放大）。
- **X**：欄位（Column），由左至右計數，範圍從 0 到 2^Z - 1。
- **Y**：列位（Row），由上至下計數，範圍從 0 到 2^Z - 1。

以 OpenStreetMap 官方伺服器為例[^osm]：

```
https://tile.openstreetmap.org/18/232798/103246.png
```

這代表縮放等級 18、X 座標 232798、Y 座標 103246 的圖磚。

## 圖磚的核心規格

- **尺寸**：每張圖磚為 256×256 像素，高解析度（Retina）顯示則可能採用 512×512 像素[^wiki]。
- **格式**：傳統為 PNG（柵格圖磚），新式地圖也支援 MVT（Mapbox Vector Tile）向量圖磚格式，向量圖磚可在客戶端自行進行樣式繪製[^wiki]。
- **投影**：Web Mercator（EPSG:3857），緯度限制在約 ±85.0511° 範圍內，整個地圖呈現正方形[^wiki]。

## 縮放等級與圖磚數量

在 Zoom Level Z 時，整個地圖由 2^Z × 2^Z 張圖磚組成[^osm]：

| Zoom Level | 圖磚網格 | 總圖磚數 |
|-----------|---------|---------|
| 0 | 1 × 1 | 1 |
| 1 | 2 × 2 | 4 |
| 2 | 4 × 4 | 16 |
| ... | ... | ... |
| 12 | 4096 × 4096 | 16,777,216 |
| 16 | 65536 × 65536 | 4,294,967,296（約43億）|
| 18 | 262144 × 262144 | 68,719,476,736（約687億）|

## 圖磚座標系的計算方式

給定經緯度座標與縮放等級，圖磚的 XYZ 編號可透過以下換算得出[^osm]：

```python
import math
def deg2num(lat_deg, lon_deg, zoom):
    lat_rad = math.radians(lat_deg)
    n = 2.0 ** zoom
    xtile = int((lon_deg + 180.0) / 360.0 * n)
    ytile = int((1.0 - math.asinh(math.tan(lat_rad)) / math.pi) / 2.0 * n)
    return (xtile, ytile)
```

反向換算（從圖磚編號回經緯度，回傳圖磚左上角座標）[^osm]：

```python
def num2deg(xtile, ytile, zoom):
    n = 2.0 ** zoom
    lon_deg = xtile / n * 360.0 - 180.0
    lat_rad = math.atan(math.sinh(math.pi * (1 - 2 * ytile / n)))
    lat_deg = math.degrees(lat_rad)
    return (lat_deg, lon_deg)
```

## 三種主要的圖磚編號方案

目前業界主要有三種不同的圖磚編號方式[^wiki][^tms]：

### 1. XYZ（Google Maps / OpenStreetMap 標準）

Y 軸由上往下數（北→南），左上角為 (0, 0)。URL 格式為 `/Z/X/Y.png`。這是目前最普遍使用的方案。

### 2. TMS（Tile Map Service）

由 OSGeo（Open Source Geospatial Foundation）制定，Y 軸由下往上數（南→北），與 XYZ 的 Y 值相反（即 `Y_TMS = 2^Z - 1 - Y_XYZ`）[^tms]。

### 3. QuadTree（四叉樹，Microsoft Bing Maps）

Microsoft Bing Maps 採用四叉樹編號，將每張圖磚編碼為單一的十進位或四進位數字字串，而非獨立的 X/Y 值[^wiki]。

## 檔案系統層級的儲存結構

雖然在網路上圖磚以 URL 路徑呈現，但在伺服器的檔案系統中，圖磚的儲存結構通常直接對應 URL 路徑格式[^osm]：

```
tiles/
├── 0/
│   ├── 0/
│   │   └── 0.png
├── 1/
│   ├── 0/
│   │   ├── 0.png
│   │   └── 1.png
│   ├── 1/
│   │   ├── 0.png
│   │   └── 1.png
├── 2/
│   ├── 0/
│   │   ├── 0.png
│   │   ├── 1.png
│   │   ├── 2.png
│   │   └── 3.png
│   ├── 1/
│   │   └── ...
│   ├── 2/
│   │   └── ...
│   └── 3/
│       └── ...
...
```

此結構的優點在於：
- 無需資料庫即可直接使用靜態檔案伺服器（如 Nginx、Apache）提供服務
- 檔案路徑與 URL ——對應，路由規則單純
- 利於快取（CDN 可直接快取靜態 PNG/MVT 檔案）

然而對全球規模的圖磚集而言，檔案數量極為龐大（Zoom 18 約 687 億張），會導致嚴重的 inode 耗盡問題，因此大型圖磚伺服器通常會採用其他儲存方式。

## 替代儲存方案

### MBTiles

MBTiles 是一種以 SQLite 資料庫為基礎的圖磚容器格式，將所有圖磚儲存在單一 `.mbtiles` 檔案中[^pmtiles]。由 MapBox 提出，設計用於本地磁碟存取，被 TileCache 等伺服器廣泛支援。

### PMTiles

PMTiles 是由 Protomaps 設計的單檔唯讀圖磚歸檔格式，將整個圖磚金字塔儲存在單一 `.pmtiles` 檔案中[^pmtiles]。不同於 MBTiles，PMTiles 專門針對 HTTP Range Request 設計，可直接放在 S3 等雲端儲存服務上，允許客戶端僅下載所需的圖磚資料，無需運行伺服器軟體。對於大型全球圖磚集，PMTiles 可透過內部去重機制將檔案體積縮減 70% 以上。其 3 版規格為當前穩定版本，支援多種語言（JavaScript、Python、Rust、Kotlin、Dart）的用戶端程式庫。

## 圖磚儲存方式比較

| 儲存方式 | 載入機制 | 優點 | 缺點 |
|---------|---------|------|------|
| 獨立檔案（Z/X/Y.png） | 直接 HTTP GET | 結構單純、可直接 CDN 快取 | 檔案數量極多、上傳成本高 |
| MBTiles（SQLite） | 本地伺服器讀取 DB | 單檔管理、支援事務寫入 | 不適合直接 HTTP 遠端讀取 |
| PMTiles（單檔歸檔）| HTTP Range Request | 單檔上傳、雲端原生、去重合併 | 唯讀、無法原地更新 |

## 結論

XYZ 圖磚地圖的核心儲存機制是透過 `{Zoom}/{X}/{Y}.png` 三層路徑來定址每一張圖磚，使用 Web Mercator 投影與 256×256 像素的 PNG（或 MVT）檔案。傳統上這些圖磚以獨立檔案存放在伺服器檔案系統中，但面對全球規模的圖磚集時，會因檔案數量過於龐大而面臨效能瓶頸。現代的替代方案如 MBTiles（SQLite 容器）與 PMTiles（HTTP Range Request 單檔歸檔）可有效解決此問題，其中 PMTiles 特別適合雲端原生與無伺服器架構的部署場景。

## 參考資料

[^wiki]: Wikipedia. (n.d.). *Tiled web map*. Retrieved 2026-09-19, from https://en.wikipedia.org/wiki/Tiled_web_map
[^osm]: OpenStreetMap Wiki. (n.d.). *Slippy map tilenames*. Retrieved 2026-09-19, from https://wiki.openstreetmap.org/wiki/Slippy_map_tilenames
[^tms]: Wikipedia. (n.d.). *Tile Map Service*. Retrieved 2026-09-19, from https://en.wikipedia.org/wiki/Tile_Map_Service
[^pmtiles]: Protomaps. (n.d.). *PMTiles Concepts*. Retrieved 2026-09-19, from https://docs.protomaps.com/pmtiles/