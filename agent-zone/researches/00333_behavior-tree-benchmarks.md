# Behavior Tree 的評測基準與評估框架

Behavior Tree (BT) 在遊戲 AI、機器人學與規劃領域中，過去缺乏統一的標準評測平台，直到 2025 年才有第一個專門的 BT 規劃基準平台出現。以下整理目前已知的 BT 評測基準、評估框架與比較研究。

---

## 1. BTPG — Behavior Tree Planning Gym

IJCAI 2025 由 Chen 等人發表，**首個專為 BT 規劃設計的基準平台與評測環境**。目標是填補 BT 規劃演算法缺乏標準平台、測試基準與全面指標的空白[^btpg-ijcai]。

**技術特點：**
- 將行為節點以**謂詞邏輯 (predicate logic)** 與分類物件表示
- 採用 **STRIPS** 風格形式化 BT 規劃
- 提供 **4 個環境 × 3 個模擬器**：RoboWaiter（餐廳服務機器人）、VirtualHome（居家機器人）、RobotHow（居家機器人測試）、OmniGibson（基於 IsaacSim 4.2 的物理模擬）
- 內建**資料集生成器**
- 測試 4 種 BT 規劃演算法（ReactivePlanning、BTExpansion、OBTEA、HOBTEA-BTPG[^btpg-gh]）

**新增指標：**
1. **Planning progress** — 規劃過程的推進效率
2. **Region distance** — 空間覆蓋率
3. **Execution robustness** — 執行階段對失敗的韌性

**既有指標亦採用：** cost、expanded nodes、action steps、timeout rate、error rate[^btpg-gh]

---

## 2. BT vs. FSM 比較框架

Iovino 等人 (2024) 的論文《Comparison between Behavior Trees and Finite State Machines》(arXiv:2405.16137) 提出了**基於指標的 BT 與 FSM 比較框架**[^bt-vs-fsm]：

| 屬性 | 使用指標 |
|---|---|
| **Modularity（模組性）** | Add/remove 操作的計算複雜度、Graph Edit Distance (GED) |
| **Reactivity（反應性）** | 結構中 active elements 數量 |
| **Readability（可讀性）** | 圖形元素數 vs. active elements 數 |
| **Design（設計）** | 修改結構所需 effort |

**量化比較結果（以 mobile manipulation robot 為實驗對象）：**

| 指標 | BT | FSM (容錯) | HFSM |
|---|---|---|---|
| 計算複雜度 (add/remove) | **O(1)** | O(n) | O(1) |
| Edit distance (tuck arm) | **6** | 51 | 2 |
| Edit distance (safe move) | **2** | 24 | 4 |
| Edit distance (dock subtree) | **8** | 85 | 17 |
| Edit distance (recharge battery) | **8** | 88 | 17 |

實驗使用 **py_trees** (BT) 與 **SMACH** (FSM)，皆與 ROS 相容[^bt-vs-fsm]。

---

## 3. BT 實作策略評測 (C# 基準)

d-bucur 的 GitHub 專案對不同程式典範的 BT 實作進行系統性基準測試[^bt-impl-bench]：

| 實作方式 | 平均時間 (10 次) | 平均時間 (100 次) | 記憶體 |
|---|---|---|---|
| Coroutines | 1,883.6 ns | 20,139.0 ns | 4,355 B |
| Functional/closures | 791.5 ns | 4,742.6 ns | 2,000 B |
| Classes (OOP) | 762.0 ns | 4,704.7 ns | 1,008 B |
| Union types | 689.1 ns | 5,142.0 ns | 888 B |

**結論：** Classes 實作方式在執行時間、記憶體效率與可擴展性之間達到最佳平衡。

---

## 4. Mario AI Benchmark

Karakovskiy & Togelius (2012) 提出的 Mario AI Benchmark 被用於遊戲 AI 中 BT 與 FSM 的比較[^bt-vs-fsm]：
- 評估指標：**節點數量**、**reward function ρ(x)**
- 發現：BT 複雜度隨任務複雜度**線性成長**，而 FSM 為**二次方或更差**

---

## 5. 基於 BT 的敵方難度自動化分析

《Breaking down the challenge: A comprehensive framework for evaluating enemy difficulty with automated behavior tree analytics》提出了一套從 BT 結構推導遊戲難度指標的分析框架[^bt-difficulty]。

---

## 6. 循環複雜度與結構指標

BT 的結構指標已有形式化定義：
- **Cyclomatic Complexity (CC)：** `CC = a + s - n + 1`（arcs、sinks、nodes）。BT 因其結構介面可達到最優模組性（CC = 1）[^bt-vs-fsm]。
- **Maintainability Index (MI)：** 結合 CC、程式行數、註解比例與 Halstead volume[^bt-survey]。

---

## 7. LLM-as-BT-Planner 評估（ICRA 2025）

評估 LLM 生成的 BT 用於機器人任務規劃，使用以下指標[^llm-bt]：
- **Success rate**（成功率）
- **Robustness**（韌性）
- **Comprehensibility**（可理解性）
- 比較不同 in-context learning 方法與 fine-tuning 方法
- 同時在**模擬與真實環境**（機器人組裝任務）中實驗

---

## 8. 常見 BT 評估平台與函式庫

Iovino 等人 (2020) 的調查論文整理了常用來評估 BT 的軟體平台[^bt-survey]：

| 平台 / 函式庫 | 領域 |
|---|---|
| **py_trees** (Python) | 機器人學（與 ROS 相容） |
| **SMACH** (ROS) | FSM 基線比較 |
| **CoSTAR** (ROS) | 圖形化 BT 編輯；可用性研究 |
| **BehaviorTree.CPP** | C++ BT 函式庫 |
| **Groot** | 圖形化 BT 編輯器 |
| **behaviac** (騰訊) | 遊戲 AI（用於《王者榮耀》等 AAA 作品） |
| **Navigation2** (ROS2) | 基於 BT 的機器人導航評估 |

---

## 總結

| 基準 / 框架 | 領域 | 主要指標 |
|---|---|---|
| **BTPG (IJCAI 2025)** | 機器人學（日常服務） | Planning progress, region distance, execution robustness, cost, expanded nodes |
| **BT vs. FSM 比較 (2024)** | 機器人學（移動操作） | Edit distance, 計算複雜度, active/graphical elements |
| **BT 實作策略基準** | 軟體工程 | 執行時間, 記憶體, GC overhead |
| **Mario AI Benchmark** | 遊戲 AI | 節點數, reward score |
| **敵方難度分析框架** | 遊戲 AI | 從 BT 結構推導難度指標 |
| **LLM-as-BT-Planner (ICRA 2025)** | 機器人學（組裝） | Success rate, robustness |
| **循環複雜度** | 形式化 / 結構 | CC = a+s-n+1 |

BTPG 是目前針對 BT **規劃**領域最完整的專用基準平台；BT vs. FSM 比較論文則提供了最詳盡的指標導向比較框架。在此之前，BT 評測多為各論文自建自訂環境，缺乏標準化。

---

[^btpg-ijcai]: Chen, et al. (2025). Behavior Tree Planning Gym: The First Platform and Benchmark for BT Planning in Everyday Service Robots. *IJCAI 2025 Proceedings*. Retrieved 2026-10-01, from https://www.ijcai.org/proceedings/2025/969

[^btpg-gh]: DIDS-EI. (2025). BTPG — Behavior Tree Planning Gym. GitHub Repository. Retrieved 2026-10-01, from https://github.com/DIDS-EI/BTPG

[^bt-vs-fsm]: Iovino, F., et al. (2024). Comparison between Behavior Trees and Finite State Machines. *arXiv preprint arXiv:2405.16137*. Retrieved 2026-10-01, from https://arxiv.org/abs/2405.16137

[^bt-survey]: Iovino, F., et al. (2020). A Survey of Behavior Trees in Robotics and AI. *arXiv preprint arXiv:2005.05842*. Retrieved 2026-10-01, from https://arxiv.org/abs/2005.05842

[^bt-impl-bench]: d-bucur. (n.d.). behavior-tree-benchmarks — Benchmark of BT Implementation Strategies. GitHub Repository. Retrieved 2026-10-01, from https://github.com/d-bucur/behavior-tree-benchmarks

[^llm-bt]: Yuan, C., et al. (2025). LLM-as-BT-Planner. *arXiv preprint arXiv:2409.10444*. Retrieved 2026-10-01, from https://arxiv.org/abs/2409.10444

[^bt-difficulty]: Anonymous. (2026). Breaking down the challenge: A comprehensive framework for evaluating enemy difficulty with automated behavior tree analytics. *ScienceDirect*. Retrieved 2026-10-01, from https://www.sciencedirect.com/science/article/pii/S1875952126000182