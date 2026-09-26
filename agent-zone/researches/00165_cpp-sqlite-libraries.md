# C++ SQLite 函式庫調查報告

## 概述

本報告調查目前主流的 C++ SQLite 包裝/ORM 函式庫，評估其功能、授權、社群活躍度及適用場景，協助開發者為專案選擇合適的函式庫。

## 函式庫列表

### 1. SQLiteCpp (SQLiteC++)

| 屬性 | 內容 |
|---|---|
| **倉庫** | SRombauts/SQLiteCpp[^sqlitecpp] |
| **Star** | ~2.8k |
| **授權** | MIT |
| **C++ 標準** | C++17 (4.x)，C++11 (3.4.x 舊分支) |
| **型態** | 編譯式函式庫 (CMake, Meson) |
| **主要文件** | Doxygen 文件與範例[^sqlitecpp_docs] |

**特色：**
- RAII 設計，以例外處理錯誤
- 封裝 SQLite C API 為直觀的 C++ 類別：`Database`、`Statement`、`Transaction`、`Column`
- 支援 range-based for 迴圈 (`RowIterator`)
- 支援 prepared statements、交易、BLOB
- 可透過 vcpkg 安裝
- 跨平台支援（Ubuntu, Windows, macOS, GCC, Clang, MSVC, MinGW）

**優點：**
- 成熟、維護良好（2012 年至今，1,182+ commits）
- MIT 授權，商用無限制
- 文件與範例齊全
- 跨平台支援廣泛
- 測試覆蓋率高（Coveralls, Coverity）

**缺點：**
- 需要連結編譯（非 header-only）
- 非 ORM — 仍需手寫 SQL 字串
- C++17 起跳，對舊專案可能有限制

---

### 2. sqlite_orm

| 屬性 | 內容 |
|---|---|
| **倉庫** | fnc12/sqlite_orm[^sqlite_orm] |
| **Star** | ~2.7k |
| **授權** | AGPL-3.0（開源）/ MIT（商用需付費 $50） |
| **C++ 標準** | C++14/17/20 |
| **型態** | Header-only（單一 header） |
| **主要文件** | 官方文件[^sqlite_orm_docs] |

**特色：**
- **無原始 SQL 字串** — 使用 C++ 運算子進行型別安全查詢建構
- 完整 CRUD：`insert`、`get`、`update`、`remove`、`replace`
- 支援 JOIN（CROSS, INNER, LEFT, LEFT OUTER）
- 交易（顯式、lambda-based、guard-based）
- 聚合函數：`AVG`、`COUNT`、`MAX`、`MIN`、`SUM`、`GROUP_CONCAT`
- WHERE 條件：`=`、`!=`、`>`、`<`、`IN`、`BETWEEN`、`LIKE`
- ORDER BY, LIMIT, OFFSET, GROUP BY, DISTINCT
- Migration / schema 同步 (`sync_schema()`)
- 自訂型別繫結、BLOB（`std::vector<char>`）
- 外鍵與複合鍵支援
- 使用者自訂函數
- 記憶體資料庫支援

**優點：**
- 完全型別安全的查詢建構，無 SQL 字串
- Header-only，無需編譯
- 功能極其豐富（JOIN, migration, 聚合函數, prepared statements）
- 語法直觀、宣告式
- STL 相容

**缺點：**
- **雙重授權** — AGPL 開源；商用需購買 MIT 授權（$50）
- Template 大量使用 → 編譯時間較長
- 需要外部 libsqlite3
- C++14 最低要求

---

### 3. sqlite_modern_cpp

| 屬性 | 內容 |
|---|---|
| **倉庫** | SqliteModernCpp/sqlite_modern_cpp[^sqlite_modern] |
| **Star** | ~949 |
| **授權** | MIT |
| **C++ 標準** | C++14（可選 C++17 功能） |
| **型態** | Header-only |

**特色：**
- 使用 `<<` 和 `>>` 運算子的串流風格 API
- Lambda-based row handler 處理 SELECT 結果
- 可重複使用的 prepared statements
- BLOB 支援（`std::vector<T>`）
- NULL 處理：`std::unique_ptr<T>` 或 `std::optional<T>`（C++17）
- `std::variant` 支援彈性欄位型別（C++17）
- **原生 SQLCipher 支援** — 加密資料庫
- 自訂 SQL 函數（C++ lambda）
- 錯誤日誌（`sqlite::error_log`）
- NDK 支援（Android）

**優點：**
- 極簡潔、表達力強的 API（`<<` / `>>` 運算子）
- Header-only，零建置設定
- 原生 SQLCipher 加密
- Prepared statement 重用支援佳
- UTF-16 與 UTF-8 字串支援

**缺點：**
- 無法一次執行多條陳述式
- 非 ORM — 無自動 schema 產生
- Header-only 增加編譯時間
- 社群較小
- 僅支援例外處理（無 error-code 替代方案）

---

### 4. hiberlite

| 屬性 | 內容 |
|---|---|
| **倉庫** | paulftw/hiberlite[^hiberlite] |
| **Star** | ~722 |
| **授權** | BSD-3-Clause |
| **C++ 標準** | C++11 |
| **型態** | 編譯式函式庫（src/） |

**特色：**
- Boost.Serialization 風格 API — 透過 `hibernate()` 方法宣告持久化
- 惰性載入（`bean_ptr`，物件在首次存取時載入）
- 一對多、多對多關聯支援
- Active-record 風格模式
- 自動 schema 產生（從類別定義建立 CREATE TABLE）
- 無需繼承基底類別
- 無需程式碼產生器或預處理器

**優點：**
- 入門極簡單 — 只需為類別加入 `hibernate()` 方法
- 內建惰性載入
- 自動建立 schema
- BSD-3 寬鬆授權
- 支援關聯

**缺點：**
- **維護不佳** — 僅 59 次 commits，多年未更新
- 需要修改類別（加入 `friend class hiberlite::access` 與 `hibernate()` 方法）
- 非 header-only
- 功能遠少於 sqlite_orm 或 SQLiteCpp
- 文件與範例有限

---

### 5. sqlite3pp

| 屬性 | 內容 |
|---|---|
| **倉庫** | iwongu/sqlite3pp[^sqlite3pp] |
| **Star** | ~643 |
| **授權** | MIT |
| **C++ 標準** | C++11 |
| **型態** | Header-only（headeronly_src） |

**特色：**
- 類別：`database`、`command`、`query`、`transaction`
- 查詢結果支援 range-based for 迴圈
- 串流風格繫結器（`cmd.binder() << value`）
- 具名與位置參數繫結
- Copy/nocopy 繫結語意
- **SQLite 備份支援**（含進度回呼）
- **回呼鉤子**（commit handler, update handler）
- **自訂函數與聚合**（`ext::function`, `ext::aggregate`）
- **可載入 extension 支援**
- 資料庫附加（ATTACH）支援

**優點：**
- 串流繫結語法簡潔可讀
- 良好的備份支援
- 函數與聚合註冊
- MIT 授權
- 備份進度回呼

**缺點：**
- 僅 `headeronly_src` 目錄持續維護
- 文件有限
- 非 ORM
- 僅 C++11
- 社群較小

---

### 6. vsqlite++

| 屬性 | 內容 |
|---|---|
| **倉庫** | vinzenz/vsqlite--[^vsqlitepp] |
| **Star** | ~32 |
| **授權** | BSD-3-Clause |
| **C++ 標準** | C++20 |
| **型態** | 編譯式函式庫（CMake） |
| **主要文件** | 官方網站[^vsqlitepp_docs] |

**特色：**
- RAII-first 設計，使用顯式工廠建立檔案/記憶體/URI 連線
- **執行緒感知連線池**
- **陳述式快取**（LRU，可設定容量）
- **WAL & WAL2 支援**（含快照工具）
- **Session & changeset 追蹤**（用於複寫）
- **序列化**（資料庫 ↔ 位元組）
- **JSON & FTS5 輔助工具**（路徑建構、match/rank 函數）
- **使用者自訂函數**（C++ lambda）
- 型別安全繫結：`std::optional`、`std::chrono::time_point`、列舉
- Prepared statements 含顯式執行狀態機
- Debian, RPM, Arch Linux 套件產生
- Conan, vcpkg 支援
- Symlink 安全政策

**優點：**
- 最先進的 C++20 設計
- 功能極豐富 — 連線池、陳述式快取、sessions、快照
- 文件完善（尤其是執行緒、連線池、安全性）
- 顯式、安全的工廠開啟方式
- 內建 JSON 與 FTS 工具

**缺點：**
- **社群極小**（32 stars）
- 需要 C++20 編譯器
- 對簡單使用案例過度設計
- 非 header-only
- 較新，尚未充分驗證

---

## 比較表格

| 函式庫 | Star | 授權 | C++ 標準 | 型態 | 主要優勢 |
|---|---|---|---|---|---|
| **SQLiteCpp** | 2.8k | MIT | C++17 | 編譯式 | 成熟、文件完善、RAII 包裝 |
| **sqlite_orm** | 2.7k | AGPL/MIT | C++17 | Header-only | 完整 ORM、無 SQL 字串、功能最豐富 |
| **sqlite_modern_cpp** | 949 | MIT | C++14 | Header-only | 簡潔 `<<`/`>>` API、SQLCipher |
| **hiberlite** | 722 | BSD-3 | C++11 | 編譯式 | 序列化風格 API、惰性載入 |
| **sqlite3pp** | 643 | MIT | C++11 | Header-only | 備份、自訂函數、整潔繫結器 |
| **vsqlite++** | 32 | BSD-3 | C++20 | 編譯式 | 連線池、WAL2、sessions、JSON/FTS |

---

## 使用場景建議

| 使用場景 | 最佳選擇 |
|---|---|
| 簡單包裝、低學習曲線 | SQLiteCpp |
| 完整 ORM、不想寫 SQL 字串 | sqlite_orm |
| 簡潔表達力（串流風格） | sqlite_modern_cpp |
| 加密資料庫（SQLCipher） | sqlite_modern_cpp |
| 商業專案（寬鬆授權） | SQLiteCpp 或 sqlite_modern_cpp |
| C++20 專案、進階功能 | vsqlite++ |
| 物件持久化 / ActiveRecord 風格 | hiberlite |
| 自訂 SQL 函數 / 聚合 | sqlite3pp 或 vsqlite++ |

---

## 結論

若專案需要成熟穩定的 SQLite 包裝函式庫且無 ORM 需求，**SQLiteCpp** 是最安全的選擇，其 MIT 授權對商用專案友善。若追求型別安全的 ORM 體驗且專案為開源，**sqlite_orm** 提供最全面的功能。加密需求則建議 **sqlite_modern_cpp**，其原生 SQLCipher 支援為主要優勢。對於追求現代 C++20 技術的專案，**vsqlite++** 雖社群尚小但功能設計最前瞻。

[^sqlitecpp]: SRombauts. (n.d.). *SQLiteCpp*. Retrieved 2026-09-25, from https://github.com/SRombauts/SQLiteCpp
[^sqlitecpp_docs]: SRombauts. (n.d.). *SQLiteCpp Documentation*. Retrieved 2026-09-25, from http://srombauts.github.io/SQLiteCpp/
[^sqlite_orm]: fnc12. (n.d.). *sqlite_orm*. Retrieved 2026-09-25, from https://github.com/fnc12/sqlite_orm
[^sqlite_orm_docs]: fnc12. (n.d.). *sqlite_orm Documentation*. Retrieved 2026-09-25, from https://fnc12.github.io/sqlite_orm/
[^sqlite_modern]: SqliteModernCpp. (n.d.). *sqlite_modern_cpp*. Retrieved 2026-09-25, from https://github.com/SqliteModernCpp/sqlite_modern_cpp
[^hiberlite]: paulftw. (n.d.). *hiberlite*. Retrieved 2026-09-25, from https://github.com/paulftw/hiberlite
[^sqlite3pp]: iwongu. (n.d.). *sqlite3pp*. Retrieved 2026-09-25, from https://github.com/iwongu/sqlite3pp
[^vsqlitepp]: vinzenz. (n.d.). *vsqlite++*. Retrieved 2026-09-25, from https://github.com/vinzenz/vsqlite--
[^vsqlitepp_docs]: vinzenz. (n.d.). *vsqlite++ Documentation*. Retrieved 2026-09-25, from https://vsqlite.virtuosic-bytes.com/