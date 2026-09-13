# Behavior Tree 作為 Minecraft Bot AI 之開源專案調查

## 摘要

本報告調查開源社群中，已使用或聲稱使用 Behavior Tree（行為樹）作為 Minecraft Bot 人工智慧架構的專案與相關討論。調查發現，雖然 Behavior Tree 在遊戲 AI 領域已是成熟技術，但直接將之應用於 Minecraft 外部 Bot 的開源專案數量極少，且多數停留於早期階段或概念驗證。主流通路多採用 Finite State Machine（有限狀態機）搭配路徑函式庫（如 Baritone）或 LLM 整合方案。

---

## 1. 直接使用 Behavior Tree 的 Minecraft Bot 專案

### 1.1 BTBot（Scala）

BTBot 是專為 Minecraft 自動化設計的 Behavior Tree 函式庫，以 Scala 撰寫。它依賴 Kunii 的 McBot 框架（針對 Minecraft 1.5.2），需修改 Minecraft 客戶端類別。專案內含一個示範 Behavior Tree 用於戰鬥與物品拾取（BTHuntAndPickup.scala）。其路徑尋找程式碼改編自 Slick2D 函式庫。

- **開發者**：yanich
- **語言**：Scala
- **Minecraft 版本**：1.5.2
- **授權**：未明確標示
- **狀態**：早期階段，部分依賴元件未獲授權發布
- **來源**：GitHub - yanich/BTBot[^btbot]

### 1.2 headlessbot（Java/Kotlin）

headlessbot 是一個無頭 Minecraft Bot，使用 HeadlessMc 與 Baritone 路徑函式庫建構。其 README 明確指出「bot 的行為以 behavior tree 表示」，並包含 Composite Node 與 Decorator Node 的實作規劃，同時計畫開發 WebUI 的樹狀檢視器與編輯器。

- **開發者**：nothub
- **語言**：Java（基於 Minecraft Forge 1.12.2）
- **授權**：未明確標示
- **狀態**：開發中，部分 Behavior Tree 節點已實作
- **來源**：GitHub - nothub/headlessbot[^headlessbot]

### 1.3 Neurorobot（Java，Fabric Mod）

Neurorobot 是一個 Fabric 模組，在 Minecraft 1.21 中加入 AI 機器人夥伴。其 README 宣稱具備「Advanced AI Behavior Tree」。然而根據原始碼分析，其實際架構是一個以 `AIState` 列舉（IDLE、FOLLOWING、MINING、EXPLORING、MOVING、WORKING）為基礎的簡單狀態機，並未使用正規的 Behavior Tree 框架。該專案的「Behavior Tree」宣稱較為浮誇，實際實作偏向 Finite State Machine。

- **開發者**：Player9753193
- **語言**：Java（Fabric）
- **Minecraft 版本**：1.21
- **授權**：All Rights Reserved
- **狀態**：早期開發，採購 Baritone 為依賴
- **來源**：GitHub - Player9753193/Neurorobot[^neurorobot]

---

## 2. 相關但非 Behavior Tree 的 Minecraft Bot 專案

### 2.1 mindcraft（JavaScript/Node.js）

mindcraft 是目前最活躍的 LLM-based Minecraft Bot 專案之一。它結合 Mineflayer 與大型語言模型，讓 Bot 能透過自然語言理解與執行任務。其架構不使用 Behavior Tree，而是以 LLM 作為決策核心。

- **來源**：GitHub - mindcraft-bots/mindcraft[^mindcraft]

### 2.2 mineflayer-statemachine（JavaScript/Node.js）

mineflayer-statemachine 是 Mineflayer 的 Finite State Machine 插件，提供高層級 API 來編寫狀態機。有趣的是，其 README 直接將此工具描述為「幫助管理 Bot Behavior Tree 的品質」，反映出社群中常將 Finite State Machine 與 Behavior Tree 混用的現象。

- **來源**：GitHub - PrismarineJS/mineflayer-statemachine[^mineflayer-sm]

### 2.3 airi-minecraft（TypeScript）

moeru-ai 開發的 AIRI 是一個 LLM 驅動的 Minecraft Bot，能理解自然語言指令並與世界互動。不使用 Behavior Tree。

- **來源**：GitHub - moeru-ai/airi-minecraft[^airi]

### 2.4 craftmind-fishing（JavaScript）

craftmind-fishing 是一個透過 RCON 協定自動釣魚的 Minecraft Bot，內建「行為引擎」（autonomous behavior engine），但其決策邏輯屬於簡單的計時器與隨機變化模式，非 Behavior Tree。

- **來源**：GitHub - Lucineer/craftmind-fishing[^craftmind]

---

## 3. Behavior Tree 在遊戲 AI 中的定位

Behavior Tree 最初是為遊戲中的 NPC（非玩家角色）設計的 AI 架構，用以取代 Finite State Machine 在複雜行為下的維護困難[^bt-history]。其核心優勢在於模組化、可重用、易於除錯與視覺化。

然而值得注意的是，在 Minecraft Bot 領域，主流開源專案並未大規模採用 Behavior Tree。可能原因包括：

1. **Baritone 的主導地位**：Baritone 是 Minecraft 最成熟的路徑尋找與自動化函式庫，其架構基於 Goal-Oriented 而非 Behavior Tree。
2. **LLM 浪潮**：2024-2025 年間，多數新專案選擇 LLM 作為決策核心（如 mindcraft、AIRI），而非傳統的 Behavior Tree。
3. **實作成本**：完整的 Behavior Tree 框架需要較多的前期基礎建設，對於小型或個人專案門檻較高。

---

## 4. 結論

截至目前（2026 年），開源社群中直接使用 Behavior Tree 作為 Minecraft Bot AI 的專案非常稀少。**BTBot** 是最純粹的範例，但已年久失修（Minecraft 1.5.2/2013 年）。**headlessbot** 是最有潛力的現代化實作，但仍處於開發階段。**Neurorobot** 雖聲稱使用 Behavior Tree，實際為 Finite State Machine。

若要在 Minecraft Bot 專案中採用 Behavior Tree，目前並無成熟、維護中的開源先例可循，可能需要從頭打造或移植既有 Behavior Tree 框架（如 Behavior3、py_trees、BehaviorTree.CPP）至 Minecraft Bot 生態系。

---

[^btbot]: yanich. (n.d.). BTBot: A behavior tree AI for playing Minecraft. GitHub. Retrieved 2026-09-13, from https://github.com/yanich/BTBot

[^headlessbot]: nothub. (n.d.). headlessbot: A headless Minecraft bot. GitHub. Retrieved 2026-09-13, from https://github.com/nothub/headlessbot

[^neurorobot]: Player9753193. (n.d.). Neurorobot: A Minecraft 1.21 fabric mod adds Neurorobot. GitHub. Retrieved 2026-09-13, from https://github.com/Player9753193/Neurorobot

[^mindcraft]: mindcraft-bots. (n.d.). mindcraft: Minecraft AI with LLMs+Mineflayer. GitHub. Retrieved 2026-09-13, from https://github.com/mindcraft-bots/mindcraft

[^mineflayer-sm]: PrismarineJS. (n.d.). mineflayer-statemachine. GitHub. Retrieved 2026-09-13, from https://github.com/PrismarineJS/mineflayer-statemachine

[^airi]: moeru-ai. (n.d.). airi-minecraft: An intelligent Minecraft bot powered by LLM. GitHub. Retrieved 2026-09-13, from https://github.com/moeru-ai/airi-minecraft

[^craftmind]: Lucineer. (n.d.). craftmind-fishing. GitHub. Retrieved 2026-09-13, from https://github.com/Lucineer/craftmind-fishing

[^bt-history]: Colledanchise, M., & Ögren, P. (2017). Behavior Trees in Robotics and AI: An Introduction. CRC Press.