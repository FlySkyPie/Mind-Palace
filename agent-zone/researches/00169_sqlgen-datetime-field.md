# sqlgen 中建立 datetime 欄位的方法

sqlgen 是 getml 開發的一套基於 C++20 反射（reflection）的 ORM 與 SQL 查詢生成器，類似 Python 的 SQLAlchemy/SQLModel 或 Rust 的 Diesel。本文說明如何在 sqlgen 中定義 datetime 類型的資料欄位。[^sqlgen-repo]

## 核心型別：`sqlgen::Timestamp<format>`

sqlgen 透過模板類別 `sqlgen::Timestamp<format>` 來表示日期/時間，其中 `format` 是 `strftime` 風格的格式字串模板參數。此類別提供兩個常用的型別別名：[^timestamp-docs]

| C++ 型別 | PostgreSQL | MySQL | SQLite |
|---|---|---|---|
| `sqlgen::Timestamp<"%Y-%m-%d">`（別名 `sqlgen::Date`） | `DATE` | `DATE` | `TEXT`（ISO8601） |
| `sqlgen::Timestamp<"%Y-%m-%d %H:%M:%S">`（別名 `sqlgen::DateTime`） | `TIMESTAMP` | `DATETIME` | `TEXT`（ISO8601） |
| `sqlgen::Timestamp<"%Y-%m-%d %H:%M:%S%z">` | `TIMESTAMP WITH TIME ZONE` | `DATETIME`（無原生時區支援） | `TEXT`（ISO8601） |

## 定義含 datetime 欄位的模型

直接在 struct 中使用 `sqlgen::Date` 或 `sqlgen::DateTime` 作為欄位型別：[^define-models]

```cpp
#include <sqlgen/sqlite.hpp>  // 或 postgres.hpp / mysql.hpp

struct Person {
    sqlgen::PrimaryKey<uint32_t> id;
    std::string first_name;
    std::string last_name;

    sqlgen::Date birthday;             // 僅日期：%Y-%m-%d
    sqlgen::DateTime created_at;       // 日期+時間：%Y-%m-%d %H:%M:%S
    sqlgen::Timestamp<"%Y-%m-%d %H:%M:%S"> updated_at;
    sqlgen::Timestamp<"%Y-%m-%d %H:%M:%S%z"> last_login; // 含時區
};
```

產生的 SQL：

```sql
CREATE TABLE IF NOT EXISTS "Person" (
    "id" INTEGER NOT NULL,
    "first_name" TEXT NOT NULL,
    "last_name" TEXT NOT NULL,
    "birthday" DATE NOT NULL,
    "created_at" TIMESTAMP NOT NULL,
    "updated_at" TIMESTAMP NOT NULL,
    "last_login" TIMESTAMP WITH TIME ZONE NOT NULL,
    PRIMARY KEY ("id")
);
```

可空欄位可搭配 `std::optional`：[^optional-fields]

```cpp
struct Person {
    // ...
    std::optional<sqlgen::Date> birthday;       // NULLABLE DATE
    std::optional<sqlgen::DateTime> deleted_at; // NULLABLE TIMESTAMP
};
```

## 賦值與取值

`sqlgen::Timestamp` 支援字串與 `std::tm` 兩種建構方式：[^timestamp-usage]

```cpp
// 字串賦值（格式不符會在執行期拋錯）
Person p{
    .id = 1,
    .first_name = "Homer",
    .birthday = "1989-12-17",
    .created_at = "2024-03-20 15:30:00",
};

// 從 std::tm 建構
std::tm tm{};
tm.tm_year = 89;
tm.tm_mon = 11;
tm.tm_mday = 17;
sqlgen::Date birthday(tm);

// 安全建構（回傳 Result）
auto result = sqlgen::DateTime::from_string("2024-03-20 15:30:00");
if (result) {
    auto dt = result.value();
}

// 取值
const std::string& s = p.birthday.str();  // "1989-12-17"
const std::tm& t = p.birthday.tm();       // 取得 std::tm 結構
```

## 查詢中的日期時間操作

sqlgen 提供豐富的日期時間函式：[^timestamp-ops]

```cpp
using namespace sqlgen;
using namespace sqlgen::literals;

// 提取年/月/日/時/星期
year("birthday"_c)   | as<"year">
month("birthday"_c)  | as<"month">
day("birthday"_c)    | as<"day">
hour("created_at"_c) | as<"hour">

// 時間算術（支援 std::chrono 持續時間）
("birthday"_c + std::chrono::days(10))   | as<"plus_10d">
("birthday"_c + std::chrono::years(1))   | as<"next_year">

// 日期差值
days_between("birthday"_c, Date("2011-01-01")) | as<"age_in_days">

// Unix epoch 轉換
unixepoch("created_at"_c) | as<"unix">
```

## 資料庫特定行為

- **PostgreSQL**：原生支援 `DATE`、`TIMESTAMP`、`TIMESTAMP WITH TIME ZONE`，時區處理完整。[^pg-docs]
- **MySQL**：使用 `DATE` 與 `DATETIME`，`DATETIME` 無原生時區支援。[^mysql-docs]
- **SQLite**：所有日期時間存為 `TEXT`（ISO8601 格式），內部透過 `datetime()` 函式處理運算。[^sqlite-docs]

## 總結

在 sqlgen 中建立 datetime 欄位只要三步：

1. 在 struct 中使用 `sqlgen::Date` 或 `sqlgen::DateTime` 作為欄位型別
2. 用 ISO8601 格式的字串或 `std::tm` 賦值
3. 在查詢中使用 `year()`、`month()`、`days_between()` 等函式進行時間操作

---

[^sqlgen-repo]: getml. (n.d.). *sqlgen: A reflection-based ORM and SQL query generator for C++-20*. GitHub. Retrieved 2026-09-25, from https://github.com/getml/sqlgen

[^timestamp-docs]: getml. (n.d.). *Timestamp type — sqlgen documentation*. Retrieved 2026-09-25, from https://getml.github.io/sqlgen/timestamp

[^define-models]: getml. (n.d.). *Defining tables — sqlgen documentation*. Retrieved 2026-09-25, from https://getml.github.io/sqlgen/defining_tables

[^optional-fields]: getml. (n.d.). *Built-in type mappings — sqlgen documentation*. Retrieved 2026-09-25, from https://deepwiki.com/getml/sqlgen/8.1-built-in-type-mappings

[^timestamp-usage]: getml. (n.d.). *Timestamp type — sqlgen documentation*. Retrieved 2026-09-25, from https://getml.github.io/sqlgen/timestamp

[^timestamp-ops]: getml. (n.d.). *Timestamp operations — sqlgen documentation*. Retrieved 2026-09-25, from https://getml.github.io/sqlgen/timestamp_operations

[^pg-docs]: PostgreSQL Global Development Group. (2024). *PostgreSQL 16 documentation: Date/Time types*. Retrieved 2026-09-25, from https://www.postgresql.org/docs/16/datatype-datetime.html

[^mysql-docs]: Oracle Corporation. (2024). *MySQL 8.0 reference manual: Date and time data types*. Retrieved 2026-09-25, from https://dev.mysql.com/doc/refman/8.0/en/date-and-time-types.html

[^sqlite-docs]: SQLite Consortium. (2024). *SQLite documentation: Date and time functions*. Retrieved 2026-09-25, from https://www.sqlite.org/lang_datefunc.html