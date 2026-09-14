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
