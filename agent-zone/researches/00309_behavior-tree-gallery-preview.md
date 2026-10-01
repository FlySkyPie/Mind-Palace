---
name: behavior-tree-gallery-preview
created_at: 2026-09-27
author: Crush (via agentic research)
---

# Behavior Tree Gallery / Inventory 預覽方式研究

## 問題陳述

要為行為樹 (behavior tree) 建立 inventory/gallery 型 GUI（類似素材瀏覽器或腳本瀏覽器），應以什麼方式呈現預覽？換句話說，如何 preview/summarize 一棵行為樹，並將它們以 grid 或 table 形式列出？

## 現有編輯器的概覽方式

現有的行為樹編輯器**全部使用文字型列表**，沒有任何 visual gallery 或 card grid[^groot2][^opsive][^behavior3][^unreal-bt]：

| 編輯器 | 概覽機制 | 備註 |
|--------|----------|------|
| **Behavior3 Editor** (Adobe/Starbound fork) | 側欄文字列表，列出所有開啟的行為樹名稱 | 可拖曳巢狀排序，但無 thumbnail[^behavior3] |
| **Groot2** (BehaviorTree.CPP 官方 IDE) | 一次開啟一個 BT 檔案；PRO 版有跨樹節點搜尋 | 無 gallery/list view[^groot2] |
| **Opsive Behavior Designer Pro** (Unity) | 下拉選單列出掛在 GameObject 上的 BT | 純文字 dropdown[^opsive] |
| **TheKiwiCoder UnityBehaviourTreeEditor** | 下拉選單 + 啟動 dialog 切換樹 | 純文字[^kiwico] |
| **Unreal Engine** | Content Browser 的 Tiles/List/Columns 檢視 | **最接近 gallery**，但 thumbnail 是通用圖示，非樹結構預覽[^unreal-content] |
| **Behaviac** (騰訊) | Project explorer 面板，純文字檔案樹 | 無視覺預覽[^behaviac] |
| **PyTrees** | 無 GUI，終端機 `unicode_tree()` / `ascii_tree()` | CLI 專用[^pytrees] |
| **AkiBT** (Unity, GraphView) | 一次開啟一棵樹，dockable editor[^akibt] |
| **BTEditor.dev** (瀏覽器) | 單棵樹編輯，無多樹管理[^btedev] |

**結論：沒有任何現有編輯器提供行為樹的濃縮視覺預覽**（mini-map、thumbnail card、節點計數 badge、文字摘要 snippet）在概覽/庫存情境中。

## 可行的 Preview / Summary 格式

以下是在學術文獻或相關領域中出現的預覽形式，按可行性分級：

### 已存在（但不在 BT 領域的 gallery 中）

| 格式 | 來源 | 說明 |
|------|------|------|
| **完整樹狀圖** | 所有編輯器 | 標準「預覽」——必須打開該樹才能看到 |
| **樹名文字** | 所有編輯器 | 最常見形式 |
| **Asset icon / thumbnail** | Unreal Engine Content Browser | 通用型 asset type icon，非樹結構視覺化 |
| **ASCII/Unicode 樹** | PyTrees | 純文字樹狀圖，在 CLI/git diff 情境有用 |
| **自然語言摘要** | IEEE 2025 論文 | LLM 生成 BT 的文字描述[^llm-summarize] |
| **XML/JSON 原始碼** | Groot2 | 編輯時即時顯示的原始碼面板 |
| **Side-by-side diff** | NCAlt 2022 | 多棵 BT 的差異對比[^ncal] |

### 不存在但可行（新的設計空間）

| 格式 | 描述 | 實作難度 |
|------|------|---------|
| **Mini-map thumbnail** | 將 BT 節點結構縮小為 icon 尺寸的圖形 | 中 — 需要 graph layout algorithm + thumbnail rendering |
| **節點計數 badge** | 在樹名旁邊顯示 `(12 nodes, 3 subtrees)` | 低 — 僅需樹遍歷 |
| **樹拓撲 icon** | 以抽象方式顯示樹的「形狀」（寬 vs 深、sequence vs selector 比例） | 中 — 特徵提取 + icon generation |
| **文字摘要 snippet** | 如 LLM 生成：「巡邏路徑→偵測→攻擊→返回」 | 中 — 需要 LLM 或 heuristic（root-to-leaf 路徑提取） |
| **彩色 status card** | 類似 CI badge，顯示樹的健康狀態（節點數、子樹數、error count、最後修改時間） | 低 |
| **縮放過的 SVG/PNG screenshot** | 預先渲染成小圖，類似 Figma 的 page preview | 高 — 需要 headless rendering engine |

## 遊戲 AI 編輯器 / Visual Scripting 工具的 Gallery 實例

這些領域（比 BT 更成熟的圖形化編輯）也**沒有**提供 visual gallery：

- **Unity Visual Scripting (Bolt)**：一次只開一個 graph，使用者曾要求 tab[^unity-vs]
- **Unreal Engine Content Browser**：BT asset 顯示為 tile 但 thumbnail 是通用的[^unreal-content]
- **一般模式**：sidebar list、dropdown、file system browser，皆無視覺卡片

## 關鍵學術文獻

| 論文 | 年份 | 關鍵貢獻 |
|------|------|---------|
| Semantic Embedding of Behavior Trees via LLM Summarization[^llm-summarize] | 2025 | LLM 生成文字摘要 + 語義嵌入用於相似性搜索 |
| Structural Segmentation and LLM Make BT Explanation[^llm-explain] | 2024 | 層級結構分割 + LLM 解釋生成 |
| NCAlt: Alternatives and Difference Visualizations for BTs[^ncal] | 2022 | 多棵 BT 的 side-by-side diff/merge 視覺化 |
| AIPaint: Sketch-Based BT Authoring[^aipaint] | 2011 | 在遊戲世界覆蓋草圖式 BT 創建 |
| Behavior Tree Visualization in Godot Engine (Treehave)[^treehave] | 2020 | Godot 引擎內的 BT 視覺編輯器 |

## 設計建議

若要建立 inventory/gallery GUI，建議採用**漸進式層級**：

```
Level 1: 文字摘要（節點數、深度、子樹數、標籤）
    → 低實作成本，立即實用

Level 2: 文字摘要 + 樹拓撲示意圖（簡化的 tree shape icon）
    → 中等成本，顯著提升識別性

Level 3: 文字摘要 + 樹拓撲示意圖 + LLM 自然語言摘要 snippet
    → 高成本，最 rich 的預覽體驗
```

Grid layout 適合卡片式呈現（每張卡片 = 一棵 BT 的 preview + 中繼資料），table layout 適合大量 BT 的批次管理（排序、過濾、批次操作）。

## 來源

[^groot2]: BehaviorTree.CPP. (n.d.). Groot2. Retrieved 2026-09-27, from https://www.behaviortree.dev/groot/
[^opsive]: Opsive. (n.d.). Behavior Designer Pro Overview. Retrieved 2026-09-27, from https://opsive.com/support/documentation/behavior-designer-pro/overview/
[^behavior3]: DeepWiki. (n.d.). Behavior3 Editor — Projects and Trees. Retrieved 2026-09-27, from https://deepwiki.com/behavior3/behavior3editor/3.1-projects-and-trees
[^unreal-bt]: Epic Games. (n.d.). Behavior Trees in Unreal Engine. Retrieved 2026-09-27, from https://dev.epicgames.com/documentation/unreal-engine/behavior-trees-in-unreal-engine?lang=en-US
[^unreal-content]: Epic Games. (n.d.). Content Browser in Unreal Engine. Retrieved 2026-09-27, from https://dev.epicgames.com/documentation/unreal-engine/content-browser-in-unreal-engine?lang=en-US
[^kiwico]: TheKiwiCoder. (n.d.). UnityBehaviourTreeEditor. Retrieved 2026-09-27, from https://github.com/thekiwicoder0/UnityBehaviourTreeEditor
[^behaviac]: Tencent. (n.d.). Behaviac — Behavior Tree AI Engine. Retrieved 2026-09-27, from https://github.com/Tencent/behaviac/
[^pytrees]: PyTrees Project. (n.d.). Trees — PyTrees Documentation. Retrieved 2026-09-27, from https://py-trees.readthedocs.io/en/devel/trees.html
[^akibt]: AkiKurisu. (n.d.). AkiBT. Retrieved 2026-09-27, from https://github.com/AkiKurisu/AkiBT
[^btedev]: BTEditor.dev. (n.d.). Behavior Tree Editor. Retrieved 2026-09-27, from https://bteditor.dev/
[^llm-summarize]: IEEE. (2025). Semantic Embedding of Behavior Trees via Large Language Model Summarization. Retrieved 2026-09-27, from https://ieeexplore.ieee.org/abstract/document/11669644
[^llm-explain]: IEEE. (2024). Structural Segmentation and Large Language Model Make Behavior Tree Explanation. Retrieved 2026-09-27, from https://ieeexplore.ieee.org/document/10505332
[^ncal]: ACM. (2022). NCAlt: Alternatives and Difference Visualizations for Behavior Trees. Retrieved 2026-09-27, from https://dl.acm.org/doi/10.1145/3549508
[^aipaint]: AAAI. (2011). AIPaint: A Sketch-Based Behavior Tree Authoring Tool. Retrieved 2026-09-27, from https://ojs.aaai.org/index.php/AIIDE/article/view/12423
[^treehave]: Springer. (2020). Behavior Tree Visualization in Godot Engine. Retrieved 2026-09-27, from https://link.springer.com/chapter/10.1007/978-981-95-6746-1_5
[^unity-vs]: Unity Technologies. (n.d.). Unity Visual Scripting. Retrieved 2026-09-27, from https://docs.unity3d.com/Packages/com.unity.visualscripting@1.8/manual/vs-graph-types.html