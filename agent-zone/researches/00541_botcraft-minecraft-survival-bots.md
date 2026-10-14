# 以 Botcraft 為基礎、且能在生存模式存活的 FOSS Minecraft 機器人調查

## 摘要

本報告調查「以 Botcraft（`https://github.com/adepierre/Botcraft`）為基礎的機器人實作」，並進一步整理「能在 Minecraft 生存模式（survival mode）中存活、且為自由開源軟體（FOSS）的機器人」。結論是：Botcraft 本身是一套以 C++ 撰寫的機器人函式庫，其官方範例（Examples）即為可在生存模式運作的機器人程式，但 GitHub 的「Used by / dependents」圖譜顯示目前沒有任何公開的第三方獨立專案依賴它；生態系主要由 Botcraft 本體、官方範例、專案模板與數個個人 fork 組成[^dependents][^clients]。若把範圍放寬到「所有能在生存模式存活的 FOSS 機器人」，則以 Baritone 為基礎的 AltoClef 是史上第一個全自動破關 Minecraft 的機器人，而 Mindcraft（基於 Mineflayer）則是目前最活躍、最完整的 LLM 驅動生存代理人[^altoclef-readme][^mindcraft-readme]。

本報告的事實陳述以 2026-10-14 檢索到的公開網頁為準；GitHub 星數與活躍度屬易變資訊，閱讀時請留意時效。

## 背景：Botcraft 是什麼

Botcraft 由 adepierre 開發，官方描述為「一個跨平台的 C++ 函式庫，用來建立能連線並與 Minecraft 伺服器互動的機器人，並可選擇整合 OpenGL 渲染器」[^botcraft-repo]。截至檢索時，該專案約有 642 顆星、採用 GPL v3 授權，最後一次提交約在 2026-10-03[^botcraft-repo][^stars]。作者自述這是一個「learn-by-doing」的個人寵物專案，定位更接近「公開的私有儲存庫」而非社群型開源專案，通常每月第一週推送更新[^botcraft-readme]。

其核心能力包括：支援所有官方版本 1.12.2 至 26.3、離線（cracked）與線上（Microsoft 帳號）連線、DNS 與 SRV 解析、壓縮與加密、1.19+ 的簽章聊天、原生物理與碰撞、完整世界資料與實體支援，以及以行為樹（behaviour tree）為基礎的機器人程式設計[^botcraft-readme]。內建行為涵蓋：尋路（含爬梯與鷹架）、方塊破壞（含正確工具與加速效果）、物品欄管理（含箱子、鐵砧、釀造台互動）、方塊放置（原版或空中放置）、實體互動、物品使用（釣竿、進食）、方塊互動（按鈕、拉桿、床）以及村民交易（1.14+）與合成[^botcraft-readme]。

值得注意的是，Botcraft 明確支援在生存模式下運作。官方 README 舉例說明：「一個由 10 個生存模式機器人協作建造像素畫的範例。它們全部處於生存模式，因此必須從箱子中挑選正確的方塊、吃食物，而且顯然不能飛行。整個過程沒有任何全域監督，它們彼此也無法通訊。」[^botcraft-readme]。README 亦明確列出「不支援」項目：反作弊規避（動作接近原版但不擬人化）、完整模組伺服器支援，以及 macOS 的渲染與 UI[^botcraft-readme]。

## 以 Botcraft 為基礎的實作

### 官方範例（Botcraft 儲存庫內的 Examples/）

Botcraft 官方提供編號 0 至 7 的範例程式，這些即為最直接的「Botcraft 驅動實作」[^botcraft-readme][^clients]：

| 編號 | 名稱 | 功能 | 生存模式運作 |
|---|---|---|---|
| 0 | HelloWorld | 連線後在聊天室送出「Hello, World!」後斷線 | 否（僅連線示範） |
| 1 | UserControlledExample | 以滑鼠鍵盤控制的玩家，可搭配 OpenGL 渲染器 | 可（本身即為可遊玩的客戶端） |
| 2 | ChatCommandExample | 透過聊天指令控制的機器人 | 是 |
| 3 | SimpleAFKExample | 極輕量的 AFK 機器人，連線後靜止不動 | 否（僅掛機） |
| 4 | MapCreatorExample | 依 NBT 結構檔協作建造像素畫，可同時啟動多個機器人 | 是 |
| 5 | MobHitterExample | 攻擊範圍內所有敵對生物 | 是（但不進食，會餓死） |
| 6 | DispenserFarmExample | 全自動發射器農場 | 是 |
| 7 | WorldEaterExample | 全自動「世界吞噬者」挖掘程式 | 是 |

其中三個範例與「生存能力」最直接相關：

- **Example 4（MapCreatorExample）**：多機器人合作建造以地圖為基礎的像素畫。官方說明它們全處於生存模式，必須從箱子挑選正確方塊、進食，且不能飛行，過程中沒有全域監督、彼此也不能通訊[^botcraft-readme]。
- **Example 6（DispenserFarmExample）**：以村民交易換取紅石與弓、並挖掘鵝卵石來自動合成發射器。官方 wiki 指出，除交易與挖礦外，還有大量工作讓腳本「能無限運行而不出問題」，包括**進食、睡覺、銷毀無用物品**等；其行為樹包含清理生怪磚輸出箱、以 `SortInventory` 整理物品欄、將毒馬鈴薯對著仙人掌銷毀、處理穿著冰霜行者靴子的生物、睡眠子樹、進食子樹（必要時購買食物）、以及合成發射器並存入輸出界伏盒[^wiki-dispenser]。
- **Example 7（WorldEaterExample）**：多機器人協作，將指定區域由上而下挖空。生存相關設計包括：以**金胡蘿蔔**為食物（存放於黃色界伏盒），工具依顏色分類存放於界伏盒（青色十字鎬、淺藍色斧、藍色鏟、紫色鋤、洋紅色剪刀），快損壞的工具會送回紅色混凝土方塊丟棄點；熔岩桶存放於橘色界伏盒並以暫時方塊填補岩漿；以**梯子**（棕色界伏盒）協助上下移動以降低摔落傷害；機器人死亡（例如熔岩桶誤用、生物）不會中止程式，但重生點必須符合與啟動時相同的位置條件[^wiki-worldeater]。此外，該範例指出機器人不具戰鬥能力，需要以「生物開關（mob switch）」或人工防生怪處理[^wiki-worldeater]。

### 專案模板

adepierre 另釋出 `botcraft-project-template`，這是一個可直接使用的 CMake 專案模板，用以在其上建立自己的 Botcraft 專案；模板本身採用 MIT 授權（Botcraft 本體仍為 GPL v3）[^template-readme]。截至檢索時約有 2 顆星，最後提交約為 2025-10-15（新增 MIT 授權）[^stars][^template-readme]。它不是機器人本身，但從它建立的專案可繼承 Botcraft 的全部生存能力。

### 個人 fork

GitHub 的 fork 清單顯示，Botcraft 有數十個 fork，但絕大多數僅是個人備份或跟進上游的鏡像，具有實質差異者如下[^forks]：

- **kingsoft7070/Botcraft-AI**：加入 OpenAI API 整合，使機器人可由 AI 控制；作者自述「我在這個專案加入了 OpenAI API，所以它可以被 AI 控制」[^botcraft-ai]。約 0 顆星。
- **SuperMalinge/BotcraftGTNH**：推測為相容於 GTNH（GregTech: New Horizons）模組包的分支，版本範圍調整為 1.12.2 至 1.21.1，約 0 顆星[^gtnh]。
- **OEOTYAN/Botcraft-xmake**：將建置系統由 CMake 改為 xmake，功能不變，約 0 顆星[^forks]。
- **coldelectrons/BotcraftExperimental**：實驗性質分支，約 0 顆星[^forks]。
- **FlySkyPie/Botcraft**：加入 Meson 建置系統（作者標示為實驗性、stage 1/4），目前僅完成 protocolCraft 的建置；約 0 顆星[^flyskypie]。
- **Lemiort/Protocolcraft**：Botcraft 早期版本的快照（支援至 1.17.1，使用較舊的 Mojang 帳號認證），約 0 顆星[^protocolcraft-fork]。
- **Gjum/Botcraft** 與 **RegularRabbit05/Botcraft**：跟進上游的活躍鏡像，各約 1 顆星[^forks]。

這些 fork 均繼承 Botcraft 的生存能力，但除建置系統與 AI 整合外，並未提出新的生存策略。

### 第三方依賴與相關專案

GitHub 的「Used by / dependents」頁面顯示 Botcraft **沒有任何 dependents**（0 個儲存庫、0 個套件），訊息為「我們尚未找到此儲存庫的任何 dependents」[^dependents]。官方 wiki 的 Clients 頁面也未列出任何第三方機器人專案；僅提及在執行 World Eater 範例時可搭配的伺服器端效能模組（如 Lithium、Staaaaaaaaaaaack）以承載更高玩家數，以及 botcraft-project-template 與內部協定子函式庫 ProtocolCraft[^clients]。

換言之，就公開可查的範圍而言，**不存在以 Botcraft 為基礎、獨立於 Botcraft 儲存庫之外的第三方機器人專案**。Botcraft 生態系即為：本體 + 官方範例 + 專案模板 + 個人 fork。

## 其他能在生存模式存活的 FOSS 機器人（非 Botcraft）

由於 Botcraft 生態系的第三方實作極少，以下整理同樣為 FOSS、且確實在生存模式中運作的其他機器人，作為對照與替代方案。此類專案語言與架構與 Botcraft 不同（多為 Java 或 JavaScript），mix 使用時需注意。

| 專案 | 授權 | 約略星數 | 生存模式能力 | 定位 |
|---|---|---|---|---|
| AltoClef（`gaucho-matrero/altoclef`，已封存） | MIT | 728 | **是**，2021-05-24 成為首個全自動破關 Minecraft 的機器人 | 客戶端 Fabric 模組，以 Baritone 為基礎 |
| AltoClef（`MiranCZ/altoclef`，活躍分支） | MIT | 100 | **是**，重寫並最佳化破關任務，支援 Fabric 1.16.5–1.21.1 | 客戶端 Fabric 模組 |
| Mindcraft（`mindcraft-bots/mindcraft`） | MIT | 約 5.9k | **是**，LLM 驅動的自主生存代理人 | Node.js + Mineflayer |
| Baritone（`cabaletta/baritone`） | LGPL-3.0 | 約 9.3k | 部分，僅為尋路引擎／工具，本身不具自主決策 | 函式庫／模組 |
| Mineflayer（`PrismarineJS/mineflayer`） | MIT | 約 7.6k | 部分，僅為框架，需自行實作決策 | JavaScript 函式庫 |
| Voyager（`MineDojo/Voyager`） | MIT | 約 7.3k | 否，官方要求創造模式 + 和平難度 | 研究型 LLM 代理人 |

（星數為 2026-10-14 前後檢索所得，屬易變資訊[^stars]。）

### AltoClef：目前最被驗證的生存自主性

AltoClef 自述為「玩家側機器人，嘗試自行破關 Minecraft」，並「以 Baritone 為動力」，於 2021-05-24 成為第一個全自動破關 Minecraft 的機器人[^altoclef-readme]。原始儲存庫（gaucho-matrero）已於 2024-06-28 封存；活躍分支由 MiranCZ 維護，重點在修復錯誤並最佳化 `BeatMinecraftTask`，透過 ReplayMod preprocessor 同時支援約 1.16.5 至 1.21.1 的 Fabric 版本[^altoclef-readme][^stars]。其 FAQ 明確指出該分支移除 `terminator` 與攻擊玩家的能力，理由是「我不認為這個機器人應該專注於伺服器用途」，且目前不支援與其他模組併用[^altoclef-readme]。

### Mindcraft：LLM 驅動的生存代理人

Mindcraft 由 LLM 與 Mineflayer 驅動，支援多種模型供應商（OpenAI、Google、Anthropic、xAI、DeepSeek、Ollama、Mistral、OpenRouter 等），並可透過任務檔執行「例如收集 4 根橡木原木」的目標導向任務[^mindcraft-readme]。它需要 Minecraft Java 版（最高 1.21.11）、Node.js 與至少一組 API 金鑰[^mindcraft-readme]。

其生存相關機制值得注意：技能層（`src/agent/library/skills.js`）提供進食（`consume`）、烹飪（`smeltItem`）、戰鬥（`attackNearest`、`attackEntity`）、自衛（`defendSelf`，會自動裝備最佳武器並在戰鬥期間暫停其他模式）、逃跑（`avoidEnemies`、`moveAwayFromEntity`）、採集方塊（`collectBlock`，會自動選用工具並處理礦物與深板岩變體）、前往地表（`goToSurface`）與床（`goToBed`）等[^mindcraft-skills]。其模式（modes）預設開啟三層敵對生物處理：`cowardice`（16 格內逃跑）、`self_defense`（8 格內反擊）與 `hunting`（閒置時獵殺被動動物）[^mindcraft-skills]。

然而，**Mindcraft 並不自動管理飢餓**：雖有 `consume` 技能，但沒有會檢查飽食度並自動進食的背景模式；`self_preservation` 模式僅檢查生命值過低與溺水、著火、岩漿，並不含食物或飽和度[^mindcraft-skills]。此外官方以醒目警告說明：在啟用 coding 功能時不要連線到公開伺服器，因為專案允許 LLM 在電腦上撰寫／執行程式碼，雖有沙盒隔離但仍可能受到注入攻擊，預設關閉（需將 `allow_insecure_coding` 設為 `true`）[^mindcraft-readme]。

### Voyager 與 Baritone 的定位澄清

Voyager 雖有約 7.3k 星，但官方設定即要求將遊戲模式設為**創造模式**、難度設為**和平**，屬開放式探索與技能學習的研究框架，並非生存模式代理人[^stars]。Baritone 則是尋路引擎，本身不具自主決策，是 AltoClef 等專案的底層依賴[^stars][^altoclef-readme]。這兩者常被誤認為「生存機器人」，實際上不符合本報告的生存模式定義。

## 討論與批判性反思

### 為何 Botcraft 缺乏獨立第三方實作

綜合上述證據，Botcraft 的 dependents 為 0、wiki Clients 頁面未列第三方機器人，且其餘公開搜尋僅能找到個人 fork[^dependents][^clients][^forks]。可能的原因包括：

1. **授權因素**：Botcraft 採用 GPL v3，作者並在 discussion #51 說明選擇此授權的理由[^botcraft-readme]。相較之下，Mineflayer（MIT）、Baritone（LGPL-3.0）與 AltoClef（MIT）對衍生專案的擴散更友善，這與 JavaScript／Java 生態系機器人眾多的現象一致。
2. **語言與門檻**：Botcraft 為 C++ 專案，需要 C++17 編譯器、git 與 CMake（相依套件由 CMake 自動取得），相對於 Node.js 或 Java 的入門門檻更高[^botcraft-readme]。
3. **專案定位**：作者自述這更接近「公開的私有儲存庫」而非社群專案，更新節奏由個人掌控[^botcraft-readme]。這使得外部貢獻與衍生專案的動機降低。

因此，「以 Botcraft 為基礎、且能在生存模式存活的 FOSS 機器人」在公開網路上，實際上就以**官方範例 4、6、7 為代表**，而非獨立的第三方專案。

### 來源可信度與時效性

- Botcraft 的功能與生存敘述主要來自其官方 README 與 wiki，屬第一手且由作者維護，可信度高，但帶有專案自我行銷的傾向（例如以 GIF 強調生存能力）[^botcraft-readme][^wiki-dispenser][^wiki-worldeater]。
- GitHub 星數、最後提交日期與 dependents 數量為檢索當下的快照，會隨時間變動；本報告一切數字均標註為「約略」或「截至檢索時」[^stars][^dependents]。
- 部分 fork（如 BotcraftGTNH）的用途是依名稱與差異推斷，未見作者完整說明，屬**推論而非確證**[^gtnh]。
- Mindcraft 的生存能力來自其原始碼檔案層級的檢視，可信度較高；但其「自動進食」缺席的結論係根據 `self_preservation` 模式不檢查飢餓值推得，屬對程式碼的合理判讀[^mindcraft-skills]。
- 低星數專案（多數 fork 為 0–1 星）在一般情況下參考價值有限；本報告納入它們，是因為研究主題（Botcraft 生態系）本身即為小眾，公開實作極少[^forks]。

## 結論與建議

1. **以 Botcraft 為基礎、能在生存模式運作的實作確實存在，但皆為官方範例**。若目標是「以 Botcraft 打造能在生存模式存活的機器人」，最直接的起點是 Example 4（MapCreator）、Example 6（DispenserFarm）與 Example 7（WorldEater）；新專案則可使用 `botcraft-project-template`。三者皆為 GPL v3 授權的 FOSS[^botcraft-readme][^template-readme][^wiki-dispenser][^wiki-worldeater]。
2. **若需要「現成可破關／高度自主」的生存機器人**，AltoClef（MiranCZ 分支）是最具體的選擇，因其已被驗證能全自動破關 Minecraft，且為 MIT 授權[^altoclef-readme]。
3. **若需要 LLM 驅動、跨模型供應商的自主生存代理人**，Mindcraft 最為活躍與完整，但須留意其飢餓管理需自行補強，且啟用 coding 功能時有安全風險[^mindcraft-readme][^mindcraft-skills]。
4. **若僅需要尋路與自動挖礦等基礎能力**，Baritone 為業界標準，但它不是自主代理人[^stars]。

## 參考文獻

adepierre. (n.d.). *Botcraft* [Computer software]. GitHub. Retrieved 2026-10-14, from https://github.com/adepierre/Botcraft

[^botcraft-repo]: adepierre. (n.d.). *Botcraft* [Computer software]. GitHub. Retrieved 2026-10-14, from https://github.com/adepierre/Botcraft

[^botcraft-readme]: adepierre. (n.d.). *README.md — Botcraft* [Computer software]. GitHub. Retrieved 2026-10-14, from https://raw.githubusercontent.com/adepierre/Botcraft/master/README.md

[^clients]: adepierre. (n.d.). *Clients*. In *Botcraft Wiki*. GitHub. Retrieved 2026-10-14, from https://github.com/adepierre/Botcraft/wiki/Clients

[^wiki-dispenser]: adepierre. (n.d.). *Dispensers example*. In *Botcraft Wiki*. GitHub. Retrieved 2026-10-14, from https://github.com/adepierre/Botcraft/wiki/Dispensers-example

[^wiki-worldeater]: adepierre. (n.d.). *World Eater example*. In *Botcraft Wiki*. GitHub. Retrieved 2026-10-14, from https://github.com/adepierre/Botcraft/wiki/World-Eater-example

[^dependents]: GitHub. (n.d.). *Dependents of adepierre/Botcraft*. Retrieved 2026-10-14, from https://github.com/adepierre/Botcraft/network/dependents

[^forks]: GitHub. (n.d.). *Forks of adepierre/Botcraft*. Retrieved 2026-10-14, from https://github.com/adepierre/Botcraft/forks

[^template-readme]: adepierre. (n.d.). *botcraft-project-template* [Computer software]. GitHub. Retrieved 2026-10-14, from https://github.com/adepierre/botcraft-project-template

[^botcraft-ai]: kingsoft7070. (n.d.). *Botcraft-AI* [Computer software]. GitHub. Retrieved 2026-10-14, from https://github.com/kingsoft7070/Botcraft-AI

[^gtnh]: SuperMalinge. (n.d.). *BotcraftGTNH* [Computer software]. GitHub. Retrieved 2026-10-14, from https://github.com/SuperMalinge/BotcraftGTNH

[^flyskypie]: FlySkyPie. (n.d.). *Botcraft* [Computer software]. GitHub. Retrieved 2026-10-14, from https://github.com/FlySkyPie/Botcraft

[^protocolcraft-fork]: Lemiort. (n.d.). *Protocolcraft* [Computer software]. GitHub. Retrieved 2026-10-14, from https://github.com/Lemiort/Protocolcraft

[^stars]: GitHub. (n.d.). *Repository star counts and latest commits for Botcraft, botcraft-project-template, altoclef, baritone, mineflayer, mindcraft, and Voyager*. Retrieved 2026-10-14, from https://github.com

[^altoclef-readme]: MiranCZ. (n.d.). *README.md — altoclef* [Computer software]. GitHub. Retrieved 2026-10-14, from https://raw.githubusercontent.com/MiranCZ/altoclef/main/README.md

[^mindcraft-readme]: mindcraft-bots. (n.d.). *README.md — mindcraft* [Computer software]. GitHub. Retrieved 2026-10-14, from https://raw.githubusercontent.com/mindcraft-bots/mindcraft/main/README.md

[^mindcraft-skills]: mindcraft-bots. (n.d.). *skills.js and modes.js* [Computer software]. GitHub. Retrieved 2026-10-14, from https://github.com/mindcraft-bots/mindcraft/tree/main/src/agent
