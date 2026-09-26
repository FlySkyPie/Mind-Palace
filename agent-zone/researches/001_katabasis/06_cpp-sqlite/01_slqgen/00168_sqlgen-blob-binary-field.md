# sqlgen 中建立 BLOB/Binary 欄位的方法

## 概述

[getml/sqlgen](https://github.com/getml/sqlgen) 是一個 reflection-based 的 C++20 ORM 與 SQL 查詢產生器，類似 Python 的 SQLAlchemy/SQLModel 或 Rust 的 Diesel。該函式庫**本身不提供原生的 BLOB/binary 欄位型別**，但可透過其 `Dynamic` 自訂型別機制來實現。

## sqlgen 的原生型別系統

`sqlgen::dynamic::Type` 定義於 [`include/sqlgen/dynamic/types.hpp`](https://github.com/getml/sqlgen/blob/816ca6ad/include/sqlgen/dynamic/types.hpp)，支援的型別僅包含[^types-header]：

`Boolean`, `Float32`, `Float64`, `Int8/16/32/64`, `UInt8/16/32/64`, `Text`, `VarChar`, `JSON`, `Date`, `Timestamp`, `TimestampWithTZ`, `Enum`, `Unknown`

**沒有任何 BLOB、Binary、Bytes 或 BYTEA 變體。**[^type-header]

## 各後端對應表

sqlgen 的三個資料庫後端在 `type_to_sql()` 函式中皆未提供 binary 型別的對應[^sqlite-sql][^postgres-sql][^mysql-sql]：

| 資料庫 | Binary 原生型別 | sqlgen 原生支援 | 使用 Dynamic 字串 |
|---|---|---|---|
| SQLite | `BLOB` | ❌ | `Dynamic{"BLOB"}` |
| PostgreSQL | `BYTEA` | ❌ | `Dynamic{"BYTEA"}` |
| MySQL | `BLOB` / `BINARY` / `VARBINARY` | ❌ | `Dynamic{"BLOB"}` |

官方文件在討論 SQLite 型別時明確提到 SQLite 本身支援 `INTEGER, REAL, TEXT, BLOB`，但 sqlgen 並未提供內建的對應[^builtin-types]。

## 解決方案：使用 Dynamic 自訂型別

sqlgen 的 [`Dynamic` 機制](https://deepwiki.com/getml/sqlgen/6.3-custom-types-with-the-dynamic-mechanism)允許定義任意 SQL 欄位型別。官方文件明確指出 BLOB 是此機制的可能使用案例：「*SQLite: Custom types stored as TEXT or **BLOB***」[^dynamic-docs]。

### 步驟一：定義你的 Binary 型別

```cpp
// 定義一個簡單的二進位資料包裝型別
struct BlobData {
    std::vector<char> data;
};
```

### 步驟二：特化 `Parser<T>`

定義於 [`include/sqlgen/parsing/Parser_default.hpp`](https://github.com/getml/sqlgen/blob/816ca6ad/include/sqlgen/parsing/Parser_default.hpp) 的預設 `Parser<T>` 不處理二進位型別，因此必須特化[^parser-default]：

```cpp
namespace sqlgen::parsing {

template <>
struct Parser<BlobData> {
    using Type = BlobData;

    static Result<BlobData> read(const std::optional<std::string>& _str) {
        if (!_str) return BlobData{};
        return BlobData{std::vector<char>(_str->begin(), _str->end())};
    }

    static std::optional<std::string> write(const BlobData& _b) {
        return std::string(_b.data.begin(), _b.data.end());
    }

    static dynamic::Type to_type() noexcept {
        return sqlgen::dynamic::types::Dynamic{"BLOB"};  // 關鍵行
    }
};

}
```

### 步驟三：（可選）特化 `ToValue<T>` 以支援 WHERE 子句

```cpp
namespace sqlgen::transpilation {

template <>
struct ToValue<BlobData> {
    dynamic::Value operator()(const BlobData& _b) const {
        return dynamic::Value{
            dynamic::String{.val = std::string(_b.data.begin(), _b.data.end())}
        };
    }
};

}
```

### 步驟四：在 struct 定義中使用

```cpp
struct Document {
    sqlgen::PrimaryKey<uint32_t, sqlgen::auto_incr> id;
    std::string                                name;
    BlobData                                   content;
};
```

產生的 SQL（以 SQLite 為例）：

```sql
CREATE TABLE "Document" (
    "id"      INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,
    "name"    TEXT NOT NULL,
    "content" BLOB NOT NULL
);
```

若你需要 PostgreSQL 則將 `Dynamic{"BLOB"}` 改為 `Dynamic{"BYTEA"}`，MySQL 亦可使用 `Dynamic{"BLOB"}` 或 `Dynamic{"VARBINARY(255)"}`。

## 注意事項

- sqlgen 不提供 `std::vector<uint8_t>` 或 `std::byte` 的現成 Parser，每次使用 binary 型別都需自訂特化[^parser-default]。
- `Dynamic` 型別的字串在 SQL 產生時直接傳入，不做轉換或驗證，因此需確保字串對目標資料庫有效。
- 此方法適用於所有三個後端（SQLite、PostgreSQL、MySQL），只需調整 `Dynamic` 中的型別名稱字串。

## 結論

sqlgen **無原生 BLOB/binary 支援**，但透過 `Dynamic` 自訂型別機制可以乾淨地實現。你需要自訂一個 C++ 型別，特化 `Parser<T>`，並在 `to_type()` 中回傳 `Dynamic{"BLOB"}`（或對應資料庫的型別名稱），然後即可在 struct 定義中當作一般欄位使用。

[^types-header]: getml/sqlgen. (n.d.). *include/sqlgen/dynamic/types.hpp*. Retrieved 2026-09-25, from https://github.com/getml/sqlgen/blob/816ca6ad/include/sqlgen/dynamic/types.hpp

[^type-header]: getml/sqlgen. (n.d.). *include/sqlgen/dynamic/Type.hpp*. Retrieved 2026-09-25, from https://github.com/getml/sqlgen/blob/816ca6ad/include/sqlgen/dynamic/Type.hpp

[^sqlite-sql]: getml/sqlgen. (n.d.). *src/sqlgen/sqlite/to_sql.cpp*. Retrieved 2026-09-25, from https://github.com/getml/sqlgen/blob/816ca6ad/src/sqlgen/sqlite/to_sql.cpp

[^postgres-sql]: getml/sqlgen. (n.d.). *src/sqlgen/postgres/to_sql.cpp*. Retrieved 2026-09-25, from https://github.com/getml/sqlgen/blob/816ca6ad/src/sqlgen/postgres/to_sql.cpp

[^mysql-sql]: getml/sqlgen. (n.d.). *src/sqlgen/mysql/to_sql.cpp*. Retrieved 2026-09-25, from https://github.com/getml/sqlgen/blob/816ca6ad/src/sqlgen/mysql/to_sql.cpp

[^builtin-types]: DeepWiki. (n.d.). *Built-in Type Mappings — sqlgen*. Retrieved 2026-09-25, from https://deepwiki.com/getml/sqlgen/8.1-built-in-type-mappings

[^dynamic-docs]: getml/sqlgen. (n.d.). *docs/content/dynamic.md*. Retrieved 2026-09-25, from https://github.com/getml/sqlgen/blob/816ca6ad/docs/content/dynamic.md

[^parser-default]: getml/sqlgen. (n.d.). *include/sqlgen/parsing/Parser_default.hpp*. Retrieved 2026-09-25, from https://github.com/getml/sqlgen/blob/816ca6ad/include/sqlgen/parsing/Parser_default.hpp