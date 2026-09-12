# C++ 依賴管理：Bazel 替代方案綜覽

## 前言

Bazel（Google 開源）是一套 hermetic（密封）、rule-based（規則式）的建置系統，擅長大規模多語言 monorepo。然而其學習曲線陡峭、協力廠商 C++ 函式庫支援不足、對小型專案過度設計等痛點，讓許多團隊尋找替代方案[^reddit-bazel-cmake]。本文以「Bazel 替代方案」為方向，系統比較當今 C++ 生態中的建置與依賴管理工具。

## 工具分類

C++ 建置工具鏈可粗略分為三類[^pkglog]：

| 類別 | 工具 | 角色 |
|---|---|---|
| 元建置系統（Meta-build） | CMake、Meson | 產生原生建置檔（Ninja、Make、MSBuild） |
| 密封式建置系統 | Bazel、Buck2、Please、GN | 自有建置引擎，多語言、圖形化、密封 |
| 套件管理器 | Conan、vcpkg、CPM.cmake | 抓取、建置、提供第三方函式庫 |

## 選項一：CMake + 套件管理器

### CMake 簡介

CMake 是 C/C++ 事實上的業界標準元建置系統，約 83% 的 C++ 專案使用[^pkglog]。它產生 Ninja/Make/Visual Studio/Xcode 等後端建置檔。

**優點**：
- 龐大的生態系——幾乎所有 C++ 函式庫都支援 CMake
- 優異的 IDE 整合（CLion、Visual Studio、VS Code）
- 成熟的跨平台與交叉編譯支援

**缺點**[^gist-comparison][^meson-compare]：
- DSL 語法笨重且有歷史包袱
- 非密封（non-hermetic），無法保證全域重現性建置
- 依賴圖粒度較粗，不如 Bazel 的構件級追蹤
- 本身無遞迴依賴解析——需仰賴外部套件管理器

### CPM.cmake

一個輕量的 CMake 腳本（僅需加入 `CPM.cmake` 一個檔案），包裝 CMake 的 `FetchContent` 並加入版本控制與快取[^cpm]。

**特色**：
- 語法簡潔：`CPMAddPackage("gh:user/repo@version")`
- 支援 Package lock 檔案、離線快取、區域覆蓋
- 約 4.1k GitHub Stars，用於 Tracy Profiler、CRoaring 等專案
- 無預編譯二進位——一律從原始碼建置

**適用場景**：需要輕量的純原始碼依賴管理，不想引入完整套件管理器。

### Conan（v2.x）

由 JFrog 維護的完整 C/C++ 套件管理器，支援任何建置系統（CMake、Meson、Bazel、Autotools、VS、Xcode 等）[^conan]。

**特色**：
- 二進位快取與預編譯套件——支援多平台/編譯器/設定組合
- 版本區間解析與鎖定檔（conan.lock）
- 分散式倉儲（ConanCenter、私有伺服器）
- SBOM 產生、弱點掃描（conan audit）
- 與建置系統徹底解耦——可搭配 CMake、Meson、Bazel 等

**適用場景**：多平台專案、需要二進位快取、需要與建置系統解耦、私有函式庫分發。

### vcpkg

微軟維護的開源 C/C++ 套件管理器，提供超過 2300 個精選開源函式庫[^vcpkg]。

**特色**：
- Manifest 模式（vcpkg.json）確保可重現性
- 版本化 baseline + vcpkg.lock 鎖定檔——衝突極少
- 預編譯二進位（支援 70+ 組態的 ABI 驗證）
- 與 CMake 及 MSBuild 無縫整合（CMAKE_TOOLCHAIN_FILE）
- 支援自訂註冊表（registries）與空中隔離快取

**適用場景**：Windows/Visual Studio 為主的專案、快速入門、微軟生態系。

## 選項二：Meson

Meson 是新一代元建置系統，Python-like 語法，輸出 Ninja 建置檔，設計目標是比 CMake 更快、更簡潔[^meson]。

**依賴管理**：
- 內建 `dependency()` 函式，支援 pkg-config、CMake find_package、config-tool（如 llvm-config）及 subproject 回退
- `dependency('zlib', version: '>=1.2.8')` 找不到時自動回退到 Meson subproject 建置

**優點**：
- 語法乾淨，學習曲線三者中最低
- 設定速度快——大型專案顯著快於 CMake
- 良好的交叉編譯支援（cross file）
- 內建測試

**缺點**：
- 生態系小於 CMake
- IDE 整合仍不如 CMake 成熟

**主要用戶**：GNOME、systemd、Mesa、Vulkan SDK、SDL、GTK[^meson]。

**與 Bazel 的比較**[^meson-compare]：Meson 官方比較頁指出 Bazel「已證明可擴展到非常大的專案，但以 Java 實作、Windows 支援差、高度集中在 Google 的做事方式」。

## 選項三：Buck2

Meta（Facebook）開發的密封式建置系統，以 Rust 重寫，是 Bazel 的直接競爭者[^buck2]。

**特色**：
- 所有規則以 Starlark 撰寫（非內建於二進位），高度可擴展
- 動態 DAG——可在建置中依動態結果決定後續依賴
- Transitive sets（類似 Bazel 的 depset 但整合到依賴圖）
- 比 Buck1 快 2 倍，支援遠端執行優先
- 多語言：C++、Python、Rust、Kotlin、Swift、OCaml、Haskell

**優點**：
- 極快、密封、可擴展到超大 monorepo
- 遠端快取與執行原生支援
- 架構設計比 Bazel 更純粹（Hacker News 評論）[^hn-buck2]

**缺點**：
- 開源尚在早期（2023 年釋出）
- Meta 以外的社群較小
- 學習曲線陡峭（需理解 Starlark 與 Buck2 規則模型）
- 非遠端執行的工作流程較不成熟

## 選項四：Please（plz）

Thought Machine 開發的密封式建置系統，以 Go 撰寫，定位為「更輕量的 Bazel 替代品」[^please-build]。

**特色**：
- 同樣使用 BUILD 檔案與 Python-like DSL（Starlark 風格）
- 外掛式語言支援——Go、Python、Java、C++、Rust
- 密封沙箱、內容尋址快取、REAPI 相容遠端執行
- `plz query` 圖形分析、`plz test` 測試分片
- 與 Bazel 共用快取與執行器

**優點**：
- 學習曲線比 Bazel 溫和
- 專為多語言 monorepo 設計
- Docker/Container 原生整合

**缺點**：
- 生態系顯著小於 Bazel
- Windows 支援為實驗性（WSL2）
- 自訂規則需自行維護

## 選項五：GN（Generate Ninja）

Google 從 Chromium 提取的元建置系統，產生 Ninja 建置檔[^gn]。

**特色**：
- 乾淨、可讀的語法
- 單一執行可針對多平台
- 正確性導向——`gn check`、`testonly`、`assert_no_deps`
- 原生支援 C、C++、Rust、Objective C、Swift

**限制**：
- GN 是建置設定產生器，**無套件管理功能**——依賴需手動宣告
- 最小設定負擔重（無預設編譯器設定）
- 不可組合——為單一大專案設計
- 無正式版本發行機制，專案需自行管理 GN 版本

**主要用戶**：Chromium、Fuchsia 及相關 Google 專案。**不建議**用於通用 C++ 開發[^gn]。

## 選項六：xmake

以 Lua 為基礎的跨平台建置工具，可直出（如 Make/Ninja）亦可產生專案檔（如 CMake/Meson）[^xmake]。

**特色**：
- Lua 設定語法直觀簡潔
- 內建套件管理系統，亦可取用 Conan/vcpkg
- 啟動快速、支援多平台
- 約 5k+ GitHub Stars，活躍於遊戲開發與嵌入式領域

**適用場景**：需要內建套件管理與簡潔語法的 C/C++ 專案，特別適合遊戲開發與嵌入式系統。

## 選項七：build2

目標是提供類似 Rust Cargo 的 C++ 一站式體驗——建置系統 + 套件管理器 + 專案管理器合一[^build2]。

**特色**：
- 跨平台（MIT 授權）、一致的編譯器介面
- 整合式建置與套件管理——自動依賴解析與抓取
- 生態系較小但持續發展

**適用場景**：希望獲得 Cargo-like 體驗的 C++ 專案。

## 對比總表

| 工具 | 類型 | 密封 | 依賴管理 | 多語言 | Monorepo | 學習曲線 | 生態系成熟度 |
|---|---|---|---|---|---|---|---|
| CMake + CPM | 元建置 | ❌ | ✅ 輕量（純原始碼） | C/C++ | ◐ | ★★★★★ | ★★★★★ |
| CMake + Conan | 元建置 + 套件管理 | ❌ | ✅ 完整（含二進位） | C/C++ | ◐ | ★★★★ | ★★★★★ |
| CMake + vcpkg | 元建置 + 套件管理 | ❌ | ✅ 完整（含二進位） | C/C++ | ◐ | ★★★★★ | ★★★★★ |
| Meson | 元建置 | ❌ | ✅ 內建 | C/C++ | ◐ | ★★★★ | ★★★★ |
| Bazel | 規則式密封 | ✅ | 需手寫 WORKSPACE | ✅ | ✅ | ★★★ | ★★★ |
| Buck2 | 規則式密封 | ✅ | 需手寫 Starlark | ✅ | ✅ | ★★ | ★★ |
| GN | 元建置（Ninja 產生） | ◐ | ❌ 全手動 | C++/Rust/Swift | ✅ | ★★ | ★★ |
| Please | 規則式密封 | ✅ | 外掛式 | ✅ | ✅ | ★★★ | ★★ |
| xmake | Lua 建置 + 套件管理 | ❌ | ✅ 內建 | C/C++ | ◐ | ★★★★ | ★★★ |
| build2 | 一站式工具鏈 | ◐ | ✅ 內建 | C/C++ | ◐ | ★★★ | ★★ |

▲ ◐ = 部分支援

## 選擇建議

| 場景 | 推薦方案 |
|---|---|
| 標準 C++ 函式庫/應用、跨平台 | **CMake + Conan** 或 **CMake + vcpkg** + Ninja |
| 想要比 CMake 更簡潔的語法 | **Meson** |
| 大型多語言 monorepo（>100 萬行） | **Bazel** 或 **Buck2** 或 **Please** |
| 遊戲開發 / 嵌入式 | **xmake** |
| Chromium/Fuchsia 開發 | **GN** |
| 想要 Cargo-like 的 C++ 體驗 | **build2** |
| 只需要輕量原始碼依賴 | **CMake + CPM** |

業界存在兩個匯流趨勢：**CMake 是開源 C++ 函式庫散播與跨平台應用的無爭議標準**；而 **Bazel/Buck2 類系統則是大型 CI 密集多語言 monorepo 組織的未來**[^pkglog][^gist-comparison]。許多團隊採用混合工作流程——CMake 用於消費函式庫、Bazel 用於 monorepo 建置協調。

## 參考資料

[^reddit-bazel-cmake]: Reddit r/cpp. (2023). Bazel or CMake?. Retrieved 2026-09-13, from https://www.reddit.com/r/cpp/comments/vxr1ap/bazel_or_cmake/
[^pkglog]: pkglog. (2024). C++ Build System Comparison. Retrieved 2026-09-13, from https://pkglog.com/en/blog/cpp-series-53-6-build-system-comparison/
[^gist-comparison]: MangaD. (2024). Build Systems Comparison. Retrieved 2026-09-13, from https://gist.github.com/MangaD/26ef92a1e1efd967c3e0188dc0591e83
[^meson-compare]: Meson. (n.d.). Comparisons — Meson documentation. Retrieved 2026-09-13, from https://mesonbuild.com/Comparisons.html
[^cpm]: CPM.cmake. (n.d.). CPM.cmake — CMake's missing package manager. Retrieved 2026-09-13, from https://github.com/cpm-cmake/CPM.cmake
[^conan]: JFrog. (n.d.). Conan 2.0 Documentation. Retrieved 2026-09-13, from https://docs.conan.io/2/
[^vcpkg]: Microsoft. (n.d.). vcpkg — C++ Library Manager. Retrieved 2026-09-13, from https://learn.microsoft.com/en-us/vcpkg/
[^meson]: Meson. (n.d.). Meson Build System. Retrieved 2026-09-13, from https://mesonbuild.com/
[^buck2]: Meta. (2023). Buck2 Build System. Retrieved 2026-09-13, from https://buck2.build/
[^hn-buck2]: Hacker News. (2023). Buck2: A Tour Around Buck2. Retrieved 2026-09-13, from https://news.ycombinator.com/item?id=36509302
[^please-build]: Thought Machine. (n.d.). Please Build System. Retrieved 2026-09-13, from https://please.build/
[^gn]: Google. (n.d.). GN — Generate Ninja. Retrieved 2026-09-13, from https://gn.googlesource.com/gn
[^xmake]: xmake. (n.d.). xmake — A Cross-platform Build Utility Based on Lua. Retrieved 2026-09-13, from https://xmake.io/
[^build2]: build2. (n.d.). build2 — Build System and Package Manager for C/C++. Retrieved 2026-09-13, from https://www.build2.org/