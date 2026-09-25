# C++ 動態函式庫單例模式：資源初始化與釋放完整指南

## 問題場景

C++ 單例（Singleton）模式在多數教學中都以`getInstance()`靜態方法展示，但這些示範往往忽略一個關鍵前提：**當單例存在於動態函式庫（.so / .dll）中時**，資源的初始化與釋放不再是程式結束時由 C++ 執行時期自動處理那麼單純。

典型情境：

- 單例持有資料庫連線、執行緒池、日誌檔案控制代碼等需要顯式釋放的資源。
- 插件系統透過 `dlopen()` / `dlclose()` 載入與卸載函式庫。
- 主程式要求動態函式庫在卸載時不能留下懸置資源或記憶體洩漏。
- 同一個動態函式庫可能被重複載入與卸載（熱重載 hot-reload）。

本文針對這些場景，分析各類單例實作在動態函式庫環境下的行為差異，並給出具可行性的解決方案。

---

## 1. Meyers Singleton 在動態函式庫中的問題

### 標準實作

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
    Singleton()  = default;
    ~Singleton() = default;
};
```

這是 Scott Meyers 在《Effective C++》中推廣的模式，C++11 起編譯器保證函式區域靜態變數的初始化是執行緒安全的[^meyers-singleton]。

### 在 .so 中遭遇的三個問題

#### 問題一：STB_GNU_UNIQUE 符號阻止函式庫卸載

GCC/Clang 會將 inline 函式與 template 函式中的靜態變數標記為 `STB_GNU_UNIQUE`。glibc 遇到此類符號時，會自動為函式庫設定 `DF_1_NODELETE` 旗標，使 `dlclose()` **永遠無法卸載該函式庫**[^gnu-unique]。

```cpp
inline Logger& getLogger() {
    static Logger instance;  // ← 產生 STB_GNU_UNIQUE 符號
    return instance;
}
```

使用 `readelf` 可以檢查：

```bash
readelf -sW plugin.so | grep UNIQUE
```

#### 問題二：靜態解構順序地獄（Static Deinitialization Order Fiasco）

Meyers Singleton 的解構子在程式結束（或函式庫卸載）時由 `__cxa_atexit` 註冊的回呼自動呼叫。問題在於[^siof-cppreference]：

- 不同函式庫中的靜態物件解構順序未定義。
- 若函式庫 A 在函式庫 B 之後卸載，但 A 的單例在 B 的某個物件解構時仍被使用，就會存取到已毀壞的物件。
- 跨動態函式庫的靜態解構順序完全不受 C++ 標準規範。

#### 問題三：musl libc 中靜態解構子永不執行

若執行環境使用 musl libc（如 Alpine Linux），`dlclose()` 與 `__cxa_finalize` 均為空操作（no-op），函式庫內的靜態解構子**永遠不會被呼叫**[^musl-noop]。

---

## 2. 不同執行環境的行為差異

| 環境 | Meyers Singleton 解構行為 | 能否卸載函式庫 |
|------|--------------------------|----------------|
| glibc + 無 STB_GNU_UNIQUE | `dlclose` 時透過 `.fini_array` 呼叫解構子 | ✅ 可卸載 |
| glibc + 有 STB_GNU_UNIQUE | 永不呼叫解構子，函式庫被鎖定在記憶體中 | ❌ 無法卸載 |
| musl libc | 解構子永不執行 | `dlclose` 為空操作 |
| Windows (MSVC) | 每個 DLL 有獨立資料段，各 DLL 各自解構 | ✅ 可卸載，但每個 DLL 有自己的實例 |

**關鍵洞察**：Meyers Singleton 在動態函式庫中的行爲強烈依賴平台實作，無法寫出跨平台一致的行為。

---

## 3. 推薦做法：顯式生命週期管理（Explicit Lifecycle Singleton）

對於動態函式庫中的單例，最可靠的策略是**放棄自動解構，改採顯式的初始化與釋放 API**[^isocpp-faq]。

### 3.1 基本模式：堆分配 + init/shutdown

```cpp
// plugin.h — C 相容介面（對 dlopen/dlsym 友善）
#pragma once
#ifdef __cplusplus
extern "C" {
#endif

int   plugin_init(void);
void  plugin_shutdown(void);

#ifdef __cplusplus
}
#endif
```

```cpp
// plugin.cpp
#include "plugin.h"
#include <atomic>

namespace {
    // 僅使用平凡解構型別（POD）作為全域變數
    std::atomic<bool> g_initialized{false};

    class Engine {
    public:
        Engine()  { /* 分配資源：開啟連線、啟動執行緒等 */ }
        ~Engine() { /* 釋放資源：關閉連線、停止執行緒等 */ }
        void doWork() { /* ... */ }

        // 禁止複製移動
        Engine(const Engine&) = delete;
        Engine& operator=(const Engine&) = delete;
    };

    Engine* g_engine = nullptr;  // 平凡指標，無需解構
}

int plugin_init() {
    if (g_initialized.exchange(true)) {
        return 1;  // 已初始化，冪等處理
    }
    g_engine = new Engine();
    return 0;
}

void plugin_shutdown() {
    if (!g_initialized.exchange(false)) {
        return;  // 已釋放，冪等處理
    }
    delete g_engine;
    g_engine = nullptr;
}
```

編譯參數（關鍵）：

```bash
g++ -fvisibility=hidden -fvisibility-inlines-hidden \
    -fno-gnu-unique -shared -o plugin.so plugin.cpp
```

- `-fvisibility=hidden`：避免 inline/template 函式中的靜態變數產生 `STB_GNU_UNIQUE`。
- `-fvisibility-inlines-hidden`：同上，針對 inline 函式。
- `-fno-gnu-unique`：直接禁用 GNU unique 符號機制（會破壞 ODR 保證，但在插件場景下可接受）。
- 僅透過 `__attribute__((visibility("default")))` 顯式匯出 `plugin_init`、`plugin_shutdown` 等 C API。

### 3.2 主程式的正確呼叫順序

```cpp
// host.cpp
#include <dlfcn.h>

void* handle = dlopen("./plugin.so", RTLD_NOW | RTLD_LOCAL);
auto init    = (int   (*)())dlsym(handle, "plugin_init");
auto shutdown = (void  (*)())dlsym(handle, "plugin_shutdown");

init();       // 1. 先初始化
// ... 使用函式庫 ...
shutdown();   // 2. 手動釋放資源（必須在 dlclose 之前）
dlclose(handle);  // 3. 再卸載函式庫
```

**順序不可顛倒**：若先 `dlclose` 再呼叫 `shutdown`，函式庫的程式碼段已被卸載，`shutdown` 將觸發段錯誤。

---

## 4. 安全存取慣用語：防護指標（Guarded Pointer）

在顯式生命週期模式中，任何使用單例的程式碼都必須先檢查單例是否仍存活。最簡單的方式是使用 `std::atomic` 旗標：

```cpp
namespace {
    std::atomic<bool> g_active{false};

    class Resource {
    public:
        void criticalOperation() {
            // 內部不再檢查，由呼叫方確保
            /* ... */
        }
    };

    Resource* g_res = nullptr;
}

void useResource() {
    if (!g_active.load(std::memory_order_acquire)) {
        return;  // 或拋出例外、回傳錯誤碼
    }
    g_res->criticalOperation();
}
```

對於需要更高安全性的場景，可考慮 LLVM `ManagedStatic` 風格的實作，將活躍狀態檢查封裝在存取函式中[^llvm-managed-static]。

---

## 5. 進階方案比較

| 方案 | 初始化控制 | 釋放控制 | 執行緒安全 | dlclose 相容 | STB_GNU_UNIQUE | 適用場景 |
|------|-----------|---------|-----------|-------------|----------------|----------|
| **Meyers Singleton**（原始） | 自動懶載入 | 自動（無法控制） | ✅ C++11+ | 視平台 | ❌ 產生 | 常駐程式，不卸載 |
| **Leak Singleton**（`static T* p = new T;`） | 自動懶載入 | **永不釋放** | ✅ | ✅ 無解構問題 | 視編譯選項 | 僅需初始化，不需釋放 |
| **顯式 init/shutdown**（本報告推薦） | 手動控制 | 手動控制 | 需自行實作 | ✅ 完全相容 | 可避免 | 插件系統、熱重載 |
| **Nifty Counter** | 自動（相同 DSO 內有序） | 自動（相同 DSO 內有序） | 不安全 | 與 Meyers 相同 | 可能產生 | 大型框架中同函式庫內依賴 |
| **`[[clang::no_destroy]]`** | 自動懶載入 | **永不呼叫解構子** | ✅ C++11+ | ✅ 無解構呼叫 | 同 Meyers | 解構無副作用 |

### Leak Singleton（ISO C++ FAQ 推薦）

```cpp
Logger& getLogger() {
    static Logger* instance = new Logger();
    return *instance;
}
```

此模式的優點[^isocpp-faq]：

- 無靜態解構順序問題（因為物件從不解構）。
- 堆疊記憶體在程式結束時由 OS 自動回收。
- 若解構子僅有無關緊要的副作用（如釋放 OS 會自動回收的記憶體），此為最簡潔安全的方案。
- 在動態函式庫中，即使「洩漏」的堆疊記憶體在 `dlclose` 時不會被回收，但若該資源（如檔案控制代碼、網路連線）必須在卸載時關閉，則此模式不適用。

---

## 6. Windows DLL 的特殊考量

在 Windows 上，每個 DLL 擁有獨立的資料段（data segment）。若單例定義在 header 中並被多個 DLL 含入，每個 DLL 會各自擁有一個實例。解決方案[^alib-module]：

1. 將單例實作集中在一個專屬 DLL 中，所有需要共用單例的模組都與該 DLL 鏈結。
2. 使用以 `std::type_info` 為鍵的程序級註冊表（process-wide registry）。

---

## 7. 實作檢核清單

若你正在設計一個以單例模式提供服務的動態函式庫，請確認以下事項：

- [ ] 所有全域變數皆為平凡型別（POD），或使用平凡指標 + 堆分配。
- [ ] 提供獨立的 `init()` 與 `shutdown()` 函式（C 相容介面）。
- [ ] `init()` 與 `shutdown()` 是冪等的（可安全重複呼叫）。
- [ ] 存取單例的函式在單例已被 `shutdown()` 後有安全的降級行為。
- [ ] 編譯選項包含 `-fvisibility=hidden` 與 `-fno-gnu-unique`。
- [ ] 主程式保證先 `shutdown()` 再 `dlclose()`。
- [ ] 單例持有的資源（檔案、連線、執行緒）在 `shutdown()` 中被明確關閉。

---

## 總結

在動態函式庫中實現單例時，**顯式生命週期管理是唯一可靠的做法**。放棄 C++ 的靜態儲存期自動管理，改用 `init()`/`shutdown()` 手動控制資源的生命週期，並搭配適當的編譯選項避免 `STB_GNU_UNIQUE` 符號，才能確保函式庫可被安全載入與卸載。對於解構無副作用的場景，Leak Singleton（`static T* p = new T`）提供最簡潔的替代方案。

---

[^meyers-singleton]: Scott Meyers. (1998). Singleton 模式實作. In *Effective C++* (2nd ed., Item 26). Addison-Wesley. 函式區域靜態變數在 C++11 後的執行緒安全性由 ISO C++ 標準 §6.7/4 保證。

[^gnu-unique]: bramoosterhuis. (2024). STB_GNU_UNIQUE & dlclose: Why Shared Libraries Refuse to Unload. Retrieved 2026-09-25, from https://gist.github.com/bramoosterhuis/bdee3df706808516206bcd5bd248bcb1

[^siof-cppreference]: cppreference.com. (n.d.). Static Initialization Order Fiasco. Retrieved 2026-09-25, from https://en.cppreference.com/w/cpp/language/siof

[^musl-noop]: maskray.me. (2024). C++ Exit-time Destructors and Dynamic Libraries. Retrieved 2026-09-25, from https://maskray.me/blog/2024-03-17-c++-exit-time-destructors

[^isocpp-faq]: ISO C++ FAQ. (n.d.). Construct on First Use Idiom. Retrieved 2026-09-25, from https://isocpp.org/wiki/faq/ctors#static-init-order-on-first-use

[^llvm-managed-static]: LLVM Project. (n.d.). ManagedStatic — LLVM Documentation. Retrieved 2026-09-25, from https://llvm.org/doxygen/classllvm_1_1ManagedStatic.html

[^alib-module]: ALib Project. (n.d.). ALib Module Singletons. Retrieved 2026-09-25, from https://www.alib.dev/alib_mod_singletons.html