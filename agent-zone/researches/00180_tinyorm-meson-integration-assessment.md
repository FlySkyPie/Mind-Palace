# TinyORM 導入 Meson 專案之可行性評估

## 概述

本報告評估將 [TinyORM](https://github.com/silverqx/TinyORM) — 一個受 Laravel Eloquent 啟發的現代 C++20 ORM 函式庫 — 導入 Meson 建置系統專案的可行性、困難與替代方案。

---

## TinyORM 專案概況

TinyORM 是由 Silver Zachara（silverqx）開發的 C++20 ORM，提供 ActiveRecord 模式、流暢查詢建構器（Fluent Query Builder）、資料庫遷移（Migrations）、資料填充（Seeders），以及 CLI 工具 `tom`[^readme]。完整的測試套件包含 3,378 個單元與功能測試，支援 MySQL、MariaDB、PostgreSQL 與 SQLite[^tests]。

### 授權條款

MIT License[^license]。

### 依賴關係

| 依賴 | 最低版本 | 用途 |
|---|---|---|
| Qt Framework | ≥6.2 | `QtCore`、`QtSql` 模組 |
| range-v3 | ≥0.11.0 | Eric Niebler 的範圍函式庫，僅標頭檔 |
| tabulate | latest | `tom` 遷移 `migrate:status` 命令的終端表格 |

Qt v5.15 支援自 v0.38.0 起已移除[^qt5removal]。

可選依賴：MySQL Connector/C 8、MariaDB Connector/C（僅供 `mysql_ping()` 使用）[^deps]。

### 目前支援的建置系統

TinyORM 官方僅支援兩種建置系統[^buildsystems]：

1. **CMake**（≥3.22）— 主要建置系統，支援 `FetchContent`、LTO、Ninja、Visual Studio 生成器
2. **qmake** — Qt 框架提供的建置系統（透過 `.env` 或 `conf.pri` 配置）

**TinyORM 不支援 Meson**，GitHub Issues 中也沒有任何關於 Meson 的討論[^mesonsearch]。

---

## Meson 專案導入方案分析

### 方案一：使用 Meson 的 cmake 模組（cmake.subproject）

Meson 內建模組可直接引入 CMake 子專案[^mesoncmake]：

```meson
cmake = import('cmake')

opt_var = cmake.subproject_options()
opt_var.add_cmake_defines({
  'CMAKE_BUILD_TYPE': get_option('buildtype'),
  'BUILD_TESTS': false,
  'TOM': true,
})

sub_proj = cmake.subproject('TinyORM', options: opt_var)
tinyorm_dep = sub_proj.dependency('TinyOrm')
```

**優點：**
- 無需重寫 TinyORM 的建置邏輯
- 直接沿用 CMake 的 517 行 `CMakeLists.txt` 與客製模組
- 隨 TinyORM 上游更新，維護成本低

**缺點／風險：**
- TinyORM 內部使用 `find_package(Qt6 ...)`、`find_package(range-v3 ...)` 尋找依賴，Meson 的 cmake 模組對 `find_package` 的處理能力有限，尤其 Qt6 本身非 CMake 子專案
- Qt6 需要透過 `dependency('Qt6Core')`、`dependency('Qt6Sql')` 在 Meson 側取得，與 cmake 子專案的 `find_package` 可能產生衝突
- cmake 子專案的相依傳遞（dependency propagation）可能不夠完整，需要手動補齊 `tinyorm_dep` 的 `dependencies:` 列表
- Windows 上 MSVC 的 Qt 二進位版本相容性（Qt 至 v6.8 才提供 MSVC 2022 二進位）可能增加配置複雜度[^qtmsvc]

**可行性：中等偏低。** 核心障礙在 Qt6 的雙邊依賴管理。

### 方案二：編寫 Meson Wrap 覆蓋層（patch_directory）

建立 `subprojects/TinyORM.wrap`：

```ini
[wrap-git]
url = https://github.com/silverqx/TinyORM.git
revision = v0.38.1
depth = 1

[provide]
TinyOrm = tinyorm_dep

[patch_directory]
directory = tinyorm
```

並在 `subprojects/packagefiles/tinyorm/meson.build` 中完整重新實作建置邏輯。

**優點：**
- 完全掌控建置流程
- 可透過 Meson WrapDB 的 `range-v3` 與 `tabulate` wrap 解決兩項依賴[^wrapdb_rang3][^wrapdb_tabulate]
- 純 Meson 體驗，無需混用 CMake

**缺點／風險：**
- TinyORM 的建置系統包含大量客製 CMake 模組（如編譯選項、功能開關、安裝規則、測試整合），全部需移植到 Meson
- 專案有 5,739 次提交、3,378 個測試，meson.build 檔案的維護量極大
- 需與上游同步更新，否則可能落後
- Qt6 仍無法透過 WrapDB 解決，需仰賴系統安裝或 `dependency('Qt6Core')`（pkg-config / CMake find module）

**可行性：低。** 除非有長期維護人力，否則不建議。

### 方案三：外部建置 + dependency() 查找

先使用 TinyORM 原生的 CMake 建置並安裝到系統，或透過 vcpkg 安裝，再從 Meson 專案中以 `dependency()` 引用[^vcpkg_port]：

```meson
# 方法 A：pkg-config（但目前 TinyORM 不產生 .pc 檔）
# tinyorm_dep = dependency('TinyOrm', required: false)

# 方法 B：自訂查找
cc = meson.get_compiler('cpp')
tinyorm_dep = cc.find_library('TinyOrm', has_headers: 'orm/db.hpp')

# 方法 C：CMake find module fallback
tinyorm_dep = dependency('TinyOrm', method: 'cmake')

# 手動補上 Qt6 與 range-v3
qt6_dep = dependency('Qt6Core')
qt6_sql_dep = dependency('Qt6Sql')
range_dep = dependency('range-v3')
```

**優點：**
- Meson 側幾乎零維護成本
- TinyORM 建置完全使用官方支援的方式
- 可搭配 vcpkg 管理安裝

**缺點／風險：**
- TinyORM 的 CMake config package 未設計供 Meson 使用，`TinyOrm::TinyOrm` target 的傳遞依賴（Qt6、range-v3）在 Meson 的 `dependency()` 中可能不會自動展開
- 若 TinyORM 以靜態函式庫建置，其 `INTERFACE_LINK_LIBRARIES` 中的 Qt6 與 range-v3 可能無法正確傳遞給 Meson 的 `executable()` 或 `library()` target
- TinyORM 未產生 pkg-config `.pc` 檔（CMakeLists.txt 中相關段落被註解）[^cmakelists513]
- 開發者需手動額外加裝一套依賴，無法享受 Meson Wrap 的自動下載

**可行性：中等。** 對已使用 Qt6 且能接受外部工具鏈管理的專案來說最可行。

### 方案四：尋找 Meson 原生替代方案

若無法克服 TinyORM + Meson 的整合障礙，可考慮下列 C++ 資料庫函式庫：

| 函式庫 | 建置系統 | 支援資料庫 | 備註 |
|---|---|---|---|
| [sqlpp11](https://github.com/rbock/sqlpp11) | CMake | SQLite, MySQL, PostgreSQL | 型別安全的 SQL 查詢建構器，非完整 ORM |
| [ODB](https://www.codesynthesis.com/products/odb/) | 自訂 | MySQL, SQLite, PostgreSQL, Oracle, SQL Server | 需程式碼產生器，非 Meson 原生 |
| [Poco Data](https://github.com/pocoproject/poco) | CMake | SQLite, MySQL, PostgreSQL | 資料層 + 其他基礎設施 |
| [SQLiteCpp](https://github.com/SRombauts/SQLiteCpp) | CMake | SQLite | 僅 SQLite |
| [SOCI](https://github.com/SOCI/soci) | CMake | SQLite, MySQL, PostgreSQL, Oracle | 較輕量的資料庫存取層 |

上述函式庫皆不原生支援 Meson，同樣面臨方案三的整合問題，但由於不依賴 Qt6，透過 `dependency()` + `cmake` method 的整合難度較低。

---

## 綜合評估結論

| 方案 | 開發成本 | 維護成本 | 整合完全度 | 建議 |
|---|---|---|---|---|
| 一：cmake.subproject() | 中 | 低 | 部分（Qt6 瓶頸） | 不優先 |
| 二：Wrap 覆蓋層 | 極高 | 高 | 完整 | 不建議 |
| 三：外部建置 + dependency() | 低 | 低 | 部分（需手動補傳遞依賴） | **較佳** |
| 四：Meson 原生替代 | 中 | 中 | 視函式庫而定 | 可考慮 |

**結論：TinyORM 導入 Meson 專案的可行性為偏低到中等。** 最可行的做法是方案三（外部 CMake/vcpkg 建置後透過 `dependency()` 引用），但前提是專案本身已經使用 Qt6，且團隊能接受混合 CMake/Meson 的工具鏈。若專案未使用 Qt6，TinyORM 的 Qt 依賴會是最大的進入障礙，建議考慮 sqlpp11 或 SOCI 等不需 Qt 的替代方案。

---

## 參考資料

[^readup]: silverqx. (n.d.). *TinyORM — Modern C++ ORM library*. GitHub. Retrieved 2026-09-25, from https://github.com/silverqx/TinyORM

[^readme]: silverqx. (n.d.). *TinyORM README*. GitHub. Retrieved 2026-09-25, from https://github.com/silverqx/TinyORM/blob/main/README.md

[^tests]: silverqx. (n.d.). *TinyORM features summary*. GitHub. Retrieved 2026-09-25, from https://github.com/silverqx/TinyORM/blob/main/docs/features-summary.mdx

[^license]: silverqx. (2024). *TinyORM LICENSE*. GitHub. Retrieved 2026-09-25, from https://github.com/silverqx/TinyORM/blob/main/LICENSE

[^deps]: silverqx. (n.d.). *TinyORM dependencies*. GitHub. Retrieved 2026-09-25, from https://github.com/silverqx/TinyORM/blob/main/docs/dependencies.mdx

[^qt5removal]: silverqx. (2024). *TinyORM v0.38.0 release — Qt 5.15 support removed*. GitHub. Retrieved 2026-09-25, from https://github.com/silverqx/TinyORM/releases

[^buildsystems]: silverqx. (n.d.). *Building TinyORM*. GitHub. Retrieved 2026-09-25, from https://github.com/silverqx/TinyORM/blob/main/docs/building/tinyorm.mdx

[^mesonsearch]: GitHub. (n.d.). *silverqx/TinyORM issues — "meson" search*. GitHub. Retrieved 2026-09-25, from https://github.com/silverqx/TinyORM/issues?q=meson

[^mesoncmake]: The Meson Build System. (n.d.). *CMake module*. MesonBuild.com. Retrieved 2026-09-25, from https://mesonbuild.com/CMake-module.html

[^wrapdb_rang3]: Meson WrapDB. (n.d.). *range-v3 0.12.0-1 wrap*. WrapDB. Retrieved 2026-09-25, from https://wrapdb.mesonbuild.com/v2/range-v3_0.12.0-1/range-v3.wrap

[^wrapdb_tabulate]: Meson WrapDB. (n.d.). *tabulate 1.5-1 wrap*. WrapDB. Retrieved 2026-09-25, from https://wrapdb.mesonbuild.com/v2/tabulate_1.5-1/tabulate.wrap

[^cmakelists513]: silverqx. (n.d.). *TinyORM CMakeLists.txt line 513 — pkg-config generation commented out*. GitHub. Retrieved 2026-09-25, from https://github.com/silverqx/TinyORM/blob/main/CMakeLists.txt

[^vcpkg_port]: Microsoft. (n.d.). *vcpkg port: tinyorm*. GitHub. Retrieved 2026-09-25, from https://github.com/microsoft/vcpkg/tree/master/ports/tinyorm

[^qtmsvc]: The Qt Company. (n.d.). *Qt for Windows — supported compilers*. Qt Documentation. Retrieved 2026-09-25, from https://doc.qt.io/qt-6/windows.html