# 行為樹 (Behavior Tree) — 開源範例與樣本調查報告

## 引言

行為樹 (Behavior Tree，簡稱 BT) 是一種用於建模非玩家角色 (NPC) 人工智慧、機器人控制與任務規劃的分層樹狀結構，自 2000 年代中期起逐漸取代有限狀態機 (FSM) 成為遊戲 AI 與機器人領域的主流方法[^wiki]。與傳統 FSM 相比，行為樹具有模組化、可重用、易於除錯等優點[^btbook]。本報告針對實際的行為樹**定義範例**（而非行為樹函式庫或框架本身）進行調查，收錄各生態系中公開可取得的 XML、JSON 及 Python 形式之行為樹樣本。

行為樹的核心結構包含四種節點類型[^btbook]：
- **複合節點 (Composite)**：控制子節點流程，如 Sequence（依序執行）、Fallback/Selector（優先選擇）、Parallel（並行）
- **修飾節點 (Decorator)**：包裹單一子節點以改變其行為，如 Inverter、RetryUntilSuccessful、Timeout
- **條件節點 (Condition)**：檢查狀態，不回傳 Running
- **動作節點 (Action)**：執行具體操作，可回傳 Running/Success/Failure

以下按生態系與應用領域分類。

---

## 1. BehaviorTree.CPP 生態系

BehaviorTree.CPP 是機器人與遊戲領域最廣泛使用的 C++ 行為樹函式庫，其行為樹定義以 XML 格式儲存[^btcpp]。

### 1.1 官方範例與測試樹

**位置**：`BehaviorTree.CPP/examples/test_files/` 與 `BehaviorTree.CPP/tests/trees/`[^btcpp_examples]

**格式**：XML (BTCPP_format="4")

**內容**：
- `subtree_test.xml` — 展示 Sequence、SequenceWithMemory、SubTree 包含、Sleep 節點
- `Check.xml` — 單一條件檢查節點
- `log_test.xml` — 日誌測試
- `subtrees/Talk.xml` — 可被子樹引用的說話行為

### 1.2 教學範例集 (oguzaltan/Behavior-Tree-Examples)

**位置**：https://github.com/oguzaltan/Behavior-Tree-Examples[^oguz]

**格式**：XML

**介紹**：16 個自包含的行為樹範例專案，每個範例資料夾內含一個 `bt_tree.xml` 檔案，涵蓋 BehaviorTree.CPP v4+ 的各項功能。核心案例包括：

**範例 1：簡單夾取序列 (01-simple-pick-sequence)**
```xml
<BehaviorTree ID="MainTree">
    <Sequence>
        <CheckBattery/>
        <OpenGripper/>
        <ApproachObject/>
        <CloseGripper/>
    </Sequence>
</BehaviorTree>
```
此樹展示最基本的 Sequence：依序檢查電量 → 開啟夾爪 → 接近物體 → 關閉夾爪。

**範例 5：CrossDoor — 備援與重試 (05-crossdoor-fallback-retry)**
```xml
<BehaviorTree ID="MainTree">
    <Sequence>
        <Fallback>
            <Inverter>
                <IsDoorClosed/>
            </Inverter>
            <SubTree ID="DoorClosed"/>
        </Fallback>
        <PassThroughDoor/>
    </Sequence>
</BehaviorTree>

<BehaviorTree ID="DoorClosed">
    <Fallback>
        <OpenDoor/>
        <RetryUntilSuccessful num_attempts="5">
            <PickLock/>
        </RetryUntilSuccessful>
        <SmashDoor/>
    </Fallback>
</BehaviorTree>
```

此範例展示典型的**越權 (guarded fallback) 模式**：主樹先確認門是否關閉（透過 Inverter 反轉 IsDoorClosed 條件），若門關閉則進入 DoorClosed 子樹，依序嘗試開門 → 開鎖（最多 5 次）→ 破門[^oguz]。

### 1.3 其他代表性範例

其他 14 個範例涵蓋：
- ReactiveSequence（反應式序列，每 tick 重新評估子節點狀態）
- Blackboard（黑板資料交換）
- Scripting（內建腳本節點）
- SubTree 重用與 port remapping
- Parallel 並行節點
- Global Blackboard（全域資料交換）
- Threading（多執行緒行為樹）

---

## 2. 遊戲 AI 行為樹範例 (Rain AI / Unity)

**位置**：`wesleywh/GameDevRepo/tree/master/Rain AI/Behavior Trees/`[^rain]

**格式**：XML 與 Unity `.asset`

**介紹**：來自 Rain AI (Unity 資產商店的行為樹工具) 的完整 NPC 行為樹套件，包含巡邏、搜索、攻擊、躲藏等完整的遊戲 AI 行為。

### 2.1 根樹 (Root_Tree.xml)

根樹採用 Parallel 並行偵測模式，將**世界感知**與**行動決策**分離：

感知層 (並行執行)：
- 聽覺偵測 (TryToHearPlayerFootsteps)
- 近戰範圍偵測 (InMeleeDistance)
- 視覺偵測 — 區分明亮 (eyes) 與黑暗 (NightVisual) 環境

行動層 (Selector 優先順序)：
1. **看見玩家且可疑** → 等待反應時間 → 記錄玩家位置 → 設定敵對狀態 → 進入 HostileAttack 子樹（含隨機切換攻擊/找掩護）
2. **看不見玩家但敵對** → 觸發 Search 行為
3. **看不見玩家且可疑** → Search → 若沒找到則還原至 Patrol
4. **平靜狀態 (calm)** → Patrol

### 2.2 巡邏樹 (Patrol.xml)

```xml
<sequencer>
    <waypointpatrol waypointsetvariable="PatrolRoute" ...>
        <move ... facetarget="nextStop" closeenoughangle="15" />
        <move ... movetarget="nextStop" movespeed="1" closeenoughdistance=".5" />
    </waypointpatrol>
</sequencer>
```

運作邏輯：從 PatrolRoute 路徑點集依序選取目標 → 先旋轉面向目標 → 移動至目標（距離 0.5 單位內視為到達）→ 重複[^rain_patrol]。

### 2.3 搜索樹 (Search.xml)

搜索行為：移動至最後已知玩家位置 → 隨機漫步（3–8 次迭代，每次含注視、移動、定時等待）。

---

## 3. 機器人巡邏範例 (PythonRobotics)

**位置**：`MissionPlanning/BehaviorTree/robot_behavior_tree.xml`[^pyrob]

**格式**：XML

**介紹**：來自知名 PythonRobotics 專案的機器人巡邏行為樹，展示**電池管理 + 任務執行 + 路徑規劃**的複合架構：

```
Selector (Robot Main Controller)
├── Sequence (Battery Management)
│   ├── Inverter
│   │   └── CheckBattery threshold=30
│   ├── Echo "Battery level low! Charging needed"
│   └── ChargeBattery charge_rate=20
└── Sequence (Patrol Task)
    ├── Sequence (Move to Position A)
    │   ├── MoveToPosition A
    │   ├── Selector (Obstacle Handling A)
    │   │   ├── Sequence: DetectObstacle → AvoidObstacle
    │   │   └── Echo "Path clear"
    │   └── PerformTask
    ├── Sequence (Move to Position B)  [含 Timeout 障礙處理]
    ├── WhileDoElse (Conditional Move to C)
    │   ├── Condition: CheckBattery threshold=50
    │   ├── Sequence: MoveTo C → ForceSuccess(PerformTask)
    │   └── Echo "Insufficient power, skipping"
    └── Return to Charging Station
```

特殊模式：
- **WhileDoElse**：根據電池電量條件判斷是否前往 C 點
- **ForceSuccess**：確保 C 點任務即使失敗也被標記為成功
- **Timeout**：限定障礙物處理時間（2 秒）以防止無限阻塞

---

## 4. ROS 2 Navigation2 生產級行為樹

**位置**：`nav2_bt_navigator/behavior_trees/`[^nav2]

**格式**：XML (BTCPP_format="4")

**介紹**：ROS 2 Navigation2 堆疊的官方行為樹集合，是部署在真實機器人上的生產級範例。Nav2 使用行為樹來組織完整的導航任務，包含路徑規劃、路徑跟隨、錯誤復原等。

### 4.1 具有重規劃與復原的導航 (NavigateToPoseWReplanningAndRecovery)

核心結構[^nav2_w_recovery]：

```
RecoveryNode (max_retries=6) — NavigateRecovery
├── PipelineSequence — NaviateWithReplanning
│   ├── ProgressCheckerSelector
│   ├── GoalCheckerSelector
│   ├── PathHandlerSelector
│   ├── ControllerSelector
│   ├── PlannerSelector
│   ├── RateController (hz=1.0)
│   │   └── RecoveryNode — ComputePathToPose
│   │       ├── Fallback
│   │       │   ├── ReactiveSequence (CheckIfNewPathNeeded)
│   │       │   │   ├── Inverter → GlobalUpdatedGoal
│   │       │   │   ├── IsGoalNearby
│   │       │   │   ├── TruncatePathLocal
│   │       │   │   └── ValidatePath
│   │       │   └── ComputePathToPose
│   │       └── Sequence (這裡 Planner 復原)
│   │           ├── WouldAPlannerRecoveryHelp
│   │           └── ClearEntireCostmap (global)
│   └── RecoveryNode — FollowPath
│       ├── FollowPath
│       └── Sequence (Controller 復原)
│           ├── WouldAControllerRecoveryHelp
│           └── ClearEntireCostmap (local)
└── Sequence (系統級復原)
    ├── Fallback (確認復原類型)
    │   ├── WouldAControllerRecoveryHelp
    │   └── WouldAPlannerRecoveryHelp
    └── ReactiveFallback — RecoveryFallback
        ├── GoalUpdated
        └── RoundRobin — RecoveryActions
            ├── ClearingActions (清除全域+局部成本地圖)
            ├── Spin (旋轉 1.57 弧度)
            ├── Wait (等待 5 秒)
            └── BackUp (後退 0.3 公尺)
```

此樹展示的行為樹模式：
- **RecoveryNode**：重試機制，失敗後執行復原子樹
- **PipelineSequence**：類似管線的 Sequence（子節點 Running 時仍持續監控輸入條件）
- **RateController**：以 1 Hz 頻率重新規劃路徑
- **RoundRobin**：輪詢復原行動列表
- **ReactiveFallback**：若目標更新則中斷復原

### 4.2 其他 Nav2 行為樹

| 檔案名稱 | 特色 |
|---|---|
| `navigate_w_replanning_time.xml` | 1 Hz 固定頻率重規劃 |
| `navigate_w_replanning_distance.xml` | 機器人移動 1m 後重規劃 |
| `navigate_w_replanning_only_if_path_becomes_invalid.xml` | 僅在路徑失效時重規劃 |
| `navigate_to_pose_w_bounds_check.xml` | 加入路徑邊界檢查 |
| `follow_point.xml` | 追蹤動態目標點 |
| `application_example.xml` | 完整任務鏈：含電池感知的對接/解除對接任務 |

---

## 5. Python 行為樹 (py_trees)

**位置**：`py_trees/demos/`[^pytrees]

**格式**：Python 程式碼（樹以程式化方式建構）

**介紹**：py_trees 是支援 ROS 的 Python 行為樹函式庫，其 `demos/` 資料夾包含多個展示行為樹設計模式的獨立範例。

| 示範檔案 | 展示模式 |
|---|---|
| `selector.py` | 優先權 Selector：高優先任務失敗 2 次後在第 3 次成功則切回 |
| `sequence.py` | SequenceWithMemory 展示（子節點在 Running 狀態時保持記憶） |
| `context_switching.py` | Parallel 並行切換：工作序列 + 背景管理器 |
| `either_or.py` | 二選一模式 |
| `eternal_guard.py` | 永恆守衛模式 |
| `pick_up_where_you_left_off.py` | 中斷後恢復模式 |

---

## 6. 線上互動式行為樹範例 (behaviortrees.com)

**位置**：https://www.behaviortrees.com/learn/behavior-tree-examples/[^btonline]

**格式**：JSON（可從線上編輯器匯出）

**介紹**：行為樹的互動式學習資源，提供 5 種常見遊戲 AI 模式，可在視覺編輯器中開啟並匯出為 JSON。

1. **敵人巡邏/追趕/攻擊** — Selector 優先權梯
2. **越權備援鏈 (Guarded Fallback)** — 進門嘗試：開門 → 用鑰匙 → 破門
3. **生存覆蓋** — 血量低時強制逃跑/補血
4. **有限重試** — RepeatUntilSuccess max=3 用於抓取
5. **冷卻門控特殊攻擊** — 火箭冷卻中 → 射擊 → 近戰

---

## 總結

| 序號 | 倉庫/位置 | 格式 | 領域 | 定義位置 |
|---|---|---|---|---|
| 1 | oguzaltan/Behavior-Tree-Examples | XML | 教學/機器人 | `*/bt_tree.xml` |
| 2 | BehaviorTree/BehaviorTree.CPP | XML | 測試/範例 | `examples/test_files/*.xml` |
| 3 | wesleywh/GameDevRepo (Rain AI) | XML + Unity | 遊戲 NPC AI | `Rain AI/Behavior Trees/*.xml` |
| 4 | PythonRobotics | XML | 機器人巡邏 | `robot_behavior_tree.xml` |
| 5 | ROS 2 Navigation2 | XML | 機器人導航 | `behavior_trees/*.xml` |
| 6 | py_trees | Python | 機器人/ROS | `py_trees/demos/*.py` |
| 7 | behaviortrees.com | JSON | 遊戲 AI 模式 | 線上編輯器匯出 |

---

[^wiki]: Wikipedia. (n.d.). Behavior tree. Retrieved 2026-09-13, from https://en.wikipedia.org/wiki/Behavior_tree

[^btbook]: Colledanchise, M., & Ögren, P. (2018). Behavior Trees in Robotics and AI: An Introduction. CRC Press. Retrieved 2026-09-13

[^btcpp]: BehaviorTree/BehaviorTree.CPP. (n.d.). GitHub repository. Retrieved 2026-09-13, from https://github.com/BehaviorTree/BehaviorTree.CPP

[^btcpp_examples]: BehaviorTree/BehaviorTree.CPP. (n.d.). examples/test_files. Retrieved 2026-09-13, from https://github.com/BehaviorTree/BehaviorTree.CPP/tree/master/examples/test_files

[^oguz]: oguzaltan. (n.d.). Behavior-Tree-Examples. Retrieved 2026-09-13, from https://github.com/oguzaltan/Behavior-Tree-Examples

[^rain]: wesleywh. (n.d.). GameDevRepo — Rain AI/Behavior Trees. Retrieved 2026-09-13, from https://github.com/wesleywh/GameDevRepo/tree/master/Rain%20AI/Behavior%20Trees

[^rain_patrol]: wesleywh. (n.d.). Patrol.xml. Retrieved 2026-09-13, from https://github.com/wesleywh/GameDevRepo/blob/master/Rain%20AI/Behavior%20Trees/Patrol.xml

[^pyrob]: Atsushi Sakai. (n.d.). PythonRobotics — MissionPlanning/BehaviorTree. Retrieved 2026-09-13, from https://github.com/AtsushiSakai/PythonRobotics/tree/master/MissionPlanning/BehaviorTree

[^nav2]: ROS 2 Navigation2. (n.d.). navigation2/nav2_bt_navigator/behavior_trees. Retrieved 2026-09-13, from https://github.com/ros-navigation/navigation2/tree/main/nav2_bt_navigator/behavior_trees

[^nav2_w_recovery]: ROS 2 Navigation2. (n.d.). navigate_to_pose_w_replanning_and_recovery.xml. Retrieved 2026-09-13, from https://github.com/ros-navigation/navigation2/blob/main/nav2_bt_navigator/behavior_trees/navigate_to_pose_w_replanning_and_recovery.xml

[^pytrees]: splintered-reality. (n.d.). py_trees — demos. Retrieved 2026-09-13, from https://github.com/splintered-reality/py_trees/tree/devel/py_trees/demos

[^btonline]: Behavior Trees Online. (n.d.). Behavior Tree Examples. Retrieved 2026-09-13, from https://www.behaviortrees.com/learn/behavior-tree-examples/