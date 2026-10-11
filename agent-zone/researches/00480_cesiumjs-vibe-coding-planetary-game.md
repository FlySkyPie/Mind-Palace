# CesiumJS 搭配 Vibe Coding 的星球遊戲專案調查

## 摘要

本報告調查使用 CesiumJS 進行「vibe coding」（AI 輔助開發）並打造星球／太空遊戲的相關專案。結果顯示，目前尚無專案明確以「vibe coding」方式，使用 CesiumJS 開發完整的行星規模遊戲，但已有數個飛行模擬、FPS 及遊戲世界匯入管線的開源專案，以及官方對 AI 輔助開發的投入，表明這條路徑是可行且正在成長中的領域。

---

## 1. 背景

CesiumJS 是一個開源的 JavaScript 函式庫，專注於 3D 地球與地理空間資料可視化，支援全球地形串流、3D Tiles 及高精度攝影機控制。雖然其本職是 GIS / 地理資訊系統，但近年開發者開始將其用於遊戲場景——尤其是需要**全球尺度地形**的遊戲。[^origin]

「Vibe coding」是 Andrej Karpathy 提出的概念，指完全依賴 AI 工具（如大型語言模型）生成程式碼，開發者僅提供高層次提示，不再手動逐行編寫。[^vibe]

---

## 2. 主要 CesiumJS 遊戲專案

### 2.1 Web Flight Simulator ⭐ 577

**GitHub:** [dimartarmizi/web-flight-simulator](https://github.com/dimartarmizi/web-flight-simulator)

目前最成熟的 CesiumJS 遊戲專案，混合渲染 **Three.js**（飛機模型、粒子特效）與 **CesiumJS**（全球地形、衛星影像串流）。功能包含 F-15 戰鬥機、武器系統（機炮、飛彈、熱焰彈）、戰術 HUD 及 NPC AI。[^webflight] [^webflight-blog]

### 2.2 Cesium Flight Simulator ⭐ 436

**GitHub:** [WilliamAvHolmberg/cesium-flight-simulator](https://github.com/WilliamAvHolmberg/cesium-flight-simulator)

Cesium + React + TypeScript 的 3D 飛行／地面載具模擬器，支援多種載具、攝影機模式、小地圖（Mapbox）、快速跳轉地標、撞地檢測。[^cesiumflight]

### 2.3 GeoStrike / CesiumFPS

**GitHub:** [YonatanKra/CesiumFPS](https://github.com/YonatanKra/CesiumFPS)

**網站:** [geo-strike.com](http://geo-strike.com/)

目前唯一使用 CesiumJS 的多人即時 FPS。採用 Angular + CesiumJS + 3D Tiles + GraphQL，支援多人遊戲房間、FPV/SemiFPV 攝影機、武器系統與建築物進入。雖然星數為 0，但已有 **495 commits**，是提交次數最多的專案。[^geostrike]

### 2.4 GeoFS（非開源）

**網站:** [www.geo-fs.com](https://www.geo-fs.com/)

成熟的免費線上多人飛行模擬器，基於 CesiumJS（早期使用 Google Earth Plugin），支援數十種飛行器、多人語音聊天、即時 ADS-B 商業航班交通與即時天氣。Cesium 官方部落格將其列為展示案例。[^geofs] [^geofs-blog]

---

## 3. 星球／太空相關 CesiumJS 專案

### 3.1 Orbit Codex — 太空可視化

**Cesium 社群:** [community.cesium.com/t/space-visualizations-in-cesiumjs-by-orbit-codex/46241](https://community.cesium.com/t/space-visualizations-in-cesiumjs-by-orbit-codex/46241)

使用 CesiumJS 可視化衛星群集、地球、月球與行星的百科資料，以及使用 **Cesium Moon** 與 **Cesium Mars** 數據呈現月球與火星任務。[^orbitcodex]

### 3.2 Cesium 官方行星表面實驗

**部落格:** [cesium.com/blog/2018/06/22/planetary-surface-experiment/](https://cesium.com/blog/2018/06/22/planetary-surface-experiment/)

Cesium 官方基於 CesiumJS 的行星表面 GIS 分析實驗。[^planetary-exp]

### 3.3 Cesium Sandcastle — Mars Demo

**URL:** [sandcastle.cesium.com/?id=mars](https://sandcastle.cesium.com/?id=mars)

官方 CesiumJS 展示火星地形渲染。[^mars-demo]

---

## 4. 「Vibe Coding」與 CesiumJS 的交集

### 4.1 benchvue/vibe-coding ⭐ 0

**GitHub:** [benchvue/vibe-coding](https://github.com/benchvue/vibe-coding)

目前唯一明確標註「vibe coding」且使用 CesiumJS 的專案。這是一套教育性質的 7 堂課教案，其中 `earthquakes-v1` 使用 CesiumJS 3D 地球可視化即時地震資料。`solar-system-v1` 則使用 Three.js（非 CesiumJS）呈現太陽系。[^benchvue]

### 4.2 Cesium 社群對 Vibe Coding 的討論

Cesium 社群有專文討論 [vibe coding 在 CesiumJS 的最佳實踐](https://community.cesium.com/t/vibe-coding-best-practice-for-cesiumjs/45399)，顯示社群對 AI 輔助開發的關注。[^vibe-practice]

### 4.3 官方 AI 輔助開發資源

Cesium 正在積極投資 AI 輔助開發：

- **cesiumjs-skills** — 提供給 Cursor / Crush 等 AI 編輯器的技能集
- **cesium-ai-integrations** — AI 整合資源
- **[Build a Flight Simulator with CesiumJS Using AI](https://cesium.com/learn/cesiumjs-learn/build-a-flight-simulator-with-cesiumjs-using-ai/)** 教學 — 官方教學教你用 AI 打造 CesiumJS 飛行模擬器[^ai-tutorial]

這表明 CesiumJS + AI vibe coding 是一塊正在成長的領域。

---

## 5. 遊戲世界匯入 CesiumJS 的技術管道

### 5.1 Elden Ring 3D Map — 技術示範

**文章:** [stevenpg.com/posts/game-map-to-cesium-3d-tileset/](https://stevenpg.com/posts/game-map-to-cesium-3d-tileset/)
**示範:** [stevenpg.com/projects/elden-ring-3d-map/](https://stevenpg.com/projects/elden-ring-3d-map/)

該專案展示如何將**任何遊戲世界**匯入 CesiumJS：

1. 從遊戲中提取高度圖（heightmap）與地圖影像
2. 透過 GDAL 「偽造座標」將遊戲地圖對應到地球上的矩形區域
3. 使用 `Cesium Terrain Builder` 建立 quantized-mesh 地形圖塊
4. 使用 `gdal2tiles` 建立影像圖塊金字塔
5. 在 CesiumJS 中指向靜態圖塊檔案

**關鍵洞察：** 只要你有高度圖 + 彩色地圖，這個流程可以套用到任何奇幻世界地圖或火星等行星表面。[^elden-ring]

---

## 6. 遊戲開發相關工具

### cesium-player-controller

**GitHub:** [hh-hang/cesium-player-controller](https://github.com/hh-hang/cesium-player-controller)

第一／第三人稱角色控制器，支援膠囊碰撞、動畫、攝影機障礙物避免——這是目前最遊戲開發相關的 CesiumJS 外掛。[^player-ctrl]

### 遊戲引擎整合（官方）

| 專案 | 平台 | GitHub | 說明 |
|------|------|--------|------|
| Cesium for Unreal | Unreal Engine | [CesiumGS/cesium-unreal](https://github.com/CesiumGS/cesium-unreal) | ⭐ ~1k+，將 3D 地理空間帶入 UE |
| Cesium for Unity | Unity | [CesiumGS/cesium-unity](https://github.com/CesiumGS/cesium-unity) | ⭐ ~800+，v1.20.0 支援 Web 部署 |
| 3D Tiles for Godot | Godot 4 | [Battle-Road-Labs/3D-Tiles-For-Godot](https://github.com/Battle-Road-Labs/3D-Tiles-For-Godot) | 社群 GDExtension，串流 3D Tiles |
| NASA 3DTilesRendererJS | Three.js | [NASA-AMMOS/3DTilesRendererJS](https://github.com/NASA-AMMOS/3DTilesRendererJS) | Three.js 的 3D Tiles 渲染器（CesiumJS 替代方案） |

---

## 7. 綜合分析

### 發現

1. **飛行模擬器佔主導** — 絕大多數 CesiumJS 遊戲專案是飛行模擬器，因為 CesiumJS 的全球地形串流天生適合飛行視角。
2. **僅有一個 FPS** — GeoStrike/CesiumFPS 是唯一非飛行類的射擊遊戲嘗試，提交量可觀（495 commits）但星數極低。
3. **混合渲染是主流架構** — 最成功的專案使用 CesiumJS 處理地球／地形層，**Three.js** 處理本地遊戲物件（飛機、武器、角色）。Web Flight Simulator 是最佳範例。
4. **「Vibe coding」+ CesiumJS 遊戲尚無先例** — 既有的 vibe coding 專案（benchvue/vibe-coding）是教育性質的地震可視化，並非遊戲。目前尚無人以 vibe coding 方式用 CesiumJS 開發行星規模遊戲並公開。
5. **技術管道存在** — Elden Ring 3D Map 示範已證明可以將任意遊戲世界（包括行星地形）匯入 CesiumJS。
6. **官方正在推動 AI 輔助開發** — Cesium 已釋出 AI 技能包與教學，降低 AI 生成 CesiumJS 程式碼的門檻。

### 結論

截至目前，**沒有任何公開的 vibe coding 專案使用 CesiumJS 打造行星遊戲**。但相關的基礎設施（地形串流、遊戲引擎整合、世界匯入管道）與官方 AI 支援均已到位，這是一塊尚未被開墾的領域，有潛力成為 vibe coding 的下一個熱門方向。

---

## 參考資料

[^origin]: CesiumGS. (n.d.). *CesiumJS — Geospatial 3D Globe & Map*. Retrieved 2026-10-10, from https://github.com/CesiumGS/cesium
[^vibe]: Karpathy, A. (2025). *Vibe Coding*. Retrieved 2026-10-10, from https://en.wikipedia.org/wiki/Vibe_coding
[^webflight]: dimartarmizi. (2026). *Web Flight Simulator*. GitHub. Retrieved 2026-10-10, from https://github.com/dimartarmizi/web-flight-simulator
[^webflight-blog]: Bright Coding. (2026). *Web Flight Simulator: The Revolutionary Browser Aviation Tool*. Retrieved 2026-10-10, from https://www.blog.brightcoding.dev/2026/06/29/web-flight-simulator-the-revolutionary-browser-aviation-tool
[^cesiumflight]: WilliamAvHolmberg. (2025). *Cesium Flight Simulator*. GitHub. Retrieved 2026-10-10, from https://github.com/WilliamAvHolmberg/cesium-flight-simulator
[^geostrike]: YonatanKra. (2025). *CesiumFPS*. GitHub. Retrieved 2026-10-10, from https://github.com/YonatanKra/CesiumFPS
[^geofs]: Tassin, X. (n.d.). *GeoFS — Free Online Flight Simulator*. Retrieved 2026-10-10, from https://www.geo-fs.com/
[^geofs-blog]: Cesium. (2021). *GeoFS Is a Flight Simulator That Showcases Global Satellite*. Retrieved 2026-10-10, from https://cesium.com/blog/2021/12/06/geofs-is-a-flight-simulator-that-showcases-global-satellite/
[^orbitcodex]: Orbit Codex. (2025). *Space Visualizations in CesiumJS by Orbit Codex*. Cesium Community. Retrieved 2026-10-10, from https://community.cesium.com/t/space-visualizations-in-cesiumjs-by-orbit-codex/46241
[^planetary-exp]: Cesium. (2018). *Planetary Surface Experiment*. Retrieved 2026-10-10, from https://cesium.com/blog/2018/06/22/planetary-surface-experiment/
[^mars-demo]: Cesium. (n.d.). *Cesium Sandcastle — Mars Demo*. Retrieved 2026-10-10, from https://sandcastle.cesium.com/?id=mars
[^benchvue]: benchvue. (2026). *vibe-coding*. GitHub. Retrieved 2026-10-10, from https://github.com/benchvue/vibe-coding
[^vibe-practice]: Cesium Community. (2025). *Vibe Coding Best Practice for CesiumJS*. Retrieved 2026-10-10, from https://community.cesium.com/t/vibe-coding-best-practice-for-cesiumjs/45399
[^ai-tutorial]: Cesium. (2025). *Build a Flight Simulator with CesiumJS Using AI*. Retrieved 2026-10-10, from https://cesium.com/learn/cesiumjs-learn/build-a-flight-simulator-with-cesiumjs-using-ai/
[^elden-ring]: stevenpg. (2025). *Game Map to Cesium 3D Tileset*. Retrieved 2026-10-10, from https://stevenpg.com/posts/game-map-to-cesium-3d-tileset/
[^player-ctrl]: hh-hang. (2025). *cesium-player-controller*. GitHub. Retrieved 2026-10-10, from https://github.com/hh-hang/cesium-player-controller