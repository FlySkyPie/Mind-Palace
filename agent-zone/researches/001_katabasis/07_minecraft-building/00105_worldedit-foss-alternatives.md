# WorldEdit 的 FOSS 替代方案

[WorldEdit](https://github.com/enginehub/worldedit) 是 Minecraft 最經典的世界編輯工具之一，支援 Bukkit/Spigot/Paper 插件、Forge/Fabric 模組，以及獨立客戶端。它以指令操作（`//set`, `//replace`, `//copy`, `//paste` 等）為核心，提供選取區域、生成幾何形狀、地形編輯等功能。

本報告整理目前仍活躍的 FOSS（自由開源）替代方案，涵蓋不同生態系與使用情境。

---

## 1. FastAsyncWorldEdit (FAWE)

- **倉庫：** [github.com/IntellectualSites/FastAsyncWorldEdit](https://github.com/IntellectualSites/FastAsyncWorldEdit)[^fawe]
- **授權：** GPL-3.0
- **類型：** Bukkit/Spigot/Paper 插件 + Fabric/Forge/Sponge 模組

FAWE 是 WorldEdit 的 **高效能分支**，API 完全相容，只需安裝便能讓依賴 WorldEdit 的插件自動獲得效能提升。特色包括：

- 非同步區塊編輯，大幅降低伺服器延遲
- 無限復原/重做，跨世界歷史記錄
- 跨伺服器剪貼簿（透過 Web 整合）
- 200+ 指令，進階筆刷、遮罩、花樣語法
- 圖片匯入、生態系混合、洞穴生成、離軸旋轉

維護狀態：**非常活躍**（806 stars，7,297 commits，持續在 Modrinth/CurseForge 發布）。

> 本質上是「WorldEdit++」，若需要 WorldEdit API 相容性且要求效能，這是首選。

---

## 2. Litematica

- **倉庫：** [github.com/maruohon/litematica](https://github.com/maruohon/litematica)[^litematica]
- **授權：** LGPL-3.0
- **類型：** Fabric/LiteLoader/Forge 用戶端模組

Litematica 是 **客戶端側的示意圖模組**，不提供 `//set` 等指令，而是專注於：

- 示意圖載入、預覽與放置
- 區域複製、移動、填充、刪除
- 豐富的 GUI 操作（而非指令）
- 材料列表計算

它是 Schematica 的精神繼承者，特別適合不想要伺服器端模組的玩家。需要 malilib 函式庫。

維護狀態：**非常活躍**（954 stars，1,037 commits，持續支援最新 Minecraft 版本）。

---

## 3. Amulet Map Editor

- **倉庫：** [github.com/Amulet-Team/Amulet-Map-Editor](https://github.com/Amulet-Team/Amulet-Map-Editor)[^amulet]
- **授權：** 自訂開源授權
- **類型：** 獨立桌面應用程式（Python/PyQt）

Amulet 是 **MCEdit 的現代替代品**，支援 **Java 1.12+ 與 Bedrock 1.7+** 的所有世界格式。功能包括：

- 方塊操作、實體編輯、NBT 編輯
- 世界格式轉換（Java ↔ Bedrock）
- 篩選器（Filters）腳本擴充
- 跨平台（Windows/macOS/Linux）

維護狀態：**活躍**（2.2k stars，2,103 commits，官方網站 [amuletmc.com](https://www.amuletmc.com) 提供安裝檔）。

---

## 4. WorldPainter

- **倉庫：** [github.com/Captain-Chaos/WorldPainter](https://github.com/Captain-Chaos/WorldPainter)[^worldpainter]
- **授權：** GPL-3.0
- **類型：** 獨立桌面應用程式（Java）

WorldPainter 是 **互動式地圖產生器**，而非區塊編輯器。它像繪圖軟體一樣讓使用者「繪製」地形：

- 雕刻地形、繪製材質/生態系
- 筆刷產生樹木、雪、冰
- 匯入匯出 `.schematic` 檔案

與 WorldEdit 的定位不同：WorldPainter 適合從零打造自訂世界地圖，而非編輯既有建築。

維護狀態：**活躍**（453 stars，2,503 commits，官方網站 [worldpainter.net](https://www.worldpainter.net)）。

---

## 5. SimpleEdit

- **Modrinth：** [modrinth.com/plugin/simpleedit](https://modrinth.com/plugin/simpleedit)[^simpleedit]
- **授權：** Apache-2.0
- **類型：** Paper 插件

SimpleEdit 是 **現代化的 WorldEdit 靈感插件**，支援 Minecraft 26.1（最新版）。功能涵蓋：

- 選取系統（木斧、pos1/pos2）
- 方塊操作（set/replace/outline/walls/hollow）
- 筆刷系統（球體/平滑/替換/示意圖）
- 幾何形狀（球體/圓柱/金字塔/圓頂/圓錐/直線）
- 地形生成（洞穴/山脈/森林/島嶼/迷宮/神廟/廢墟）
- 進階操作（平滑/自然化/變形/侵蝕/漸層）
- 完整復原/重做與遮罩系統

號稱單次操作可處理 5000 萬個方塊，採用非同步處理（8000 blocks/tick）。

維護狀態：**活躍**（1.2k 下載量，2 個月前更新）。注意：公開 GitHub 倉庫不易找到，但宣稱 Apache-2.0 授權。

---

## 6. NusaWEdit

- **SpigotMC：** [spigotmc.org/resources/nusawedit.125695](https://www.spigotmc.org/resources/nusawedit.125695)[^nusawedit]
- **授權：** 免費插件（授權條款未明確標示）
- **類型：** Spigot/Paper 插件

NusaWEdit 是 **輕量、對玩家友好的 WorldEdit 替代品**，專為生存/創造伺服器設計：

- **虛擬物品欄**：操作消耗虛擬物品欄中的材料
- **預覽模式**：提交前可預覽變更
- **粒子視覺化**：顯示選取範圍
- **保護插件整合**：支援 WorldGuard、Towny、GriefPrevention、PlotSquared、SuperiorSkyblock
- ** Rank 限制**：不同玩家組別有不同方塊數量限制
- 簡潔指令：`/nwe set`, `/nwe replace`, `/nwe undo`

維護狀態：**活躍**，支援 1.20–1.21。

---

## 7. Makkit

- **倉庫：** [github.com/ejektaflex/Makkit](https://github.com/ejektaflex/Makkit)[^makkit]
- **授權：** GPL-3.0
- **類型：** Fabric 模組

Makkit 是 **GUI 驅動的輕量世界編輯器**，而非指令操作：

- 滑鼠驅動的即時選取框（多人模式下彼此可見）
- 複製貼上（支援旋轉與鏡像）
- 花樣工具（tiling / 重複方塊）
- 復原/重做歷史
- 選取框可滑鼠拖曳移動與縮放

開發者明確表示「不打算取代 WorldEdit，而是相輔相成」。

維護狀態：**低活動 / Beta**（6 stars，273 commits，最後更新約 3 年前，僅支援 1.16.5）。不建議用於現代 Minecraft 版本。

---

## 8. BlockEdit

- **倉庫：** [github.com/YouHaveTrouble/BlockEdit](https://github.com/YouHaveTrouble/BlockEdit)[^blockedit]
- **授權：** GPL-2.0
- **類型：** Paper 插件

BlockEdit 是 **為現代 Paper 伺服器設計的輕量替代品**，特色：

- **無 NMS**：不依賴 Minecraft 內部程式碼，跨版本無需更新
- **Work Splitter**：按區塊分散運算至不同 Tick，減少延遲
- **簡單 API**：易於擴充自訂操作

開發者明確表示「這只是個興趣專案，永遠不會跟 WorldEdit 一樣好」。

維護狀態：**低活動 / 業餘專案**（14 stars，73 commits，仍在開發中）。不建議用於生產環境。

---

## 9. MCEdit 2（已淘汰）

- **倉庫：** [github.com/mcedit/mcedit2](https://github.com/mcedit/mcedit2)[^mcedit2]
- **授權：** BSD
- **類型：** 獨立桌面應用程式

MCEdit 曾是經典的獨立世界編輯器，可編輯世界所有面向、匯入匯出 `.schematic` 檔案。然而：

- 最後一次提交：**2018 年 8 月**
- 不支援 Minecraft 1.13+ 的世界格式
- **Amulet Map Editor** 是公認的現代替代品

---

## 總結比較

| 專案 | 類型 | 授權 | 支援版本 | 維護狀態 |
|---|---|---|---|---|
| **FAWE** | Bukkit/Fabric/Forge/Sponge | GPL-3.0 | 1.20–1.21+ | 🟢 非常活躍 |
| **Litematica** | Fabric/LiteLoader/Forge | LGPL-3.0 | 1.21+ | 🟢 非常活躍 |
| **Amulet Editor** | 獨立桌面 | 自訂開源 | 1.12–1.21+ | 🟢 活躍 |
| **WorldPainter** | 獨立桌面 | GPL-3.0 | 全版本 | 🟢 活躍 |
| **SimpleEdit** | Paper 插件 | Apache-2.0 | 26.1 | 🟢 活躍 |
| **NusaWEdit** | Spigot/Paper 插件 | 免費 | 1.20–1.21 | 🟢 活躍 |
| **Makkit** | Fabric 模組 | GPL-3.0 | 1.16.5 | 🟡 Beta/低活動 |
| **BlockEdit** | Paper 插件 | GPL-2.0 | 現代 Paper | 🟡 業餘專案 |
| **MCEdit 2** | 獨立桌面 | BSD | 1.12 以前 | 🔴 已淘汰 |

---

## 快速推薦

- **想要完整 WorldEdit 體驗但更快** → **FAWE**（相容 API，效能大幅提升）
- **想要用戶端示意圖操作（不需伺服器模組）** → **Litematica**
- **想要現代獨立桌面編輯器** → **Amulet Map Editor**（MCEdit 的繼承者）
- **想要對生存玩家友好的編輯工具** → **NusaWEdit**
- **想要從零繪製/生成地形** → **WorldPainter**
- **想要現代 Paper 插件且功能完整** → **SimpleEdit**（但原始碼可取得性不明）

---

## 非 FOSS 的知名選擇（僅供參考）

- **Axiom**（[modrinth.com/mod/axiom](https://modrinth.com/mod/axiom)）[^axiom] — 熱門 Fabric 模組，Blender 風格介面。**非開源**（ARR 授權），多人模式需 Patreon/商業授權。

---

[^fawe]: IntellectualSites. (n.d.). FastAsyncWorldEdit. Retrieved 2026-09-22, from https://github.com/IntellectualSites/FastAsyncWorldEdit

[^litematica]: maruohon. (n.d.). Litematica. Retrieved 2026-09-22, from https://github.com/maruohon/litematica

[^amulet]: Amulet-Team. (n.d.). Amulet-Map-Editor. Retrieved 2026-09-22, from https://github.com/Amulet-Team/Amulet-Map-Editor

[^worldpainter]: Captain-Chaos. (n.d.). WorldPainter. Retrieved 2026-09-22, from https://github.com/Captain-Chaos/WorldPainter

[^simpleedit]: SimpleEdit. (n.d.). Retrieved 2026-09-22, from https://modrinth.com/plugin/simpleedit

[^nusawedit]: NusaWEdit. (n.d.). Retrieved 2026-09-22, from https://www.spigotmc.org/resources/nusawedit.125695

[^makkit]: ejektaflex. (n.d.). Makkit. Retrieved 2026-09-22, from https://github.com/ejektaflex/Makkit

[^blockedit]: YouHaveTrouble. (n.d.). BlockEdit. Retrieved 2026-09-22, from https://github.com/YouHaveTrouble/BlockEdit

[^mcedit2]: mcedit. (n.d.). mcedit2. Retrieved 2026-09-22, from https://github.com/mcedit/mcedit2

[^axiom]: Axiom. (n.d.). Retrieved 2026-09-22, from https://modrinth.com/mod/axiom