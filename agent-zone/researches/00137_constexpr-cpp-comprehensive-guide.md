# C++ `constexpr` 完整解析

## 什麼是 `constexpr`？

`constexpr`（constant expression 的縮寫）是 C++11 引入的關鍵字，用來表示某個**變數、函式或建構子可以在編譯期求值**，成為常數表達式（constant expression）的一部分[^cppref-constexpr]。

> `constexpr` 修飾詞宣告了「在編譯期對該實體求值是**可能**的」，而非強制要求。`constexpr` 函式同時也能在執行期以執行期參數呼叫。

[^cppref-constexpr]: cppreference. (n.d.). `constexpr` specifier (since C++11). Retrieved 2026-09-25, from https://en.cppreference.com/cpp/language/constexpr

---

## 解決的問題

| 問題 | `constexpr` 如何解決 |
|---|---|
| **模板元編程晦澀難懂** — 編譯期計算需要遞迴模板（`std::integral_constant`），語法痛苦 | `constexpr` 函式寫起來像一般程式碼（有迴圈、有變數），但執行在編譯期 |
| **缺乏編譯期函式** — 巨集（`#define SQUARE(x) ((x)*(x))`）無型別安全，容易雙重求值出錯 | `constexpr` 函式有型別安全、有作用域、可除錯 |
| **魔術數字充斥** — 陣列大小、模板參數需要寫死數值或複雜模板技巧 | `constexpr` 變數與函式產生的值可用於任何常數表達式語境 |
| **執行期開銷** — 本可在編譯期完成的工作卻在執行期重複計算 | 計算移到建置期 → 執行期零成本 |
| **靜態初始化順序問題** — 跨編譯單元的全域變數動態初始化順序不確定 | `constexpr`/`constinit` 保證編譯期初始化，完全避開此問題 |

**核心概念**：`constexpr` 將編譯期計算的能力帶入主流 C++ 語法，無需模板元編程的技巧即可使用[^modernescpp-intro]。

[^modernescpp-intro]: Grimm, R. (2020). C++20: consteval and constinit. Retrieved 2026-09-25, from https://www.modernescpp.com/index.php/c-20-consteval-and-constinit/

---

## `constexpr` vs `const`

| 面向 | `const` | `constexpr` |
|---|---|---|
| **初始化時機** | 可延遲至執行期 | 必須在編譯期完成 |
| **不可變性** | 是 — 初始化後不可修改 | 是 — 且隱含 `const` |
| **編譯期保證** | 無 — `const int x = rand()` 合法 | 有 — 初始值必須是常數表達式 |
| **用於函式** | 僅限成員函式（「不修改 `this`」之意） | 任意函式（「可在編譯期求值」之意） |
| **用於變數** | 唯讀值，可能是執行期 | 編譯期常數，可用於模板參數、陣列大小等 |
| **整數常數** | `const int x = 5;` 在某些場合可用於常數表達式（特殊規則） | `constexpr double x = 3.14;` 對任何字面型別都成立 |
| **範例** | `const int x = std::rand();` ✅ | `constexpr int x = std::rand();` ❌ |

簡言之：**所有 `constexpr` 變數都是 `const`，但不是所有 `const` 變數都是 `constexpr`**[^ms-constexpr]。

[^ms-constexpr]: Microsoft. (n.d.). constexpr (C++). Retrieved 2026-09-25, from https://learn.microsoft.com/en-us/cpp/cpp/constexpr-cpp

---

## `constexpr` 跨標準版本的演進

### C++11 — 誕生

極度嚴格，`constexpr` 函式幾乎像另一種語言：

- 函式本體**只能有一條 `return` 語句**
- 不能有區域變數、不能有迴圈、不能有 `if`/`else`
- 遞迴是唯一的迭代方式
- 非靜態成員函式隱含 `const`
- 只能操作字面型別（literal type）
- 不允許 `constexpr` 解構子

```cpp
// C++11 constexpr — 只能一條 return
constexpr int factorial(int n) {
    return n <= 1 ? 1 : (n * factorial(n - 1));
}
```

### C++14 — 大幅放寬（Relaxed constexpr）

重大擴展，`constexpr` 函式終於可以寫得像一般函式：

- 允許**多條 `return` 語句**
- 允許**區域變數**（必須以常數表達式初始化）
- 允許**迴圈**（`for`、`while`、`do-while`）
- 允許**條件判斷**（`if`、`else`、`switch`）
- 不再強制使用 `? :` 三元運算子
- 非 `const` 成員函式可行（`constexpr` 不再隱含 `const`）
- 變數模板（variable template）可為 `constexpr`

```cpp
// C++14 constexpr — 寫起來像一般程式碼！
constexpr int factorial(int n) {
    int res = 1;
    while (n > 1)
        res *= n--;
    return res;
}
```

### C++17 — 工具箱擴充

- **`if constexpr`** — 編譯期條件分支，未選取的分支直接被丟棄（徹底改變了模板元編程）
- **`constexpr` lambda** — lambda 可在編譯期求值
- `constexpr` 函式隱含 `inline`；靜態資料成員亦同
- 以值捕獲（capture by value）`constexpr` 物件

```cpp
// C++17: if constexpr — false 分支在編譯期直接被丟棄
template<typename T>
auto get_value(T x) {
    if constexpr (std::is_pointer_v<T>) {
        return *x;  // 只有 T 是指標時才編譯
    } else {
        return x;   // 只有 T 不是指標時才編譯
    }
}

// C++17: constexpr lambda
constexpr auto square = [](int n) { return n * n; };
static_assert(square(5) == 25);
```

### C++20 — 重大突破

迄今為止最顯著的擴展[^cppstories-constexpr-new][^cppstories-constexpr-vector]：

- **`constexpr` 虛擬函式** — 編譯期多型
- **`constexpr` 動態配置記憶體** — `new`/`delete` 可在 `constexpr` 函式中使用（暫時性配置，transient allocation）
- **`constexpr` `std::vector` 與 `std::string`** — 完整的容器編譯期支援
- **`consteval`** — **立即函式（immediate function）**，**必須**在編譯期執行
- **`constinit`** — 保證靜態/執行緒區域變數在編譯期初始化（但變數本身可變）
- **`std::is_constant_evaluated()`** — 偵測目前是否在常數求值語境中
- 可在常數求值中改變聯集（union）的作用中成員
- `try`/`catch` 區塊可用（但 `throw` 在常數求值中仍被禁止）
- 超過 100 個 STL 演算法變成 `constexpr`

```cpp
// C++20: constexpr 動態配置（暫時性）
constexpr int sum_up_to(int n) {
    auto p = new int[n];             // 編譯期配置
    for (int i = 0; i < n; ++i)
        p[i] = i + 1;
    int sum = 0;
    for (int i = 0; i < n; ++i)
        sum += p[i];
    delete[] p;                       // 必須在結束前釋放！
    return sum;
}
static_assert(sum_up_to(10) == 55);

// C++20: constexpr std::vector
constexpr int max_element() {
    std::vector<int> v = {1, 2, 4, 3};
    std::sort(v.begin(), v.end());
    return v.back();
}
static_assert(max_element() == 4);

// C++20: constexpr 虛擬函式（編譯期多型）
struct Base {
    constexpr virtual int get() const { return 1; }
};
struct Derived : Base {
    constexpr int get() const override { return 2; }
};
constexpr int test() {
    const Derived d;
    const Base& b = d;
    return b.get();  // 編譯期的虛擬函式分派！
}
static_assert(test() == 2);

// C++20: consteval — 立即函式，強制編譯期執行
consteval int sqr(int n) { return n * n; }
constexpr int x = sqr(5);  // ✅
// int y = 5;
// int z = sqr(y);         // ❌ y 不是常數表達式

// C++20: constinit — 編譯期初始化但非 const
constinit int global = 42; // 編譯期初始化
global = 100;              // ✅ 可以修改
```

[^cppstories-constexpr-new]: C++ Stories. (2021). constexpr dynamic memory allocation in C++20. Retrieved 2026-09-25, from https://www.cppstories.com/2021/constexpr-new-cpp20/
[^cppstories-constexpr-vector]: C++ Stories. (2022). constexpr vector and string in C++20. Retrieved 2026-09-25, from https://www.cppstories.com/2022/const-options-cpp20/

### C++23 — 進一步放寬

- `constexpr` 函式中可使用**非字面型別**（只要位於不被求值（non-evaluated）的分支中）
- `constexpr` 函式中可使用 `static`/`thread_local` 變數
- 允許 `goto` 與標籤（但跨越生命週期邊界的 `goto` 仍禁止）
- 回傳型別與參數型別不再需要是字面型別
- 顯式默認（explicitly defaulted）的函式若滿足條件，在其首次宣告時隱含 `constexpr`
- **`constexpr std::unique_ptr`** — 智慧指標在編譯期可用
- **`if consteval`** — 語言層級的常數求值語境偵測
- `constexpr` 對 `<cmath>` 與 `<cstdlib>` 的部分支援
- `constexpr std::bitset` 完整 API
- `constexpr std::optional`、`std::variant`
- `constexpr` 整數版本的 `std::to_chars`/`std::from_chars`[^sandordargo-cpp23]

[^sandordargo-cpp23]: Dargo, S. (2023). C++23 constexpr. Retrieved 2026-09-25, from https://www.sandordargo.com/blog/2023/05/24/cpp23-constexpr

### C++26（即將到來）

- `constexpr` `void*` 轉型 — 朝向編譯期型別消除
- `constexpr` 虛擬繼承
- 建構子/解構子的進一步放寬

---

## `const` 家族關鍵字比較（C++20）

| 關鍵字 | 用於變數 | 用於函式 | 編譯期保證 | 是否為 const |
|---|---|---|---|---|
| `const` | ✅（任意作用域、任意生命週期） | ✅（僅成員函式） | ❌（可能是執行期） | 是 |
| `constexpr` | ✅（必須是字面型別） | ✅（可在編譯期或執行期執行） | 「可能」在編譯期 | 是 |
| `consteval` | ❌ | ✅（立即函式，**必須**執行在編譯期） | 永遠 | 不適用（函式） |
| `constinit` | ✅（僅 static/thread_local） | ❌ | 初始化保證在編譯期，但值可變 | 否 |

---

## 實用範例

### 編譯期查詢表

```cpp
constexpr std::array<int, 10> build_squares() {
    std::array<int, 10> arr{};
    for (int i = 0; i < 10; ++i)
        arr[i] = i * i;
    return arr;
}
constexpr auto squares = build_squares();  // 編譯期計算
static_assert(squares[5] == 25);
```

### `if constexpr` 型別分派

```cpp
template<typename T>
constexpr std::string_view type_name() {
    if constexpr (std::is_same_v<T, int>) return "int";
    else if constexpr (std::is_same_v<T, double>) return "double";
    else if constexpr (std::is_same_v<T, std::string>) return "string";
    else return "unknown";
}
static_assert(type_name<int>() == "int");
```

### 編譯期字串雜湊

```cpp
constexpr unsigned fnv1a(const char* s, size_t n) {
    unsigned hash = 2166136261u;
    for (size_t i = 0; i < n; ++i) {
        hash ^= static_cast<unsigned>(s[i]);
        hash *= 16777619u;
    }
    return hash;
}
constexpr unsigned hash_hello = fnv1a("hello", 5);
```

### `std::is_constant_evaluated()` — 雙重路徑最佳化

```cpp
constexpr double power(double base, int exp) {
    if (std::is_constant_evaluated()) {
        // 編譯期路徑：可用 constexpr 專屬功能
        double result = 1.0;
        for (int i = 0; i < exp; ++i) result *= base;
        return result;
    } else {
        // 執行期路徑：可用 std::pow 等
        return std::pow(base, exp);
    }
}
```

---

## 限制與何時不該使用

### 主要限制

| 限制 | 說明 |
|---|---|
| **暫時性配置（transient allocation）**（C++20） | 在 `constexpr` 語境中配置的記憶體**必須在同一求值結束前釋放** |
| **無 I/O** | `std::cout`、檔案 I/O、網路操作不能在編譯期執行 |
| **無 `reinterpret_cast`** | 指標重新解釋在常數表達式中被禁止 |
| **無未定義行為** | 有號整數溢位、越界存取等會在編譯期被捕獲 → 編譯錯誤 |
| **無 `std::shared_ptr`** | 缺乏編譯期的原子操作支援 |
| **編譯時間增加** | 大量使用 `constexpr` 進行大型計算會顯著拖慢編譯 |
| **除錯困難** | 編譯期執行的程式碼對除錯器不可見 |

### 何時不該使用 `constexpr`

1. **純執行期值** — 依賴使用者輸入、系統時鐘、網路等，`constexpr` 不可能
2. **大型編譯期計算** — 若計算耗時數秒且執行期執行完全沒問題，編譯期成本可能不值得
3. **複雜 I/O 邏輯** — 任何涉及檔案、網路、系統呼叫的函式
4. **需要執行期多型** — `dynamic_cast` 與帶多型型別的 `typeid` 仍受限
5. **過早最佳化** — 到處加 `constexpr`「以防萬一」只會增加編譯時間而無實質效益。應在需要時才使用（如 `static_assert`、模板參數）
6. **刻意使用副作用** — 若函式故意記錄日誌、變動全域狀態或依賴求值順序，`constexpr` 可能會強制不想要的編譯期求值
7. **需要 `std::shared_ptr`** — 即使在 C++23 仍不支援 `constexpr`

---

## 總結

`constexpr` 從 C++11 極度受限的單行 return 函式，發展到 C++20 支援虛擬函式、動態配置記憶體、`std::vector`/`std::string` 的完整編譯期程式設計工具，是現代 C++ 中最重要的語言特性之一。搭配 `consteval`（強制編譯期執行）與 `constinit`（保證編譯期初始化但不鎖定值），C++20 提供了一套完整的編譯期計算生態系統[^modernescpp-consteval]。

[^modernescpp-consteval]: Grimm, R. (2020). constexpr and consteval functions in C++20. Retrieved 2026-09-25, from https://www.modernescpp.com/index.php/constexpr-and-consteval-functions-in-c-20/