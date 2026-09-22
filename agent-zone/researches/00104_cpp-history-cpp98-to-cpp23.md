# C++ 標準演化史：從 C++98 到 C++23

## 概述

C++ 語言自 1979 年由 Bjarne Stroustrup 在貝爾實驗室開發的「C with Classes」萌芽以來，歷經四十餘年發展，已成為世界最關鍵的系統程式語言之一。本文追溯其 ISO 標準化後的演化歷程——從 1998 年的第一個國際標準，歷經 2003 年的修訂、2011 年的現代革命，直到最新的 2023 標準。

---

## C++98（ISO/IEC 14882:1998）——第一個 ISO 標準

**發布年份：1998 年**

C++98 是 C++ 的第一個國際標準，正式奠定了語言的核心框架。在此之前，C++ 的規格主要依賴於 1990 年出版的《Annotated C++ Reference Manual》（ARM）作為事實標準[^cppreference-history]。

### 主要新增語言特性

- **RTTI（Runtime Type Identification）**——`dynamic_cast` 與 `typeid` 提供執行期型別識別
- **例外處理**——`try` / `catch` / `throw` 結構化錯誤處理
- **命名空間（namespace）**——避免名稱衝突
- **模板（template）實例化與成員模板**
- **運算子重載（operator overloading）**
- **`bool` 型別、`mutable`、共變返回型別、轉型運算子**
- **條件式中的宣告（declarations in conditions）**

### 主要新增函式庫

- **STL（Standard Template Library）**——容器（`vector`、`list`、`map` 等）、演算法（`sort`、`find` 等）、迭代器、函式物件
- **`bitset`、`valarray`、`auto_ptr`、模板化 `string`**
- **I/O 串流（iostream）、複數（complex）**
- **在地化（locale）支援**

C++98 尚未包含：`auto` 型別推導、range-based for 迴圈、移動語意、lambda、標準函式庫中的智慧指標、以及執行緒支援[^cppreference-cpp98]。

---

## C++03（ISO/IEC 14882:2003）——小幅度修訂

**發布年份：2003 年**

C++03 本質上是技術勘誤（technical corrigendum），未引入任何新語言特性。它修復了 C++98 中的 **92 個核心缺陷報告**與 **125 個函式庫缺陷報告**[^cppreference-history]。

### 主要變更

- **值初始化（value initialization）**的正式定義
- 各項缺陷的修正

C++03 通常與 C++98 合稱為「C++98/03」，兩者差異極小[^cppreference-history]。

---

## C++11（ISO/IEC 14882:2011）——現代革命

**發布年份：2011 年**（原代號「C++0x」，因延遲發布而得名）

C++11 是 C++98 以來最重要的版本更新，被廣泛認為是「Modern C++」的開端。開發耗時 **8 年**，是歷史上間隔最長的一次標準更迭[^cppreference-cpp11]。

### 主要語言特性

- **`auto` 型別推導**——編譯器自動推導型別
- **`decltype`**——取得表達式的型別
- **Range-based `for` 迴圈**——`for (auto& x : vec)` 取代迭代器繁瑣寫法
- **移動語意（Move Semantics）**——右值引用（`&&`）、移動建構子、移動賦值運算子
- **Lambda 表達式**——匿名函式，用於回呼與演算法中
- **`nullptr`**——型別安全的空指標，取代 `NULL`
- **`constexpr`**——編譯期函式求值
- **智慧指標**——`std::unique_ptr`、`std::shared_ptr`、`std::weak_ptr`
- **`enum class`**——限定範圍且強型別的列舉
- **一致初始化（Uniform Initialization）**——`{}` 語法適用於所有型別
- **可變參數模板（Variadic Templates）**——接受任意數量引數的模板
- **`static_assert`**——編譯期斷言
- **`noexcept`** 指定符與運算子
- **`override` 與 `final`** 指定符
- **型別別名（Type Aliases）**——`using` 宣告取代 `typedef`
- **委託建構子（Delegating Constructors）與繼承建構子（Inherited Constructors）**
- **使用者自訂字面值（User-defined Literals）**
- **屬性（Attributes）**——`[[...]]` 語法
- **多執行緒記憶體模型**、執行緒區域儲存（`thread_local`）
- **`long long`、`char16_t`、`char32_t`**

### 主要函式庫新增

- **並行支援**——`std::thread`、`std::mutex`、`std::condition_variable`、`std::future`、`std::promise`、`std::async`
- **新容器**——`std::array`、`std::forward_list`、`std::unordered_map`、`std::unordered_set`
- **`std::tuple`、`std::regex`、`std::chrono`、`std::random`、`std::ratio`**
- **`std::begin()` / `std::end()`** 自由函式

> C++11 根本性地改變了 C++ 的寫法，被稱為「新 C++」[^wikipedia-cpp]。

---

## C++14（ISO/IEC 14882:2014）——打磨與精煉

**發布年份：2014 年**

C++14 是次要修訂——作為「檢查點版本」，修復 C++11 的痛點而未引入顛覆性特性[^cppreference-cpp14]。

### 主要語言特性

- **泛型 Lambda（Generic Lambdas）**——Lambda 參數可以使用 `auto`：`auto add = [](auto a, auto b) { return a + b; };`
- **函式返回型別推導**——編譯器自動推導一般函式的返回型別
- **放寬的 `constexpr`**——`constexpr` 函式中允許更多程式碼（迴圈、條件判斷等）
- **變數模板（Variable Templates）**——不僅型別與函式，變數也可模板化：`template<typename T> constexpr T pi = T(3.14);`
- **Lambda init-capture**——`[x = std::move(obj)]{ ... }` 允許在 capture 中初始化變數
- **二進位字面值（Binary Literals）**——`0b1010` 語法
- **數字分隔符（Digit Separators）**——`1'000'000` 增加可讀性
- **`decltype(auto)`**——`auto` 與 `decltype` 的混合
- **`[[deprecated]]`** 屬性

### 主要函式庫新增

- **`std::make_unique`**——C++11 意外遺漏的重要工廠函式
- **`std::shared_timed_mutex` 與 `std::shared_lock`**——共享鎖定
- **`std::integer_sequence`**
- **`std::exchange`**
- **型別特性（Type Traits）的別名版本**——如 `std::remove_const_t`

> C++14 讓 C++11 更好用——可視為「C++11 完成版」[^cppreference-cpp14]。

---

## C++17（ISO/IEC 14882:2017）——務實擴充

**發布年份：2017 年**

C++17 被描述為 C++11 之後的第一個主要修訂，聚焦於開發者實際需要的日常改良[^cppreference-cpp17]。

### 主要語言特性

- **`if constexpr`**——編譯期條件分支（對模板元程式設計影響深遠）
- **結構化繫結（Structured Bindings）**——`auto [key, value] = my_pair;`
- **折疊表達式（Fold Expressions）**——`(args + ...)` 用於可變參數模板的參數包展開
- **類別模板引數推導（CTAD）**——`std::pair{1, 2.0}` 無需明確指定模板引數
- **內聯變數（Inline Variables）**——`inline constexpr int val = ...;`
- **`constexpr` Lambda**——Lambda 可用於常數表達式
- **保證複製省略（Guaranteed Copy Elision）**——特定情況下的強制 RVO
- **簡化巢狀命名空間**——`namespace A::B::C {}`
- **新屬性**——`[[fallthrough]]`、`[[maybe_unused]]`、`[[nodiscard]]`
- **`__has_include`**——預處理器檢查標頭檔可用性
- **求值順序保證**——特定運算子改為左到右求值

### 主要函式庫新增

- **`std::filesystem`**——跨平臺檔案系統操作
- **`std::optional`**——可能不存在的值
- **`std::variant`**——型別安全的聯合體（union）
- **`std::any`**——型別消除的值容器
- **`std::string_view`**——非擁有權的字串視圖
- **`std::byte`**——明確的位元組型別
- **`<charconv>`**——高效、與在地化無關的數字與字串轉換
- **並行演算法（Parallel Algorithms）**——執行策略（`std::execution::par`、`std::execution::seq`）
- **多型分配器（Polymorphic Allocators）**——`std::pmr::memory_resource`

> C++17 被認為是今日大多數新 C++ 專案的合理最低標準[^cppreference-cpp17]。

---

## C++20（ISO/IEC 14882:2020）——重大擴充

**發布年份：2020 年**

C++20 是僅次於 C++11 的第二重要更新，加入了多項從根本上重塑 C++ 程式碼撰寫方式的大型特性[^cppreference-cpp20]。

### 主要語言特性

- **概念（Concepts）**——對模板引數的約束（`template<Numeric T> T add(T a, T b);`）
- **Ranges**——可組合的範圍演算法，支援管線語法（`numbers | views::filter(...) | views::transform(...)`）
- **協同程式（Coroutines）**——無堆疊協同程式，使用 `co_await`、`co_yield`、`co_return`
- **模組（Modules）**——取代標頭檔的現代方案（`import std;`）
- **三向比較運算子 `<=>`**——「太空船運算子」與預設比較
- **`consteval`**——立即函式，必須在編譯期執行
- **`constinit`**——保證常數初始化但不要求編譯期求值
- **指定初始化（Designated Initializers）**——`.name = value` 語法
- **`std::format`**——型別安全字串格式化（源自 {fmt} 函式庫）
- **`std::span`**——非擁有權的連續資料視圖
- **`char8_t`**——UTF-8 字元型別
- **`[[no_unique_address]]`、`[[likely]]`、`[[unlikely]]`** 屬性
- **Range-for 中的初始化語句**——`for (auto v = vec; auto& e : v)`
- **進一步放寬的 `constexpr`**——允許虛擬函式呼叫、`dynamic_cast`、`try`/`catch` 在 constexpr 中
- **有號整數保證為二補數（2's complement）**

### 主要函式庫新增

- **Ranges 函式庫**——`std::ranges::sort`、views、range adaptors、`std::views`
- **Concepts 函式庫**——`<concepts>` 標頭檔
- **格式化函式庫**——`std::format`、`std::format_to`
- **日曆與時區**——`<chrono>` 擴充
- **執行緒協調類別**——`std::barrier`、`std::latch`、`std::counting_semaphore`
- **`std::jthread`**——協作式執行緒，自動加入並支援停止權杖
- **`std::endian`**——偵測位元組序
- **`std::bit_cast`** 與 `<bit>` 中的二的冪運算
- **數學常數**——`std::numbers::pi`、`std::numbers::e` 等
- **`std::bind_front`、`std::ssize`、`std::midpoint`、`std::lerp`**
- **統一容器刪除**——`std::erase` / `std::erase_if`

> C++20 標誌著 C++ 真正開始與 C++17 產生顯著差異，引入了 Concepts、Coroutines、Ranges 三大特性（有時加上 Modules 合稱「C++20 四大」）[^cppreference-cpp20]。

---

## C++23（ISO/IEC 14882:2023）——最新標準

**發布年份：2023 年**

C++23 是截至目前最新發布的標準。它遵循 C++14 的模式——打磨 C++20 引入的大型特性，並填補標準函式庫中的缺口[^cppreference-cpp23]。

### 主要語言特性

- **Deducing `this`**——顯式物件參數，允許在成員函式中推導 `this` 的型別
- **`if consteval`**——偵測是否在編譯期求值上下文中
- **多維下標運算子**——`v[1, 3, 7] = 42;`
- **`static operator()` 與 `static operator[]`**——靜態呼叫/下標運算子
- **`auto(x)` 與 `auto{x}`**——語言層級的 decay-copy
- **`[[assume(expression)]]`**——最佳化提示屬性
- **`#elifdef` / `#elifndef` / `#warning`**——新的預處理器指令
- **擴充浮點型別**——`std::float16_t`、`std::float32_t`、`std::float64_t`、`std::float128_t`、`std::bfloat16_t`
- **字面值後綴 `z`/`Z`**——代表 `std::size_t`
- **更簡潔的隱式移動（Simpler Implicit Move）**
- **命名通用字元跳脫（Named Universal Character Escapes）**——`"\N{CAT FACE}"`
- **分隔跳脫序列（Delimited Escape Sequences）**——`"\x{C0DE}"`
- **進一步放寬的 `constexpr`**——允許 non-literal 變數、標籤、`goto`、`static` 與 `thread_local`

### 主要函式庫新增

- **`std::expected`**——無需例外機制的錯誤處理（回報預期/非預期結果的詞彙型別）
- **`std::print` / `std::println`**——簡單、型別安全的輸出（在多數情況下取代 `std::cout`）
- **`std::mdspan`**——非擁有權的多維陣列視圖
- **`std::flat_map` / `std::flat_set`**——排序扁平容器配接器
- **`std::generator`**——用於 ranges 的同步協同程式生成器
- **`std::stacktrace`**——堆疊追蹤
- **`std::move_only_function`**——僅可移動的可呼叫包裝器
- **`std::bind_back`**——反向引數綁定
- **`std::out_ptr` / `std::inout_ptr`**——用於 C 語言互操作的智慧指標配接器
- **`std::to_underlying`**——取得列舉的底層值
- **`std::unreachable`**——標記不可到達的程式碼路徑
- **`std::optional` 與 `std::expected` 的單子操作**——`.transform()`、`.or_else()`、`.and_then()`
- **新的 range adaptors**——`views::zip`、`views::adjacent`、`views::cartesian_product`、`views::chunk`、`views::slide`、`views::stride`、`views::enumerate`、`views::repeat`、`views::as_rvalue`、`views::as_const` 等
- **`ranges::to`**——將 range 轉換為容器
- **`ranges::fold_left`** 等 fold 演算法
- **`import std;`**——以模組形式匯入整個標準函式庫
- **`std::bitset`、`std::unique_ptr`、`std::to_chars`/`std::from_chars` 的 `constexpr` 支援**

> C++23 仍在逐步獲得編譯器支援。GCC 13+、Clang 16+、MSVC 19.36+ 支援大部分特性[^cppreference-cpp23]。

---

## 時間線總覽

| 標準 | 年份 | ISO 編號 | 主題 |
|---|---|---|---|
| **C++98** | 1998 | ISO/IEC 14882:1998 | 第一個 ISO 標準，語言基礎奠定 |
| **C++03** | 2003 | ISO/IEC 14882:2003 | 小幅度錯誤修正，值初始化 |
| **C++11** | 2011 | ISO/IEC 14882:2011 | **現代革命**：auto、移動語意、lambda、執行緒 |
| **C++14** | 2014 | ISO/IEC 14882:2014 | 打磨：泛型 lambda、放寬 constexpr、`std::make_unique` |
| **C++17** | 2017 | ISO/IEC 14882:2017 | 務實：`if constexpr`、filesystem、結構化繫結 |
| **C++20** | 2020 | ISO/IEC 14882:2020 | **重大擴充**：concepts、coroutines、ranges、modules |
| **C++23** | 2023 | ISO/IEC 14882:2023 | 最新：deducing `this`、`std::expected`、`std::print` |

自 C++11 以來，標準化週期穩定為 **3 年一版**（C++14、C++17、C++20、C++23，C++26 預計 2026 年發布）。每個標準皆為累積性——C++23 包含 C++20 的全部內容，C++20 包含 C++17，以此類推。

---

## 演化趨勢與觀察

1. **從沈寂到爆發**：C++98 到 C++11 之間間隔 13 年，而 C++11 之後確立了 3 年的節奏。

2. **配對模式**：每兩個版本形成一組——重大版本（C++11、C++17、C++23）引入大型特性，隨後的小版本（C++14、C++20、C++26）進行打磨與補充。C++98/03 也遵循此模式。

3. **函式庫先於語言**：許多特性先在 Boost 或 TS（Technical Specification）中驗證，再納入標準——例如 TR1 → C++11，Filesystem TS → C++17，Ranges TS → C++20。

4. **編譯期運算能力持續增強**：從 C++11 的 `constexpr`、C++14 的放寬、C++17 的 `if constexpr`、C++20 的 `consteval`/`constinit`，到 C++23 的 `if consteval`，C++ 的編譯期程式設計能力不斷擴展。

5. **安全性與表達力提升**：從 `auto_ptr` 到 `unique_ptr`，從 `enum` 到 `enum class`，從 `NULL` 到 `nullptr`，從例外到 `std::expected`——C++ 持續朝向更安全、更具表達力的方向演進。

---

## 參考資料

[^cppreference-history]: cppreference.com. (n.d.). *History of C++*. Retrieved 2026-09-20, from https://en.cppreference.com/w/cpp/language/history

[^cppreference-cpp98]: cppreference.com. (n.d.). *C++98*. Retrieved 2026-09-20, from https://en.cppreference.com/w/cpp/98

[^cppreference-cpp11]: cppreference.com. (n.d.). *C++11*. Retrieved 2026-09-20, from https://en.cppreference.com/w/cpp/11

[^cppreference-cpp14]: cppreference.com. (n.d.). *C++14*. Retrieved 2026-09-20, from https://en.cppreference.com/w/cpp/14

[^cppreference-cpp17]: cppreference.com. (n.d.). *C++17*. Retrieved 2026-09-20, from https://en.cppreference.com/w/cpp/17

[^cppreference-cpp20]: cppreference.com. (n.d.). *C++20*. Retrieved 2026-09-20, from https://en.cppreference.com/w/cpp/20

[^cppreference-cpp23]: cppreference.com. (n.d.). *C++23*. Retrieved 2026-09-20, from https://en.cppreference.com/w/cpp/23

[^wikipedia-cpp]: Wikipedia contributors. (n.d.). *C++*. In Wikipedia, The Free Encyclopedia. Retrieved 2026-09-20, from https://en.wikipedia.org/wiki/C%2B%2B