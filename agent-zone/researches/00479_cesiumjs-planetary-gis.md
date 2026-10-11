# CesiumJS 對其他行星 GIS 支援能力研究

## 摘要

CesiumJS 作為一個 3D 地球/球體視覺化引擎，本質上支援**地球以外天體**的 GIS 建置，但需要手動配置。引擎內建了月球（Moon）與火星（Mars）的橢球體常數，且在 2024–2025 年間，Cesium 官方推出了商業化的 Cesium Moon 與 Cesium Mars 立體瓦片（3D Tiles）資料集。然而，部分內建功能（如 CZML、大氣效果）仍預設綁定地球（WGS84），且三軸橢球體存在已知錯誤。

## 1. 橢球體（Ellipsoid）支援

CesiumJS 提供以下內建橢球體常數[^ellipsoid-doc]：

| 常數 | 描述 |
|------|------|
| `Cesium.Ellipsoid.WGS84` | 地球（預設） |
| `Cesium.Ellipsoid.MOON` | 月球半徑的球體 |
| `Cesium.Ellipsoid.MARS` | 火星平均半徑的球體（EPSG:104905） |
| `Cesium.Ellipsoid.UNIT_SPHERE` | 單位球體 |

自 v1.120 起（約 2024 年中），可透過以下方式切換預設橢球體[^ellipsoid-doc]：

```javascript
Cesium.Ellipsoid.default = Cesium.Ellipsoid.MOON;
// 或
Cesium.Ellipsoid.default = Cesium.Ellipsoid.MARS;
```

亦可建立任意天體的自訂橢球體：

```javascript
var ellipsoid = new Cesium.Ellipsoid(x, y, z);
```

## 2. 自訂影像與地形圖層

支援使用任何 `ImageryProvider`（WMS、WMTS、單張影像等），並在 `tilingScheme` 中指定自訂橢球體[^community-mars]：

```javascript
imageryLayers.addImageryProvider(new Cesium.WebMapServiceImageryProvider({
    url: 'https://planetarymaps.usgs.gov/cgi-bin/mapserv?map=/maps/mars/mars_simp_cyl.map&service=WMS',
    layers: 'MDIM21_color',
    tilingScheme: new Cesium.GeographicTilingScheme({ ellipsoid: ellipsoidMars }),
}));
```

地形可使用 `EllipsoidTerrainProvider`。切換行星時需注意：
- 移除預設地球影像層：`imageryLayers.removeAll()`
- 關閉大氣效果：`viewer.scene.skyAtmosphere.show = false`
- 關閉地面大氣：`viewer.scene.globe.showGroundAtmosphere = false`
- 隱藏內建月球：`viewer.scene.moon.show = false`

## 3. 官方資料集

### Cesium Moon（2024 年 8 月推出）

由 NASA 月球軌道探測器（LRO）資料產生的完整 3D 瓦片集[^cesium-moon-blog]。可透過 Cesium ion 取得。

### Cesium Mars（2025 年 9 月推出）

結合 NASA MOLA 與 ESA Mars Express HRSC 地形資料的 3D 瓦片集[^cesium-mars-blog]：

- 全球影像解析度：232m，全球地形：200m
- Jezero 撞擊坑：6m 影像、20m 地形
- Gale 撞擊坑：25cm 影像、1m 地形
- CRS：IAU_2015:49901

使用範例[^cesium-mars-blog]：

```javascript
Cesium.Ellipsoid.default = Cesium.Ellipsoid.MARS;
const viewer = new Cesium.Viewer("cesiumContainer", {
    globe: false,
    sceneModePicker: false,
    baseLayerPicker: false,
    geocoder: false,
});
const tileset = await Cesium.Cesium3DTileset.fromIonAssetId(3644333);
viewer.scene.primitives.add(tileset);
```

### 月球與火星以外的天體

Cesium 官方未提供地球、月球、火星以外的資料集，需自行準備。常見資料來源[^usgs-wms]：

- **USGS Astrogeology WMS**：`https://planetarymaps.usgs.gov/cgi-bin/mapserv`
- **NASA Moon Trek / Mars Trek WMTS**
- **PDS（Planetary Data System）**
- **JPL Horizons / SPICE kernels**

## 4. 已知限制

1. **三軸橢球體錯誤**：GitHub Issue #12110 指出 CesiumJS 假設 `radii.x === radii.y`，火星的 x 與 y 半徑（3398627 vs 3393760 公尺）略有差異，使用真實值會導致錯誤，需使用球體近似（取平均值）[^issue-12110]。

2. **內部仍是 WGS84**：Issue #4245 透過 PR #12000 引入 `Ellipsoid.default`，但部分內部程式碼仍寫死 `Ellipsoid.WGS84` 與地球半徑常數，在切換橢球體時會產生問題[^issue-4245]。

3. **CZMLDataSource 鎖定 WGS84**：Issue #11185 指出 CZMLDataSource 明確使用 `Ellipsoid.WGS84`，不適用於非地球天體[^issue-11185]。

4. **單場景不支援多行星**：CesiumJS 一次只能渲染一個球體，無法原生支援同場景多行星。需自訂程式碼實現[^reddit-multiplanet]。

5. **地球專屬資料提供者無法使用**：`createGooglePhotorealistic3DTileset`、OSM 影像、Ion 世界地形等均為地球專用。

## 5. 已知專案

| 專案 | 說明 |
|------|------|
| Orbit Codex | 使用 CesiumJS 展示地球、月球與火星的太空活動百科[^orbit-codex] |
| AlphaPixel Mars Demo | 詳細教學展示火星視覺化，使用 USGS WMS 與自訂塵埃覆蓋層[^alphapixel] |
| Buendea / NASA XOSS | NASA 任務規劃與太空人訓練系統，使用 Cesium Moon 與 Cesium Mars |
| Ansys STK | 使用 Cesium 3D Tiles 進行月球地形通訊系統分析 |
| MAVEN Mission | NASA MAVEN 火星任務以 CesiumJS 視覺化[^maven] |
| Omniverse | 基於 CesiumJS 的多天體 3D 球體渲染開源庫 |

## 結論

CesiumJS 完全支援地球以外天體的 GIS 應用，但需大量手動配置。火星與月球有官方資料集，其他天體需自備資料。主要限制在於三軸橢球體錯誤、CZML 相容性，以及部分內部程式碼仍鎖定 WGS84。

[^ellipsoid-doc]: Cesium. (n.d.). Ellipsoid. CesiumJS Ref Doc. Retrieved 2026-09-10, from https://cesium.com/learn/cesiumjs/ref-doc/Ellipsoid.html
[^cesium-moon-blog]: Cesium. (2024-08-06). Introducing Cesium Moon Terrain. Cesium Blog. Retrieved 2026-09-10, from https://cesium.com/blog/2024/08/06/introducing-cesium-moon-terrain/
[^cesium-mars-blog]: Cesium. (2025-09-03). Lift Off to Cesium Mars. Cesium Blog. Retrieved 2026-09-10, from https://cesium.com/blog/2025/09/03/lift-off-to-cesium-mars/
[^community-mars]: Cesium Community. (n.d.). How can I use Mars to replace Earth in CesiumJS? Cesium Community Forum. Retrieved 2026-09-10, from https://community.cesium.com/t/how-can-i-use-mars-to-replace-the-earth-in-cesiumjs/21332
[^issue-12110]: CesiumGS. (n.d.). Overly restrictive assumption about ellipsoid radii being equal. GitHub Issue #12110. Retrieved 2026-09-10, from https://github.com/CesiumGS/cesium/issues/12110
[^issue-4245]: CesiumGS. (n.d.). Easy API to select a non-earth ellipsoid. GitHub Issue #4245. Retrieved 2026-09-10, from https://github.com/CesiumGS/cesium/issues/4245
[^issue-11185]: CesiumGS. (n.d.). CZMLDataSource with custom ellipsoids. GitHub Issue #11185. Retrieved 2026-09-10, from https://github.com/CesiumGS/cesium/issues/11185
[^reddit-multiplanet]: Reddit. (n.d.). Has anyone extended CesiumJS to include multiple planets? r/webgl. Retrieved 2026-09-10, from https://www.reddit.com/r/webgl/comments/21it44/has_anyone_ever_extended_cesiumjs_to_include/
[^usgs-wms]: USGS Astrogeology. (n.d.). Planetary WMS. USGS Astrogeology Science Center. Retrieved 2026-09-10, from https://planetarymaps.usgs.gov/
[^orbit-codex]: Cesium Community. (n.d.). Space visualizations in CesiumJS by Orbit Codex. Cesium Community Forum. Retrieved 2026-09-10, from https://community.cesium.com/t/space-visualizations-in-cesiumjs-by-orbit-codex/46241
[^alphapixel]: AlphaPixel. (n.d.). Space-Domain 3D Visualization, Part 1: Mars. AlphaPixel Dev Blog. Retrieved 2026-09-10, from https://alphapixeldev.com/we-solve-your-difficult-problems-space-domain-3d-visualization-part-1-mars/
[^maven]: ADS. (2017). 3D Planetary Data Visualization with CesiumJS (MAVEN mission). AGU Fall Meeting. Retrieved 2026-09-10, from https://ui.adsabs.harvard.edu/abs/2017AGUFMIN31D..04L/abstract