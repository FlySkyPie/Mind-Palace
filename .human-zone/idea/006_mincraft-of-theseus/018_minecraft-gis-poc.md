# Minecraft GIS - POC of Basic Integration

關注於 Lua-Cpp-HTTP-SQLite-File System 的資料驗證。

## EPSG:3857 近似

以 256px 標準圖磚瑋基準，Z=17 時，一像素近似一米，可作為 Minecraft 一個方塊的近似基準。
因此在 Z=17 時，16 個 Chunk = 一個圖磚。

> [!NOTE] 該誤差會累積造成巨大的差異，但是 POC 階段可暫時忽略誤差，僅用於驗證資料儲存與存取的實作。

## 理想上的資料儲存實務

雙儲存結構：
- MBTiles
- `/Z/X/Y.png` 檔案系統

遊戲事件觸發並使用遊戲資料更新 MBTiles；
檔案系統則透過排程事件批次運算，讀取 MBTiles 來更新圖磚。

涉及更新運算時，需要額外的資訊來檢索或標記資料，因此需要關聯式資料庫提供這一類的需求；
但是客戶端 GET 圖磚的流量相對大，並不適合直接讀取關聯式資料庫的內容，
因此採用雙軌並行的措施。

## Todo Items

- Create hello world of Cuberite Lua Plugin.
- Create hello world of Lua-Cpp project.
- Find Cpp image library to process PNG
- Find Cpp http library to serve image tiles.
- Find Cpp SQLite library to store image tiles in SQLite.

## Out of Scope

- 以取得地圖資料為第一優先。
    - 不考慮正確或恰當的 Cuberite 事件。
    - 不考慮累增運算...等優化措施。
- 不考慮 1 block = 1 meter = 1 px 帶來的誤差。
