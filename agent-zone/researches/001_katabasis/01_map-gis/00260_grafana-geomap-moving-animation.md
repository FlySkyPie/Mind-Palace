# Grafana Geomap 動態移動點位視覺化可行性研究

## 問題

Grafana 的 Geomap 面板能否顯示「在地圖上移動的物件」？亦即點位會隨著時間推進而移動，而非僅呈現靜態的點資料。

## 結論

**Grafana 原生的 Geomap 面板不支援真正的時間軸動畫**——沒有時間軸滑桿或播放按鈕讓點位隨著時間平滑移動。但可透過以下方式實現近似效果。

---

## 1. 原生 Geomap 面板的能力與限制

### 可做到的事

- 顯示靜態點位（markers）、熱力圖、路線（Route）、GeoJSON 圖層、網路圖
- 透過 **儀表板自動重新整理（auto-refresh）** 呈現即時移動資料——當儀表板定時重新查詢（如每 5 秒），點位位置會更新[^grafana-docs]
- **Route 圖層（beta）** 可將 GPS 軌跡歷史渲染為地圖上的線條/路徑

### 限制

- 沒有時間軸滑桿或播放功能讓點位隨時間動畫
- Route 圖層有一個 **已知限制**：無法在同一個面板中區分多條路線/實體——它會將所有資料視為單一時間序列，在不同實體之間畫出 zigzag 線條。這是一個已記錄的 GitHub issue[^route-issue]
- 若要顯示不同實體的獨立路線，必須為每個實體建立單獨的圖層（或使用 panel repeat）

官方文件指出：「Geomap 也適用在有**即時變化**的位置資料時，透過 auto-refresh 可視化物件移動位置。」[^grafana-docs] 但這並非真正的動畫，而是快照式更新。

## 2. 實驗性 Alpha 圖層

在 Grafana 組態中啟用 `enable_alpha = true` 後可使用的兩種實驗性圖層[^aws-geomap]：

- **Icon at last point（Alpha）**——僅在最後一個資料點渲染圖示，適合顯示追蹤物件的當前位置
- **Dynamic GeoJSON（Alpha）**——根據查詢結果動態設定 GeoJSON 檔案的樣式

這些功能同樣**不提供**時間軸動畫。

## 3. 外掛：GeoLoop Panel

- **倉庫**：[CitiLogics/citilogics-geoloop-panel](https://github.com/CitiLogics/citilogics-geoloop-panel)[^geoloop]
- **目的**：專門用於**動畫地圖**——將 GeoJSON 與時間序列資料結合，循環動畫顯示地理特徵（多邊形、點、線）
- **使用案例**：視覺化氣象站每小時溫度變化、降雨隨時間演變、水質變化
- **需求**：InfluxDB + MapBox API key + 以 JSONP 提供的 GeoJSON
- **注意**：動畫速度/循環選項在 README 中標示為「尚未實作」

## 4. 外掛：TrackMap Panel

- **市集**：[pr0ps-trackmap-panel](https://grafana.com/grafana/plugins/pr0ps-trackmap-panel/)[^trackmap]
- **目的**：將 GPS 點位渲染為互動地圖上的線條/軌跡
- **關鍵功能**：當滑鼠懸停在其他面板時，在地圖上對應時間位置顯示一個點（cross-highlighting）
- **支援**：Time series 或 table 資料、多種地圖背景（OpenStreetMap、OpenTopoMap、Satellite）
- **適合**：檢視單一車輛/追蹤物的歷史路徑

## 5. 外部工具：grafanimate

- **倉庫**：[grafana-toolbox/grafanimate](https://github.com/grafana-toolbox/grafanimate)[^grafanimate]
- **目的**：外部 Python CLI 工具，透過程式化操縱 Grafana 儀表板的時間範圍控制項並擷取螢幕截圖，來動畫化**任何** Grafana 儀表板
- **輸出**：動畫 GIF、影片（MP4）、或 PNG 序列
- **運作方式**：定義「場景」（scenarios）包含開始/結束時間與步進間隔；使用 Firefox 瀏覽器自動化渲染每一幀
- **使用案例**：動畫地圖顯示歐洲空氣品質感測器覆蓋範圍隨月份的變化、天氣圖
- **⚠️ 警告**：可能對 Grafana 伺服器和資料庫造成顯著負載

## 6. 實用變通方案

根據 QuestDB 的 Grafana 地圖教學[^questdb]：

| 方法 | 運作方式 | 最佳用途 |
|------|---------|---------|
| **Auto-refresh + `LATEST BY`** | 儀表板自動重新整理以顯示最新點位 | 即時車隊追蹤 |
| **每個實體一個 Route 圖層** | 透過 panel repeat 搭配變數為每輛車建立獨立圖層 | 歷史軌跡 |
| **Route + Marker 圖層組合** | Route 顯示歷史軌跡 + Marker 顯示當前位置 | 同時顯示路徑與當前位置 |
| **Marker 旋轉** | 使用 `bearing` 欄位顯示箭頭方向 | 車輛行進方向 |
| **依欄位設定 Marker 顏色** | 依 `speed` 或其他指標設定顏色 | 狀態指示 |

該教學也指出：「目前 Grafana 無法區分多條路線，即使使用 transformation 也無法……這是 Grafana 目前的限制。」[^questdb]

## 7. 社群意見與需求

Grafana 社群論壇有多則討論確認了使用者需求[^community1][^community2]：

- 「是否可以在時間軸上顯示標記？」——使用者希望 Geomap 有時間軸滑桿
- 「如何顯示車隊的即時位置？」——即時車隊追蹤需求
- 「顯示座標位置」——在游標時間位置顯示點位的功能請求

共識是：**此功能未內建，官方也未宣布計畫加入。**

---

## 總結對照表

| 方法 | 原生？ | 動畫？ | 可用於生產？ | 最佳用途 |
|------|--------|--------|-------------|---------|
| Geomap + auto-refresh | ✅ 內建 | ❌ 快照更新 | ✅ 是 | 即時位置更新 |
| Geomap Route 圖層 | ✅ 內建（beta） | ❌ 靜態路徑 | ⚠️ 有限制（單一路線） | 歷史 GPS 軌跡（單一實體） |
| Icon at last point（alpha） | ✅ 內建（alpha） | ❌ 靜態 | ⚠️ Alpha | 顯示當前位置 |
| GeoLoop 外掛 | ❌ 社群外掛 | ✅ 循環動畫 | ⚠️ 有限 | 天氣/氣候資料動畫 |
| TrackMap 外掛 | ❌ 社群外掛 | ❌ 靜態附 cross-highlight | ✅ 是 | GPS 軌跡視覺化 |
| grafanimate（外部工具） | ❌ 外部工具 | ✅ 影片/GIF 動畫 | ✅ 是 | 製作可分享的動畫地圖影片 |
| Panel repeat + Route 圖層 | ✅ 內建 | ❌ 每個車輛靜態 | ✅ 是 | 多車輛路線（變通方案） |

---

## 結語

若你的需求是**真正的時間軸動畫**（點位隨時間滑桿推進而平滑移動），Grafana 原生 Geomap 無法達成。最佳替代方案為：(1) **GeoLoop 社群外掛**用於循環 GeoJSON 動畫；(2) **grafanimate** 用於從儀表板產出動畫影片/GIF；(3) **auto-refresh** 用於即時（非歷史）位置更新。

---

[^grafana-docs]: Grafana Labs. (n.d.). Geomap visualization. Retrieved 2026-09-27, from https://grafana.com/docs/grafana/latest/visualizations/panels-visualizations/visualizations/geomap/
[^route-issue]: Grafana Labs. (n.d.). Route layer: simplify data or allow panel repeat to separate multiple routes in single panel [#72878]. Retrieved 2026-09-27, from https://github.com/grafana/grafana/issues/72878
[^aws-geomap]: Amazon Web Services. (n.d.). Using Geomap panels in Grafana (v10). Retrieved 2026-09-27, from https://docs.aws.amazon.com/grafana/latest/userguide/v10-panels-geomap.html
[^geoloop]: CitiLogics. (n.d.). citilogics-geoloop-panel. Retrieved 2026-09-27, from https://github.com/CitiLogics/citilogics-geoloop-panel
[^trackmap]: Grafana Labs. (n.d.). TrackMap Panel. Retrieved 2026-09-27, from https://grafana.com/grafana/plugins/pr0ps-trackmap-panel/
[^grafanimate]: grafana-toolbox. (n.d.). grafanimate: Animate timeseries data with Grafana. Retrieved 2026-09-27, from https://github.com/grafana-toolbox/grafanimate
[^questdb]: QuestDB. (n.d.). Working with Grafana map markers and geomaps. Retrieved 2026-09-27, from https://questdb.com/blog/working-with-grafana-maps-markers/
[^community1]: Grafana Community. (n.d.). Is it possibile to show markers along the time? (asking for hints). Retrieved 2026-09-27, from https://community.grafana.com/t/is-it-possibile-to-show-markers-along-the-time-asking-for-hints/128031
[^community2]: Grafana Community. (n.d.). Worldmap Panel: how to display the live position of vehicles in a fleet. Retrieved 2026-09-27, from https://community.grafana.com/t/worldmap-panel-how-to-display-the-live-position-of-vehicles-in-a-fleet/29239