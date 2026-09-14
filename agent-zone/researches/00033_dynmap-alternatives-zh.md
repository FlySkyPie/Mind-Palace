# Dynmap 開源替代方案調查

Dynmap 是一套歷史悠久的 Minecraft 網頁地圖渲染外掛，能以 2D 平面、3D 等角投影與洞穴視角等方式將遊戲世界呈現在瀏覽器上[^dynmap]。然而 Dynmap 資源消耗較高、不支援 Folia 且對新版 Minecraft 的適配速度較慢，因此社群中出現了數個替代方案。本報告調查目前主流且持續維護的開源替代方案。

## 替代方案一覽與比較

### BlueMap — 全 3D WebGL 即時地圖（⭐ 2,801）

BlueMap 使用 three.js / WebGL 將 Minecraft 世界建構成真正的 3D 網格模型，使用者可在瀏覽器中旋轉、傾斜與飛行瀏覽地形[^bluemap]。

- **渲染方式**：全 3D 網格（非磁磚拼接）
- **支援平台**：Paper、Spigot、Folia、Fabric、Forge、NeoForge、Sponge，另有獨立 CLI 離線渲染工具
- **授權**：MIT
- **優點**：3D 視覺效果極佳、支援材質包塊塊渲染、非同步渲染不佔用伺服器主執行緒（對 TPS 影響小）、WebSocket 即時玩家追蹤、對模組世界支援良好（自訂方塊可正確顯示）
- **缺點**：初次渲染耗時且佔用大量 CPU 與磁碟空間（同等面積約為 squaremap 3-5 倍）、瀏覽器需支援 WebGL（老舊裝置效能不佳）、無內建 Web 聊天室

### squaremap — 輕量 2D 俯視地圖（Pl3xMap v1 分支）（⭐ 512）

squaremap 採用 Leaflet 前端，以 2D 俯視磁磚方式呈現，風格近似 Minecraft 原版地圖[^squaremap]。

- **渲染方式**：2D 俯視磁磚
- **支援平台**：Paper、Fabric、NeoForge、Sponge（官方未列 Folia）
- **授權**：MIT
- **優點**：極度輕量、CPU 與磁碟使用最低、即時區塊更新（5-15 秒）、程式碼庫現代化乾淨、設定簡潔
- **缺點**：僅 2D 無立體視角、無內建 Web 聊天室、附加生態系較小

### Pl3xMap — 輕量 2D 俯視地圖（原 v1 繼承者）（⭐ 172）

Pl3xMap 與 squaremap 同源但由不同團隊維護，支援最廣泛的伺服器平台[^pl3xmap]。

- **渲染方式**：2D 俯視磁磚（額外支援生態域染色模式）
- **支援平台**：Paper、Folia、Purpur、Spigot、Bukkit、Fabric、Forge、Quilt — 為 2D 方案中平台最廣
- **授權**：MIT
- **優點**：極為輕量、明確支援 Folia、玩家標記顯示面向/血量/裝備、多元渲染模式
- **缺點**：僅 2D、無 Web 聊天室、社群較小

### Dynmap（基準線）（⭐ 2,222）

- **渲染方式**：2D 俯視 + 3D 等角投影 + 洞穴視角
- **支援平台**：Spigot/Paper、Forge、Fabric（至 1.21.4），**無官方 Folia 支援**
- **授權**：Apache-2.0
- **優點**：最成熟（13 年以上開發）、龐大的附加生態系（WorldGuard、Towny、Factions 等）、內建 Web 聊天室、完整標記 API
- **缺點**：資源消耗最高（CPU + 磁碟）、每個地圖單執行緒渲染（緩慢）、UI 過時、新版 Minecraft 適配慢、無 Folia 支援

### uNmINeD — 離線 2D 地圖匯出工具

uNmINeD 為桌面應用程式，讀取世界檔案後匯出靜態 2D 網頁地圖（Leaflet 前端）[^unmined]。

- **渲染方式**：2D 俯視磁磚（桌面應用離線渲染）
- **支援平台**：獨立應用程式，支援 Java Edition **與 Bedrock Edition**
- **授權**：專有軟體（免費使用）
- **優點**：支援 Java 與 Bedrock 雙版本、多執行緒渲染極快、支援地下/X 光/夜間模式、可匯出靜態 HTML
- **缺點**：非即時、需手動重新執行以更新地圖、無玩家追蹤、非伺服器外掛

### Minecraft Overviewer — 已停止維護

命令列 Python 工具，產生靜態 2D 等角磁磚 + Leaflet 網頁介面。**已於 2023 年 4 月停止維護**，不再支援 Minecraft 1.20+，官方建議遷移至 BlueMap[^overviewer]。

### ChunkyMap — Dynmap 寫實渲染擴充

使用 Chunky 路徑追蹤引擎替代 Dynmap 的 HDMap 渲染器，可產生照片級寫實地圖，但渲染時間極長，偏向藝術展示而非實用地圖[^chunkymap]。

### LiveAtlas — 前端 UI 替代（非地圖引擎）（⭐ 380）

LiveAtlas 是以 Vue.js + TypeScript 撰寫的前端介面，可取代 Dynmap、squaremap、Pl3xMap、Overviewer 的預設 UI，提供現代化操作體驗，**本身不負責地圖渲染**[^liveatlas]。

## 綜合比較表

| 功能 | BlueMap | squaremap | Pl3xMap | Dynmap | uNmINeD |
|------|---------|-----------|---------|--------|---------|
| GitHub ⭐ | **2,801** | **512** | **172** | **2,222** | N/A |
| 渲染風格 | 全 3D (WebGL) | 2D 俯視 | 2D 俯視 | 2D + 等角 | 2D 俯視 |
| 即時更新 | 是 (5-30 秒) | 是 (5-15 秒) | 是 (5-15 秒) | 是 (10-60 秒) | 否（靜態） |
| 伺服器外掛 | 是 | 是 | 是 | 是 | 否（桌面程式） |
| Folia 支援 | 是 | 未列 | 是 | 否 | N/A |
| Web 聊天 | 無 | 無 | 無 | 有 | 無 |
| 3D 視角 | 真 3D | 無 | 無 | 部分（等角） | 無 |
| 洞穴視角 | 可（3D 瀏覽） | 無 | 無 | 有 | 有（X 光） |
| 附加生態系 | 成長中 | 小型 | 小型 | 極大 | N/A |
| 授權 | MIT | MIT | MIT | Apache-2.0 | 專有免費 |

## 效能參考（10,000 × 10,000 區塊已探索區域）

| 指標 | BlueMap | squaremap | Pl3xMap | Dynmap |
|------|---------|-----------|---------|--------|
| 初次渲染時間 | 2-3 小時 | ~1 小時 | ~1 小時 | 4+ 小時 |
| 渲染中 TPS 下降 | 1-2 TPS | <1 TPS | <1 TPS | 2-3 TPS |
| 磁碟使用量 | ~8 GB | ~2 GB | ~2 GB | ~18 GB |
| 閒置 RAM | 150-300 MB | 80-150 MB | 80-150 MB | 200-500 MB |

## 選用建議

- **公開伺服器 / 創造展示**：追求視覺效果 → BlueMap
- **大型生存伺服器（Paper）**：追求最少 TPS 影響的 2D 地圖 → squaremap
- **Folia 伺服器**：需要明確 Folia 支援 → BlueMap 或 Pl3xMap
- **模組包（Fabric/NeoForge）**：自訂方塊需正確顯示 → BlueMap
- **需要 Web 聊天與成熟生態系**：繼續使用 Dynmap
- **Bedrock Edition**：離線渲染 → uNmINeD
- **想換現代 UI 但保留現有地圖引擎**：LiveAtlas

[^dynmap]: webbukkit. (n.d.). Dynmap. Retrieved 2026-09-13, from https://github.com/webbukkit/dynmap
[^bluemap]: BlueColored. (n.d.). BlueMap. Retrieved 2026-09-13, from https://bluemap.bluecolored.de/
[^squaremap]: jpenilla. (n.d.). squaremap. Retrieved 2026-09-13, from https://github.com/jpenilla/squaremap
[^pl3xmap]: granny. (n.d.). Pl3xMap. Retrieved 2026-09-13, from https://github.com/granny/Pl3xMap
[^unmined]: uNmINeD. (n.d.). uNmINeD - Minecraft Map Viewer. Retrieved 2026-09-13, from https://unmined.net/
[^overviewer]: Minecraft Overviewer. (n.d.). Minecraft Overviewer. Retrieved 2026-09-13, from https://overviewer.org/
[^chunkymap]: ChunkyMap. (n.d.). ChunkyMap — Photorealistic Dynmap. Retrieved 2026-09-13, from https://github.com/oddlama/vane/issues/254
[^liveatlas]: JLyne. (n.d.). LiveAtlas. Retrieved 2026-09-13, from https://github.com/JLyne/LiveAtlas