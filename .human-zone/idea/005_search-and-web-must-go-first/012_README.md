# 網際網路訪問能力是開源陣營使用 LLM 的第一要件

2025 年 10 月左右，我開始以 [Local Deep Research](https://github.com/LearningCircuit/local-deep-research#local-deep-research) 和 [Cline](https://github.com/cline/cline) 搭配 `openai/gpt-oss-20b` 的形式使用開始使用 LLM，不過尚未真的投入太多的時間在使用它們，一來是 Local Deep Research 的表現時好時壞，而我又對使用基於 [ReAct](../002_project-nymphs/006_ReAct.md) 的工具有所顧忌。當時為了運行 Local Deep Research 便在 Homelab 架設 SearXNG 和 YaCy 的服務。

2026 年 7 月左右，在直覺與運氣的作用之下選擇並開始使用 [Crush](https://github.com/charmbracelet/crush)，經過兩個月的使用後，因為不支援 subagent 而調查並試用其他的同類工具 (CLI Coding Agent)，卻發現沒有一個用得比 Crush 趁手，原因有不少，這裡只挑兩個重點出來講：

- Crush 執行指令後會顯示頭幾行回傳的資訊。
- Crush 內部實做了呼叫 DuckDuckGo 的功能，因此 Crush 無須額外安裝 MCP 或插件本身就自帶訪問與搜尋網際網路的能力。

在尚未知曉 Crush 能力的前提下，我設定了兩個 MCP 來讓它獲得訪問網際網路的能力，因為當 Agent 可以訪問網路，廣義來講就是一種 RAG 架構，其中一個用來 Fetch，另外一個透過我的 SearXNG 實例進行搜尋。

而 Crush 的第一個功能讓我能在第一時間知道，呼叫 SearXNG 搜尋之後，是不是得到了空結果？是不是只回傳了 YaCy 的結果？這兩個資訊代表 SearXNG 被外部的搜尋引擎阻擋訪問了，在 Local Deep Research 我可以從 LLM 可觀測的基礎設施調閱日誌來知道這件事，但是資訊不夠即時、充足跟易於理解。

於是在我配置的搜尋引擎失效後，我觀察到 Agent 使用 Crush 內建的搜尋能力，而且是運行在獨立 Session (Subagent) 中進行搜尋，因此搜尋過程累積的 Context 不會直接累加到主 Session 的 Context 中。

---

網路搜尋與訪問能力本質上就是廣義的 RAG 範式，然而對於開源陣營而言這裡有兩個難題：

目前方便使用的搜尋引擎依然在寡頭手裡，只要它們開啟機器人過濾，來自外部的請求就會像我遇到的這樣，無法使用，而自建搜尋引擎需要耗費資源與心思去建立索引以及調校參數，實務上無法達到寡頭搜尋引擎的水準。

二來是 LLM 氾濫的時代，造成人心惶惶，不只是寡頭企業有動機去阻止機器人訪問，開放陣營的網站也紛紛掛起 [Anubis](https://github.com/techaroHQ/anubis) 來阻止機械人爬蟲。

兩個難題都會反過來削弱開放陣營的 LLM 系統，當開放陣營的 LLM 系統不敵商業陣營，開放陣營的力量就會在 LLM 時代中可預期的被削弱。

## 潛在方案

- 利用 LLM 來自動化調校 YaCy。
- 利用 LLM 來自動觸發 YaCy 檢索。
- 利用本專案透過 Crush + DuckDuckGo 產生的報告連結，ETL 到 YaCy 觸發檢索。
- 探索開源搜尋引擎的最新進展（YaCy 替代方案）。
- 使用 LLM 接續 YaCy Grid 自 2022 年擱置的微服務尚未完成的工作。
- 探索開源專案使用 LLM 強化或搭配搜尋引擎的專案。

## 選擇 Crush 的直覺與運氣

個人在軟體工程的道路上基本上使用一個準則：

> 業餘探索、工作利用

也就是工作時使用無聊的技術決策，確保產品符合產業常態、穩定、可靠；
業餘的時間則探索一些較為冷門的技術或工具。

於是在選擇 CLI Coding Agent 這個問題上我給的第一個條件是：使用編譯語言實做，性能第一，
一部份的原因是出於自己轉職成需要重視效率的嵌入式系統工程師。

第二個條件是剛剛提到的：探索冷門選項，於是當時嘗試了使用 Rust 實做的 [Goose](https://github.com/aaif-goose/goose) 以及用 Go 實做的 Crush，Goose 因為在設定 OpenAI-Compatible API 有一些摩擦，Crush 就被選上並持續使用了。
