# C++ SQLite 資料庫連接與操作函式庫調查

## 概述

SQLite 是全球部署最廣的資料庫引擎，作為嵌入式資料庫廣泛應用於桌面、行動裝置與 IoT 環境。C++ 開發者有多種方式操作 SQLite，從輕量級的 C API 包裝、類型安全的查詢建構器，到完整的 ORM 框架，各有所長。本報告整理目前主流且活躍維護的 C++ SQLite 函式庫，供技術選型參考。

## 函式庫總覽

### 1. SQLiteCpp (SQLiteC++)

- **GitHub**：[SRombauts/SQLiteCpp](https://github.com/SRombauts/SQLiteCpp)[^sqlitecpp-repo]
- **授權**：MIT License
- **活躍度**：⭐ ~2,800，持續活躍維護（2025 年仍有更新）
- **C++ 版本需求**：C++11

**簡介**：輕量級的 C++11 RAII 包裝，圍繞 SQLite3 C API 建構。使用 RAII 手法管理資源，並以例外處理錯誤。

**主要特色**：
- `Database`、`Statement`、`Transaction` 三大類別
- 支援 range-based `for` loop（透過 `RowIterator`）
- Prepared statements 與參數綁定
- 交易支援（begin/commit/rollback）
- 欄位 metadata 查詢
- CMake、Meson、vcpkg 安裝支援
- 依賴極少（C++11 STL + SQLite3）

### 2. sqlpp11

- **GitHub**：[rbock/sqlpp11](https://github.com/rbock/sqlpp11)[^sqlpp11-repo]
- **SQLite 連接器**：[rbock/sqlpp11-connector-sqlite3](https://github.com/rbock/sqlpp11-connector-sqlite3)[^sqlpp11-sqlite]
- **授權**：BSD 2-Clause License
- **活躍度**：⭐ ~2,600，作者建議新專案移往 [sqlpp23](https://github.com/rbock/sqlpp23)
- **C++ 版本需求**：C++14/17

**簡介**：嵌入式領域特定語言（EDSL），在編譯期對 SQL 查詢進行語法、型別與名稱檢查，減少執行期錯誤。

**主要特色**：
- 編譯期型別安全的 SQL 查詢建構
- 支援靜態與動態查詢
- 查詢結果以具名成員的結構體回傳
- 支援 JOIN、子查詢、聚合函數、ORDER BY、GROUP BY
- 資料庫無關核心，需搭配各資料庫連接器
- 透過 `ddl2cpp` Python 腳本自動產生 DDL

### 3. sqlite_orm

- **GitHub**：[fnc12/sqlite_orm](https://github.com/fnc12/sqlite_orm)[^sqlite-orm-repo]
- **授權**：AGPL-3.0（開源版）/ MIT（付費版，$50）
- **活躍度**：⭐ ~2,700，持續活躍
- **C++ 版本需求**：C++14/17/20

**簡介**：輕量的 header-only ORM，直接將 C++ 結構體映射到 SQLite 表格，完全以純 C++ 表達式操作資料庫，無需撰寫原始 SQL 字串。

**主要特色**：
- 單一 header file，極簡整合
- 完整 CRUD 操作（get、insert、update、remove、replace）
- `sync_schema()` 自動 schema 遷移（比對並更新）
- 支援 INNER、LEFT、CROSS、LEFT OUTER JOIN
- 強型別的 prepared statements
- 交易支援（顯式、lambda、guard 三種形式）
- 聚合函數（COUNT、AVG、SUM、MIN、MAX、GROUP_CONCAT、TOTAL）
- 複雜 WHERE 條件（`=`、`!=`、`>`、`<`、`IN`、`BETWEEN`、`LIKE`、`IS NULL`）
- BLOB、自訂型別綁定、使用者自訂函數

### 4. SOCI

- **GitHub**：[SOCI/soci](https://github.com/SOCI/soci)[^soci-repo]
- **授權**：Boost Software License 1.0
- **活躍度**：⭐ ~1,600，成熟穩定（v4.x 系列）* 開發於 CERN
- **C++ 版本需求**：C++14（4.x 系列）

**簡介**：抽象資料庫存取層，提供統一的物件導向介面，支援多種後端資料庫。適合需要在不同資料庫間切換的專案。

**主要特色**：
- 單一 API 操作 SQLite、PostgreSQL、MySQL、Oracle、DB2、Firebird、ODBC
- SQLite3 後端可選用系統 `libsqlite3` 或內建 SQLite3
- Row-by-row 與 bulk 操作
- BLOB 支援
- 型別安全的 `into` 與 `use` 語法
- Session pool
- 成熟專案，在 CERN 控制系統中使用

### 5. sqlite3pp (iwongu)

- **GitHub**：[iwongu/sqlite3pp](https://github.com/iwongu/sqlite3pp)[^sqlite3pp-repo]
- **授權**：MIT License
- **活躍度**：⭐ ~643，僅 `headeronly_src` 目錄持續維護
- **C++ 版本需求**：C++11

**簡介**：C++11 SQLite3 包裝，提供 `database`、`command`、`query`、`transaction` 類別，並支援 iterator 風格的資料提取。

**主要特色**：
- Stream-like 參數綁定（`cmd.binder() << value`）
- 具名參數綁定
- Range-based `for` loop 查詢結果
- 備份支援（含進度回呼）
- 資料庫 attach
- 自訂 SQL 函數與聚合（lambda 支援）
- 可載入延伸模組

### 6. sqlite_modern_cpp

- **GitHub**：[SqliteModernCpp/sqlite_modern_cpp](https://github.com/SqliteModernCpp/sqlite_modern_cpp)[^sqlite-modern-repo]
- **授權**：MIT License
- **活躍度**：⭐ ~949，維護模式
- **C++ 版本需求**：C++14/17

**簡介**：Header-only C++14 包裝，使用 `operator<<` / `operator>>` 進行直觀的查詢建構與結果提取，搭配 lambda 處理結果集。

**主要特色**：
- Header-only（單一 include 目錄）
- Stream-like 語法（`db << "query" >> callback`）
- Lambda 式 row 處理
- Prepared statements（可保持與重複使用）
- 共享資料庫連線管理
- NULL 處理支援 `std::unique_ptr<T>` 與 `std::optional<T>`（C++17）
- BLOB 支援 `std::vector<T>`
- Variant 型別支援（`std::variant`）
- SQLCipher 支援（加密資料庫）
- 自訂 SQL 函數（`db.define()`）
- NDK（Android）支援

### 7. ODB (Code Synthesis)

- **網站**：[codesynthesis.com/products/odb/](https://www.codesynthesis.com/products/odb/)[^odb-site]
- **GitHub 鏡像**：[codesynthesis-com/odb](https://github.com/codesynthesis-com/odb)
- **授權**：GPL-2.0（開源版）/ NCUEL（非商業）/ CPL（商業授權，約 $1,500+）/ FPL（≤10K 行程式碼免費專屬授權）
- **活躍度**：持續活躍（v2.6.0 於 2026 年釋出）

**簡介**：完整的跨資料庫 ORM 系統，使用真實 C++ 編譯器（GCC plugin）從 C++ 類別宣告自動產生資料庫操作程式碼。

**主要特色**：
- 完整的 ORM — 無需手寫 mapping 程式碼
- 自動資料庫 schema 產生
- 型別安全的查詢 API（物件導向，非字串型）
- 支援 views、容器、智慧指標（Boost、Qt profiles）
- 資料庫 schema 演進（migration）
- 跨資料庫可攜性（SQLite、PostgreSQL、MySQL、Oracle、SQL Server）
- 零 per-object 記憶體開銷
- 商業級支援

### 8. CppSQLite (NeoSmart)

- **GitHub**：[NeoSmart/CppSQLite](https://github.com/NeoSmart/CppSQLite)
- **授權**：BSD License
- **活躍度**：⭐ ~161，維護模式

**簡介**：極簡的跨平台 C++ SQLite 包裝，單一 `.cpp`/`.h` 檔案即可加入專案。源自知名的 Code Project 函式庫。

## 比較總表

| 函式庫 | 設計理念 | ⭐ | 授權 | C++ 標準 | header-only | 是否需要 SQL 字串 |
|---|---|---|---|---|---|---|
| SQLiteCpp | 輕量 RAII 包裝 | 2.8k | MIT | C++11 | 否 | 是 |
| sqlpp11 | 編譯期型別安全 EDSL | 2.6k | BSD-2 | C++14/17 | 否 | 否（EDSL） |
| sqlite_orm | ORM / 物件映射 | 2.7k | AGPL-3.0 / MIT($) | C++14/17/20 | 是 | 否 |
| SOCI | 多資料庫抽象層 | 1.6k | Boost 1.0 | C++14 | 否 | 是 |
| sqlite3pp | 輕量包裝 + 延伸 | 0.6k | MIT | C++11 | 是 | 是 |
| sqlite_modern_cpp | Header-only 包裝 | 0.9k | MIT | C++14/17 | 是 | 是 |
| ODB | 全功能 ORM | — | GPL / 商業 | C++98~17 | 否 | 否（ORM） |
| CppSQLite | 極簡包裝 | 0.1k | BSD | C++98 | 否 | 是 |

## 技術選型建議

### 依專案需求

| 需求場景 | 推薦函式庫 | 理由 |
|---|---|---|
| 簡單的 SQLite 操作，最少學習成本 | SQLiteCpp | 輕量、MIT 授權、RAII 設計直觀、生態成熟 |
| 編譯期型別安全，減少 SQL 錯誤 | sqlpp11 | EDSL 設計，編譯期捕獲語法/型別錯誤 |
| 不想寫任何 SQL 字串，用 C++ struct 直接映射 | sqlite_orm | Header-only ORM，開發速度快 |
| 需要支援多種資料庫（PostgreSQL/MySQL/SQLite 切換） | SOCI | 統一 API，後端可插拔 |
| 資源受限環境（嵌入式/IoT） | sqlite3pp 或 CppSQLite | 極簡、依賴少 |
| 企業級專案，需要完整 ORM + 商業支援 | ODB | 成熟、跨資料庫、有商業授權選項 |
| 喜歡 stream 語法、lambda 處理 | sqlite_modern_cpp | 直觀的 operator<< 語法 |

### 授權注意事項

- **sqlite_orm** 的 AGPL-3.0 授權對商業軟體有較嚴格的 copyleft 要求。若需閉源商用，須支付 $50 購買 MIT 授權。選用前務必確認合規性。
- **ODB** 的 GPL-2.0 開源版同樣有 copyleft 限制，商用需購買授權。另有 FPL（Free Proprietary License）方案提供 ≤10K 行程式碼的免費商用。
- **SQLiteCpp、sqlpp11、SOCI、sqlite3pp、sqlite_modern_cpp、CppSQLite** 均為寬鬆授權（MIT/BSD/Boost），商用友善。

## 注意事項

- SQLite 本身為 public domain（[sqlite.org/copyright.html](https://www.sqlite.org/copyright.html)），不受授權限制，但 C++ 包裝函式庫各有其授權條款，需分別考量。
- 多數函式庫底層仍依賴 SQLite3 C API，因此部份函式庫需額外安裝 `libsqlite3-dev` 或內建 amalgamation 原始碼。
- 本報告所列 stars 數與活躍度為 2026-09 近似值，實際數字可能隨時間變動。

## 參考資料

[^sqlitecpp-repo]: SRombauts. (n.d.). SQLiteCpp. GitHub. Retrieved 2026-09-19, from https://github.com/SRombauts/SQLiteCpp
[^sqlpp11-repo]: rbock. (n.d.). sqlpp11. GitHub. Retrieved 2026-09-19, from https://github.com/rbock/sqlpp11
[^sqlpp11-sqlite]: rbock. (n.d.). sqlpp11-connector-sqlite3. GitHub. Retrieved 2026-09-19, from https://github.com/rbock/sqlpp11-connector-sqlite3
[^sqlite-orm-repo]: fnc12. (n.d.). sqlite_orm. GitHub. Retrieved 2026-09-19, from https://github.com/fnc12/sqlite_orm
[^soci-repo]: SOCI. (n.d.). SOCI. GitHub. Retrieved 2026-09-19, from https://github.com/SOCI/soci
[^sqlite3pp-repo]: iwongu. (n.d.). sqlite3pp. GitHub. Retrieved 2026-09-19, from https://github.com/iwongu/sqlite3pp
[^sqlite-modern-repo]: SqliteModernCpp. (n.d.). sqlite_modern_cpp. GitHub. Retrieved 2026-09-19, from https://github.com/SqliteModernCpp/sqlite_modern_cpp
[^odb-site]: Code Synthesis. (n.d.). ODB — C++ Object-Relational Mapping. Retrieved 2026-09-19, from https://www.codesynthesis.com/products/odb/