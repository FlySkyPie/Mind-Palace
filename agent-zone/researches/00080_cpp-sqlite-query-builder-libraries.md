# C++ SQLite Query Builder 函式庫調查（非 ORM）

## 概述

在 C++ 生態系中，操作 SQLite 有多種選擇。開發者若想**程式化建構 SQL 查詢**（而非手寫 SQL 字串），但**不想使用完整 ORM**，有以下幾個選項。本報告聚焦於此特定類別。

## 函式庫總覽

### 1. sqlpp11 — 類型安全 EDSL 查詢建構器

- **GitHub**：[rbock/sqlpp11](https://github.com/rbock/sqlpp11)[^sqlpp11-repo]
- **SQLite 連接器**：[rbock/sqlpp11-connector-sqlite3](https://github.com/rbock/sqlpp11-connector-sqlite3)[^sqlpp11-sqlite]
- **授權**：BSD 2-Clause License
- **活躍度**：⭐ ~2,600，目前處於維護模式（作者建議新專案移往 sqlpp23）
- **C++ 標準需求**：C++14/17

**簡介**：嵌入式領域特定語言（EDSL），在**編譯期**對 SQL 查詢進行語法、型別與名稱檢査。這是最符合「query builder, not ORM」需求的函式庫——無需撰寫原始 SQL 字串，純粹以 C++ 表達式程式化建構查詢。

**查詢建構能力**：
```cpp
// 編譯期型別安全的查詢建構
auto result = db(select(foo.name, foo.hasFun)
                  .from(foo)
                  .where(foo.id > 17 and foo.name.like("%bar%"))
                  .order_by(foo.name.asc())
                  .limit(10));
```

- 完整的 `SELECT`、`INSERT`、`UPDATE`、`DELETE` 子句建構
- 編譯期檢查語法、型別、名稱錯誤
- 支援 JOIN、子查詢、聚合函數、ORDER BY、GROUP BY、HAVING、LIMIT/OFFSET
- 靜態與動態查詢皆可
- 結果迭代為具名 struct 成員
- 提供 `ddl2cpp` 腳本從 DDL 自動產生 C++ table 定義

**注意事項**：
- 核心與資料庫連接器分離（需額外引入 `sqlpp11-connector-sqlite3`）
- SQLite3 連接器功能完整（支援 transactions、prepared statements、BLOB）
- 另支援 PostgreSQL、MySQL、MariaDB、SQLCipher 連接器
- 編譯時間較長（大量模板）

---

### 2. sqlpp23 — 下一代版本

- **GitHub**：[rbock/sqlpp23](https://github.com/rbock/sqlpp23)[^sqlpp23-repo]
- **授權**：BSD 2-Clause License
- **活躍度**：⭐ ~179，活躍開發中（2,361 commits）
- **C++ 標準需求**：C++23（需 clang 20.1 / gcc 14.2 / MSVC 19.44+）

**簡介**：sqlpp11 的重新實作版本，充分利用 C++23 語言特性。相同的 query builder 哲學，但更現代、更好的編譯錯誤訊息、支援 C++20 modules。

**與 sqlpp11 的差異**：
- 更簡潔的 API 設計
- 更好的編譯期錯誤訊息
- 支援 C++20 modules（更快的編譯）
- 仍在活躍開發，API 尚未完全穩定
- 同樣支援 SQLite3、PostgreSQL、MySQL 等連接器

---

### 3. SOCI — 多資料庫抽象層（部分查詢建構）

- **GitHub**：[SOCI/soci](https://github.com/SOCI/soci)[^soci-repo]
- **授權**：Boost Software License 1.0
- **活躍度**：⭐ ~1,600，成熟穩定（v4.x 系列），起源於 CERN
- **C++ 標準需求**：C++14

**簡介**：統一的資料庫存取抽象層。不是純正的 query builder——查詢結構仍使用**原始 SQL 字串**，但提供**型別安全的參數綁定與結果提取**。

**查詢建構相關能力**：
```cpp
sql << "select name, age from persons where id = :id",
    into(name), into(age), use(id);
```

- 動態查詢組合：使用 `sql << "SELECT ..."` 串接子句
- 型別安全的 `into()` 與 `use()` 綁定
- 支援 prepared statements
- Row-by-row 迭代與 bulk 操作
- Session pool、交易管理

**與 sqlpp11 的關鍵差異**：
- ❌ 查詢語法本身仍是原始 SQL 字串
- ✅ 綁定參數與提取結果是型別安全的
- ✅ 後端可插拔（SQLite、PostgreSQL、MySQL、Oracle、ODBC 等）
- 適合需要在多種資料庫間切換的專案

---

### 4. ODB Query API — ORM 框架內的查詢建構器

- **網站**：[codesynthesis.com/products/odb/](https://www.codesynthesis.com/products/odb/)[^odb-site]
- **授權**：GPL-2.0（開源版）/ NCUEL / CPL（商業授權）/ FPL（≤10K 行程式碼免費）
- **C++ 標準需求**：C++98~17

**簡介**：ODB 是完整的 ORM，但其查詢 API 提供了一個程式化查詢建構機制，可以在不用完整的 ORM mapping 的情況下使用。

**查詢建構 API**：
```cpp
query q = query::id > 17 && query::name.like("%bar%");
```

然而，由於 ODB 本質上是 ORM，且依賴外部程式碼產生器（GCC plugin），對純 query builder 需求而言過於厚重。

---

## 比較總表

| 函式庫 | Query Builder | 原始 SQL 字串 | ORM | SQLite 支援 | 授權 | C++ 標準 | ⭐ | 狀態 |
|---|---|---|---|---|---|---|---|---|
| **sqlpp11** | ✅ EDSL，完整 | 不需要 | ❌ 非 ORM | ✅ 內建連接器 | BSD-2 | C++14/17 | 2.6k | 維護模式 |
| **sqlpp23** | ✅ EDSL，完整 | 不需要 | ❌ 非 ORM | ✅ 內建連接器 | BSD-2 | C++23 | 179 | 活躍開發 |
| **SOCI** | ⚠️ 部分（SQL 字串+型別安全綁定） | 需要 | ❌ 非 ORM | ✅ 後端 | Boost 1.0 | C++14 | 1.6k | 成熟穩定 |
| **ODB Query** | ✅ 完整查詢建構 | 不需要 | ✅ 完整 ORM | ✅ 後端 | GPL/商業 | C++98~17 | — | 成熟穩定 |

## 技術選型建議

| 需求場景 | 推薦 | 理由 |
|---|---|---|
| 不想寫任何 SQL 字串，編譯期型別安全 | **sqlpp11** | 最純正的 query builder EDSL，編譯期捕獲錯誤 |
| 新專案，可以使用 C++23 | **sqlpp23** | 下一代 sqlpp，更現代的設計 |
| 需要多資料庫支援，可接受 SQL 字串 | **SOCI** | 統一的跨資料庫 API，CERN 長期維護 |
| 僅需簡單查詢，不想學新 DSL | SQLiteCpp 等 | 直接寫 SQL 字串，RAII 風格包裝 |

## 重點結論

若你尋找的是「**純 query builder，非 ORM**」，sqlpp11（或 sqlpp23）是唯一真正符合條件的選擇。它完全消除原始 SQL 字串，以類型安全的 C++ 表達式建構查詢，且沒有任何 ORM 的包袱。

SOCI 可視為折衷方案：查詢結構仍是 SQL 字串，但參數與結果的綁定型別安全。

## 參考資料

[^sqlpp11-repo]: rbock. (n.d.). sqlpp11. GitHub. Retrieved 2026-09-20, from https://github.com/rbock/sqlpp11

[^sqlpp11-sqlite]: rbock. (n.d.). sqlpp11-connector-sqlite3. GitHub. Retrieved 2026-09-20, from https://github.com/rbock/sqlpp11-connector-sqlite3

[^sqlpp23-repo]: rbock. (n.d.). sqlpp23. GitHub. Retrieved 2026-09-20, from https://github.com/rbock/sqlpp23

[^soci-repo]: SOCI. (n.d.). SOCI. GitHub. Retrieved 2026-09-20, from https://github.com/SOCI/soci

[^odb-site]: Code Synthesis. (n.d.). ODB — C++ Object-Relational Mapping. Retrieved 2026-09-20, from https://www.codesynthesis.com/products/odb/