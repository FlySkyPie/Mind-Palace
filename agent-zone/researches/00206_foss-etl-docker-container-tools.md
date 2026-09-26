# FOSS ETL 工具在 Docker 容器中執行任務的調查：工具鏈隔離方案

## 摘要

本報告調查支援在 Docker 容器中執行個別任務（task/step）的 FOSS（自由與開源）ETL 與工作流程編排工具。此類功能對於隔離複雜工具鏈、管理相依性衝突、以及確保可重現的資料管線至關重要。調查重點為該工具是否允許不同任務使用不同的 Docker 映像檔分別執行，以達成真正的工作負載隔離。

## 重點工具比較

| 工具 | Docker 隔離層級 | Docker 支援成熟度 | 需 Kubernetes | ETL 專用？ | 授權 |
|---|---|---|---|---|---|
| **Apache Airflow** | 每個任務 | ✅ 非常成熟 | 否 | ✅ 是 | Apache 2.0 |
| **Dagster** | 每個資產 | ✅ 成熟 | 否 | ✅ 是 | Apache 2.0 |
| **Prefect** | 每個流程 | ✅ 成熟 | 否 | ✅ 是 | Apache 2.0 |
| **Argo Workflows** | 每個步驟（原生容器） | ✅ 非常成熟 | ✅ 需要 | ✅ 是 | Apache 2.0 |
| **Luigi** | 每個任務（社群支援） | ⚠️ 有限 | 否 | ✅ 是 | Apache 2.0 |
| **Dagu** | 每個步驟（原生） | ✅ 中等 | 否 | ⚠️ 部分 | AGPL-3.0 |
| **Temporal** | 每個工作者（非每個活動） | ⚠️ 僅繞行方案 | 否 | ❌ 通用用途 | MIT |

## 工具詳細說明

### 1. Apache Airflow

**授權：** Apache 2.0 | **語言：** Python

Apache Airflow 是目前最成熟的 FOSS 工作流程編排工具之一，以 DAG（有向無環圖）定義管線。其 `DockerOperator` 是實現每個任務獨立容器執行的主要機制。

`DockerOperator` 支援的特性包括：
- 為每個任務指定不同的 Docker 映像檔（例如 `python:3.12-slim` 與 `postgres:16` 交替使用）
- 環境變數、磁碟區掛載、資源限制（CPU/記憶體）
- 網路模式、連接埠綁定、ulimit 設定
- 私有倉庫認證（透過 Docker Connection ID）
- GPU 裝置請求
- 模板化欄位支援（可在執行期動態決定映像檔與指令）

限制：需要 Airflow Worker 能存取 Docker daemon（通常透過 Docker socket 掛載或 Docker-in-Docker）；對於不熟悉 DockerOperator 參數配置的團隊有學習曲線。[^airflow-docker]

### 2. Dagster

**授權：** Apache 2.0 | **語言：** Python

Dagster 以「軟體定義資產」（Software-defined Assets）的理念著稱，強調資料產品的可追蹤性。其 `PipesDockerClient`（位於 `dagster-docker` 套件）可直接從 Dagster 資產或操作（ops）啟動 Docker 容器。

PipesDockerClient 的特色：
- 為每個資產指定獨立的 Docker 映像檔
- 容器可即時回傳日誌、資產檢查結果與具體化事件至 Dagster
- 容器端程式碼幾乎不需修改即可整合
- 也支援 `PipesSubprocessClient` 用於更簡潔的本機執行

限制：PipesDockerClient 較 Airflow 的 DockerOperator 新穎，生態系與文件累積較少；完全隔離需要每個資產各自配置 PipesDockerClient。[^dagster-pipes]

### 3. Prefect

**授權：** Apache 2.0 | **語言：** Python

Prefect 提供 Python 原生語法（`@flow`、`@task` 裝飾器）來定義工作流程。其 Docker 整合以 **Docker Work Pools** 與 **Docker Workers** 為核心。

Docker Workers 的運作方式：
- Docker Work Pool 儲存基礎設施配置（基底映像檔、資源限制、環境變數）
- Docker Worker 輪詢 Prefect API，將每個流程執行（flow run）啟動為獨立的 Docker 容器
- 容器在完成後自動移除
- 不同部署可指定不同的 Docker 映像檔
- 支援 `.deploy()` 自動建置含程式碼的 Docker 映像檔

限制：容器隔離是在**流程層級**（每個流程執行一個容器），而非個別任務層級；要達到任務層級隔離需拆分為子流程（subflows）。[^prefect-docker]

### 4. Argo Workflows

**授權：** Apache 2.0 | **語言：** Go（Kubernetes 原生）

Argo Workflows 採取容器原生設計：工作流程中的**每個步驟本身就是一個容器**。這使其成為隔離強度最高的方案。

每個步驟的 YAML 定義範例：
```yaml
templates:
  - name: my-step
    container:
      image: python:3.12-slim
      command: ["python", "script.py"]
```

每個步驟作為完全獨立的 Kubernetes Pod 執行，擁有自己的 Docker 容器。支援完整的 Docker 功能：磁碟區掛載、環境變數、資源限制。步驟間的資料傳遞透過成品倉庫（S3、GCS 等）處理。

限制：**必須有 Kubernetes 叢集**（即使是本機的 minikube 或 Docker Desktop 內建的 K8s）；對純 Docker 環境的使用者而言基礎設施開銷過大；工作流程以 YAML 而非程式碼定義，缺乏 Python 原生語法。[^argo]

### 5. Luigi

**授權：** Apache 2.0 | **語言：** Python

Luigi 是輕量級的依賴解決與管線排程工具。然而，其**沒有內建的 Docker 支援**。社群貢獻者（如 Open Targets）曾貢獻過一個 Docker Runner 模組，透過 `docker-py` 與 Docker API 直接溝通。

社群 Docker Runner 可做到：
- 為任務啟動任何 Docker 容器
- 控制磁碟區掛載
- 平行執行多個容器化工作

限制：無官方維護的 DockerOperator；容器編排屬於「外掛」而非第一線功能；即時日誌串流無法回傳至 Luigi UI。[^luigi-docker]

### 6. Dagu

**授權：** AGPL-3.0 | **語言：** Go

Dagu 是輕量級、自我託管的工作流程引擎，單一二進位檔即可執行。其原生支援在 YAML 工作流程中直接定義 Docker 容器執行。

特性：
- 無需外部資料庫
- 內建 Web UI
- 除了 Docker 容器，也支援 Shell 指令、Kubernetes Jobs、SSH 指令
- 適合不需要大型平台的小型團隊

限制：社群較小（GitHub 約 4,100 星）；整合與外掛生態系較弱；不適合大規模 ETL；僅支援 YAML 定義。[^dagu]

### 7. Temporal

**授權：** MIT | **語言：** Go（伺服器），多語言 SDK

Temporal 是持久執行引擎，採用工作流程（Workflow）與活動（Activity）模型。它不是 ETL 專用工具，但常被用於工作流程編排。

其 Docker 相關支援：
- Temporal Server 本身可透過 Docker Compose 部署
- 工作者（Workers）可打包為容器
- 但 Temporal **不原生支援**每個活動（Activity）在獨立容器中執行

限制：每個活動的容器執行需要自訂實作繞行方案；需要運作 Temporal Server 做為基礎設施；學習曲線較高。[^temporal]

## 建議

- **最成熟的每個任務 Docker 隔離** → **Apache Airflow**（DockerOperator）或 **Argo Workflows**（在 Kubernetes 環境中）
- **現代開發體驗 + Docker 外部程式碼執行** → **Dagster**（PipesDockerClient）
- **最簡單的流程層級 Docker 隔離** → **Prefect**（Docker Worker + Work Pools）
- **輕量、無資料庫的 Docker 管線執行** → **Dagu**
- **Kubernetes 原生的容器化工作流程** → **Argo Workflows**

## 參考資料

[^airflow-docker]: Apache Airflow. (n.d.). DockerOperator. Retrieved 2026-09-25, from https://airflow.apache.org/docs/apache-airflow-providers-docker/stable/_api/airflow/providers/docker/operators/docker/index.html

[^dagster-pipes]: Dagster. (n.d.). Dagster Pipes. Retrieved 2026-09-25, from https://docs.dagster.io/concepts/dagster-pipes

[^prefect-docker]: Prefect. (n.d.). Docker Workers. Retrieved 2026-09-25, from https://docs.prefect.io/v3/how-to-guides/deployment_infra/docker

[^argo]: Argo. (n.d.). Argo Workflows. Retrieved 2026-09-25, from https://argoproj.github.io/workflows/

[^luigi-docker]: Open Targets. (2017). Using containers with Luigi. Retrieved 2026-09-25, from https://blog.opentargets.org/using-containers-with-luigi/

[^dagu]: Dagu. (n.d.). Dagu — A lightweight workflow engine. Retrieved 2026-09-25, from https://github.com/dagucloud/dagu/

[^temporal]: Temporal. (n.d.). Temporal Worker best practices. Retrieved 2026-09-25, from https://docs.temporal.io/best-practices/worker