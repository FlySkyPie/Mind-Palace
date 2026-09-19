# Cuberite 外掛 HTTP 伺服器與 PNG 圖片生成 API 研究

## 問題一：外掛是否有建立 HTTP 伺服器的 API？

**有，Cuberite 提供兩種方式讓外掛建立 HTTP 伺服器或處理 HTTP 請求。**

### 方式一：使用 `cNetwork:Listen()` 建立獨立 TCP/HTTP 伺服器

Cuberite 的 `cNetwork` 類別提供了低階網路 API，允許外掛在任意埠監聽 TCP 連線[^cnetwork]。使用方式如下：

```lua
local Server = cNetwork:Listen(8080, ListenCallbacks)
```

`ListenCallbacks` 是一個回呼表，包含以下事件：

- **`OnIncomingConnection(a_RemoteIP, a_RemotePort)`** — 有新的連線進入，須回傳 `LinkCallbacks` 或 `nil`（拒絕連線）
- **`OnAccepted(a_TCPLink)`** — 新連線已被接受，可開始傳送資料
- **`OnError(a_ErrorCode, a_ErrorMsg)`** — 監聽失敗

官方 `NetworkTest` 外掛示範了如何實作 HTTP 伺服器：在 `OnReceivedData` 中解析 HTTP 請求頭，偵測到 `\r\n\r\n` 後回傳 HTTP 回應[^networktest]。

```lua
OnReceivedData = function (a_Link, a_Data)
    IncomingData = IncomingData .. a_Data
    if (IncomingData:find("\r\n\r\n")) then
        local Content = os.date()
        a_Link:Send("HTTP/1.0 200 OK\r\nContent-type: text/plain\r\nContent-length: " .. #Content .. "\r\n\r\n" .. Content)
        a_Link:Shutdown()
    end
end
```

此方式需要自行處理 HTTP 協定細節（請求解析、標頭、Content-Length 等），但可完全自訂伺服器行為，且不依賴 Cuberite 內建的 WebAdmin。

### 方式二：使用 `cWebAdmin:AddWebTab()` 註冊 WebAdmin 頁面

Cuberite 內建的 WebAdmin 提供 HTTP 伺服器，外掛可透過 `cWebAdmin:AddWebTab()` 註冊自訂頁面[^cwebadmin]：

```lua
cWebAdmin:AddWebTab("Title", "UrlPath", HandlerFn)
```

`HandlerFn` 簽章為：

```lua
function (a_Request, a_UrlPath) return Content, ContentType end
```

其中 `a_Request` 是 `HTTPRequest` 物件，包含請求方法、路徑、參數、表單資料等[^httprequest]。`ContentType` 預設為 `text/html`。

此方式不需自行處理 HTTP 協定，整合在既有 WebAdmin 中，適合儀表板、管理介面等用途。但路徑受限於 `/webadmin/{PluginName}/{UrlPath}`，且依賴 WebAdmin 的啟用狀態與認證機制。

## 問題二：外掛是否有建立 PNG 圖片的 API？

**沒有，Cuberite 核心 API 不包含任何 PNG 圖片產生或編碼的功能。**

具體來說：

- Cuberite 的 Lua API 中不存在 PNG 編碼器（encoder）、點陣圖畫布（canvas）或任何 2D 繪圖功能
- `cColor` 類別僅表示 RGB 顏色值，用於遊戲內物品（如盔甲），不具備圖片產生能力[^ccolor]
- `cJson` 類別可處理 JSON 序列化，但與圖片無關[^cjson]
- `cFile` 類別提供檔案系統操作，但僅是讀寫原始位元組，不包含任何圖片格式處理[^cfile]

### 最接近的內建功能：`cMap:SetPixel()`

Cuberite 的 `cMap` 類別允許操作遊戲內 Minecraft 地圖物品的像素[^cmap]。可使用 `SetPixel(x, z, ColorID)` 設定像素顏色，但這僅作用於 Minecraft 地圖物品（128×128 解析度，約 64 色限制色盤），並非產生真正的 PNG 圖片檔。

### 第三方解決方案：純 Lua PNG 程式庫

若外掛需要產生 PNG 圖片，需自行帶入純 Lua 的 PNG 編碼程式庫。已知的相關資源：

- **Cuberite-CustomMaps 外掛**[^custommaps] 使用了純 Lua 的 PNG 解碼器（`png.lua`，源自 [png-lua](https://github.com/Didericis/png-lua)），但該程式庫僅能「讀取」PNG，無法「產生」PNG
- 社群（LuaRocks）中存在純 Lua 的 PNG 編碼器，但 Cuberite 未內建，需外掛自行整合
- Cuberite-CustomMaps 的 README 警告：純 Lua 圖片處理在大圖片下可能觸發 Cuberite 的 deadlock 偵測器

### 使用 `cUrlClient` 代理圖片產生

另一種迂迴方式：外掛可使用 `cUrlClient` 呼叫外部 HTTP 服務來產生圖片[^curlclient]，但這需要外部依賴。

## 結論

| 問題 | 答案 |
|------|------|
| 外掛可建立 HTTP 伺服器嗎？ | ✅ **可以**。透過 `cNetwork:Listen()` 建立獨立伺服器，或透過 `cWebAdmin:AddWebTab()` 註冊 WebAdmin 頁面 |
| 外掛可產生 PNG 圖片嗎？ | ❌ **否**。Cuberite 核心無內建 PNG 編碼器或繪圖 API，需自行帶入純 Lua 程式庫或使用外部服務 |

[^cnetwork]: Cuberite 團隊。 (n.d.). cNetwork Class. Retrieved 2026-09-13, from https://api.cuberite.org/cNetwork.html
[^networktest]: Cuberite 團隊。 (n.d.). NetworkTest.lua. Retrieved 2026-09-13, from https://raw.githubusercontent.com/cuberite/cuberite/master/Server/Plugins/NetworkTest/NetworkTest.lua
[^cwebadmin]: Cuberite 團隊。 (n.d.). cWebAdmin Class. Retrieved 2026-09-13, from https://api.cuberite.org/cWebAdmin.html
[^httprequest]: Cuberite 團隊。 (n.d.). HTTPRequest Class. Retrieved 2026-09-13, from https://api.cuberite.org/HTTPRequest.html
[^ccolor]: Cuberite 團隊。 (n.d.). cColor Class. Retrieved 2026-09-13, from https://api.cuberite.org/cColor.html
[^cjson]: Cuberite 團隊。 (n.d.). cJson Class. Retrieved 2026-09-13, from https://api.cuberite.org/cJson.html
[^cfile]: Cuberite 團隊。 (n.d.). cFile Class. Retrieved 2026-09-13, from https://api.cuberite.org/cFile.html
[^cmap]: Cuberite 團隊。 (n.d.). cMap Class. Retrieved 2026-09-13, from https://api.cuberite.org/cMap.html
[^curlclient]: Cuberite 團隊。 (n.d.). cUrlClient Class. Retrieved 2026-09-13, from https://api.cuberite.org/cUrlClient.html
[^custommaps]: Mike Jagdis。 (n.d.). Cuberite-CustomMaps. Retrieved 2026-09-13, from https://git.eris-associates.co.uk/active/Cuberite-CustomMaps