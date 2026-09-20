# 行為樹樣本庫：資料儲存結構建議報告

## 摘要

本報告探討建立「行為樹樣本庫」(Behavior Tree Sample Library) 時應採用的資料儲存結構。經調查主流行為樹框架（BehaviorTree.CPP、ROS2/Nav2、py_trees、Unreal Engine、Unity Behavior、behavior3 系列、behaviac 等）的序列化格式、生態系成熟度與工具鏈支援後，提出以 **XML（BT.CPP 方言）為主要收錄格式、JSON（behavior3 格式）為輔、YAML 描述中繼資料** 的分層儲存架構，作為兼顧標準化、通用性與生態系廣度的最佳方案。

## 1. 背景與動機

行為樹（Behavior Tree, BT）已廣泛應用於遊戲 AI、機器人控制、模擬系統等領域，但 **不存在跨框架、跨引擎的統一標準格式**。市面上也缺乏專門的行為樹樣本資料庫——現有的資源散見於各框架的範例目錄、教學文件與研究論文中。建立一個統一樣本庫的首要決策，便是選擇合適的資料儲存結構。

## 2. 主要行為樹框架的序列化格式一覽

| 框架 / 生態系 | 儲存格式 | 特點 |
|---|---|---|
| **BehaviorTree.CPP**（機器人/ROS2） | **XML**（自訂 DSL） | 事實上的業界標準；格式版本 v3 → v4；支援 `<SubTree>`、`<include>`、port remapping；搭配 Groot/Groot2 視覺化編輯器 |
| **ROS2 / Nav2** | **XML**（BT.CPP 方言） | 直接使用 BT.CPP XML；生產環境驗證；大量可參考的實際行為樹 |
| **py_trees**（Python/ROS2） | **程式碼為主** + XML 解析器（實驗性） | 新 XML 解析器與 BT.CPP 格式相容；另有 DOT/ASCII/Unicode 視覺化輸出 |
| **Unreal Engine** | **二進位**（`.uasset`，專有格式） | 透過 UPROPERTY 序列化，不可攜帶、不可人工讀寫 |
| **Unity Behavior** | **Unity YAML**（ScriptableObject） | 引擎綁定；Unity 序列化引擎將物件圖轉為 YAML 文字資產 |
| **behavior3 系列**（behavior3js/py/editor） | **JSON**（開放格式） | 明確設計為跨語言、跨工具可攜帶；保留編輯器排版資訊（`display.x/y`） |
| **behaviac**（騰訊） | XML 匯出 | 編輯器專有格式，但可匯出 XML |
| **Godot**（beehave, LimboAI） | GDScript / Godot 資源 | 引擎綁定 |
| **Game AI Pro** 系列 | 程式碼（Lua/C++） | 無共用序列化標準 |

## 3. 格式評估

### 3.1 XML（BT.CPP 方言）

**優勢：**
- 最大生態系：ROS2/Nav2、Groot/Groot2、py_trees、behaviac 皆採用或相容
- 工具鏈成熟：有 Groot/Groot2 視覺化編輯器、BT.CPP 的 `writeTreeXSD()` 可生成 XSD Schema（v4.5+）[^btcpp-xsd]
- 可讀性佳：標籤結構清晰，適合 Git diff/版本控制
- 支援模組化：`<SubTree>` 與 `<include>` 實現樹的組合與重用
- 大量現成樣本：Nav2 出廠行為樹、BT.CPP 測試檔案、ROS2 教學範例

**劣勢：**
- 語法較冗長
- 節點詞彙表框架綁定（`<Sequence>` 可攜帶，但 `<MoveTo>` 是特定框架自訂節點）
- 格式版本差異（BT.CPP v3 ↔ v4 不相容）

### 3.2 JSON（behavior3 格式）

**優勢：**
- 明確設計為跨語言交換格式[^behavior3-json]
- Web/JS 生態系原生支援，易於解析
- 相對簡潔，保留編輯器佈局資訊
- 適合當作前端工具／網頁展示的中間格式

**劣勢：**
- 生態系遠小於 XML（無 Groot 等級的專用編輯器）
- 無 `<include>`/subtree 檔案重用慣例
- 手寫較困難（巢狀括號）

### 3.3 YAML

**優勢：**
- 最適合人類手寫與審閱
- 最適合描述中繼資料（領域、標籤、授權、框架版本）
- 在 BTGenBot 研究專案中被用作外層容器[^btgenbot-yaml]

**劣勢：**
- 沒有任一個主要 BT 框架原生使用 YAML 作為樹定義格式
- 縮排敏感，容易出錯

### 3.4 二進位格式（Unreal/其他引擎專有）

完全不適合開放樣本庫——不可 diff、不可手寫、不可攜帶。僅適合以引擎專案形式收錄。

### 3.5 程式碼（Lua/C++/Python）

適合收錄「以程式碼建構行為樹」的範例模式（如 py_trees idioms、Game AI Pro 範例），但不適合作為主要儲存格式——無法被非程式人員檢視或編輯。

## 4. 建議儲存架構

### 分層混合方案

每一筆行為樹樣本應包含三個層次：

```
behavior-tree-samples/
├── index.yaml                          # 樣本庫索引
├── samples/
│   ├── <sample-id>/
│   │   ├── manifest.yaml               # 中繼資料（必備）
│   │   ├── tree.xml                    # BT.CPP 格式（主要）
│   │   ├── tree.json                   # behavior3 格式（選用，轉換版本）
│   │   ├── tree.bt                     # 自訂簡潔格式（選用，見 §5）
│   │   ├── blackboard.json             # 黑板初始值（選用）
│   │   ├── assets/                     # 相依資源（選用）
│   │   │   ├── config.yaml
│   │   │   └── nodes_manifest.xml
│   │   └── README.md                   # 說明文件
│   └── ...
└── schemas/
    ├── manifest.schema.json            # manifest.yaml 的 JSON Schema
    ├── tree-btcpp.xsd                  # BT.CPP XML 的 XSD
    └── tree-behavior3.schema.json      # behavior3 JSON 的 JSON Schema
```

### manifest.yaml 欄位設計

```yaml
id: navigate-to-pose-w-replanning
title: "含重規劃與復原的導航行為樹"
description: "ROS2 Nav2 預設導航行為樹，包含路徑規劃、路徑跟隨、碰撞復原與重規劃"
format:
  primary: btcpp-xml               # 主要格式：btcpp-xml | behavior3-json | code
  version: "4"                     # 格式版本
  compatible_parsers:              # 相容解析器
    - "BehaviorTree.CPP >= 4.0"
    - "py_trees >= 2.5 (XML parser)"
    - "Groot2"
domain:                            # 應用領域
  - robotics
  - navigation
tags:
  - sequence
  - fallback
  - recovery
  - replanning
node_vocabulary:                   # 使用的節點集（跨框架對照用）
  builtin:
    - Sequence
    - Fallback
    - ReactiveFallback
    - PipelineSequence
    - RecoveryNode
    - RateController
    - RoundRobin
  custom:
    - ComputePathToPose
    - FollowPath
    - ClearEntireCostmap
    - Spin
    - BackUp
source:
  framework: ROS2 Nav2
  repository: "https://github.com/ros-navigation/navigation2"
  license: Apache-2.0
references:                        # 學術或技術參考
  - Colledanchise & Ögren (2018). Behavior Trees in Robotics and AI.
```

## 5. 補充：BT.CPP XML 格式範例

```xml
<root BTCPP_format="4" main_tree_to_execute="NavigateToPoseWReplanningAndRecovery">
    <BehaviorTree ID="NavigateToPoseWReplanningAndRecovery">
        <RecoveryNode>
            <Sequence>
                <RecoveryNode>
                    <RoundRobin name="ComputePathToPose">
                        <ComputePathToPose goal="{goal}"/>
                        <ClearEntireCostmap name="ClearLocalCostmap" service_name="local_costmap"/>
                    </RoundRobin>
                    <FollowPath path="{path}" controller_id="FollowPath"/>
                </RecoveryNode>
                <ClearEntireCostmap name="ClearGlobalCostmap" service_name="global_costmap"/>
            </Sequence>
        </RecoveryNode>
    </BehaviorTree>
</root>
```

## 6. 推行建議

| 優先級 | 行動 | 說明 |
|---|---|---|
| **P0** | 定義 manifest.yaml 的 JSON Schema | 確保所有樣本中繼資料格式一致 |
| **P0** | 以 XML（BT.CPP v4）收錄第一批樣本 | 從 Nav2、BT.CPP examples、py_trees idioms 匯入 |
| **P1** | 建立 XML → JSON（behavior3）轉換器 | 增加樣本庫的跨生態系價值 |
| **P1** | 收錄程式碼式樣本（py_trees idioms, Game AI Pro） | 保留原有程式碼，另附 XML/JSON 表示 |
| **P2** | 設計自訂簡潔格式 `.bt` | 減少手寫負擔，參考 BT.CPP v3 → v4 轉換腳本的語法 |
| **P3** | 研究跨框架節點詞彙對照表 | 讓單一樣本能對應到多個框架 |

## 7. 結論

最務實且生態系最廣的方案是 **以 BT.CPP XML 為主要儲存格式、YAML 為中繼資料格式**。XML 擁有最成熟的工具鏈（Groot/Groot2）、最大的既有樣本庫（ROS2/Nav2），且任何行為樹概念都能以 XML 表示。JSON（behavior3 格式）可作為次要／轉換格式，以服務 Web/JS 生態系使用者。**不建議自行發明新的 BT 序列化格式**，因為樣本庫的價值在於與既有生態系的相容性，而非創造另一個孤立標準。

## 參考資料

[^btcpp-xsd]: BehaviorTree.CPP. (n.d.). XML Format - BehaviorTree.CPP documentation. Retrieved 2026-09-20, from https://www.behaviortree.dev/docs/learn-the-basics/xml_format

[^btcpp-github]: BehaviorTree.CPP. (n.d.). GitHub repository. Retrieved 2026-09-20, from https://github.com/BehaviorTree/BehaviorTree.CPP

[^groot]: BehaviorTree. (n.d.). Groot - Graphical Editor for Behavior Trees. Retrieved 2026-09-20, from https://github.com/BehaviorTree/Groot

[^behavior3-json]: behavior3js. (n.d.). behavior3js GitHub repository. Retrieved 2026-09-20, from https://github.com/behavior3/behavior3js

[^nav2-trees]: ROS2 Navigation. (n.d.). Nav2 Behavior Trees. Retrieved 2026-09-20, from https://github.com/ros-navigation/navigation2/tree/main/nav2_bt_navigator/behavior_trees

[^py-trees]: py_trees. (n.d.). py_trees documentation. Retrieved 2026-09-20, from https://py-trees.readthedocs.io/en/devel/

[^ue-bt]: Epic Games. (n.d.). Behavior Trees in Unreal Engine. Retrieved 2026-09-20, from https://docs.unrealengine.com/5.0/en-US/behavior-tree-in-unreal-engine/

[^unity-behavior]: Unity Technologies. (n.d.). Unity Behavior package serialization. Retrieved 2026-09-20, from https://docs.unity3d.com/Packages/com.unity.behavior@1.0/manual/serialize-behavior.html

[^colledanchise]: Colledanchise, M., & Ögren, P. (2018). Behavior Trees in Robotics and AI: An Introduction. CRC Press. Retrieved 2026-09-20, from https://arxiv.org/abs/1709.00084

[^btgenbot-yaml]: AIRLab POLIMI. (n.d.). BTGenBot - Behavior Tree Generation for Robots. Retrieved 2026-09-20, from https://github.com/AIRLab-POLIMI/BTGenBot

[^behaviac]: Tencent. (n.d.). behaviac - Behavior tree solution for games. Retrieved 2026-09-20, from https://github.com/Tencent/behaviac

[^omg]: Object Management Group. (n.d.). OMG Specifications Catalog. Retrieved 2026-09-20, from https://www.omg.org/spec/

[^game-aipro]: Game AI Pro. (n.d.). Game AI Pro: Collected Wisdom of Game AI Professionals. Retrieved 2026-09-20, from https://www.gameaipro.com/

[^behavior-designer]: Opsive. (n.d.). Behavior Designer - Behavior Trees for Unity. Retrieved 2026-09-20, from https://opsive.com/support/documentation/behavior-designer/overview/