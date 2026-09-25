# Meyers Singleton 教學

## 簡介

**Meyers Singleton**（又稱 **Meyers' Singleton**）是 C++ 中一種實作 Singleton 設計模式的經典手法，由知名 C++ 專家 **Scott Meyers** 在其著作《Effective C++》第二版（1998）Item 26 中推廣[^meyers-singleton]。

它的核心概念非常簡單：利用**函式區域靜態變數（function-local static variable）**來達成**懶初始化（lazy initialization）**、**執行緒安全（thread safety）**以及**自動解構**的 Singleton。

## 核心程式碼模式

```cpp
class Singleton {
public:
    static Singleton& getInstance() {
        static Singleton instance;  // 關鍵：函式區域靜態變數
        return instance;
    }

private:
    Singleton() = default;                          // 私有建構子
    Singleton(const Singleton&) = delete;           // 不可複製
    Singleton& operator=(const Singleton&) = delete;
    Singleton(Singleton&&) = delete;                // 不可移動
    Singleton& operator=(Singleton&&) = delete;
    ~Singleton() = default;
};
```

### 運作機制

1. **懶初始化**：`static Singleton instance` 只在第一次執行到這行時才被建構，不會在程式啟動時就初始化，從而避免了 C++ 經典的「靜態初始化順序地獄（Static Initialization Order Fiasco）」[^cppref-static-init]。
2. **執行緒安全**：C++11 起，標準保證多個執行緒同時初始化同一個函式區域靜態變數時，只會發生一次初始化。其他執行緒會阻塞直到初始化完成[^cppref-static-block]。
3. **自動解構**：程式結束時，該靜態變數的解構子會自動被呼叫，並按建構的反順序進行。

## 執行緒安全的保證

### C++11 之前

C++ 標準在 C++11 之前**沒有執行緒的概念**，因此 Meyers Singleton **不具備執行緒安全性**。不同執行緒同時呼叫 `getInstance()` 可能導致重複建構或資料競爭。

### C++11 之後

C++11 引入了正式的記憶體模型（memory model），並明確規定[^cppref-static-block]：

> 「如果多個執行緒同時嘗試初始化同一個靜態區域變數，初始化只會發生**恰好一次**。」

實務上，編譯器通常使用雙重檢查鎖定（double-checked locking）的變體來實現此機制，初始化完成後的開銷僅僅是一次非原子的布林值比較[^cppref-static-block]。功能測試巨集 `__cpp_threadsafe_static_init`（值為 `200806L`）可用於檢查編譯器是否支援此功能[^cppref-feature-test]。

## 優點 ✅

| 特性 | 說明 |
|------|------|
| **極簡實作** | 只需約 5 行核心程式碼，不需要手動加鎖或原子變數 |
| **執行緒安全** | 由語言保證，不會有人為錯誤空間 |
| **執行效率高** | 初始化後相當於直接存取靜態變數，比 mutex 版本快得多[^grimm-singleton] |
| **避免靜態初始化順序問題** | 因為是第一次使用時才初始化 |
| **自動記憶體管理** | 無需 `new`/`delete`，沒有堆積碎片問題 |
| **例外安全** | 若建構子拋出例外，變數視為未初始化，下次呼叫會重試 |

## 缺點 ❌

| 特性 | 說明 |
|------|------|
| **無法控制解構順序** | 程式結束時的解構順序難以預測 |
| **難以測試** | Singleton 本質上是全域狀態，不利於單元測試或注入 mock |
| **無法參數化建構** | 無法在建構時傳入執行期參數 |
| **無法重新建立** | 無法在執行期中銷毀並重新建立實例 |
| **遞迴未定義行為** | 若建構子間接呼叫自身 `getInstance()` 會導致未定義行為 |

## 何時使用

- **日誌系統（Logger）**：單一日誌實例
- **設定管理（Configuration Manager）**：應用程式全域設定
- **硬體抽象（Hardware Abstraction）**：單一裝置管理員
- **資源快取（Resource Cache）**：共用的資源池

## 何時避免

- 需要**可測試性**時（建議改用依賴注入）
- 需要在建構時傳入**執行期參數**
- 需要精細控制**生命週期**（在特定時間點建立/銷毀）

## 總結

Meyers Singleton 是 C++ 中實作 Singleton 的首選方式，特別是在 C++11 及之後的標準中，它提供了語言級別的執行緒安全保證。它的簡單性與效率使其成為「如果你**真的**要用 Singleton，這是最好的寫法」。

[^meyers-singleton]: Meyers, S. (1998). Effective C++: 55 Specific Ways to Improve Your Programs and Designs (2nd ed.). Item 26: "Make sure that objects are initialized before they're used." Addison-Wesley.
[^cppref-static-init]: cppreference.com. (n.d.). Initialization. Retrieved 2026-09-25, from https://en.cppreference.com/w/cpp/language/initialization
[^cppref-static-block]: cppreference.com. (n.d.). Storage duration — Static block variables. Retrieved 2026-09-25, from https://en.cppreference.com/w/cpp/language/storage_duration#Static_block_variables
[^cppref-feature-test]: cppreference.com. (n.d.). Feature testing. Retrieved 2026-09-25, from https://en.cppreference.com/w/cpp/feature_test#cpp_threadsafe_static_init
[^grimm-singleton]: Grimm, R. (n.d.). Thread-Safe Initialization of a Singleton. Modernes C++. Retrieved 2026-09-25, from https://www.modernescpp.com/index.php/thread-safe-initialization-of-a-singleton/