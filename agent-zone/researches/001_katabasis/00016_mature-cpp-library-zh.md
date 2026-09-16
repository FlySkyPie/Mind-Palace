# 成熟 C++ 函式庫應具備的特性

> 研究時間：2026-09-12 | 方法：Web 提取驗證標準文件

---

## 摘要

一個成熟的 C++ 函式庫不僅需要正確的 API 設計，還需涵蓋專案結構、建置系統、測試、文件、CI/CD、套件管理、跨平台支援與長期維護等面向。本報告彙整 C++ Core Guidelines、C++Alliance (前 Boost) 貢獻者指南、Pitchfork 專案佈局標準，以及現代 C++ 生態系常見實踐，歸納出核心特性。

---

## 1. API 設計

### 1.1 C++ Core Guidelines 介面規範 (I 系列規則)

C++ Core Guidelines（2026 年 6 月版）對介面設計給出一系列具體規則[^core-guidelines]：

- **I.1**：讓介面明確，避免隱式全域依賴
- **I.4**：精確強型別 — 避免 `void*`，使用具體型別或 template
- **I.6**：使用 `Expects()` 陳述前置條件
- **I.7**：使用 Concepts（C++20）約束 template 參數
- **I.8**：使用例外處理錯誤，錯誤不應被忽略
- **I.11**：不用 raw pointer 傳所有權，用 value / `unique_ptr` / `shared_ptr`
- **I.13**：不用單一指標傳陣列，用 `span<T>`
- **I.23**：函式參數少於 4 個，將相關參數組合成 struct
- **I.26**：需要跨編譯器 ABI 時使用 C subset
- **I.30**：封裝規則違反需要理由

### 1.2 設計最佳實踐（C++Alliance 貢獻者指南）

C++Alliance 的設計最佳實踐文件建議[^boost-design]：

- 首要追求清晰與正確性，最佳化為次要考量
- 優先使用 ISO 標準 C++，避免非標準編譯器擴充
- Header 應為良好的鄰居（不汙染全域 namespace、最小化 `#include`）
- 使用 C++ Standard Library 或其他 Boost 函式庫，但僅在效益大於成本時使用
- 遵循 Scott Meyers 在《Effective C++》系列中提出的品質程式設計實踐

---

## 2. 資源管理與例外安全

C++ Core Guidelines 的 R 系列規則涵蓋資源管理[^core-guidelines]：

- **RAII**：資源取得即初始化，是 C++ 最系統化的洩漏預防方式
- **R.1**：使用 RAII 管理資源，避免裸 `new`/`delete`
- **Rule of Zero**：優先使用標準型別，避免自定義解構/複製/移動函式
- **noexcept**：解構函式、`swap()`、移動操作必須標記 `noexcept`
- **Const Correctness**：預設物件、成員函式、參數都應為 `const`

例外安全有三個層級：basic guarantee、strong guarantee（透過 copy-and-swap）、nothrow guarantee。

---

## 3. 專案結構 — Pitchfork 佈局標準

Pitchfork 是 C++ 專案目錄結構的事實參考標準，由 vector-of-bool 制定[^pitchfork]：

```
<project>/
├── include/          # 公開 API header（消費者只加此路徑）
│   └── <project>/
│       └── module.hpp
├── src/              # 實作原始碼 + 私有 header
│   └── <project>/
│       ├── module.cpp
│       └── module.test.cpp   # 合併測試（推薦）
├── tests/            # 非單元測試
├── examples/         # 範例程式碼
├── docs/             # 文件
├── external/         # 嵌入的第三方專案
├── extras/           # 選擇性子模組（語言綁定、外掛）
├── tools/            # 開發工具/腳本
├── data/             # 非程式資源
├── CMakeLists.txt
├── README.md
└── LICENSE
```

**關鍵原則**：目錄結構應對應 namespace 結構，讓元件可透過 namespace 路徑定位。

---

## 4. 建置系統與套件管理

### 4.1 建置系統

CMake 是跨平台的事實標準，現代 CMake（3.15+）採用 target-based 設計，透過 `target_include_directories`、`target_link_libraries` 等指令明確表達相依關係。

### 4.2 套件管理整合

常見選項包含：

| 套件管理器 | 說明 |
|-----------|------|
| vcpkg     | Microsoft 維護，支援 Windows/Linux/macOS |
| Conan     | 去中心化 C/C++ 套件管理器，支援多平台及多建置系統 |
| CPM.cmake | CMake-native 相依管理，無需額外安裝 |
| Hunter    | 跨平台 CMake 套件管理器 |

**最佳實踐**：支援 vcpkg + Conan 雙整合，並提供 CPM.cmake 作為輕量備案。

來自 C++Alliance 的相關指南指出：相依管理應謹慎，只有當效益大於成本時才引入外部相依[^boost-design]。

---

## 5. 測試與品質保證

### 5.1 Boost 測試政策要求

C++Alliance 的測試政策列出必要條件[^boost-test]：

1. 每個函式庫應提供一個或多個合適的測試程式，納入回歸測試套件
2. 測試程式執行錯誤必須以非零回傳值報告（非零回傳值是回歸測試框架辨識錯誤的唯一方式）
3. 執行時間過長的測試應拆分為：快速基礎測試（用於狀態表）與完整覆蓋測試（用於詳盡測試）
4. 如需偏離一般測試政策，必須說明並實作替代測試策略

### 5.2 現代 C++ 測試生態系

| 框架 | 適用場景 |
|------|----------|
| GoogleTest | 最廣泛使用，支援 mock（GoogleMock） |
| Catch2 | 現代 C++、header-only、支援 BDD 風格 |
| doctest | 最輕量的 header-only 框架 |
| Fuzz Testing | libFuzzer / afl 測試邊界案例 |
| Property-based | rapidcheck（QuickCheck 風格） |
| Sanitizers | AddressSanitizer / UBSan / TSan / LSan |

C++Alliance 貢獻者指南特別將 sanitizers 與 fuzzing 列為獨立的主題章節[^boost-sanitizers][^boost-fuzzing]，顯示其在現代 C++ 品質保證中的重要性。

### 5.3 CI/CD 建議管線

```
CI Pipeline（GitHub Actions）:
├── Build（GCC / Clang / MSVC 多編譯器）
├── Test（完整測試套件）
├── Code Coverage（gcovr + Codecov）
├── Static Analysis（clang-tidy / cppcheck）
├── Sanitizers（AddressSanitizer / UBSan / TSan / LSan）
├── Code Formatting（clang-format 強制一致風格）
└── Security Analysis（CodeQL）
```

---

## 6. 文件

C++Alliance 文件指南要求[^boost-design]：

- 文件應包含通用介紹與設計理念
- 類別與函式說明（前置條件、效果、回傳值、拋出例外）
- 錯誤處理策略
- 使用範例
- 編譯與連結說明
- 版本歷史與變更記錄

常見工具與平台搭配：

| 工具/標準 | 說明 |
|-----------|------|
| Doxygen   | C++ 文件生成事實標準，從程式碼註解自動產生 |
| GitHub Pages | 自動部署 Doxygen 輸出 |
| README.md | 函式庫的快速入門、CI 徽章、安裝說明 |

---

## 7. 版本管理

### 7.1 語意化版本（SemVer）

| 版本 | 變更類型 |
|------|----------|
| MAJOR | 不相容的 API 變更 |
| MINOR | 向後相容的功能新增 |
| PATCH | 向後相容的錯誤修復 |

### 7.2 C++Alliance 命名慣例

來自 C++Alliance 設計最佳實踐的命名建議[^boost-design]：

| 元素 | 規則 |
|------|------|
| 通用名稱 | `lowercase_with_underscores` |
| 縮寫 | 視為一般單字（`xml_parser` 而非 `XML_parser`） |
| 模板參數 | 大寫開頭 |
| 巨集 | 全大寫 + 函式庫前綴 |
| 函式庫名稱 | 單數、描述性、無晦澀縮寫 |

---

## 8. 效能考量

C++ Core Guidelines 的 Per 系列規則涵蓋效能[^core-guidelines]：

- **Per.1**：不要盲目地最佳化
- **Per.5**：在最佳化前使用效能分析工具
- **Per.7**：不要讓設計受限於假設的效能瓶頸
- 傳入大物件用 const reference 而非 by value
- 最小化 `#include` 依賴（forward declaration / Pimpl）
- 不要內聯直到效能分析證明需要

---

## 9. 跨平台與可移植性

C++Alliance 的可移植性要求明確指出[^boost-portability]：

- 至少通過 2 種不同 C++ 編譯器（不同作業系統）的編譯與執行
- 提供跨平台介面，不綁定特定編譯器或 OS
- 函式庫應使用 ISO 標準 C++，避免非標準編譯器擴充[^boost-design]

| 層面 | 要求 |
|------|------|
| 作業系統 | Linux / macOS / Windows 三平台建置通過 |
| 編譯器 | GCC / Clang / MSVC 至少 2 種 |
| 工具鏈 | CMake 管理跨平台建置、sanitizers 捕捉未定義行為 |
| WebAssembly | 現代 C++ 函式庫可考慮 WASM 支援 |

---

## 10. 維護與治理

C++Alliance 的函式庫要求標準[^boost-library-req]：

| 要求 | 說明 |
|------|------|
| 授權條款 | 必須簡單、無償、商業友善（推薦 Boost Software License） |
| 版權清晰 | 所有重要檔案須有版權聲明 |
| 通用有用性 | 必須對廣泛受眾有用，不限於狹隘領域 |
| 長期維護 | 作者須願意參與郵件列表討論並根據回饋改進 |
| 所有權確認 | 著作權必須清晰，避免僱主擁有程式碼的爭議 |

---

## 總結檢查清單

- [ ] API 設計：遵循 C++ Core Guidelines I 系列、強型別、concept 約束
- [ ] 例外安全：RAII、noexcept 解構式、strong guarantee
- [ ] 資源管理：Rule of Zero、smart pointers
- [ ] 專案結構：Pitchfork 佈局（include/ + src/ 分離）
- [ ] 建置系統：現代 CMake + vcpkg / Conan 支援
- [ ] 測試：GoogleTest/Catch2 + 單元/模糊/屬性測試 + sanitizers
- [ ] CI/CD：多編譯器矩陣 + static analysis + coverage
- [ ] 文件：Doxygen + README + GitHub Pages 自動部署
- [ ] 版本：SemVer + CHANGELOG
- [ ] 跨平台：Linux / macOS / Windows + GCC/Clang/MSVC
- [ ] ABI 穩定：Pimpl idiom（需要時）
- [ ] 執行緒安全：設計時即考慮並發存取
- [ ] 授權與維護：清晰版權 + 商業友善授權 + 長期維護承諾

---

## 參考來源

[^core-guidelines]: Stroustrup, B., & Sutter, H. (2026, June 14). C++ Core Guidelines. Retrieved 2026-09-12, from https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines.html

[^boost-library-req]: C++Alliance. (n.d.). Library Requirements — Contributor Guide. Retrieved 2026-09-12, from https://docs.cppalliance.org/contributor-guide/requirements/library-requirements.html

[^boost-design]: C++Alliance. (n.d.). Design Best Practices — Contributor Guide. Retrieved 2026-09-12, from https://docs.cppalliance.org/contributor-guide/design-guide/design-best-practices.html

[^boost-test]: C++Alliance. (n.d.). Test Policy — Contributor Guide. Retrieved 2026-09-12, from https://docs.cppalliance.org/contributor-guide/testing/test-policy.html

[^boost-sanitizers]: C++Alliance. (n.d.). Sanitizers — Contributor Guide. Retrieved 2026-09-12, from https://docs.cppalliance.org/contributor-guide/testing/sanitizers.html

[^boost-fuzzing]: C++Alliance. (n.d.). Fuzzing — Contributor Guide. Retrieved 2026-09-12, from https://docs.cppalliance.org/contributor-guide/testing/fuzzing.html

[^boost-portability]: C++Alliance. (n.d.). Portability Requirements — Contributor Guide. Retrieved 2026-09-12, from https://docs.cppalliance.org/contributor-guide/requirements/portability-requirements.html

[^pitchfork]: vector-of-bool. (n.d.). Pitchfork — A set of conventions and rules for C++ project layout. Retrieved 2026-09-12, from https://github.com/vector-of-bool/pitchfork

[^awesome-modern-cpp]: rigtorp. (n.d.). Awesome Modern C++. Retrieved 2026-09-12, from https://github.com/rigtorp/awesome-modern-cpp