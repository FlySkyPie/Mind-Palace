# C++ 依賴注入容器 (Dependency Injection Container) 函式庫調查

## 概述

C++ 的依賴注入（DI）不同於 Java/C# 有成熟的工業標準（如 Spring、Guice），在 C++ 社群中長期缺乏主流共識。隨著 C++11/14/17/20 的演進，基於模板元程式設計的編譯期 DI 逐漸成熟，本文調查目前主要的 C++ DI 容器函式庫。

## 三大主流函式庫

### Boost.DI

Boost.DI 是純編譯期的 DI 容器，由 boost-ext 社群維護，為 Boost 函式庫的候補專案[^boost-di-gh]。

**核心特性：**
- C++14 起，純 header-only（單一 `include/boost/di.hpp`）
- **零執行期開銷** — 編譯期解析依賴圖，最終展開為直接的建構式呼叫
- 編譯期完整驗證依賴圖的正確性
- 無巨集、無 RTTI，支援 `-fno-exceptions -fno-rtti`
- Scope：預設 `unique`（每次新實體），可手動指定 `di::singleton`
- 介面綁定：`di::bind<Interface>.to<Implementation>()`
- 命名注入：用 `di::named` 區分同類型的參數
- 擴充模組：assisted injection、lazy injection、XML/UML dump 等

**優點：** 零開銷、強型別安全、社群最大（1,300+ stars）、積極維護
**缺點：** 模板語法對新手較陡、無執行期重組態能力

### Google Fruit

Google Fruit 是 Google 開發的混合式 DI 框架，編譯期檢查 + 執行期注入[^fruit-gh]。

**核心特性：**
- C++11 起，需編譯（非 header-only）
- **Component 架構** — 每個 component 宣告「提供哪些型別」及「需要哪些型別」
- 使用 `INJECT` 巨集標記建構式
- 編譯期檢查依賴圖 + 執行期建立 injector
- Component 可組合：`.install()`
- 介面綁定：`.bind<Interface, Implementation>()`

**優點：** 適合大團隊、明確的 component 契約、Google 生產環境驗證
**缺點：** 需編譯（非 header-only）、`INJECT` 巨集較侵入、boilerplate 較多、學習曲線較高

### Kangaru

Kangaru 是輕量、非侵入的 header-only DI 容器，最新 v4 版本推出 Autowire API[^kangaru-gh]。

**核心特性：**
- C++11/14+，header-only，無外部相依，MIT 授權
- **Service Map 模式** — 透過 function overload 解析服務映射
- **Autowire API（v4）** — `friend auto service_map(...)` 自動推導建構式參數
- Scope：`kgr::single_service<T>`（singleton）、`kgr::service<T>`（unique）
- 非侵入 — 不需修改既有類別
- 支援 setter injection、function parameter injection

**優點：** API 最簡潔、學習曲線最低、完全非侵入、MIT 授權
**缺點：** 社群較小、SFINAE 錯誤訊息可能不易閱讀

## 其他值得注意的函式庫

| 函式庫 | 描述 | 備註 |
|---|---|---|
| **Ichor**[^ichor-gh] | C++20 微服務框架，DI + 事件迴圈 + thread confinement | 不只是 DI，而是完整的應用框架，需 C++20 編譯器 |
| **Injec++or**[^injecttor-gh] | 模仿 C# DI 風格的執行期容器 | 非常新（2024）、星星數極少（~2）、尚未成熟 |
| **Hypodermic** | 較早期的 C++11 DI 函式庫 | 已無維護或更名 |
| **cpp-effects**[^cpp-effects-gh] | 代數效應處理器（algebraic effect handler） | 非傳統 DI，但被歸類在 DI 相關話題 |

## 比較總表

| 特性 | Boost.DI | Google Fruit | Kangaru | Ichor | Injec++or |
|---|---|---|---|---|---|
| **C++ 標準** | C++14 | C++11 | C++11/14+ | C++20 | C++17 |
| **Header-only** | ✅ 是 | ❌ 需編譯 | ✅ 是 | ❌ 需編譯 | ✅ 是 |
| **零執行期開銷** | ✅ 完全 | ⚠️ 極小 | ✅ 完全 | ❌ 執行期 | ❌ 執行期 |
| **編譯期檢查** | ✅ 完整 | ✅ 部分 | ✅ 完整 | ❌ 無 | ❌ 無 |
| **需巨集** | ❌ 無 | ✅ INJECT | ❌ 無 | ❌ 無 | ❌ 無 |
| **Singleton Scope** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Factory/Transient** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Scoped（Request）** | ✅ 擴充 | ❌ | ✅ | ✅ | ✅ |
| **建構式注入** | ✅ 自動 | ✅ INJECT | ✅ 自動 | ✅ | ✅ 手動 |
| **介面綁定** | `bind<I>.to<C>()` | `.bind<I,C>()` | Service map | 註冊 | `Register<I,C>()` |
| **學習曲線** | 中 | 中高 | 低中 | 高 | 低 |
| **授權** | Boost 1.0 | Apache 2.0 | MIT | MIT | MIT |
| **GitHub Stars** | ~1,300 | ~1,900 | ~553 | ~237 | ~2 |

## Scope（生命週期）機制

### Boost.DI
- 預設 scope 由型別推導：value type 為 `unique`，`shared_ptr`/reference 為 `singleton`
- 可手動指定：`di::bind<T>.in(di::singleton)` 或 `di::unique`
- 自訂 scope 透過 extension system[^boost-di-scope]

### Google Fruit
- 預設為 lazy singleton（共享實體）
- 使用 factory 模式建立 unique instance[^fruit-scope]

### Kangaru
- 明確透過 service trait 指定：
  - `kgr::single_service<T>` = singleton（回傳 reference）
  - `kgr::service<T>` = unique（回傳 value）
  - 自訂 scope 可繼承 `kgr::service` 實作[^kangaru-scope]

## 建構式注入方式

### Boost.DI — 完全自動
```cpp
class app {
public:
    // Boost.DI 自動推導參數型別並注入
    app(ilogger& log, idatabase& db) { }
};

auto injector = di::make_injector(
    di::bind<ilogger>.to<console_logger>(),
    di::bind<idatabase>.to<sql_database>()
);
auto obj = injector.create<app>();
```

### Google Fruit — INJECT 巨集
```cpp
class app {
public:
    INJECT(app(ASSISTED(ilogger) log, idatabase* db))
        : log_(std::move(log)), db_(db) { }
};
```

### Kangaru — Autowire API（v4）
```cpp
struct app_service : kgr::service<app_service> {
    // autowire 自動推導建構式
    app_service(ilogger& log, idatabase& db) { }
};

auto container = kgr::container{};
auto& svc = container.service<app_service>();
```

## 選擇建議

| 使用場景 | 推薦選擇 |
|---|---|
| 追求最大編譯期安全、零開銷 | **Boost.DI** |
| 大型團隊、需要明確架構契約 | **Google Fruit** |
| C++17+ 現代專案、重視易用性 | **Kangaru**（尤其是 Autowire API） |
| 嵌入式/即時系統、禁用例外/RTTI | **Boost.DI** 或 **Kangaru** |
| 完整微服務框架、含執行緒管理 | **Ichor** |
| 快速原型開發 | **Kangaru** |
| 從 C#/Java 轉 C++ 的開發者 | **Injec++or** 或 **Boost.DI** |

## 關鍵觀察

1. **C++ DI 的生態仍以編譯期為主流**。不同於 Java 的執行期容器，C++ DI 傾向於利用模板在編譯期解析依賴，以達到零開銷。

2. **三大主流各自佔據不同定位**：Boost.DI 是純模板元編程的巔峰、Fruit 是 Google 規模的工程實踐、Kangaru 則是最接近「現代 C++ 語感」的選擇。

3. **新趨勢**：Ichor 代表 C++20 coroutine + DI 的整合方向，但代價是失去輕量性。

4. **尚未有標準化進展**：截至目前，C++ 標準委員會尚未將 DI 容器納入標準函式庫的討論議程。

---

[^boost-di-gh]: Boost.Extension. (n.d.). *Boost.DI*. Retrieved 2026-09-25, from https://github.com/boost-ext/di
[^fruit-gh]: Google. (n.d.). *Fruit*. Retrieved 2026-09-25, from https://github.com/google/fruit
[^kangaru-gh]: Gracicot. (n.d.). *Kangaru*. Retrieved 2026-09-25, from https://github.com/gracicot/kangaru
[^ichor-gh]: Volt-Software. (n.d.). *Ichor*. Retrieved 2026-09-25, from https://github.com/volt-software/Ichor
[^injecttor-gh]: Fabrizio86. (n.d.). *Injec++or*. Retrieved 2026-09-25, from https://github.com/Fabrizio86/Injecttor
[^cpp-effects-gh]: Maciejpirog. (n.d.). *cpp-effects*. Retrieved 2026-09-25, from https://github.com/maciejpirog/cpp-effects
[^boost-di-scope]: Boost.Extension. (n.d.). *Boost.DI — Scopes*. Retrieved 2026-09-25, from https://boost-ext.github.io/di/tutorial.html#scopes
[^fruit-scope]: Google. (n.d.). *Fruit — Scopes*. Retrieved 2026-09-25, from https://github.com/google/fruit/wiki
[^kangaru-scope]: Gracicot. (n.d.). *Kangaru — Scopes*. Retrieved 2026-09-25, from https://github.com/gracicot/kangaru