# Prefect 對 Docker 容器執行 Task 的支援度調查

## 結論

**Prefect 並沒有內建的 `DockerTaskRunner`，無法讓 Flow 內的個別 Task 各自跑在獨立的 Docker 容器中。**

Prefect 的 Task Runner 僅提供四種：`ThreadPoolTaskRunner`（預設）、`ProcessPoolTaskRunner`、`DaskTaskRunner` 與 `RayTaskRunner` [^task-runners]。沒有任何一個可以將個別 Task 派發至 Docker 容器執行。

## Docker 在 Prefect 的實際使用層級

### 1. Flow Run 層級（完整支援）

Prefect 的 Docker 整合主要在 **Flow Run 層級**，即整支 Flow 跑在一個 Docker 容器內：

- 建立 Docker Work Pool → Docker Worker 輪詢 Flow Run → 每個 Flow Run 在獨立的 ephemeral Docker 容器中執行
- 容器內的所有 Task 共享同一個執行環境，不會各自開新容器 [^docker-worker]

```bash
# 建立 Docker Work Pool
prefect work-pool create --type docker my-docker-pool

# 啟動 Docker Worker
prefect worker start --pool my-docker-pool
```

```python
# 部署 Flow 至 Docker 容器
from prefect import flow

@flow(log_prints=True)
def buy():
    print("Buying securities")

if __name__ == "__main__":
    buy.deploy(
        name="my-deployment",
        work_pool_name="my-docker-pool",
        image="my_registry/my_image:my_image_tag"
    )
```

### 2. Subflow 層級（透過變通方式支援）

可將 Subflow 包裝成另外的 Deployment，透過 `run_deployment()` 呼叫不同 Work Pool，使其在另一個 Docker 容器中執行：

```python
from prefect.deployments import run_deployment

@task
def trigger_subflow(data):
    run_deployment(
        name="my-subflow/deployment-name",
        parameters={"data": data},
        work_pool_name="subflow-pool"  # 不同的 Docker Pool
    )
```

這是最接近「個別 Task 跑在獨立容器」的正規做法，但粒度是 Subflow 而非 Task [^subflow-example]。

### 3. Background Task Worker 層級（部分支援）

Prefect 支援 Background Task Worker 在 Docker 容器中執行，但這是一個長期運行的 Process，不會每個 Task 啟動一個新容器：

```python
from prefect import task
from prefect.task_worker import serve

@task
def my_task(name: str):
    print(f"Hello, {name}!")

if __name__ == "__main__":
    serve(my_task)
```

透過 `.delay()` 呼叫的背景 Task 會由此 Worker 處理，但 Worker 本身是一個在容器內長期運行的 Process，不是每呼叫一次就起一個新容器 [^bg-task-examples]。

## 若需將個別 Task 隔離在 Docker 容器中的實作方式

| 方式 | 支援度 | 說明 |
|------|--------|------|
| 整支 Flow 跑在 Docker 容器 | ✅ 完整支援 | Docker Work Pool + Worker，所有 Task 共用容器 |
| Subflow 放在不同 Docker 容器 | ✅ 可透過多個 Deployment 達成 | 使用 `run_deployment()` 搭配不同 Work Pool |
| **個別 Task 各自獨立容器** | ❌ 無原生支援 | 需透過兩種變通方式 |

### 兩種變通方案

1. **將每個 Task 重構成 Subflow Deployment** — 每個 Subflow 各自有一個 Deployment，透過 `run_deployment()` 叫用，即可讓它們各別跑在獨立的 Docker 容器中 [^subflow-example]。

2. **在 Task 內手動執行 Docker** — 在 Task 中使用 `subprocess.run(["docker", "run", ...])` 或 Docker Python SDK 來啟動容器。但這會繞過 Prefect 的編排功能，失去自動重試、狀態追蹤與 Artifact 收集等能力 [^discussion-16160]。

## 限制

1. **不支援 Task 層級的 Docker 隔離** — 無法對 Flow 內的個別 Task 指定各自的容器環境
2. **Subflow 方案需要重構** — 若要做到 Task 層級的隔離，需要將每個 Task 獨立成 Flow
3. **手動 Docker 執行無編排支援** — `subprocess` 方式喪失 Prefect 的編排能力
4. **容器啟動開銷** — 對細粒度 Task 來說，每次啟動容器的延遲可能過高
5. **跨容器資料傳遞** — 需依賴外部儲存（物件儲存、共享 Volume 等）而非 Python 記憶體物件

## 參考資料

[^task-runners]: Prefect. (n.d.). Task Runners. Retrieved 2026-09-25, from https://docs.prefect.io/v3/concepts/task-runners
[^docker-worker]: Prefect. (n.d.). How to Run Flows in Docker Containers. Retrieved 2026-09-25, from https://docs.prefect.io/v3/how-to-guides/deployment_infra/docker
[^subflow-example]: EnigmaticMachine. (n.d.). prefect-modular-isolated-subflows. Retrieved 2026-09-25, from https://github.com/EnigmaticMachine/prefect-modular-isolated-subflows
[^bg-task-examples]: Prefect. (n.d.). Prefect Background Task Examples. Retrieved 2026-09-25, from https://github.com/PrefectHQ/prefect-background-task-examples
[^discussion-16160]: Prefect. (n.d.). Running tasks in Docker containers — GitHub Discussion #16160. Retrieved 2026-09-25, from https://github.com/PrefectHQ/prefect/discussions/16160
[^stackoverflow]: Stack Overflow. (n.d.). Docker run as Prefect task. Retrieved 2026-09-25, from https://stackoverflow.com/questions/62509406/docker-run-as-prefect-task