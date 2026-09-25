# C++ 非同步非阻塞 SQLite 讀寫支援研究

## 摘要

本報告探討 C++ 環境下 SQLite 資料庫的非同步（Asynchronous）與非阻塞（Non-Blocking）讀寫支援現況。研究發現 SQLite 核心 C API 本質上為同步阻塞設計，並無內建的 async/await 或 callback 機制。然而，SQLite 提供了多種底層工具——WAL 模式、Busy Handler、Unlock-Notify、Async VFS（已棄用）——可供開發者在其上建構非同步模式。社群實務上最常見的做法是「專屬工作執行緒 + 工作佇列」架構，並搭配 WAL 模式以最大化並發效能。

## 1. SQLite 核心 API 的同步本質

SQLite 的 C 語言 API（`sqlite3_step()`、`sqlite3_prepare_v2()`、`sqlite3_exec()` 等）全部是同步阻塞的——呼叫執行緒會一直等待作業完成才返回。SQLite 本身並未提供內建的 async/await、基於回呼（callback）或基於 future 的非同步介面[^sqlite-threadsafe]。

SQLite 支援三種執行緒模式（single-thread、multi-thread、serialized），可透過編譯期 `SQLITE_THREADSAFE` 巨集、啟動期 `sqlite3_config()` 或執行期 `SQLITE_OPEN_NOMUTEX`/`SQLITE_OPEN_FULLMUTEX` 旗標控制。預設為 **serialized** 模式，所有 API 呼叫內部都有 mutex 保護，可安全從多執行緒呼叫[^sqlite-threadsafe]。

## 2. WAL（Write-Ahead Logging）模式

WAL 模式是 SQLite 目前最主要的並發改善機制，其核心概念是將變更寫入獨立的 `-wal` 檔案，而非直接修改主資料庫檔案。

**並發特性**[^sqlite-wal]：

- **讀取不阻塞寫入**：讀取器可以與寫入器同時進行
- **寫入不阻塞讀取**：寫入器不會阻塞讀取器
- **寫入阻塞寫入**：同一時間只能有一個寫入器
- **無限並發讀取器**：支援任意數量的同時讀取

**臨界配置**：

```sql
PRAGMA journal_mode=WAL;         -- 啟用 WAL 模式
PRAGMA synchronous=NORMAL;       -- commit 時不執行 fsync()（略為犧牲持久性）
PRAGMA busy_timeout=5000;        -- BUSY 時自動重試 5 秒
```

WAL 檔案需要定期透過「檢查點（Checkpoint）」折回主資料庫。SQLite 預設在累積 1000 頁後自動觸發檢查點。相關 API 包含 `sqlite3_wal_checkpoint_v2()`、`sqlite3_wal_hook()` 等[^sqlite-wal]。

## 3. Busy Handler：`sqlite3_busy_timeout()` 與 `sqlite3_busy_handler()`

當資料表被鎖定時，SQLite 會返回 `SQLITE_BUSY`。Busy Handler 提供重試機制[^sqlite-busy-handler]。

### `sqlite3_busy_timeout()`

最簡潔的並發輔助函式。設定 SQLite 在遇到 `SQLITE_BUSY` 時自動休眠並重試，直到指定毫秒數過期：

```c
int sqlite3_busy_timeout(sqlite3*, int ms);
```

傳入 `<= 0` 可停用。內部實作基於 `sqlite3_busy_handler()`[^sqlite-busy-timeout]。

### `sqlite3_busy_handler()`

允許應用程式自訂回呼函式，在資料表被鎖定時被呼叫：

```c
int sqlite3_busy_handler(sqlite3*, int(*)(void*, int), void*);
```

回呼函式接收使用者指標與呼叫次數，返回非零代表繼續重試，返回零則放棄並返回 `SQLITE_BUSY`。**注意**：回呼內部不得呼叫任何 `sqlite3_xxx()` 函式（不可重入）[^sqlite-busy-handler]。

## 4. `sqlite3_unlock_notify()`：共享快取的通知機制

此 API 提供基於回呼的等待機制，僅在啟用共享快取模式（Shared-Cache Mode）且編譯時定義了 `SQLITE_ENABLE_UNLOCK_NOTIFY` 時可用[^sqlite-unlock-notify]。

**運作流程**：

1. 連線 A 在共享快取中持有某資料表鎖
2. 連線 B 呼叫 `sqlite3_step()` 得到 `SQLITE_LOCKED`（延伸錯誤碼為 `SQLITE_LOCKED_SHAREDCACHE`）
3. 連線 B 呼叫 `sqlite3_unlock_notify()` 註冊回呼
4. 連線 A 結束交易（commit/rollback）時，SQLite 觸發連線 B 的回呼
5. 回呼應通知等待執行緒（例如透過 condition variable），讓它重試

**標準 pthreads 範例**[^sqlite-unlock-notify-doc]：

```c
struct UnlockNotification {
    int fired;
    pthread_mutex_t mutex;
    pthread_cond_t cond;
};

static void unlock_notify_cb(void **apArg, int nArg) {
    for (int i = 0; i < nArg; i++) {
        UnlockNotification *un = (UnlockNotification*)apArg[i];
        pthread_mutex_lock(&un->mutex);
        un->fired = 1;
        pthread_cond_signal(&un->cond);
        pthread_mutex_unlock(&un->mutex);
    }
}

int wait_for_unlock_notify(sqlite3 *db) {
    UnlockNotification un = {0};
    pthread_mutex_init(&un.mutex, NULL);
    pthread_cond_init(&un.cond, NULL);
    int rc = sqlite3_unlock_notify(db, unlock_notify_cb, (void*)&un);
    if (rc == SQLITE_OK) {
        pthread_mutex_lock(&un.mutex);
        if (!un.fired) pthread_cond_wait(&un.cond, &un.mutex);
        pthread_mutex_unlock(&un.mutex);
    }
    pthread_cond_destroy(&un.cond);
    pthread_mutex_destroy(&un.mutex);
    return rc;
}
```

**限制**：
- 僅適用於共享快取模式（SQLite 團隊在 WAL 出現後已不推薦此模式）
- 回呼不可重入（不得在回呼內呼叫任何 SQLite API）
- 若系統進入死結狀態（A 等 B、B 等 A），則返回 `SQLITE_LOCKED`

## 5. 非同步 I/O VFS（已棄用）

SQLite 曾提供一個非同步 I/O 虛擬檔案系統（Async VFS），源碼位於 `ext/async/sqlite3async.c`。其設計是將所有寫入請求交由背景執行緒處理，寫入資料先進入記憶體佇列，控制權立即返回呼叫者[^sqlite-asyncvfs]。

**核心 API**：

```c
int sqlite3async_initialize(const char *zParent, int isDefault);
void sqlite3async_run();          // 背景執行緒進入點
int sqlite3async_control(int op, ...);
void sqlite3async_shutdown();
```

**代價**：失去 ACID 的 **Durability（持久性）**。若程式在背景執行緒將資料寫入磁碟前崩潰或斷電，變更將遺失。此外寫入佇列可能無限增長。官方文件明確指出：**「WAL 模式很大程度上消除了對此非同步 I/O 模組的需求。因此該模組已不再受支援。」**[^sqlite-asyncvfs]

## 6. C++ 第三方函式庫

### 6.1 DelegateMQ / Async-SQLite

倉庫：[github.com/DelegateMQ/Async-SQLite](https://github.com/DelegateMQ/Async-SQLite)[^async-sqlite]

專為非同步 SQLite 操作設計的 C++ 函式庫，透過專屬工作者執行緒代理所有 SQLite 呼叫。提供三種 API 風格：

| 風格 | 機制 | 說明 |
|---|---|---|
| **同步（阻塞）** | `async::sqlite3_exec()` | 呼叫者阻塞直到完成 |
| **Future（非阻塞）** | `async::sqlite3_exec_future()` | 立即返回 `std::future<int>`，可稍後 `.get()` |
| **Callback（非阻塞）** | Delegate 委派 | 透過回呼傳遞結果 |

```cpp
// Future 風格範例
auto future = async::sqlite3_exec_future(db, sql.c_str(), nullptr, nullptr, nullptr);
// ... 做其他事 ...
int rc = future.get();
```

### 6.2 SQLiteCpp

倉庫：[github.com/SRombauts/SQLiteCpp](https://github.com/SRombauts/SQLiteCpp)[^sqlitecpp]

現代 C++17 RAII 封裝，提供例外處理與 STL 整合。本身不提供非同步機制，但可搭配 `std::async`、`std::jthread` 或工作佇列模式使用。

### 6.3 vsqlite++

網站：[vsqlite.virtuosic-bytes.com](https://vsqlite.virtuosic-bytes.com/)[^vsqlite]

C++20 header-only 函式庫，具備 RAII、連線池、JSON 輔助功能。其連線池模式自然支援非同步使用（從池中借用連線至工作者執行緒）。

### 6.4 其他封裝

- **SQLiteXX**[^sqlitexx]：C++14 物件導向封裝
- **msqlite**[^msqlite]：C++20 函式庫，支援 pipe-operator 鏈式呼叫

## 7. C++ 實務上的非同步模式

由於 SQLite 本身不提供非同步 API，社群已收斂出幾種典型模式：

### 模式 A：專屬工作者執行緒 + 工作佇列（最常見）

所有資料庫操作派發至單一背景執行緒，序列化所有存取，搭配 `std::future` 或回呼將結果傳回呼叫者[^stackoverflow-worker-thread]。

```
主執行緒
    │
    ▼
[執行緒安全工作佇列]  (e.g., moodycamel::ConcurrentQueue)
    │
    ▼
[專屬 SQLite 工作者執行緒]  ← 擁有 sqlite3* 控制代碼
    │
    ▼
  SQLite DB
```

**C++ future 風格範例**：

```cpp
class AsyncSQLite {
    std::thread worker_;
    moodycamel::ConcurrentQueue<std::function<void()>> queue_;
    std::atomic<bool> running_{true};
    sqlite3* db_;

public:
    std::future<int> execAsync(const std::string& sql) {
        auto promise = std::make_shared<std::promise<int>>();
        auto future = promise->get_future();
        queue_.enqueue([this, sql, promise]() {
            char* err = nullptr;
            int rc = sqlite3_exec(db_, sql.c_str(), nullptr, nullptr, &err);
            promise->set_value(rc);
        });
        return future;
    }
};
```

### 模式 B：WAL + Busy Timeout（簡易型）

不需要完整非同步架構時，僅 WAL 模式加上 busy timeout 即可處理大部分並發場景。呼叫執行緒仍然會阻塞，但阻塞時間大幅縮短且有自動重試機制。

### 模式 C：Unlock-Notify + 共享快取（舊型）

適用於傳統的共享快取場景，但 SQLite 團隊已不推薦此方式。

### 模式 D：多連線序列化寫入 + 並發讀取（高效能型）

WAL 模式下可使用多個讀取連線（`SQLITE_OPEN_NOMUTEX`）並發讀取，同時透過單一寫入連線序列化所有寫入請求。

## 8. 綜合建議

| 使用情境 | 推薦方案 | 關鍵工具 |
|---|---|---|
| 簡單單執行緒應用 | WAL + busy_timeout | `PRAGMA journal_mode=WAL; PRAGMA busy_timeout=5000;` |
| 多執行緒需非阻塞 UI | 工作者執行緒 + 佇列 | `sqlite3_open_v2()` with `SQLITE_OPEN_NOMUTEX`、`std::future` |
| 多程序同主機 | WAL 模式 | `PRAGMA journal_mode=WAL; PRAGMA busy_timeout=5000;` |
| C++20 Coroutine 友善 | 工作者佇列 + `std::future` co_await | 自訂 `awaitable` 封裝 `std::future` |
| 最大化寫入吞吐量 | WAL + 批次寫入 + 背景檢查點 | `sqlite3_wal_checkpoint_v2()`、`sqlite3_wal_hook()` |

## 結論

**C++ 中不存在 SQLite 的原生非阻塞讀寫 API。** SQLite 的 C API 本質上為同步設計。達到「非阻塞」效果的最佳實務途徑是：(1) 啟用 WAL 模式減少鎖定衝突時間，(2) 使用專屬工作者執行緒派發所有資料庫操作，(3) 透過 `std::future` 或回呼將結果非同步傳回呼叫者。該模式已廣泛用於生產環境，並有 DelegateMQ/Async-SQLite 等專用 C++ 函式庫提供現成實作。

---

[^sqlite-threadsafe]: SQLite. (n.d.). Thread Safety. Retrieved 2026-09-25, from https://sqlite.org/threadsafe.html
[^sqlite-wal]: SQLite. (n.d.). Write-Ahead Logging. Retrieved 2026-09-25, from https://sqlite.org/wal.html
[^sqlite-busy-handler]: SQLite. (n.d.). Busy Handler. Retrieved 2026-09-25, from https://sqlite.org/c3ref/busy_handler.html
[^sqlite-busy-timeout]: SQLite. (n.d.). Busy Timeout. Retrieved 2026-09-25, from https://sqlite.org/c3ref/busy_timeout.html
[^sqlite-unlock-notify]: SQLite. (n.d.). Unlock Notification. Retrieved 2026-09-25, from https://sqlite.org/c3ref/unlock_notify.html
[^sqlite-unlock-notify-doc]: SQLite. (n.d.). Using the SQLite Unlock Notification Feature. Retrieved 2026-09-25, from https://sqlite.org/unlock_notify.html
[^sqlite-asyncvfs]: SQLite. (n.d.). An Asynchronous I/O Module For SQLite. Retrieved 2026-09-25, from https://sqlite.org/asyncvfs.html
[^async-sqlite]: DelegateMQ. (n.d.). Async-SQLite. Retrieved 2026-09-25, from https://github.com/DelegateMQ/Async-SQLite
[^sqlitecpp]: SRombauts. (n.d.). SQLiteCpp. Retrieved 2026-09-25, from https://github.com/SRombauts/SQLiteCpp
[^vsqlite]: Virtuosic Bytes. (n.d.). vsqlite++. Retrieved 2026-09-25, from https://vsqlite.virtuosic-bytes.com/
[^sqlitexx]: Böhme, M. (n.d.). SQLiteXX. Retrieved 2026-09-25, from https://maxxboehme.github.io/SQLiteXX/
[^msqlite]: Cosme, R. (n.d.). msqlite. Retrieved 2026-09-25, from https://github.com/ricardocosme/msqlite
[^stackoverflow-worker-thread]: Stack Overflow. (n.d.). Best way to write data to SQLite from multiple threads. Retrieved 2026-09-25, from https://stackoverflow.com/questions/39785185/best-way-to-write-data-to-a-sqlite-from-multiple-threads