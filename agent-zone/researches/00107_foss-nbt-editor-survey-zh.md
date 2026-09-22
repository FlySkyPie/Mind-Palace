# FOSS Minecraft NBT 編輯器調查：聚焦 Web UI 方案

## 調查背景

Minecraft 的 NBT（Named Binary Tag）格式用於儲存玩家資料、世界資料、結構方塊等遊戲元素。本篇調查範圍限定在**自由與開源軟體**（FOSS, Free and Open Source Software）領域內能夠編輯 `.nbt` 檔案的編輯器，並優先考量 Web UI（瀏覽器介面）方案。

## Web UI 方案（FOSS）

以下為完全開源且可在瀏覽器中執行（或可自行代管）的 NBT 編輯器。

### 1. Bedrock NBT Editor（w1zardz）⭐ 首選

- **網址**：https://w1zardz.github.io/bedrock-nbt-editor/
- **原始碼**：https://github.com/w1zardz/bedrock-nbt-editor
- **授權**：MIT
- **活躍度**：持續維護中，多語言支援（20+ 語言）[^bedrock-nbt-editor]
- **技術**：純靜態 HTML 頁面，可佈署至 GitHub Pages
- **支援格式**：
  - Java 版（big-endian, gzip）
  - Bedrock 版（little-endian + 8-byte header）
  - Varint network NBT
  - `.mcstructure`、`.schem`、`.schematic`、`.litematic`、`level.dat`、`playerdata`
  - 所有五種 NBT 編碼自動偵測
- **核心特色**：
  - 100% 客戶端處理，檔案不上傳伺服器（使用 FileReader + in-memory Blob）
  - 離線載入後可離線使用
  - 真實 64-bit `BigInt` 支援
  - 惰性分頁渲染（大檔案不卡頓）
  - 正確區分 UTF-8 / CESU-8 字串編碼
  - 腳本式 API 暴露於 `window.NBT`
  - 樹狀編輯、搜尋、增刪標籤、SNBT 匯出、格式轉換
  - 行動裝置友善（44px 觸控目標）
- **限制**：無法直接開啟 Java `.mca` region 檔（僅能處理已解壓的區塊承載）、無 Bedrock LevelDB 世界容器、SNBT 匯出唯讀（無法匯入解析）

### 2. webNBT（iRath96）

- **網址**：https://irath96.github.io/webNBT/
- **原始碼**：https://github.com/iRath96/webNBT
- **授權**：原始碼公開，未明確標示授權條款[^webnbt]
- **活躍度**：最後提交 2022-01-15，已停滯
- **技術**：Emscripten（C++ 編譯為 JavaScript）
- **支援格式**：基本 `.nbt`、`.dat`
- **核心特色**：簡單拖放即可使用，GitHub Pages 可自行代管
- **限制**：功能極簡，無 Bedrock 支援、無 SNBT 編輯、無搜尋 / 復原 / 重做

### 3. webNBT-next（dmitibrr）

- **網址**：https://dmitibrr.github.io/webNBT-next/
- **原始碼**：https://github.com/dmitibrr/webNBT-next
- **本質**：webNBT 的分支更新版[^webnbt-next]
- **限制**：與 webNBT 相同的功能侷限

## 桌面端 FOSS 方案（輔助參考）

### 1. NBTExplorer（jaquadro）

- **原始碼**：https://github.com/jaquadro/NBTExplorer
- **授權**：MIT
- **語言**：C#（.NET / Mono）
- **平台**：Windows（原生）、Linux（Mono）、macOS（原生 Cocoa）
- **星數**：~3,000（最受歡迎）[^nbtexplorer]
- **支援格式**：`.nbt`、`.dat`、`.schematic`、`.mcr`（region）、`.mca`（anvil）、Cubic Chunks
- **狀態**：最後提交 2017-11-24，已停止維護
- **限制**：無 Bedrock 支援、Linux 需依賴 Mono

### 2. NBT Studio（tryashtar）

- **原始碼**：https://github.com/tryashtar/nbt-studio
- **授權**：原始碼公開[^nbtstudio]
- **語言**：C#（.NET）
- **平台**：Windows（Microsoft Store + GitHub releases）、跨平台 .NET
- **星數**：~820
- **支援格式**：Java NBT（`.dat`）、Java region（`.mca` / `.mcr`）、**Bedrock NBT**（`.mcstructure` 小端序）、SNBT
- **特色**：NBTExplorer 的精神繼承者，支援復原重做、拖放、多選、陣列十六進位編輯、正規表達式搜尋、剪貼簿 SNBT

### 3. NBTEditor（Sueh-Tam）

- **原始碼**：https://github.com/Sueh-Tam/NBTEditor
- **授權**：MIT
- **語言**：Python（PySide6/Qt）
- **平台**：跨平台（Windows, macOS, Linux）
- **支援格式**：無壓縮 / GZIP / ZLIB 的 `.nbt`、`.dat`、`.schematic`
- **狀態**：最後提交 2026-03-19，持續維護中（最新）
- **限制**：無 region 檔支援、尚無社群採用

## 非開源 Web UI 方案（僅供對照）

| 工具 | 網址 | 特色 | 開源 |
|------|------|------|:---:|
| Mcgic NBT Editor | mcgic.com/nbt/ | 最豐富功能：樹狀+SNTP+十六進位+主題+模板+`.mca` 支援 | ❌ |
| CraftMC NBT Editor | craftmc.net/tools/minecraft-nbt-editor | Java+Bdetails 自動格式偵測、64-bit 安全 | ❌ |
| Chunkweave | nbteditor.org | Region 檔 `.mca`/`.mcr` 瀏覽、非破壞性編輯 | ❌ |
| Nodecraft NBT Editor | nodecraft.com/tools/minecraft/nbt-editor | 物品圖示渲染（含模組）、豐富視覺化 | ❌ |
| MC Tools NBT Editor | mctools.gg/nbt-editor | 簡潔樹狀+SNBT 編輯，僅 Java 版 | ❌ |

## 綜合建議

### 首選：Bedrock NBT Editor（w1zardz）

在所有 FOSS 方案中，**Bedrock NBT Editor** 是唯一同時滿足以下條件的工具：

1. **MIT 授權** — 真正的自由軟體，可自由使用、修改、散佈
2. **Web UI** — 純瀏覽器執行，無需安裝，可佈署至 GitHub Pages 自行代管
3. **持續維護** — 非停滯專案，持續獲得更新
4. **廣泛格式支援** — 同時支援 Java 版與 Bedrock 版 NBT
5. **完整 NBT 功能** — 64-bit 整數、樹狀編輯、格式轉換、搜尋
6. **離線可用** — 載入後不依賴網路
7. **隱私保障** — 100% 客戶端處理

若需要編輯 Java 版 `.mca` / `.mcr` region 檔案，則需搭配桌面端 **NBT Studio**（較新、有 Bedrock 支援）或 **NBTExplorer**（老牌、已停維護）。但純粹編輯 `.nbt` 檔案而言，Bedrock NBT Editor 已足夠勝任日常需求。

---

[^bedrock-nbt-editor]: w1zardz. (n.d.). Bedrock NBT Editor. Retrieved 2026-09-22, from https://github.com/w1zardz/bedrock-nbt-editor
[^webnbt]: iRath96. (n.d.). webNBT. Retrieved 2026-09-22, from https://github.com/iRath96/webNBT
[^webnbt-next]: dmitibrr. (n.d.). webNBT-next. Retrieved 2026-09-22, from https://github.com/dmitibrr/webNBT-next
[^nbtexplorer]: jaquadro. (n.d.). NBTExplorer. Retrieved 2026-09-22, from https://github.com/jaquadro/NBTExplorer
[^nbtstudio]: tryashtar. (n.d.). NBT Studio. Retrieved 2026-09-22, from https://github.com/tryashtar/nbt-studio