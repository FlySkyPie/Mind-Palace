# 可觀測性

最簡單且直覺在 Minecraft 伺服器上建立實時可觀測的方式是使用諸如 [Dynmap](https://github.com/webbukkit/dynmap) 或 [BlueMap](https://github.com/BlueMap-Minecraft/BlueMap) 之類的插件，能夠在網頁上顯式玩家的位置，然而 Cuberite 生態並不存在這類的插件。

## 計畫（方案一）

忠於 Minecraft 生態的路線。

1. 現有實作解機階段
    - 調查幾個實作呼叫 Plugin Framework 的界面，如：
        - https://github.com/webbukkit/dynmap
        - https://github.com/BlueMap-Minecraft/BlueMap
        - https://github.com/jpenilla/squaremap
        - https://github.com/granny/Pl3xMap
2. 調查 Cuberite 的 Plugin API
    - https://api.cuberite.org/
3. 評估移植的可能性，Cuberite 是否已經提供所有必要 API。

## 計畫（方案二）

使用 Grafana 的 GIS 功能，如此一來 Cuberite 端只需要實做 API 提供圖資與資料，而無須考慮 Web UI 的問題。缺點是即時性受限，可觀測體系的工具並不具有很高的即時性，資料存在一定程度的延遲。反過來說，Grafana 具有 Minecraft 原生方案所沒有時間軸資料回朔以及過濾能力。

### 圖磚化地圖

雖然 Leaflet 提供像 L.CRS.Simple 這樣的非 GIS 標準空間，方便用來呈現缺乏精確經緯度的資料，但是非標準化意味著將圖資遷移到其他框架會較為困難。[XYZ Tile](https://en.wikipedia.org/wiki/Tiled_web_map) 是一個簡單且經典的 GIS 圖資格式，並且被多數 GIS 前端或是渲染函式庫支援。「地球表面」這個尺度對於大部分用例都不會輕易超過，常規的 Minecraft 玩家活動範圍也是如此，因此使用 XYZ Tile 儲存或表現圖資依然很有吸引力。

將 Minecraft 地圖以 1 Block = 1 Meter 的比例轉換成普通的 GIS 圖磚，便可將問題簡化成「轉換與儲存 PNG 圖片」以及「提供 HTTP API 下載 PNG 圖磚」，GIS 前端則可直接使用包含 Grafana 在內的其他工具，實現關注點分離。

### Lua 與動態函式庫

Cuberite 內 Lua 環境提供的 API，僅有 TCP 能力，且無圖片操作相關的能力，也就是說建立 HTTP 伺服器需要從 TCP 往上堆砌，圖片生成也存在障礙。以 Cuberite 社群內比較有名的地圖專案 [StaticMap](https://github.com/KrystilizeNevaDies/StaticMap)為例，它處理圖片的方式在插件內嵌入 ImageMagick 的執行檔。

Cuberite 使用的 PUC-Rio Lua 5.1 同時也具備載入動態函式庫的能力，因此更為實際的方式是在 C++ 實做 GIS 所需的圖片處理與 HTTP 機能，再透過 Lua 薄封裝和 Cuberite 整合。
