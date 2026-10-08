# 機器學習與基因演算法領域：研究者是否使用 GUI 工具檢索不同迭代分支的「基因」？

## 摘要

本報告調查機器學習（ML）與基因演算法（GA）領域的研究者及工程師，在實務上是否使用圖形化使用者介面（GUI）工具來檢索、瀏覽、比較管理不同演化迭代分支所產生的「基因」（個體／超參數組合）。調查結果顯示：**目前不存在單一成熟、專門用於瀏覽演化樹狀譜系（genealogy tree）的 GUI 工具**，但生態系已分化出兩條路徑——(1) 傳統 GA 框架依賴程式碼層級的系譜追蹤搭配靜態繪圖；(2) ML 超參數最佳化領域則發展出成熟的 Web 儀表板，用於瀏覽、比較、管理不同實驗分支。兩者之間存在顯著的功能斷層。

---

## 1. 問題背景

基因演算法與演化式計算（Evolutionary Computation, EC）的核心運作方式是：透過選擇（selection）、交配（crossover）、突變（mutation）反覆產生新一代個體（individuals），每一代構成一次**迭代**，而不同超參數設定或隨機種子則可能產生**分支（branch）**。研究者常需要：

- 跨越世代檢視某個個體的**祖先鏈**（ancestry line / genealogy）；
- 比較不同分支中表現最佳的「基因型」（genotype）；
- 從特定中繼世代**恢復演化**（checkpoint resume）；
- 視覺化整個群體的**譜系樹**（genealogy tree）。

這些需求近似於版本控制系統（如 Git）對原始碼的管理，但物件是「基因」而非檔案。

---

## 2. 傳統 GA 框架的工具現狀

### 2.1 DEAP 的 History 類別與 NetworkX 譜系樹

Python 生態系中最活躍的 GA 框架 **DEAP (Distributed Evolutionary Algorithms in Python)** 並未內建 GUI，但其 `deap.tools.History` 類別提供了完整的譜系追蹤能力[^deap-history]：

- `genealogy_tree`：字典形式的父子索引對映（parent → children 列表）
- `genealogy_history`：依產生序儲存所有個體的索引列表
- 搭配 `history.decorator` 裝飾器，在 `mate` 與 `mutate` 操作時自動記錄親緣關係

範例用法[^deap-history-example]：
```python
history = History()
toolbox.decorate("mate", history.decorator)
toolbox.decorate("mutate", history.decorator)

# 演化完成後繪製譜系樹
graph = networkx.DiGraph(history.genealogy_tree).reverse()
colors = [toolbox.evaluate(history.genealogy_history[i])[0] for i in graph]
networkx.draw(graph, node_color=colors)
plt.show()
```

節點顏色可編碼適應度（fitness），藍色低、紅色高——這是目前 GA 領域**最接近「可視覺瀏覽譜系」**的開箱方案。然而，其輸出為**靜態 matplotlib 圖片**，不具縮放、搜尋、節點展開等互動能力，且當族群規模大（>1000 個體）時圖形難以閱讀。

DEAP 的 GitHub 倉儲擁有 **~5,800 顆星**[^deap-github]，是 Python GA 領域事實上的標準。但其使用者多半透過 Jupyter Notebook 進行互動式探索，而非依賴專用 GUI。

### 2.2 Watchmaker Framework（Java）的 EvolutionMonitor

Watchmaker Framework 是 Java 生態系的 GA 框架，內建 **Swing EvolutionMonitor** 元件[^watchmaker]：

- 可直接附加至任何 `EvolutionEngine`，即時顯示最佳個體與適應度隨時間的曲線；
- 提供「互動式演化」（Interactive Evolutionary Algorithm）—使用者可手動選擇偏好個體；
- 包含多種範例（Mona Lisa 演化多邊形、TSP、數獨等）。

但 Watchmaker 自 **2010 年**（v0.7.1）後未再更新，社群已不活躍。

### 2.3 GAlib（C++）的 GA View

GAlib（Matthew 的 Genetic Algorithms Library）提供了 Windows MFC 應用程式 **GAlib View**[^galib]：

- 即時調整交配率、突變率、表示法與演算法；
- 2D/3D 函數最大化視覺化（使用 OpenGL）；
- 旅行推銷員問題（TSP）的族群可視化。

但 GAlib 最後更新約在 **2007 年**，且 GUI 高度綁定於特定範例，非通用型譜系瀏覽工具。

### 2.4 NEAT-Python 的靜態譜系繪圖

NEAT（NeuroEvolution of Augmenting Topologies）實作 **neat-python** 提供了 `visualize.py` 模組[^neat-python]：

- `draw_net()`：渲染個體神經網路結構；
- `plot_stats()`：最佳／平均適應度隨世代變化；
- `plot_species()`：物種（species）群體大小隨世代變化；
- `StatisticsReporter`：追蹤每世代統計資料；
- `Checkpointer`：儲存／還原檢查點，支援分支層級的恢復。

同樣地，上述輸出均為靜態圖表，缺乏互動式 GUI。

---

## 3. ML 超參數最佳化領域的 GUI 儀表板

在 ML 領域（尤其是深度學習），研究者更常面對的是**超參數組合的比較**而非個體譜系。這方面的工具生態相對成熟。

### 3.1 Optuna Dashboard

**Optuna** 是日本 Preferred Networks 開發的超參數最佳化框架，其 **Optuna Dashboard**[^optuna-dashboard]（~805 顆星[^optuna-dashboard-gh]）提供了完整的 Web 儀表板：

- **最佳化歷史圖**（Optimization History plot）：目標值 vs. trial 編號；
- **平行座標圖**（Parallel Coordinate plot）：高維超參數關係，以成效編碼顏色；
- **等高線／切片圖**（Contour / Slice plots）；
- **參數重要性**（Parameter Importances）；
- **時間軸**（Timeline view）；
- **EDF 圖**（Empirical Distribution Function）。

啟動方式：
```bash
pip install optuna-dashboard
optuna-dashboard sqlite:///db.sqlite3
```

可用於即時比較不同 trial（≈ GA 中的個體）的表現，但其設計將每個 trial 視為獨立，缺乏父子親緣關係模型——無法回答「這個 trial 的『父母』是誰」的問題。

### 3.2 Weights & Biases (W&B) Sweeps

**Weights & Biases**[^wandb-sweeps] 是商業化的實驗追蹤平台，其 Sweeps 模組提供：

- 平行座標圖、散點圖、參數重要性排序；
- 即時串流更新；
- 從瀏覽器暫停、恢復、取消 sweep；
- 支援 **HyperBand** 早停策略。

W&B 的介面最為精緻，適合協作團隊。但其抽象層是「run」（一次執行），而非 GA 語意中的「世代」與「譜系」。

### 3.3 TensorBoard HParams Dashboard

TensorFlow 內建的 TensorBoard 提供 HParams 儀表板[^tensorboard-hparams]：平行座標、表格檢視、散點圖、session group 分組等功能。深度學習研究者廣泛使用，但不支援譜系樹。

### 3.4 MLflow Tracking UI

MLflow[^mlflow] 的 UI 提供 run 比較、平行座標、參數重要性計算、artifact 瀏覽。為 ML 領域實驗管理的標準工具之一，但同樣缺乏譜系樹。

---

## 4. 學術界的譜系視覺化研究

學術界對 GA 譜系視覺化的關注持續成長，以下為代表性文獻：

### 4.1 Gavel（2001）

Hart 與 Ross 提出的 **Gavel**[^gavel] 是較早專注於 GA 譜系可視化的工具：

> 「其新穎之處在於使用祖先系譜（ancestry）… 來視覺化演化演算法的過程與狀態，包含對偶基因（alleles）與適應度。」

被引 **116 次**。

### 4.2 視覺分析與演化演算法實驗分析（2011）

Lutton 與 Fekete[^lutton-fekete] 提出「系譜學家工具具有可適用於 EA 視覺化的特徵」，並提倡建構更通用的可視化系統。

### 4.3 ParetoTracker（2024）

Zhang 等人發表的 **ParetoTracker**[^paretotracker]（IEEE VIS 2024）聚焦於多目標演化演算法的**譜系連線（lineage connections）**：

> 「使多個個體的譜系連線成為可能… 然而，在散點圖中視覺化譜系樹存在挑戰。」

### 4.4 可追蹤演化演算法（T-EA, 2026）

Benecke 的博士論文[^benecke] 提出了 **traceable evolutionary algorithm (T-EA)**，專注於：

> 「視覺化與評估群體經由搜尋空間的路徑… 系譜資訊的應用日益重要。」

### 4.5 VR 譜系視覺化（2018）

Dolson 與 Ofria[^dolson] 在 GECCO 2018 發表 VR 環境中的**生命之河**（Tape of Life）視覺化，展現譜系跨越適應度地圖的軌跡。

---

## 5. 綜合分析

### 5.1 回答核心問題：研究者是否使用 GUI 來檢索不同迭代分支的基因？

**部分是，但方式高度分化：**

| 使用者類型 | 常用工具 | GUI 類型 | 譜系瀏覽能力 |
|------------|----------|----------|--------------|
| GA 研究者（Python） | DEAP + Jupyter/NetworkX | 靜態繪圖（matplotlib） | ✅ 核心能力（但靜態） |
| GA 研究者（Java） | Watchmaker（已停滯） | Swing 桌面 GUI | ⚠️ 僅適應度曲線 |
| ML 超參數調校者 | Optuna Dashboard | Web 儀表板 | ❌ 無譜系，但有分支比較 |
| ML 實驗管理者 | W&B / MLflow | Web 儀表板 | ❌ 無譜系 |
| 深度學習者 | TensorBoard | Web 儀表板 | ❌ 無譜系 |

### 5.2 關鍵缺口

目前**不存在**一款成熟的 GUI 工具，同時滿足：
1. ✅ 譜系樹（genealogy tree）瀏覽
2. ✅ 跨代個體比較
3. ✅ 分支層級的恢復（checkpoint resume）
4. ✅ 互動式（zoom / search / filter / expand）
5. ✅ 活躍維護

最接近需求的是 **DEAP 的 History + NetworkX**，但缺乏互動性。ML 領域的 Optuna Dashboard 雖提供互動 Web UI，但譜系模型不存在。

### 5.3 為何這個缺口存在？

- **GA 社群規模相對小**：與深度學習相比，使用純 GA 的研究者與工程師數量少得多，市場不足以支撐專用 GUI 產品。
- **個體表徵多樣性**：基因型可以是浮點數向量、二叉樹（GP）、排列（TSP）、神經網路拓樸（NEAT）——統一視覺化極困難。
- **「分支」語意模糊**：GA 中的分支可能來自不同隨機種子、不同超參數、不同選擇壓力、甚至不同交叉算子——與 Git 分支不同，缺乏標準化的分支模型。
- **ML 領域轉向量化**：2010 年代後，ML 研究者大量轉向可微分的梯度下降法，GA 在 ML 中的地位（除 NEAT 與某些強化學習場景外）相對邊緣。

---

## 6. 結論

機器學習與基因演算法的研究者在實務上**有需求但缺乏專用 GUI**來檢索不同迭代分支的基因。當前解決方案分為兩極：

- 對需要**譜系樹語意**的 GA 研究者，唯一的原生方案是 DEAP History + NetworkX 的靜態繪圖；
- 對需要**跨實驗分支比較**的 ML 超參數調校者，Optuna Dashboard、W&B、TensorBoard、MLflow 等 Web 儀表板提供了成熟且廣為使用的互動環境，但代價是捨棄了譜系親緣資訊。

兩者之間的**譜系互動瀏覽器**仍是未滿足的需求。若有新工具能將 DEAP 的譜系模型與 Optuna/W&B 的互動 Web UI 結合，將可能填補此缺口。

---

## 參考文獻

[^deap-history]: DEAP. (n.d.). `deap.tools.History` — DEAP 1.4.3 documentation. Retrieved 2026-10-08, from https://deap.readthedocs.io/en/master/api/tools.html#deap.tools.History

[^deap-history-example]: DEAP. (n.d.). History example — DEAP documentation. Retrieved 2026-10-08, from https://deap.readthedocs.io/en/master/examples/ga_onemax_history.html

[^deap-github]: DEAP. (n.d.). DEAP — Distributed Evolutionary Algorithms in Python. GitHub. Retrieved 2026-10-08, from https://github.com/DEAP/deap

[^watchmaker]: Watchmaker Framework. (n.d.). Uncommons Watchmaker Framework for Evolutionary Computation. Retrieved 2026-10-08, from https://watchmaker.uncommons.org/

[^galib]: GAlib. (n.d.). Matthew's Genetic Algorithms Library — Screenshots. Retrieved 2026-10-08, from http://lancet.mit.edu/ga/ScreenShots.html

[^neat-python]: neat-python. (n.d.). NEAT (NeuroEvolution of Augmenting Topologies) in Python. GitHub. Retrieved 2026-10-08, from https://github.com/CodeReclaimers/neat-python

[^optuna-dashboard]: Optuna. (n.d.). Optuna Dashboard — Real-time Web Dashboard. Retrieved 2026-10-08, from https://github.com/optuna/optuna-dashboard

[^optuna-dashboard-gh]: Optuna. (n.d.). optuna/optuna-dashboard. GitHub. Retrieved 2026-10-08, from https://github.com/optuna/optuna-dashboard

[^wandb-sweeps]: Weights & Biases. (n.d.). Sweeps — Hyperparameter Optimization. Retrieved 2026-10-08, from https://wandb.ai/site/sweeps

[^tensorboard-hparams]: TensorFlow. (n.d.). Hyperparameter Tuning with the HParams Dashboard. Retrieved 2026-10-08, from https://www.tensorflow.org/tensorboard/hyperparameter_tuning_with_hparams

[^mlflow]: MLflow. (n.d.). MLflow Tracking UI. Retrieved 2026-10-08, from https://mlflow.org/docs/latest/tracking.html

[^gavel]: Hart, E., & Ross, P. (2001). Gavel: A new tool for genetic algorithm visualization. *IEEE Transactions on Evolutionary Computation*. Retrieved 2026-10-08, from https://ieeexplore.ieee.org/abstract/document/942528/

[^lutton-fekete]: Lutton, E., & Fekete, J. D. (2011). Visual analytics and experimental analysis of evolutionary algorithms (INRIA Research Report). Retrieved 2026-10-08, from https://inria.hal.science/inria-00587170/

[^paretotracker]: Zhang, Z., Yang, F., Cheng, R., & Ma, Y. (2024). ParetoTracker: Understanding population dynamics in multi-objective evolutionary algorithms. *IEEE Conference on Visualization*. Retrieved 2026-10-08, from https://ieeexplore.ieee.org/abstract/document/10670520/

[^benecke]: Benecke, T. (2026). Exploring the population dynamics of evolutionary algorithms using gene heritage (Doctoral dissertation). Otto-von-Guericke-Universität Magdeburg. Retrieved 2026-10-08, from https://repo.bibliothek.uni-halle.de/handle/1981185920/125323

[^dolson]: Dolson, E., & Ofria, C. (2018). Visualizing the tape of life: Exploring evolutionary history with VR. *GECCO 2018*. Retrieved 2026-10-08, from https://dl.acm.org/doi/10.1145/3205651.3208287