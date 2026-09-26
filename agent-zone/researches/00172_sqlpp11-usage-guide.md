# sqlpp11 使用指南

## 概述

sqlpp11 是一個**型別安全的嵌入式領域特定語言**（EDSL），用於在 C++ 中編寫 SQL 查詢並處理查詢結果。它是一個 header-only 的樣板函式庫，讓你能以 C++ struct 和 function 操作資料庫，而非撰寫原始 SQL 字串——編譯器會在編譯期檢查語法、型別、名稱與部分語意錯誤[^main]。

核心設計理念：
- 每個資料表與欄位都是獨立的 C++ 型別
- 查詢結果的 row 是強型別 struct，成員名稱對應欄位別名
- 支援**靜態查詢**（編譯期完整檢查）與**動態查詢**（執行期動態組合）
- 資料庫無關的核心層，透過 Connector 串接特定資料庫

## 環境需求與安裝

### 編譯器需求

需要支援 C++11 的現代編譯器：
- Clang 3.4+
- GCC 4.8+
- MSVC 2015 Update 1+
- Xcode 7+
- AppleClang 14+ (macOS)

### 依賴項目

- **Howard Hinnant 的 date 函式庫**：處理 `date` 與 `date_time` 資料型別。預設 sqlpp11 透過 CMake FetchContent 自動下載；設 `USE_SYSTEM_DATE=ON` 則改為使用系統安裝版[^main]。
- **Connector 函式庫**：依你的資料庫選擇 — MySQL client、MariaDB client、PostgreSQL client、SQLite3、SQLCipher。

### 安裝方式

#### 方法一：從原始碼建置並安裝

```bash
cmake -B build \
  -DBUILD_POSTGRESQL_CONNECTOR=ON \
  -DBUILD_SQLITE3_CONNECTOR=ON \
  -DDEPENDENCY_CHECK=OFF \
  -DBUILD_TESTING=OFF
cmake --build build --target install
```

Connector 對應的 CMake 選項：
- `BUILD_MYSQL_CONNECTOR`
- `BUILD_MARIADB_CONNECTOR`
- `BUILD_POSTGRESQL_CONNECTOR`
- `BUILD_SQLITE3_CONNECTOR`
- `BUILD_SQLCIPHER_CONNECTOR`

#### 方法二：套件管理員

**macOS (Homebrew)：**
```bash
brew install marvin182/zapfhahn/sqlpp11
```

**vcpkg：**
```bash
vcpkg install 'sqlpp11[mysql]'
```

### CMake 整合

#### 方式 A：FetchContent（推薦，免安裝）

```cmake
include(FetchContent)
FetchContent_Declare(sqlpp11
  GIT_REPOSITORY https://github.com/rbock/sqlpp11
  GIT_TAG origin/main
)
FetchContent_MakeAvailable(sqlpp11)

target_link_libraries(MyTarget PRIVATE sqlpp11::sqlite3)
```

#### 方式 B：find_package（需先 install）

```cmake
find_package(Sqlpp11 REQUIRED COMPONENTS SQLite3)
target_link_libraries(MyTarget PRIVATE sqlpp11::sqlpp11)
```

可用的 CMake Target：
- `sqlpp11::sqlpp11`（核心）
- `sqlpp11::mysql`
- `sqlpp11::mariadb`
- `sqlpp11::sqlite3`
- `sqlpp11::sqlcipher`
- `sqlpp11::postgresql`

### 範例專案結構

完整可執行的範例見官方 repository 的 `examples/` 目錄，包含 FetchContent 與 find_package 兩種整合方式[^ex_fetch][^ex_find]。

## 資料表定義

### 自動產生（推薦）

使用 `ddl2cpp` 工具從資料庫 DDL 轉換為 C++ header：

```bash
# 先取得 DDL
mysqldump --no-data MyDatabase > MyDatabase.sql

# 轉換為 C++ header
sqlpp11-ddl2cpp ~/temp/MyTable.ddl ~/temp/MyTable MyNamespace
```

若 DDL 中有不支援的型別，可用 `--datatype-file` 參數指定型別對應 CSV：

```csv
Boolean, one_or_zero
Text, url, uuid
```

### 手動定義

每個資料表是一個繼承 `sqlpp::table_t` 的 struct，其 column 定義在巢狀 namespace 中[^sample]：

```cpp
#include <sqlpp11/table.h>
#include <sqlpp11/data_types.h>
#include <sqlpp11/char_sequence.h>

namespace test {
namespace TabBar_ {
  struct Alpha {
    struct _alias_t {
      static constexpr const char _literal[] = "alpha";
      using _name_t = sqlpp::make_char_sequence<sizeof(_literal), _literal>;
      template <typename T>
      struct _member_t { T alpha; T& operator()() { return alpha; } };
    };
    using _traits = sqlpp::make_traits<sqlpp::bigint,
                     sqlpp::tag::must_not_insert,
                     sqlpp::tag::must_not_update,
                     sqlpp::tag::can_be_null>;
  };
  struct Beta {
    struct _alias_t { /* ... literal = "beta" ... */ };
    using _traits = sqlpp::make_traits<sqlpp::varchar,
                                       sqlpp::tag::can_be_null>;
  };
  struct Gamma {
    struct _alias_t { /* ... literal = "gamma" ... */ };
    using _traits = sqlpp::make_traits<sqlpp::boolean,
                                       sqlpp::tag::require_insert>;
  };
} // namespace TabBar_

struct TabBar : sqlpp::table_t<TabBar,
                               TabBar_::Alpha,
                               TabBar_::Beta,
                               TabBar_::Gamma> {
  struct _alias_t {
    static constexpr const char _literal[] = "tab_bar";
    using _name_t = sqlpp::make_char_sequence<sizeof(_literal), _literal>;
    template <typename T>
    struct _member_t { T tabBar; };
  };
};
} // namespace test
```

### Column 型別標籤

- `sqlpp::tag::can_be_null` — 欄位可為 NULL
- `sqlpp::tag::must_not_insert` — 禁止出現在 INSERT 中
- `sqlpp::tag::must_not_update` — 禁止出現在 UPDATE 中
- `sqlpp::tag::require_insert` — INSERT 時必須設定

### 支援的資料型別對應

| C++ 型別 | SQL 型別 |
|---|---|
| `sqlpp::boolean` | BOOLEAN |
| `sqlpp::integer` | INTEGER |
| `sqlpp::bigint` | BIGINT |
| `sqlpp::bigint_unsigned` | BIGINT UNSIGNED |
| `sqlpp::serial` | SERIAL |
| `sqlpp::floating_point` | FLOAT/DOUBLE |
| `sqlpp::varchar` | VARCHAR |
| `sqlpp::text` | TEXT |
| `sqlpp::blob` | BLOB |
| `sqlpp::day_point` | DATE |
| `sqlpp::time_point` | DATETIME/TIMESTAMP |
| `sqlpp::time_of_day` | TIME |

## 資料庫連線

### SQLite3 連線範例

```cpp
#include <sqlpp11/sqlite3/connection.h>

auto config = std::make_shared<sqlpp::sqlite3::connection_config>();
config->path_to_database = "my_database.db";
sqlpp::sqlite3::connection db(config);
```

### MySQL / MariaDB / PostgreSQL

各 Connector 提供對應的 `connection` 類別與 `connection_config`，用法模式相同。

## 查詢操作

以下範例假設已定義兩個資料表 `foo`（欄位：id bigint, name varchar, hasFun boolean）與 `bar`（欄位：alpha bigint, beta varchar, gamma boolean, delta integer），並已建立連線 `db`。

### SELECT

#### 基本查詢

```cpp
for (const auto& row :
     db(select(foo.name, foo.hasFun)
            .from(foo)
            .where(foo.id > 17 and foo.name.like("%bar%")))) {
  if (row.name.is_null())
    std::cerr << "name is null\n";
  std::string name = row.name;  // 隱含轉換
  bool hasFun = row.hasFun;     // 隱含轉換
}
```

**關鍵規則**：使用 `where()` 或 `unconditionally()` 擇一，遺漏會造成編譯錯誤。

#### 選取所有欄位

```cpp
for (const auto& row :
     db(select(all_of(foo)).from(foo).where(foo.id == 17))) {
  int64_t id = row.id;
}
```

#### 使用別名

```cpp
SQLPP_ALIAS_PROVIDER(cheese);

if (const auto& row =
        db(select(foo.name.as(cheese)).from(foo).where(foo.id == 17))) {
  std::cerr << "found: " << row.cheese << "\n";
}
```

#### 聚合函數與 GROUP BY

```cpp
for (const auto& row :
     db(select(foo.name, avg(foo.id))
            .from(foo)
            .where(foo.id > 17)
            .group_by(foo.name))) {
  std::cerr << row.name << ": " << row.avg << "\n";
}
```

#### ORDER BY、LIMIT、OFFSET

```cpp
db(select(all_of(foo))
       .from(foo)
       .unconditionally()
       .order_by(foo.name.asc())
       .limit(10u)
       .offset(20u));
```

#### JOIN

```cpp
// INNER JOIN
foo.join(bar).on(foo.id == bar.alpha);

// LEFT OUTER JOIN
foo.join(bar)
    .on(foo.id == bar.alpha)
    .left_outer_join(baz)
    .on(bar.alpha == baz.delta);
```

#### 子查詢（Subquery）

```cpp
SQLPP_ALIAS_PROVIDER(cheese_cake);

for (const auto& row :
     db(select(all_of(foo),
               select(sum(bar.delta))
                   .from(bar)
                   .where(bar.alpha > foo.id),
               select(bar.delta.as(cheese_cake))
                   .from(bar)
                   .where(bar.alpha > foo.id))
            .from(foo))) {
  const int id = row.id;
  const int64_t sum_val = row.sum;
  const int cheese = row.cheese_cake;
}
```

#### 子查詢作為 FROM

```cpp
SQLPP_ALIAS_PROVIDER(sub);

auto sub_select =
    select(all_of(foo)).from(foo).where(foo.id == 42).as(sub);

db(select(all_of(sub_select)).from(sub_select).unconditionally());
```

#### SELECT 旗標

```cpp
// ALL / DISTINCT
sqlpp::select().flags(sqlpp::all).columns(foo.id, foo.name);
select(foo.id, foo.name).flags(sqlpp::all);

// FOR UPDATE
db(select(all_of(foo)).from(foo).where(foo.id != 17).for_update());
```

### INSERT

#### 單行插入

```cpp
db(insert_into(foo)
       .set(foo.id = 17, foo.name = "bar", foo.hasFun = true));

// 時間型別
db(insert_into(tabDateTime)
       .set(tabDateTime.colTimePoint =
                std::chrono::system_clock::now()));
```

#### 多行插入

```cpp
auto multi_insert =
    insert_into(t).columns(t.gamma, t.beta, t.delta);

multi_insert.values.add(t.gamma = true,
                        t.beta = "cheesecake",
                        t.delta = 1);

multi_insert.values.add(
    t.gamma = sqlpp::default_value,
    t.beta = sqlpp::default_value,
    t.delta = sqlpp::default_value);

multi_insert.values.add(
    t.gamma = sqlpp::value_or_null(true),
    t.beta = sqlpp::value_or_null("pie"),
    t.delta =
        sqlpp::value_or_null<sqlpp::integer>(sqlpp::null));

db(multi_insert);
```

**注意**：`add()` 的參數型別必須與 column 型別完全吻合（例如微秒精度 `time_point`）。

### UPDATE

```cpp
db(update(foo)
       .set(foo.hasFun = not foo.hasFun)
       .where(foo.name != "nobody"));

db(update(tab).set(tab.gamma = false).where(tab.alpha.in(1)));
```

### DELETE（使用 `remove_from`，因 `delete` 是 C++ 關鍵字）

```cpp
db(remove_from(foo).where(not foo.hasFun));

db(remove_from(tab).where(tab.alpha == tab.alpha + 3));
```

#### 多表刪除

```cpp
db(remove_from(usr_forms)
       .using_(usr, form_, usr_forms)
       .where(usr_forms.iduser == usr.id and
              usr.username == username and
              usr_forms.idform == form_.id and
              form_.name == form_name));
```

### Prepared Statements

```cpp
SQLPP_ALIAS_PROVIDER(cheese);

auto prepared_insert = db.prepare(
    insert_into(tab).set(tab.alpha = parameter(tab.alpha),
                         tab.beta =
                             parameter(sqlpp::text(), cheese)));

for (const auto& input : input_values) {
  prepared_insert.params.alpha = input.first;
  prepared_insert.params.cheese = input.second;
  db(prepared_insert);
}
```

參數定義方式：
- `parameter(const ValueType&, const AliasProvider&)` — 例如 `parameter(sqlpp::bigint(), cheese)`
- `parameter(const NamedExpression&)` — 從 column 自動推定型別

### 交易（Transactions）

```cpp
auto tx = start_transaction(db);
try {
  // 執行操作
  tx.commit();
} catch (...) {
  tx.rollback();
}

// 隱藏解構時的自動 rollback 警告
auto tx2 = start_transaction(db, sqlpp::quiet_auto_rollback);

// 設定隔離層級
auto tx3 = start_transaction(
    db, ::sqlpp::isolation_level::repeatable_read);
```

## 動態查詢

當查詢結構要到執行期才能確定時，使用 `dynamic_select`。結果以字串索引存取欄位[^dynamic]：

```cpp
auto dynamic_query =
    dynamic_select(db, all_of(foo)).from(foo).dynamic_where();

if (someCondition)
  dynamic_query.where.add(foo.name.like("%bar%"));

for (const auto& row : db(std::move(dynamic_query)))
  std::cout << row.at("id") << "\n";
```

## 核心概念說明

### 型別安全

sqlpp11 最突出的特性是**編譯期型別檢查**：
- 不能拿字串欄位與整數比較 — 編譯器直接拒絕
- `FROM` 子句中未出現的資料表，其欄位無法在 `SELECT` 中使用
- 結果 row 的成員型別與名稱由查詢精確決定

### 編譯期檢查項目

1. **語法錯誤** — 格式不正確的查詢無法編譯
2. **型別錯誤** — 比較或賦值時型別不匹配
3. **名稱錯誤** — 欄位/資料表名稱必須存在於已定義的型別中
4. **語意錯誤** — 例如遺漏 `FROM`、或 `WHERE`/`unconditionally()` 兩者皆未使用

### SQL 方言處理

核心層為資料庫無關。各 Connector 負責：
- 資料庫特定的 SQL 語法差異（例如 `||` vs `concat()`）
- 以編譯期錯誤回報不支援的功能
- 依資料庫需求轉譯表達式

官方提供的 Connector：MySQL、MariaDB、SQLite3、SQLCipher、PostgreSQL。另有社群維護的 ODBC Connector[^main]。

### NULL 處理

- 用 `sqlpp::tag::can_be_null` 標記可為 NULL 的欄位
- 結果 row 提供 `.is_null()` 方法檢查 NULL
- 插入時使用 `sqlpp::value_or_null()` 表達可能為 NULL 的值

## 可編譯的最小範例

以下程式碼僅需 sqlpp11 核心（不需 Connector）即可編譯：

```cpp
#include <sqlpp11/select.h>
#include <sqlpp11/alias_provider.h>

int main() {
  select(sqlpp::value(false).as(sqlpp::alias::a));
  return 0;
}
```

## 遷移建議

根據作者在 2025-06-29 的公告，建議新專案考慮遷移至 sqlpp23（sqlpp11 的後繼版本）。sqlpp11 仍會持續維護，但新功能僅會加入 sqlpp23[^main]。

---

[^main]: rbock. (n.d.). sqlpp11: A type safe embedded domain specific language for SQL queries and results in C++. Retrieved 2026-09-25, from https://github.com/rbock/sqlpp11

[^sample]: rbock. (n.d.). Sample.h — Example table definition. Retrieved 2026-09-25, from https://github.com/rbock/sqlpp11/blob/main/tests/core/usage/Sample.h

[^dynamic]: rbock. (n.d.). Dynamic Select documentation. Retrieved 2026-09-25, from https://github.com/rbock/sqlpp11/blob/main/docs/Dynamic-Select.md

[^ex_fetch]: rbock. (n.d.). FetchContent example CMakeLists.txt. Retrieved 2026-09-25, from https://github.com/rbock/sqlpp11/blob/main/examples/usage_fetch_content/dependencies/CMakeLists.txt

[^ex_find]: rbock. (n.d.). find_package example CMakeLists.txt. Retrieved 2026-09-25, from https://github.com/rbock/sqlpp11/blob/main/examples/usage_find_package/CMakeLists.txt