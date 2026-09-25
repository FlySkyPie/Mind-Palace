# C++ 單例模式（Singleton）完整指南：儲存位置、記憶體配置與現代實作

## 問題

在 C++ 中實現單例（Singleton）時，開發者常面臨幾個核心問題：

1. 單例實例究竟應該存放在哪裡？全域變數？靜態區域？堆上？
2. 如何避免靜態初始化順序災難（Static Initialization Order Fiasco）？
3. 多執行緒環境下的執行緒安全如何保證？
4. 有沒有最簡單、最現代的寫法？

本文針對 C++11 以後的現代 C++（含 C++17/20）提供完整解答。

---

## 1. 最推薦做法：Meyers Singleton（C++11+ 執行緒安全）

```cpp
class Singleton {
public:
    static Singleton& getInstance() {
        static Singleton instance;  // 函式區域靜態變數
        return instance;
    }

    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;

private:
    Singleton() = default;
    ~Singleton() = default;
};
```

### 為什麼 C++11 以後是執行緒安全的？

C++11 標準明確規定：

> 「若多個執行緒同時嘗試初始化同一個靜態區域變數，初始化只會發生一次。」[^cppref-static]

編譯器內部通常使用雙重檢查鎖定（Double-Checked Locking）的變體來實作，且初始化完成後的每次呼叫僅需**單次非原子布林比較**（near-zero overhead）。

- 若初始化過程中拋出例外，該變數視為未初始化，下次呼叫會重試
- 若初始化遞迴地重新進入該區塊，則行為未定義

---

## 2. 實例存在哪裡？記憶體區域分析

### Meyers Singleton：資料段（Data Segment）

`static Singleton instance` 是**靜態儲存期（Static Storage Duration）** 變數，存放在程式的**資料段**（`.bss` 或 `.data`），**不在堆上，也不在堆疊上**。

| 實作方式 | 儲存期 | 記憶體區域 | 分配開銷 |
|---------|--------|-----------|---------|
| Meyers（靜態區域變數） | 靜態 | `.bss` / `.data` 資料段 | 無（零開銷） |
| 靜態指標 + `new` | 指標：靜態；物件：動態 | 指標在資料段，物件在**堆**上 | 有堆分配開銷 |
| C++17 `inline static` | 靜態 | 資料段 | 無 |

- `.bss` 段存放零初始化的變數，磁碟上不佔空間，由 OS 載入程式時清零[^cppref-init]
- Meyers Singleton 既無堆分配開銷，也無碎片化風險，OS 在程式結束時自動回收

### 與全域變數的比較

| 特性 | 全域變數 | Meyers Singleton |
|------|---------|-----------------|
| 初始化時機 | 程式啟動時（或動態初始化時） | 首次呼叫 `getInstance()` 時（惰性初始化） |
| 初始化的執行緒安全 | 無保證（C++11 之前） | C++11 保證 |
| 初始化順序問題 | 有（跨編譯單元不定） | 無（因為是函式內靜態變數） |
| 可控性 | 低 | 高 |

---

## 3. 靜態初始化順序災難（Static Initialization Order Fiasco）

### 問題本質

不同編譯單元（`.cpp` 檔案）中的非區域靜態變數，**初始化順序是未定義的**[^isocpp-fiasco]。

```cpp
// file1.cpp
int square(int n) { return n * n; }
auto staticA = square(5);   // 動態初始化

// file2.cpp
extern int staticA;
auto staticB = staticA;      // 可能是 0 或 25，取決於連結順序！
```

`staticB` 的值取決於 `file1.cpp` 和 `file2.cpp` 的**連結順序**——這是最難除錯的 bug 之一。

### 解決方案：Construct On First Use Idiom

ISO C++ FAQ 推薦的做法——將靜態變數移到函式內部，確保首次使用時才初始化[^isocpp-cofu]：

```cpp
// 錯誤：全域靜態變數（有初始化順序問題）
// Fred x;

// 正確：首次使用建構
Fred& x() {
    static Fred ans;  // 呼叫 x() 時才初始化
    return ans;
}
```

Meyers Singleton 正是這個思路的標準應用。

---

## 4. 其他實作模式比較

### a) `std::call_once` + `std::once_flag`

```cpp
class Singleton {
    static std::once_flag initFlag;
    static Singleton* instance;
public:
    static Singleton& getInstance() {
        std::call_once(initFlag, []{ instance = new Singleton(); });
        return *instance;
    }
};
```

- 執行緒安全、顯式控制初始化
- 缺點：堆分配、比 Meyers 稍慢

### b) 靜態指標 + 互斥鎖（最慢）

```cpp
static Singleton& getInstance() {
    std::lock_guard<std::mutex> lock(mutex);
    if (!instance) instance = new Singleton();
    return *instance;
}
```

- 每次呼叫都上鎖，效能極差[^grimm-bench]

### c) 雙重檢查鎖定（DCLP）with Atomics

```cpp
static Singleton* getInstance() {
    Singleton* sin = instance.load(std::memory_order_acquire);
    if (!sin) {
        std::lock_guard<std::mutex> lock(mutex);
        sin = instance.load(std::memory_order_relaxed);
        if (!sin) {
            sin = new Singleton();
            instance.store(sin, std::memory_order_release);
        }
    }
    return sin;
}
```

- C++11 之前不安全（缺乏記憶體序保證）
- C++11 之後可用，但 Meyers Singleton 已經內部實現了類似機制
- **不需要自己寫**

### d) CRTP 泛型 Singleton（Template）

```cpp
template<typename T>
class Singleton {
public:
    static T& getInstance() {
        static T instance;
        return instance;
    }
protected:
    Singleton() = default;
    ~Singleton() = default;
};

class MyService : public Singleton<MyService> {
    friend class Singleton<MyService>;
};
```

- 可重用、減少樣板程式碼
- 需要 `friend` 宣告；部分開發者認為繼承方式違反直覺

### e) Nifty Counter（Schwarz Counter）

- 標準函式庫內部使用（如 `std::cout`、`std::cin`）
- 每個編譯單元有一個計數器，從 0 到 1 時初始化
- 實作複雜、不建議用於使用者程式碼

---

## 5. 效能比較

Rainer Grimm 的基準測試（4000 萬次存取、4 個執行緒）[^grimm-bench]：

| 實作方式 | Linux (GCC) | Windows (MSVC) |
|---------|------------|---------------|
| **Meyers Singleton** | **最快** | **最快** |
| `std::call_once` | ~1.5x 慢 | ~1.5x 慢 |
| 雙重檢查鎖定（Atomics） | ~1.2x 慢 | ~1.2x 慢 |
| 每次呼叫上 Mutex 鎖 | **~10-50x 慢** | **~10-50x 慢** |

Meyers Singleton 不僅**最簡單**，而且**最快**。

---

## 6. 解構順序與清理

### 解構順序

靜態儲存期物件在程式結束時，以**建構完成的反順序**解構[^cppref-exit]。

```cpp
struct A { ~A() { /* 可能存取 B */ } };
struct B { ~B() { /* 可能存取 A */ } };

static A a;  // 先建構 → 後解構
static B b;  // 後建構 → 先解構
```

當 `b` 先解構後，`a` 的解構函式若試圖存取 `b`，就會發生**未定義行為**——這就是靜態解構順序問題。

### 解決方案

1. **保持解構函式簡單**：不要在解構函式中存取其他單例
2. **使用「洩漏式」單例**：堆分配、永不 `delete`，OS 會自動回收，但解構函式不會執行
3. **使用 `std::atexit` / `std::quick_exit`** 進行明確的關閉順序控制

Meyers Singleton 的 `static instance` 會在程式結束時正確解構，是安全性最高的選擇。

---

## 7. 單例是反模式嗎？替代方案

### 爭議

Rainer Grimm 的調查顯示：**59% 使用單例，41% 不使用**[^grimm-pros-cons]。

**反對理由**：
- 隱藏依賴：`Database::getInstance().update(...)` 的呼叫者不知道自己在用資料庫
- 難以測試：無法單元測試，需要整合測試
- 全域狀態：單例是「穿了馬甲的全域變數」
- 違反單一職責原則：同時控制生命週期和商業邏輯

### 改進方案

將單例依賴**明確化**，作為函式參數傳入[^grimm-alternatives]：

```cpp
// 壞——隱藏依賴
void func() {
    DataBase::getInstance().update("something");
}

// 好——明確依賴
void func(DataBase& db) {
    db.update("something");
}
```

### 替代模式

1. **依賴注入（Dependency Injection）**：透過建構子、setter 或模板參數傳入依賴
2. **Monostate 模式（Borg Idiom）**：所有實例共享靜態資料，但使用者不知道是單例：
   ```cpp
   class Monostate {
   public:
       void addEntry(const std::string& name, int val) { data[name] = val; }
   private:
       static std::unordered_map<std::string, int> data;
   };
   Monostate a, b;  // a 和 b 共享同一個 data
   ```
3. **一般物件**：直接傳遞，最簡單也最可測試

---

## 總結

| 問題 | 答案 |
|------|------|
| 單例應存在哪裡？ | 函式內靜態變數（資料段 `.bss`/`.data`） |
| 用全域變數可以嗎？ | 不建議，有初始化順序災難 |
| 執行緒安全嗎？ | C++11 開始，Meyers Singleton 自帶執行緒安全 |
| 最佳實作是什麼？ | Meyers Singleton（函式內 `static` + 回傳參考） |
| 解構呢？ | 自動反序解構，保持解構函式簡單 |
| 什麼時候該避免？ | 需要頻繁測試或依賴不明確時，考慮依賴注入 |

**一句話建議**：使用 Meyers Singleton，回傳參考，刪除拷貝/移動建構子，把單例當參數傳入以保持可測試性。

---

## 參考資料

[^cppref-static]: cppreference.com. (n.d.). *Static local variables*. Retrieved 2026-09-25, from https://en.cppreference.com/w/cpp/language/storage_duration#Static_block_variables

[^cppref-init]: cppreference.com. (n.d.). *Non-local variable initialization*. Retrieved 2026-09-25, from https://en.cppreference.com/w/cpp/language/initialization

[^cppref-exit]: cppreference.com. (n.d.). *std::exit — Destruction order*. Retrieved 2026-09-25, from https://en.cppreference.com/w/cpp/utility/program/exit

[^isocpp-fiasco]: ISO C++ FAQ. (n.d.). *What is the "static initialization order fiasco"?* Retrieved 2026-09-25, from https://isocpp.org/wiki/faq/ctors#static-init-order

[^isocpp-cofu]: ISO C++ FAQ. (n.d.). *Construct On First Use Idiom*. Retrieved 2026-09-25, from https://isocpp.org/wiki/faq/ctors#static-init-order-on-first-use

[^grimm-bench]: Grimm, R. (2023). *Thread-Safe Initialization of a Singleton — Performance Benchmarks*. Modernes C++. Retrieved 2026-09-25, from https://www.modernescpp.com/index.php/thread-safe-initialization-of-a-singleton/

[^grimm-pros-cons]: Grimm, R. (2023). *Singleton — Pros and Cons*. Modernes C++. Retrieved 2026-09-25, from https://www.modernescpp.com/index.php/singleton-pros-and-cons/

[^grimm-alternatives]: Grimm, R. (2023). *Alternatives to the Singleton Pattern*. Modernes C++. Retrieved 2026-09-25, from https://www.modernescpp.com/index.php/the-singleton-the-alternatives/