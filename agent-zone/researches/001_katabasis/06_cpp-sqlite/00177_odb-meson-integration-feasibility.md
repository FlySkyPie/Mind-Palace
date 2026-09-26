# ODB (C++ ORM) 導入 Meson 專案的可行性評估

## 摘要

本報告評估將 ODB（Object-Relational Mapping for C++）整合至使用 Meson 建置系統之專案的技術可行性。ODB 是一個全自動的 C++ ORM 系統，透過 GCC 外掛架構解析帶有 `#pragma db` 註解的 C++ 標頭檔，自動產生資料庫存取碼。評估重點涵蓋技術整合可行性、授權限制、相依性管理與實際採用的障礙。

## 1. ODB 概述

ODB 由三個核心元件組成：ODB 編譯器 (`odb`)、通用執行時期函式庫 (`libodb`)，以及資料庫專用執行時期函式庫 (`libodb-<database>`)。ODB 編譯器本質上是一個真正的 C++ 編譯器（作為 GCC 外掛執行），解析標準 C++ 程式碼並產生 C++ 輸出檔，而非組合語言或機器碼[^odb-arch]。

ODB 支援五種資料庫後端：
- SQLite（3.7.4+）
- PostgreSQL（7.4+，透過 `libpq`）
- MySQL/MariaDB（5.0.3+，透過 `libmysqlclient`）
- Oracle（10.1+，透過 OCI）
- Microsoft SQL Server（2005+，透過 ODBC）[^odb-features]

ODB 的 C++ 標準支援涵蓋 C++98/03 至 C++26，產生的輸出碼為標準 C++，可被任何 C++ 編譯器編譯[^odb-manual-1.3]。

## 2. 建置系統相容性分析

### ODB 官方建置系統

ODB 專案自身使用 **`build2`** 作為其建置與套件管理系統，這是 ODB 官方唯一支援的建置方式。ODB 不提供官方的 CMake 模組、GNU Makefile 或 Meson wrap 檔案[^odb-build2]。

### ODB 編譯器作為命令列工具

ODB 編譯器可作為獨立命令列工具呼叫，基本呼叫語法為：

```
odb --database <db> [options] <header-file>
```

產生的輸出檔包含：
- `<name>-odb.hxx`（標頭檔）
- `<name>-odb.ixx`（行內函式檔）
- `<name>-odb.cxx`（原始碼檔）
- `<name>.sql`（選擇性產生的資料庫綱要）[^odb-manual-2.2]

這使得 ODB 可以透過任何建置系統的「自訂建置步驟」機制整合。

### Meson 的 `custom_target()` 整合路徑

Meson 可透過 `custom_target()` 或 `generator()` 來呼叫 ODB 編譯器。理論上的整合方式如下：

```meson
odb_compiler = find_program('odb', required: true)

person_odb = custom_target('person-odb',
  input: 'person.hxx',
  output: ['person-odb.hxx', 'person-odb.ixx', 'person-odb.cxx'],
  command: [
    odb_compiler,
    '--std', 'c++17',
    '--database', 'sqlite',
    '--generate-query',
    '--generate-schema',
    '--output-dir', meson.current_build_dir(),
    '@INPUT@'
  ],
)
```

產生後的 `.cxx` 檔可作為一般原始檔加入 `library()` 或 `executable()` 目標[^meson-custom-target]。

### ODB 函式庫的連結

ODB 不提供 `pkg-config` (`.pc`) 檔案或 CMake config 模組。在 Meson 中需使用編譯器的 `find_library()` 方法手動查找：

```meson
cpp = meson.get_compiler('cpp')
libodb_dep = cpp.find_library('odb', required: true)
libodb_sqlite_dep = cpp.find_library('odb-sqlite', required: true)
```

標頭檔路徑則需透過 `include_directories()` 手動指定，因為 ODB 預設安裝至 `/usr/local`[^odb-install]。

## 3. 現有整合案例

截至 2026 年 9 月，**沒有任何已知的公開專案**將 ODB 與 Meson 建置系統結合使用。ODB 不在 Meson WrapDB 中[^meson-wrapdb]。在 GitHub、Stack Overflow、Reddit 等平台上均未發現相關討論或程式碼範例。唯一找到的相關提及為一篇中國 CSDN 文章提到 Scraf 後端系統同時使用了 Meson 與 ODB，但未提供可重複使用的整合方案或公開程式碼[^csdn-scraf]。

## 4. 技術障礙與限制

### 4.1 GCC 外掛依賴

ODB 編譯器以 GCC 外掛形式實作，這意味著：
- 必須使用具備外掛支援的 GCC（需要 `gcc-N-plugin-dev` 或 `gcc-plugin-devel` 套件）
- 無法使用 Clang 或 MSVC 來編譯 ODB 編譯器本身
- 但 ODB 產生的 *輸出碼* 可在任何 C++ 編譯器上編譯[^odb-platforms]

### 4.2 缺少 Meson Wrap 或 Pkg-config 支援

ODB 不提供 `pkg-config` 檔案，也無 CMake `FindODB.cmake` 模組，更沒有 Meson wrap 檔案。這使得 Meson 整合需完全手動處理：
- 依賴偵測（需使用 `find_library()`）
- 標頭檔路徑管理（需使用 `include_directories()`）
- 版本相容性檢查（需自行實作）

### 4.3 安裝流程冗長

使用 ODB 的標準安裝流程需先安裝 `build2` 工具鏈，再透過 `bpkg` 編譯安裝 ODB 及其函式庫。這與 Meson 專案常見的 `subproject()` / wrap 依賴管理流程不一致。

### 4.4 授權限制

ODB 採用雙重授權模式：
- **GPL v2**（開放原始碼資料庫適用）
- **NCUEL**（非商業使用與評估授權）
- **CPL**（商業專屬授權，需付費，起價約 $1,500 USD）
- **FPL**（免費專屬授權，限 10,000 行產出碼，約 10-20 個類別）[^odb-license]

若專案為閉源商業軟體且不使用 FPL 資格，則需購買 CPL 授權。

## 5. 可行性評分

| 面向 | 評分 (1-5) | 說明 |
|------|-----------|------|
| 技術可行性 | ⭐⭐⭐ | `custom_target()` 可呼叫 ODB 編譯器，但無自動化支援 |
| 相依性管理 | ⭐⭐ | 需手動管理安裝與連結，無 wrap/pkg-config 支援 |
| 社群資源 | ⭐ | 無公開的 Meson 整合資源或範例 |
| 授權相容性 | ⭐⭐⭐ | 開源專案可行，閉源專案需付費 |
| 維護成本 | ⭐⭐ | 每次 ODB 升級或建置環境變更需手動調整 |
| 開發效率 | ⭐⭐⭐⭐ | ODB 本身減少手寫 SQL/ORM 程式碼的效益顯著 |

## 6. 結論與建議

### 可行但不便利

ODB 在技術層面上**可以**導入 Meson 專案。ODB 編譯器的命令列介面使其可以透過 `custom_target()` 或 `generator()` 與 Meson 整合，產生的 C++ 程式碼可正常編譯與連結。

### 主要成本

導入的主要障礙並非技術不可行，而是：
1. **無任何現成的整合套件或範本**，需從頭建立維護
2. 相依性管理完全手動，無法享受 Meson wrap/subproject 的自動化依賴解析
3. 若專案為閉源且超過 FPL 門檻，需考慮授權成本

### 替代方案

若團隊決定採用 ODB，建議：
- 考慮先從 SQLite 後端開始（無需資料庫伺服器，開發門檻低）
- 建立內部共用的 Meson 整合腳本（自訂 wrap 或 extracto）
- 評估其他 C++ ORM 方案（如 SOCI、sqlpp11、LimeReport）與 Meson 的相容性

### 結論

ODB 導入 Meson 專案在技術上**可行但尚未成熟**，需要自行處理建置整合、依賴管理與授權問題。對於對 ORM 功能有強烈需求的團隊，投入這些前期成本可能是合理的；但對於尋求開箱即用方案的小型專案，短期內建議優先考慮具備原生 Meson 支援的替代方案。

---

[^odb-arch]: Code Synthesis. (n.d.). ODB Architecture and Workflow. Retrieved 2026-09-25, from https://www.codesynthesis.com/products/odb/doc/manual.xhtml#1.1
[^odb-features]: Code Synthesis. (n.d.). ODB Features. Retrieved 2026-09-25, from https://www.codesynthesis.com/products/odb/features.xhtml
[^odb-manual-1.3]: Code Synthesis. (n.d.). Supported C++ Standards. Retrieved 2026-09-25, from https://www.codesynthesis.com/products/odb/doc/manual.xhtml#1.3
[^odb-manual-2.2]: Code Synthesis. (n.d.). Generating Database Support Code. Retrieved 2026-09-25, from https://www.codesynthesis.com/products/odb/doc/manual.xhtml#2.2
[^odb-build2]: Code Synthesis. (n.d.). Installing ODB with build2. Retrieved 2026-09-25, from https://codesynthesis.com/products/odb/doc/install-build2.xhtml
[^meson-custom-target]: The Meson Build System. (n.d.). Reference manual — custom_target. Retrieved 2026-09-25, from https://mesonbuild.com/Reference-manual_functions.html#custom_target
[^odb-install]: Code Synthesis. (n.d.). ODB Installation. Retrieved 2026-09-25, from https://www.codesynthesis.com/products/odb/download.xhtml
[^meson-wrapdb]: The Meson Build System. (n.d.). WrapDB Projects. Retrieved 2026-09-25, from https://mesonbuild.com/Wrapdb-projects.html
[^csdn-scraf]: CSDN. (n.d.). Scraf 後端系統架構. Retrieved 2026-09-25, from https://wenku.csdn.net/doc/4s8arxo13n
[^odb-platforms]: Code Synthesis. (n.d.). ODB Supported Platforms. Retrieved 2026-09-25, from https://www.codesynthesis.com/products/odb/platforms.xhtml
[^odb-license]: Code Synthesis. (n.d.). ODB License Information. Retrieved 2026-09-25, from https://www.codesynthesis.com/products/odb/license.xhtml