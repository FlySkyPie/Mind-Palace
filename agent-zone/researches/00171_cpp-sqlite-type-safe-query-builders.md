# C++ SQLite 型別安全查詢建構器庫研究報告

## 概述

本報告針對 C++ SQLite 函式庫進行系統性調查，重點評比 **編譯期型別安全 (compile-time type safety)** 與 **查詢建構器 (query builder)** 模式的函式庫，而非完整 ORM (Object-Relational Mapping) 方案。

## 評比標準

1. **編譯期型別安全**：能否在編譯期擷取 SQL 語法錯誤、型別不匹配、欄位名稱錯誤
2. **查詢建構器風格**：是否以 C++ 表達式建立查詢，而非撰寫原始 SQL 字串
3. **資料庫支援**：是否僅支援 SQLite 或支援多後端
4. **授權條款**：是否適合商業/開源專案使用
5. **生態活躍度**：GitHub 星數、提交數、維護狀態

## 前三名推薦 (型別安全 + 查詢建構器)

### 1. sqlpp11 — 編譯期型別安全的業界標竿

| 指標 | 數值 |
|---|---|
| GitHub | [rbock/sqlpp11](https://github.com/rbock/sqlpp11)[^sqlpp11] |
| Stars | ⭐ 2,600 |
| Forks | 362 |
| Commits | 1,540 |
| C++ 標準 | C++11 (核心), C++14+ 建議 |
| 授權 | BSD 2-Clause |
| Header-Only | 是 |
| 支援後端 | SQLite3, PostgreSQL, MySQL, MariaDB, SQLCipher |

**設計哲學**：嵌入式領域特定語言 (Embedded Domain Specific Language, EDSL)。SQL 語法透過 C++ 模板元程式設計直接嵌入 C++ 中。這是最接近「查詢建構器」而非 ORM 的方案。[^sqlpp11_docs]

**編譯期型別安全**：所有受評函式庫中最強的。編譯期驗證包括：
- SQL 語法錯誤
- 型別不匹配（如字串欄位與整數比較）
- 欄位/表格名稱錯誤
- 語義錯誤（如 FROM 缺少表格、INSERT 缺少必要欄位）
- 部分 SQL 方言差異

**範例**：
```cpp
for (const auto& row : db(select(foo.name, foo.hasFun)
    .from(foo).where(foo.id > 17 and foo.name.like("%bar%")))) {
    std::string name = row.name;
    bool hasFun = row.hasFun;
}
```

**⚠️ 2025-06 作者公告**：作者建議新專案考慮遷移至 sqlpp23 (C++23 後繼者)。sqlpp11 仍會持續維護，但新功能開發集中在 sqlpp23。[^sqlpp11_readme]

---

### 2. sqlite_orm — SQLite 專用流暢查詢建構器

| 指標 | 數值 |
|---|---|
| GitHub | [fnc12/sqlite_orm](https://github.com/fnc12/sqlite_orm)[^sqlite_orm] |
| Stars | ⭐ 2,700 |
| Forks | 343 |
| Commits | 3,359 |
| 最新版本 | v1.9.1 |
| C++ 標準 | C++14/17/20/23 |
| 授權 | AGPL（開源免費）或 MIT（$50 購買） |
| Header-Only | 是（單一 header） |
| 支援後端 | SQLite 專用 |

**設計哲學**：流暢查詢建構器鏈 (fluent chain)，類似 SQLAlchemy 或 LINQ。刻意避開成為完整 ORM，同時提供 CRUD 便利性。所有查詢均以 C++ 表達式建構，無原始 SQL 字串。[^sqlite_orm_docs]

**編譯期型別安全**：欄位引用透過成員指標 (member pointer) 對應至資料庫欄位，編譯器可拒絕不存在的欄位或型別不匹配。但深度不如 sqlpp11 — 側重於型別安全的欄位引用而非完整 SQL 語法驗證。

**範例**：
```cpp
auto storage = make_storage("db.sqlite",
    make_table("users",
        make_column("id", &User::id, primary_key().autoincrement()),
        make_column("name", &User::firstName)));
auto users = storage.get_all<User>(where(c(&User::id) < 10));
```

**⚠️ 授權注意事項**：AGPL 授權對商業使用有限制，若需 MIT 授權需支付 $50 美元。此授權模式可能不適合部分專案。[^sqlite_orm_license]

---

### 3. sqlpp23 — 下一代 (C++23)

| 指標 | 數值 |
|---|---|
| GitHub | [rbock/sqlpp23](https://github.com/rbock/sqlpp23)[^sqlpp23] |
| Stars | ⭐ 180 |
| Forks | 19 |
| Commits | 2,361 |
| C++ 標準 | C++23 |
| 授權 | BSD 2-Clause |
| Header-Only | 是 |
| C++ Modules | 支援 |

**設計哲學**：與 sqlpp11 相同的 EDSL 方法，但為 C++23 重寫。支援 C++20 modules。[^sqlpp23_docs]

**編譯期型別安全**：與 sqlpp11 相同的編譯期保證，加上：
- 改善的錯誤訊息
- `std::optional<std::string_view>` 用於可為 NULL 的欄位
- 現代 C++ 特性整合

**關鍵改進**：
- C++23 modules 支援
- 更好的 `std::optional` 整合處理 NULL
- `ON CONFLICT ... DO UPDATE` / `DO NOTHING` 支援
- 更簡潔的 API

**編譯器需求**：clang 20.1, gcc 14.2, MSVC 19.44.35219

**⚠️ 狀態**：活躍開發中，但生態系不如 sqlpp11 成熟。

---

## 次要方案 (非查詢建構器，但仍值得注意)

### 4. SQLiteCpp — 輕量 RAII 封裝

| 指標 | 數值 |
|---|---|
| GitHub | [SRombauts/SQLiteCpp](https://github.com/SRombauts/SQLiteCpp)[^sqlitecpp] |
| Stars | ⭐ 2,800 |
| 授權 | MIT |
| 設計 | 現代 C++ RAII 封裝 SQLite C API |
| 型別安全 | ❌ 僅執行期 — SQL 為原始字串 |
| 查詢建構器 | ❌ 無 |

最受歡迎的 SQLite C++ 封裝庫，但**不是查詢建構器**。適合需要現代 C++ 物件安全但願意寫原始 SQL 的專案。

### 5. sqlite_modern_cpp — C++14 串流語法封裝

| 指標 | 數值 |
|---|---|
| GitHub | [SqliteModernCpp/sqlite_modern_cpp](https://github.com/SqliteModernCpp/sqlite_modern_cpp)[^sqlite_modern_cpp] |
| Stars | ⭐ 949 |
| 授權 | MIT |
| 設計 | 串流運算子 `<<`/`>>` 語法 |
| 型別安全 | ⚠️ 部分 — 繫結與結果型別推導 |
| 查詢建構器 | ❌ 無 — 仍為原始 SQL 字串 |

### 6. SOCI — 多後端資料庫抽象層

| 指標 | 數值 |
|---|---|
| GitHub | [SOCI/soci](https://github.com/SOCI/soci)[^soci] |
| Stars | ⭐ 1,600 |
| 授權 | Boost Software License 1.0 |
| 設計 | JDBC 風格，8+ 資料庫後端 |
| 型別安全 | ⚠️ 僅繫結層 (`into()`/`use()`) |
| 查詢建構器 | ❌ 無 |

成熟的多後端方案（用於 CERN），但非查詢建構器。SQL 仍為原始字串。

### 7. sqlite3pp — 現代 C++ 封裝含疊代器

| 指標 | 數值 |
|---|---|
| GitHub | [iwongu/sqlite3pp](https://github.com/iwongu/sqlite3pp)[^sqlite3pp] |
| Stars | ⭐ 643 |
| 授權 | MIT |
| 設計 | 幾乎支援所有 SQLite3 功能 |
| 型別安全 | ❌ 無 |
| 查詢建構器 | ❌ 無 |

### 8. nanodbc — ODBC 封裝（非 SQLite 原生）

| 指標 | 數值 |
|---|---|
| GitHub | [nanodbc/nanodbc](https://github.com/nanodbc/nanodbc)[^nanodbc] |
| Stars | ⭐ 386 |
| 授權 | MIT |
| 設計 | 最小 ODBC 封裝 |
| 型別安全 | ❌ 無 |
| 查詢建構器 | ❌ 無 |

透過 ODBC 驅動程式存取 SQLite，**非 SQLite 原生函式庫**，不建議作為型別安全 SQLite 方案。

---

## 對比總表

| 函式庫 | 型別安全 | 查詢建構器 | 原始 SQL | 資料庫後端 | C++ 標準 | GitHub Stars | 授權 | Header-Only |
|---|---|---|---|---|---|---|---|---|
| **sqlpp11** | ✅ 完整編譯期 | ✅ EDSL | ❌ 無 | 多後端 | C++11+ | ⭐ 2,600 | BSD-2 | ✅ |
| **sqlpp23** | ✅ 完整編譯期 | ✅ EDSL | ❌ 無 | 多後端 | C++23 | ⭐ 180 | BSD-2 | ✅ |
| **sqlite_orm** | ✅ 欄位層編譯期 | ✅ Fluent chain | ❌ 無 | SQLite 專用 | C++14+ | ⭐ 2,700 | AGPL/MIT$ | ✅ |
| **SQLiteCpp** | ❌ 僅執行期 | ❌ 無 | ✅ 是 | SQLite 專用 | C++17 | ⭐ 2,800 | MIT | ❌ |
| **sqlite_modern_cpp** | ⚠️ 部分 | ❌ 無 | ✅ 是 | SQLite 專用 | C++14+ | ⭐ 949 | MIT | ✅ |
| **SOCI** | ⚠️ 執行期繫結 | ❌ 無 | ✅ 是 | 多後端 (8+) | C++14 | ⭐ 1,600 | BSL-1.0 | ❌ |
| **sqlite3pp** | ❌ 僅執行期 | ❌ 無 | ✅ 是 | SQLite 專用 | C++11 | ⭐ 643 | MIT | ✅ |
| **nanodbc** | ❌ 僅執行期 | ❌ 無 | ✅ 是 | 多後端 (ODBC) | C++17 | ⭐ 386 | MIT | ❌ |

---

## 結論與建議

### 若優先考量型別安全查詢建構器（本報告核心主題）：

1. **sqlpp11** → 現階段最佳選擇。最成熟 (2,600⭐)、最廣泛的後端支援、最強的編譯期型別安全。BSD 授權無商業疑慮。但 schema 定義較為繁瑣（ddl2cpp 工具可協助），且作者建議新專案考慮 sqlpp23。[^sqlpp11]

2. **sqlite_orm** → SQLite 專案的最佳選擇。流暢 API 直觀 (2,700⭐)，無原始字串。但 **AGPL/MIT$ 授權模式** 是顯著考量 — 商業使用需付費。[^sqlite_orm]

3. **sqlpp23** → 未來方向，但需 C++23 編譯器且仍在成熟中 (180⭐)。適合全新專案搭配最新編譯器。[^sqlpp23]

### 若可接受無編譯期型別安全的輕量封裝：

SQLiteCpp (2,800⭐, MIT) 是最受歡迎的現代 C++ SQLite 封裝，但需要手寫 SQL 字串。

### 若需要多後端支援：

SOCI (BSL 授權，用於 CERN) 是最成熟的多後端方案，但無查詢建構器。

---

[^sqlpp11]: rbock. (n.d.). sqlpp11 — A type safe SQL template library for C++. Retrieved 2026-09-25, from https://github.com/rbock/sqlpp11
[^sqlpp11_docs]: rbock. (n.d.). sqlpp11 Documentation — Type System and Compile-time Validation. Retrieved 2026-09-25, from https://github.com/rbock/sqlpp11/wiki
[^sqlpp11_readme]: rbock. (2025-06-29). sqlpp11 README — Migration note for sqlpp23. Retrieved 2026-09-25, from https://github.com/rbock/sqlpp11#attention
[^sqlpp23]: rbock. (n.d.). sqlpp23 — A type safe SQL template library for C++23. Retrieved 2026-09-25, from https://github.com/rbock/sqlpp23
[^sqlpp23_docs]: rbock. (n.d.). sqlpp23 Documentation. Retrieved 2026-09-25, from https://github.com/rbock/sqlpp23
[^sqlite_orm]: fnc12. (n.d.). SQLite ORM — SQLite for C++ developers. Retrieved 2026-09-25, from https://github.com/fnc12/sqlite_orm
[^sqlite_orm_docs]: fnc12. (n.d.). SQLite ORM Documentation. Retrieved 2026-09-25, from https://sqliteorm.com/
[^sqlite_orm_license]: fnc12. (n.d.). SQLite ORM — License (AGPL / MIT purchase). Retrieved 2026-09-25, from https://github.com/fnc12/sqlite_orm?tab=License-1-ov-file
[^sqlitecpp]: SRombauts. (n.d.). SQLiteCpp — SQLite3 C++ wrapper. Retrieved 2026-09-25, from https://github.com/SRombauts/SQLiteCpp
[^sqlite_modern_cpp]: SqliteModernCpp. (n.d.). sqlite_modern_cpp — Modern C++ wrapper for SQLite. Retrieved 2026-09-25, from https://github.com/SqliteModernCpp/sqlite_modern_cpp
[^soci]: SOCI. (n.d.). SOCI — The C++ Database Access Library. Retrieved 2026-09-25, from https://github.com/SOCI/soci
[^sqlite3pp]: iwongu. (n.d.). sqlite3pp — C++ wrapper of SQLite3 API. Retrieved 2026-09-25, from https://github.com/iwongu/sqlite3pp
[^nanodbc]: nanodbc. (n.d.). nanodbc — A small C++ wrapper for the native C ODBC API. Retrieved 2026-09-25, from https://github.com/nanodbc/nanodbc