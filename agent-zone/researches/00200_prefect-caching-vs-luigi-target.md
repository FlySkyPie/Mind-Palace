# Prefect 的快取機制：有別於 Luigi 的 Makefile 式檔案偵測

## 問題概述

Luigi 採用類似 Makefile 的設計邏輯，通過偵測輸出檔案的存在與否來判斷 Job 是否已完成。例如，下載任務完成後會產生一個檔案，下次執行時 Luigi 檢查該檔案是否存在，若存在則跳過任務[^luigi-target]。

相比之下，Prefect 的任務定義方式幾乎是純函數風格，沒有明顯的「輸出目標」宣告點。那麼 Prefect 如何處理這類「任務已完成，無須重複執行」的快取需求？

## Prefect 的快取機制

Prefect 不使用檔案存在與否作為完成與否的訊號，而是透過 **快取金鑰（cache key）** 系統搭配 **結果儲存（result storage）** 來實現等同功能[^prefect-caching]。

### 執行流程

1. 任務執行前，Prefect 計算出一個**快取金鑰**——這是一個由任務輸入、原始碼、或流程參數等可配置元素決定的確定性字串
2. Prefect 在該任務的**結果儲存後端**（預設為本機檔案系統 `~/.prefect/storage/`，可換成 S3、GCS 等）中查詢此金鑰
3. 若找到**未過期**的記錄，任務進入 `Cached` 狀態，直接載入先前持久化的結果，**不執行任務本體**
4. 若未找到記錄（或記錄已過期），任務正常執行並將結果寫入儲存[^prefect-caching-flow]

### 快取原則（Cache Policies）

Prefect 內建五種快取原則，控制快取金鑰的組成內容[^prefect-policies]：

| 原則 | 金鑰依據 | 行為 |
|------|----------|------|
| `DEFAULT` | 任務輸入 + 任務原始碼 + 所屬流程執行 ID | 同一流程內，相同輸入即命中快取 |
| `INPUTS` | 僅任務的輸入參數 | 跨流程執行，相同輸入即命中 |
| `TASK_SOURCE` | 僅任務原始碼內容 | 原始碼未修改即命中（忽略輸入） |
| `FLOW_PARAMETERS` | 僅所屬流程的參數值 | 流程參數不變即命中 |
| `NO_CACHE` | 固定回傳 `None` | 永不快取 |

原則之間可以用 `+` 組合（擴大快取覆蓋範圍），用 `-` 排除特定輸入參數[^prefect-compose]：

```python
from prefect.cache_policies import INPUTS, TASK_SOURCE

@task(cache_policy=INPUTS + TASK_SOURCE)
def my_task(x: int):
    return x + 1

@task(cache_policy=INPUTS - 'debug')
def my_task(x: int, debug: bool = False):
    return x + 1  # debug 參數不影響快取
```

### 快取有效期

可設定 `cache_expiration`（`datetime.timedelta`）來自動失效快取[^prefect-expiration]：

```python
from datetime import timedelta

@task(cache_policy=INPUTS, cache_expiration=timedelta(hours=1))
def fetch_data(url: str):
    return requests.get(url).json()
```

### 結果持久化（Result Persistence）

快取**必須**啟用結果持久化，此功能預設為關閉。可透過環境變數或任務裝飾器啟用[^prefect-persist]：

```bash
prefect config set PREFECT_RESULTS_PERSIST_BY_DEFAULT=true
```

或逐任務啟用：

```python
@task(persist_result=True)
def add_one(x: int):
    return x + 1
```

### 自訂快取金鑰函數

進階用法可傳入 `cache_key_fn` 完全自訂金鑰邏輯[^prefect-custom-key]：

```python
def static_cache_key(context, parameters):
    return "always-the-same-key"

@task(cache_key_fn=static_cache_key)
def my_cached_task(x: int):
    return x + 1
```

### 隔離層級與並發控制

Prefect 支援兩種隔離層級[^prefect-isolation]：

- **`READ_COMMITTED`**（預設）：讀取最新已提交的快取，允許並發執行
- **`SERIALIZABLE`**：確保同一快取金鑰同時只有一個任務執行（透過鎖管理器）

鎖管理器支援 `MemoryLockManager`（執行緒/協程）、`FileSystemLockManager`（同機器行程）、`RedisLockManager`（跨機器）。

## 與 Luigi 的哲學對比

| 面向 | Prefect | Luigi |
|------|---------|-------|
| **偵測方式** | 自動查詢快取金鑰於結果儲存 | 手動定義 `.output()` 回傳檔案路徑，檢查檔案是否存在 |
| **快取範圍** | 輸入 + 原始碼 + 執行上下文（可配置） | 僅輸出檔案路徑 |
| **狀態** | 豐富狀態追蹤（`Cached`、`Completed`、`Failed`） | 檔案存在/不存在（二元判定） |
| **有效期** | 內建 `cache_expiration` | 無——須手動刪除目標檔案 |
| **儲存後端** | 可插拔：本機 FS、S3、GCS、Azure Blob 等 | 典型為本機 FS 或 S3（透過 `Target` 子類） |
| **序列化** | 可配置：pickle、JSON、壓縮、自訂 | `Target` 使用 pickle 為基礎 |
| **程式碼變更偵測** | 內建——`TASK_SOURCE` 原則自動將原始碼納入金鑰 | 無——須手動刪除舊目標檔案 |
| **跨任務協調** | 交易（Transaction）、隔離層級、鎖管理器 | 無——各任務獨立寫入目標 |
| **分散式快取** | 內建支援 S3/GCS 等 | 須手動實作共享檔案系統 |

### 核心差異

**Luigi 的 Makefile 哲學**[^luigi-target]：「任務是否產生預期的輸出檔案？若產生，任務完成；若未產生，執行它。」輸出檔案**就是**完成與否的訊號。使用者必須為每個任務定義一個 `output()` 方法回傳 `Target` 物件，框架檢查該目標是否存在。

**Prefect 的現代哲學**[^prefect-caching]：「從任務的參數與上下文計算出一個確定性的快取金鑰。檢查該金鑰是否已有持久化的結果。若有且未過期，跳過執行；若無，執行並持久化。」結果與狀態追蹤分離，允許更豐富的行為。

### 實務影響

1. **程式碼變更處理**：Luigi 無法自動感知任務程式碼已修改——若任務邏輯改變但輸出路徑不變，Luigi 不會重新執行，使用者需手動清理舊目標。Prefect 的 `TASK_SOURCE` 原則可將任務原始碼納入快取金鑰，程式碼變更時**自動失效快取**[^prefect-policies]。

2. **依賴鏈快取**：Luigi 需層層定義 `requires()` 與 `output()` 形成 DAG，每個環節都依賴檔案存在來觸發。Prefect 透過快取金鑰與 Prefect Server/Cloud 的狀態追蹤自動管理依賴，無需手動定義輸出目標。

3. **跨執行快取**：Luigi 的目標檔案預設路徑包含參數，相同參數自然指向相同檔案，但跨機器共享需共用檔案系統。Prefect 可輕鬆指定 S3 作為結果儲存後端，實現原生分散式快取[^prefect-distributed]。

4. **時效性控制**：Luigi 無原生機制讓快取「在 X 時間後自動過期」，Prefect 的 `cache_expiration` 直接解決此需求[^prefect-expiration]。

## 結論

Prefect 並非沒有快取機制，而是將快取從「檔案存在性檢查」抽象為「快取金鑰查詢系統」。這使得 Prefect 在維持純函數式 API 的同時，達成比 Luigi 更靈活、更自動化的任務跳過能力。關鍵在於：

- **Luigi** 將「完成訊號」綁定於**輸出檔案的存在性**，是一種副作用（side effect）驅動的設計
- **Prefect** 將「完成訊號」抽象為**快取金鑰的查詢結果**，快取金鑰可由輸入、原始碼、流程參數等決定性因素組成，無需使用者宣告輸出目標

兩種設計都能實現相同目標（避免重複計算），但 Prefect 的方案在自動化程度、可配置性、分散式支援與程式碼變更處理上更具優勢。

[^luigi-target]: Spotify. (n.d.). Luigi — Targets. Retrieved 2026-09-25, from https://luigi.readthedocs.io/en/stable/api/luigi.target.html
[^prefect-caching]: Prefect Technologies. (n.d.). Task Caching — Prefect Documentation. Retrieved 2026-09-25, from https://docs.prefect.io/v3/develop/write-pipelines/task-caching/
[^prefect-caching-flow]: Prefect Technologies. (n.d.). Caching Concepts — Prefect Documentation. Retrieved 2026-09-25, from https://docs.prefect.io/v3/concepts/caching
[^prefect-policies]: Prefect Technologies. (n.d.). Built-in Cache Policies — Prefect Documentation. Retrieved 2026-09-25, from https://docs.prefect.io/v3/concepts/caching#built-in-cache-policies
[^prefect-compose]: Prefect Technologies. (n.d.). Composing Cache Policies — Prefect Documentation. Retrieved 2026-09-25, from https://docs.prefect.io/v3/concepts/caching#composing-cache-policies
[^prefect-expiration]: Prefect Technologies. (n.d.). Cache Expiration — Prefect Documentation. Retrieved 2026-09-25, from https://docs.prefect.io/v3/concepts/caching#cache-expiration
[^prefect-persist]: Prefect Technologies. (n.d.). Configuring Result Persistence — Prefect Documentation. Retrieved 2026-09-25, from https://docs.prefect.io/v3/develop/write-pipelines/task-caching/#requirements
[^prefect-custom-key]: Prefect Technologies. (n.d.). Custom Cache Key Functions — Prefect Documentation. Retrieved 2026-09-25, from https://docs.prefect.io/v3/develop/write-pipelines/task-caching/#custom-cache-key-function
[^prefect-isolation]: Prefect Technologies. (n.d.). Cache Isolation — Prefect Documentation. Retrieved 2026-09-25, from https://docs.prefect.io/v3/concepts/caching#cache-isolation-levels
[^prefect-distributed]: Prefect Technologies. (n.d.). Distributed Caching — Prefect Documentation. Retrieved 2026-09-25, from https://docs.prefect.io/v3/develop/write-pipelines/task-caching/#distributed-caching