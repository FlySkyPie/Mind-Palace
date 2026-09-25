# Monostate 模式（Borg Idiom）實現單例時的資源釋放問題

## 問題定義

Monostate 模式（又稱 Borg Idiom）透過讓所有實例共享同一份狀態來實現「概念上的單例」。但與經典 Singleton 不同，Monostate 允許多個實例存在，這帶來了資源釋放的模糊性：當多個實例共享同一個連線、檔案控制代碼或資料庫連線時，誰該負責清理？何時清理？

## Monostate 與 Singleton 的差異

| 面向 | Singleton | Monostate / Borg |
|---|---|---|
| 實例數量 | 恰好一個 | 允許多個實例 |
| 實例身分 | `a is b` → `True` | `a is b` → `False` |
| 狀態共享 | 狀態綁在單一物件上 | 透過共享 `__dict__`（Python）或靜態欄位（Java）共享 |
| 生命週期 | 單一實例，需特殊處理 | 每個實例有正常生命週期 |
| 子類化 | 困難；子類容易意外的共享父類實例 | 較容易；子類可獨立覆寫 `_shared_state` |
| 測試性 | 難以 mock／重置 | 較容易；可在測試間清除共享狀態 |

## 核心困難：共享狀態的所有權模糊

Monostate 的核心悖論在於：**沒有一個實例真正「擁有」共享資源**。Borg 模式將每個實例的 `__dict__` 指向同一個類別層級的字典，因此任一個實例被 GC 回收時，共享狀態中的資源仍然存在[^faif]。

> 「Borg 模式提供了一個優雅的狀態共享方案……但無論是模式的典型實作還是常見討論，都幾乎不觸及資源清理問題，因為共享狀態的生命週期本來就是無界的。」—— faif/python-patterns（隱含意涵）[^faif]

## 資源釋放的五種策略（依推薦程度排列）

### 1. 🥇 明確的 `cleanup()` / `shutdown()` 類別方法（最推薦）

最可靠的作法：提供一個**冪等（idempotent）**的類別方法來統一清理共享資源。

```python
class DatabasePool:
    _shared_state = {"connection": None, "pool": None}

    def __init__(self):
        self.__dict__ = self._shared_state
        if self._shared_state["pool"] is None:
            self._shared_state["pool"] = self._create_pool()

    @classmethod
    def shutdown(cls):
        """明確關閉所有共享資源。冪等（可安全重複呼叫）。"""
        conn = cls._shared_state.get("connection")
        if conn:
            conn.close()
            cls._shared_state["connection"] = None
        pool = cls._shared_state.get("pool")
        if pool:
            pool.close()
            cls._shared_state["pool"] = None
```

**優點**：確定性（deterministic）、跨語言通用、可測試。**缺點**：呼叫端必須記得呼叫；可搭配 `atexit` 或框架生命週期鉤子註冊。

---

### 2. 🥈 情境管理器（Context Manager）— Python 最佳實踐

透過 `with` 陳述式提供確定性的進入與離開處理：

```python
class BorgResource:
    _shared_state = {}

    def __init__(self):
        self.__dict__ = self._shared_state

    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        # 在這裡清理共享狀態
        if "connection" in self._shared_state:
            self._shared_state["connection"].close()
            self._shared_state["connection"] = None
```

此作法在 Python 社群中獲得最廣泛共識[^devgex]：

> 「優先使用 Context Manager：對於需要資源管理的物件，永遠實作 `__enter__` 和 `__exit__` 方法。」—— DevGex

**優點**：清理確定性高、例外安全（即使發生例外也會執行 `__exit__`）、所有權語義清晰（`with` 區塊定義了作用域）。

---

### 3. ✅ `atexit` 註冊 — 作為安全網

對於應該存活於整個行程生命週期的資源：

```python
import atexit

class BorgWithCleanup:
    _shared_state = {}

    def __init__(self):
        self.__dict__ = self._shared_state
        if not hasattr(type(self), '_cleanup_registered'):
            atexit.register(self._final_cleanup)
            type(self)._cleanup_registered = True

    @classmethod
    def _final_cleanup(cls):
        """於直譯器關閉時執行"""
        if "connection" in cls._shared_state:
            cls._shared_state["connection"].close()
```

> 「僅將 `atexit` 用於綁定於整個行程生命週期的資源。」—— UniversoPython[^universopython]

**注意**：此方法僅在正常關機時執行；`SIGKILL`、`os._exit()`、或重大內部錯誤時不會執行[^universopython]。最穩健的作法是把 `atexit` 當作**安全網**，而非主要的清理機制，與明確的 `cleanup()` 方法搭配使用。

---

### 4. ⚠️ `weakref.finalize` — 物件生命週期繫結（進階）

作為 `__del__` 的安全替代方案：

```python
import weakref

def _cleanup_connection(conn_id):
    """外部函式——不持有對 Borg 實例的參照"""
    print(f"Closing connection {conn_id}")

class BorgConnection:
    _shared_state = {"connection": None}

    def __init__(self):
        self.__dict__ = self._shared_state
        if self._shared_state["connection"] is None:
            self._shared_state["connection"] = self._open()
        self._finalizer = weakref.finalize(
            self, _cleanup_connection, id(self._shared_state["connection"])
        )
```

> 「不同於 `object.__del__`，`weakref.finalize` 保證在直譯器關閉時被呼叫，因為直譯器會特別追蹤它們。」—— Stack Overflow[^so_finalize]

**關鍵陷阱**：回呼函式**不得持有對被 finalize 之物件的強參照**，否則永遠不會被 GC 回收[^weakref_docs]。

> 「確保 func、args 和 kwargs 不以直接或間接方式擁有任何對 obj 的參照，否則 obj 永遠不會被垃圾回收。」—— Python 官方文件[^weakref_docs]

---

### 5. ❌ `__del__`（解構子）— 強烈不建議

這是風險最高的作法，多個生產事故文件中都有記載：

> 「一個團隊使用了帶有 `__del__` finalizer 的 Singleton 連線池，但因循環參照導致 finalizer 從未執行，每小時洩漏 500 條連線。」—— TheCodeForge（生產事故檢討）[^thecodeforge]

> **「規則：永遠不要依賴 `__del__` 來清理 Singleton 的資源——請使用明確的 `close()` 和 Context Manager，否則連線池會默默洩漏。」**—— TheCodeForge[^thecodeforge]

`__del__` 在 Monostate 中的具體問題：
- **執行順序無法保證**——全域變數／模組可能已部分關閉
- `__del__` 中的例外被**默默忽略**（僅輸出一條 stderr 警告）
- **循環參照**會導致 `__del__` 完全不執行
- 多個實例各自觸發 `__del__`，導致雙重釋放（double-free）錯誤

## 總結建議

| 使用情境 | 推薦作法 |
|---|---|
| 短期、有明確作用域的資源（如暫存檔） | **Context Manager**（`with` 區塊） |
| 長期、行程級資源（如設定、快取） | **模組層級狀態** + `atexit` 作為最終安全網 |
| 測試 | **明確的 `reset()` 方法**，清除 `_shared_state` |
| 物件生命週期繫結的清理（最後安全網） | **`weakref.finalize`**（使用靜態回呼，不持有 self 參照） |
| **主要資源清理** | **永遠不用 `__del__`**——使用 Context Manager 或明確的 `close()` |

### 最穩健的組合作法

將上述策略組合使用，形成多層防護：

```python
import atexit

class RobustBorg:
    _shared_state = {}

    def __init__(self):
        self.__dict__ = self._shared_state
        # 註冊 atexit 作為安全網（僅一次）
        if not hasattr(type(self), '_cleanup_registered'):
            atexit.register(type(self).cleanup)
            type(self)._cleanup_registered = True

    @classmethod
    def cleanup(cls):
        """明確的類別方法清理。冪等。"""
        conn = cls._shared_state.get("conn")
        if conn:
            conn.close()
            cls._shared_state["conn"] = None

    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        """Context Manager 支援"""
        type(self).cleanup()
```

這個模式適用於絕大多數需要 Monostate 且需要資源清理的真實場景。

---

## 參考文獻

[^faif]: faif. (n.d.). python-patterns: borg.py. GitHub. Retrieved 2026-09-25, from https://github.com/faif/python-patterns/blob/master/patterns/creational/borg.py

[^devgex]: DevGex. (n.d.). Proper Python Object Cleanup: From `__del__` to Context Managers. Retrieved 2026-09-25, from https://devgex.com/en/article/00006351

[^universopython]: UniversoPython. (n.d.). Python `atexit`: Register Cleanup Functions. Retrieved 2026-09-25, from https://universopython.com/en/blog/python-atexit-cleanup

[^so_finalize]: Stack Overflow. (n.d.). How do I correctly clean up a Python object? Retrieved 2026-09-25, from https://stackoverflow.com/questions/865115/how-do-i-correctly-clean-up-a-python-object

[^weakref_docs]: Python Documentation. (n.d.). weakref — Weak references: Comparing finalizers with `__del__()` methods. Retrieved 2026-09-25, from https://runebook.dev/en/docs/python/library/weakref/comparing-finalizers-with-del-methods

[^thecodeforge]: TheCodeForge. (n.d.). Python Design Patterns — Singleton Connection Pool Leak. Retrieved 2026-09-25, from https://thecodeforge.io/python/python-design-patterns/

[^py4u]: py4u. (n.d.). Singleton vs. Borg: Python Design Pattern Alternatives. Retrieved 2026-09-25, from https://www.py4u.org/python-design-patterns/singleton-vs-borg-python-design-pattern-alternatives/

[^python3_info]: python3.info. (n.d.). Borg Pattern. Retrieved 2026-09-25, from https://python3.info/design-patterns/creational/borg.html

[^deepwiki]: DeepWiki. (n.d.). faif/python-patterns: Borg Pattern. Retrieved 2026-09-25, from https://deepwiki.com/faif/python-patterns/2.1.5-borg-pattern

[^c2_wiki]: Cunningham & Cunningham, Inc. (n.d.). Monostate Pattern. WikiWikiWeb. Retrieved 2026-09-25, from https://wiki.c2.com/?MonostatePattern