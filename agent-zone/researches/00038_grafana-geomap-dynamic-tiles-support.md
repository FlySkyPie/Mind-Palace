# Grafana Geomap 面板對動態地圖瓦片（Dynamic Map Tiles）的支援研究

## 摘要

本文研究 Grafana Geomap 面板是否支援根據時間範圍動態切換的地圖瓦片。結論是：Grafana 原生 Geomap 面板僅有**間接支援**，可透過儀表板變數（dashboard variables）在 XYZ 瓦片圖層 URL 中達成時間動態效果，但原生不支援 WMS/WMTS 協定或專屬的 `TIME` 參數傳遞。

## 背景

Grafana 的 Geomap 面板提供多種圖層類型，包括 Markers、Heatmap、GeoJSON、Night/Day、Route、Network 等資料圖層，以及 OpenStreetMap、CARTO、ArcGIS MapServer、XYZ Tile layer、MapLibre Style 等底圖圖層。[^geomap-docs]

當使用者希望地圖瓦片隨時間變化（例如顯示不同時間點的天氣雷達、衛星影像等），需要一個能將時間參數傳遞給瓦片伺服器的機制。

## 原生支援形式

### XYZ Tile Layer 支援變數（已修復）

Grafana 的 **XYZ Tile layer** 接受自訂 URL 模板，例如 `https://tile.openstreetmap.org/{z}/{x}/{y}.png`。根據 Grafana 官方文件（v13.2.2），該 URL 模板「支援儀表板變數」（Dashboard variables are supported），例如 `https://example.com/maps/${version}/{z}/{x}/{y}.png`。[^geomap-docs-xyz]

歷史上，此功能長期缺失。2022 年 11 月有使用者回報 Issue #58482，指出 XYZ URL 模板中的變數被忽略，以字面字串傳遞而非被求值。[^issue-58482] 該 Issue 目前仍處於 Open 狀態。

2025 年 4 月，相關 PR #104587 為 **GeoJSON URL** 添加了變數支援，並於 2025 年 5 月合併至主分支（里程碑 v12.1.x）。PR 描述中明確提到「可能可修復 Issue #58482」（指 XYZ Tile 部分的變數支援）。[^pr-104587]

### 實際運用方式

雖然無內建 `$__from`/`$__to` 這類時間範圍變數在瓦片 URL 中的文件支援，但使用者可以建立自訂儀表板變數（如 `Custom` 類型），將時間範圍格式化為目標瓦片伺服器接受的格式（如 ISO 8601 時間字串），然後在 XYZ URL 中使用該變數：

```
https://example.com/tiles/${my_time_formatted}/{z}/{x}/{y}.png
```

這使得儀表板的時間範圍選擇器可間接影響地圖瓦片的選擇。

## 原生不支援的部分

### WMS / WMTS 協定

Grafana Geomap 原生**不支援 WMS（Web Map Service）或 WMTS（Web Map Tile Service）** 圖層類型。使用者無法直接在 Geomap 面板中設定 WMS 端點並傳遞 `LAYERS`、`TIME`、`BBOX` 等 OGC 標準參數。

社群討論中多次提及此需求。2024 年有使用者在 Grafana 社群論壇發問「在 Geomap 面板中使用動態 WMS 的可能性」，期望能在 Grafana 面板中以「更原生/整合的方式連接動態（含參數）WMS 端點」，並希望「使用者可使用 Grafana 變數或面板 UI 互動式設定 WMS 請求參數（如 layers、time、BBOX 等）——而非硬編碼它們」。截至研究日期，該討論未有官方回覆。[^community-wms]

### Night / Day 圖層

Geomap 提供 **Night / Day 圖層**，可基於目前時間範圍顯示日照/陰影區域。此圖層可選擇使用 `From` 或 `To` 時間端點來驅動日夜區域。[^geomap-docs-night] 但此圖層顯示的是基於天文計算的日照陰影，而非動態地圖瓦片，與本問題的研究範疇不同。

## 第三方插件

### Geomap Panel WMS 插件

Grafana 官方市集中有 **[Geomap Panel WMS](https://grafana.com/grafana/plugins/felixrelleum-geomapwms-panel/)** 插件（作者：felixrelleum），為 Geomap 面板新增 OGC WMS 1.3.0 和 **WMTS** 支援。主要功能包括：多個 WMS 端點作為底圖圖層、空間篩選器工具、資料連結支援。[^wms-plugin]

**然而**，該插件並未宣傳支援**基於時間的 WMS**（即向 WMS 傳遞 `TIME` 參數以根據儀表盤時間範圍獲取不同瓦片）。其文件中未提及任何時間維度支援。[^wms-plugin-repo]

## 可行的解決方案

| 方案 | 方法 | 適用場景 |
|------|------|----------|
| XYZ URL + 自訂變數 | 建立格式化時間的儀表板變數，在 XYZ URL 中使用 | 瓦片伺服器支援在 URL 路徑/查詢參數中指定時間 |
| Geomap Panel WMS 插件 | 安裝第三方插件取得 WMS/WMTS 支援，但無時間參數自動傳遞 | 需要 WMS/WMTS 但時間參數可硬編碼或由變數間接處理 |
| Business Text 插件 + 自訂 Leaflet | 使用 Business Text 插件嵌入自訂 Leaflet 地圖，JavaScript 直接控制 WMS 請求 | 需要完全控制 WMS `TIME` 參數傳遞 |
| 動態 GeoJSON（Alpha） | 使用 `enable_alpha = true` 啟用 Dynamic GeoJSON 圖層，基於時間查詢動態載入 | 時間感知的 GeoJSON 資料而非瓦片 |

## 結論

Grafana 的 Geomap 面板**不直接支援「地圖瓦片隨時間自動切換」** 的原生機制。最接近的可行方式是：(1) 在 XYZ Tile Layer 中使用自訂儀表板變數將時間範圍格式化後傳入瓦片 URL；(2) 如需要 WMS 協定支援，可考慮第三方插件或自訂嵌入方案。此限制在社群中已被多次提出，但目前缺乏原生實作。

## 參考來源

[^geomap-docs]: Grafana Labs. (n.d.). Geomap visualization. Retrieved 2026-09-14, from https://grafana.com/docs/grafana/latest/visualizations/panels-visualizations/visualizations/geomap/

[^geomap-docs-xyz]: Grafana Labs. (n.d.). Geomap visualization — XYZ Tile layer. Retrieved 2026-09-14, from https://grafana.com/docs/grafana/latest/visualizations/panels-visualizations/visualizations/geomap/#xyz-tile-layer

[^geomap-docs-night]: Grafana Labs. (n.d.). Geomap visualization — Night / Day layer. Retrieved 2026-09-14, from https://grafana.com/docs/grafana/latest/visualizations/panels-visualizations/visualizations/geomap/#night--day-layer

[^issue-58482]: adageable. (2022). Geomap: Support template variable interpolation in XYZ tile layer URL (Issue #58482). GitHub. Retrieved 2026-09-14, from https://github.com/grafana/grafana/issues/58482

[^pr-104587]: drew08t. (2025). Geomap: Add variable support for GeoJSON url (PR #104587). GitHub. Retrieved 2026-09-14, from https://github.com/grafana/grafana/pull/104587

[^community-wms]: Grafana Community. (2024). Possibilities for using dynamic WMS (Web Mapping Services) in Grafana. Retrieved 2026-09-14, from https://community.grafana.com/t/possibilities-for-using-dynamic-wms-web-mapping-services-in-grafana/154225

[^wms-plugin]: felixrelleum. (n.d.). Geomap Panel WMS Plugin. Grafana Plugins. Retrieved 2026-09-14, from https://grafana.com/grafana/plugins/felixrelleum-geomapwms-panel/

[^wms-plugin-repo]: felix-mu. (n.d.). Geomap WMS Panel (GitHub repository). Retrieved 2026-09-14, from https://github.com/felix-mu/geomap-wms-panel