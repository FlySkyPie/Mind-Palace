# Nymphs 專案

Nymphs 專案本身是一個純粹的願景：

> 基於行為樹 (Behavior Tree) 的 LLM 智能體

它建立在[LangGraph 災難](./007_LangGraph-disaster.md)和[LLM 供應商的代理問題](./008_llm-provider-proxy-problem.md)...等認知之上。

基於這個願景，產生了其他包括但不限於的想像：

- 一個輕量級、不仰賴特定遊戲引擎的行為樹解決方案。
  - 行為樹 Runtime
  - 視覺化行為樹編輯器
  - 行為樹日誌與視覺化回放
- 行為樹商店/交流平台

## 潛在方案

- 採用 [BehaviorTree.CPP](https://github.com/behaviortree/behaviortree.cpp)
- Fork [Groot](https://github.com/BehaviorTree/Groot) 並使用 LLM 維護或改進。
  - 註：Groot 僅支援 BehaviorTree.CPP 3.* 版本，支援 4.* 版本的 Groot 並沒有開源。
- 學習利用與探索、蒙地卡羅樹...等古典 AI 的範式，並使用行為樹實做這些古典算法，使複數古典演算法能夠在行為樹的框架下複合使用。

## 具體作為

- 學習行為樹領域模型與範式
  - https://flyskypie.github.io/behaviortrees-docs/
- 建構基於 Typescript 的 POC
  - https://github.com/FlySkyPie/tiddlyrag-poc/tree/poc/type-c
  - 使用行為樹透過 Gtea HTTP API 遍歷檔案。
  - 使用 ECS (Entity component system) 取代黑板模式以避免其缺陷。
  - 深度優先與廣度優先遍歷遺鍵切換（改一個行為樹動作節點）。

## 命名

Nymphs 是古希臘民間傳說中的一位次要的女性自然神祇。與其他希臘女神不同，Nymphs 通常被認為是自然的化身；她們通常與特定的地點、地形或樹木相關聯，並且通常被描繪成少女的形象。
