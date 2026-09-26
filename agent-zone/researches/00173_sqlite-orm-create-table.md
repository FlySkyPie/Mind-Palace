# sqlite_orm 建立表格指南

## 概述

[sqlite_orm](https://github.com/fnc12/sqlite_orm) 是一個 C++ 的 header-only SQLite ORM 函式庫，使用 C++14 以上標準，只有一個相依套件：系統上需安裝 `libsqlite3`。它利用模板與成員函式指標，將 C++ struct 靜態映射至 SQLite 資料表，無需撰寫 SQL DDL。[^github-readme]

使用前只需引入單一 header：

```cpp
#include <sqlite_orm/sqlite_orm.h>
```

## 建立表格的基本流程

### 1. 定義 C++ struct

每個資料表對應一個 C++ struct，struct 必須可以預設建構，且映射到資料庫的欄位不能是 `const`：[^wiki-storage]

```cpp
struct User {
    int id;
    std::string firstName;
    std::string lastName;
    int birthDate;
    std::unique_ptr<std::string> imageUrl;  // 可空（nullable）
    int typeId;
};

struct UserType {
    int id;
    std::string name;
    std::string comment;
};
```

### 2. 使用 DSL 定義映射

透過 `make_storage`、`make_table`、`make_column` 三個核心函式建立映射：[^wiki-storage]

```cpp
using namespace sqlite_orm;

auto storage = make_storage(
    "db.sqlite",
    make_table("users",
        make_column("id",         &User::id,         primary_key().autoincrement()),
        make_column("first_name", &User::firstName),
        make_column("last_name",  &User::lastName),
        make_column("birth_date", &User::birthDate),
        make_column("image_url",  &User::imageUrl),
        make_column("type_id",    &User::typeId)),
    make_table("user_types",
        make_column("id",      &UserType::id,      primary_key().autoincrement()),
        make_column("name",    &UserType::name,    default_value("name_placeholder")),
        make_column("comment", &UserType::comment, default_value("user")))
);
```

SQL 型別由 C++ 型別自動推導，無須手動指定。

### 3. 實際建立資料表

呼叫 `sync_schema()` 將結構定義同步至資料庫檔案：[^wiki-storage]

```cpp
storage.sync_schema();        // 同步結構
// 或
storage.sync_schema(true);    // 保留模式：盡量保留既有資料
```

`symc_schema()` 的行為：
- 若資料表不存在則建立
- 若缺少欄位則執行 `ALTER TABLE ADD COLUMN`
- 若欄位型別、PK 或 NOT NULL 有異且 `preserve=false`，則刪除重建
- 忽略資料庫中多餘的表格

記憶體資料庫可傳入 `":memory:"` 或 `""` 作為檔名。

## 欄位型別對應

| C++ 型別 | SQLite 型別 |
|---|---|
| `int`, `long`, `long long` | `INTEGER` |
| `float`, `double` | `REAL` |
| `std::string`, `const char*` | `TEXT` |
| `bool` | `INTEGER`（0/1） |
| `std::vector<char>` | `BLOB` |

可空欄位使用 `std::shared_ptr<T>`、`std::unique_ptr<T>` 或 `std::optional<T>`（C++17，需定義 `SQLITE_ORM_OPTIONAL_SUPPORTED`）。[^wiki-make-column]

## 約束條件

### 主鍵與自增

```cpp
// 單欄位主鍵
make_column("id", &User::id, primary_key())

// 自增主鍵
make_column("id", &User::id, primary_key().autoincrement())

// 複合主鍵（在 make_table 層級）
make_table("ratings",
    make_column("user_id", &Rating::userId),
    make_column("book_id", &Rating::bookId),
    primary_key(&Rating::userId, &Rating::bookId))
```

### NOT NULL / NULL

由 C++ 型別自動推導：
- 值型別（`int`, `std::string` 等）→ `NOT NULL`
- 指標型別（`shared_ptr`, `unique_ptr`, `optional`）→ 允許 `NULL`

也可手動指定：[^wiki-constraints]

```cpp
make_column("name", &User::name, not_null())
make_column("nickname", &User::nickname, null())
```

### UNIQUE

單欄位：[^wiki-constraints]
```cpp
make_column("email", &User::email, unique())
```

複合唯一（在 `make_table` 層級）：
```cpp
make_table("enrollments",
    make_column("student_id", &Enrollment::studentId),
    make_column("course_id",  &Enrollment::courseId),
    unique(&Enrollment::studentId, &Enrollment::courseId))
```

### DEFAULT

```cpp
make_column("name", &UserType::name, default_value("name_placeholder"))
make_column("is_active", &User::isActive, default_value(true))
make_column("score", &User::score, default_value(0))
```

### CHECK

```cpp
make_column("age", &User::age, check(c(&User::age) >= 18))
```

### 外鍵

外鍵定義在 `make_table` 層級，與欄位同位。需 SQLite >= 3.6.19。[^wiki-constraints][^example-fk]

**基本外鍵：**
```cpp
struct Artist {
    int artistId;
    std::string artistName;
};

struct Track {
    int trackId;
    std::string trackName;
    std::unique_ptr<int> trackArtist;  // 可空 FK
};

auto storage = make_storage("db.sqlite",
    make_table("artist",
        make_column("artistid",   &Artist::artistId, primary_key()),
        make_column("artistname", &Artist::artistName)),
    make_table("track",
        make_column("trackid",    &Track::trackId, primary_key()),
        make_column("trackname",  &Track::trackName),
        make_column("trackartist",&Track::trackArtist),
        foreign_key(&Track::trackArtist).references(&Artist::artistId))
);
```

**ON UPDATE / ON DELETE 動作：**
```cpp
// ON UPDATE CASCADE
foreign_key(&Track::trackArtist)
    .references(&Artist::artistId)
    .on_update.cascade()

// ON DELETE SET NULL
foreign_key(&Track::trackArtist)
    .references(&Artist::artistId)
    .on_delete.set_null()

// ON DELETE SET DEFAULT（需在 FK 欄位加上 default_value）
foreign_key(&Track::trackArtist)
    .references(&Artist::artistId)
    .on_delete.set_default()

// ON DELETE CASCADE
foreign_key(&Track::trackArtist)
    .references(&Artist::artistId)
    .on_delete.cascade()
```

### COLLATE

```cpp
make_column("name", &User::name, collate_nocase())
// 可用：collate_binary(), collate_nocase(), collate_rtrim()
```

### Generated Always（SQLite >= 3.31.0）

```cpp
make_column("full_name", &User::fullName,
    generated_always_as("first_name || ' ' || last_name").stored())
// 或 .virtual()
```

## 完整範例

```cpp
#include <sqlite_orm/sqlite_orm.h>
#include <string>
#include <memory>
#include <vector>

struct User {
    int id;
    std::string firstName;
    std::string lastName;
    int birthDate;
    std::unique_ptr<std::string> imageUrl;
    int typeId;
};

struct UserType {
    int id;
    std::string name;
    std::string comment;
};

int main() {
    using namespace sqlite_orm;

    auto storage = make_storage("app.db",
        make_table("users",
            make_column("id",         &User::id,         primary_key().autoincrement()),
            make_column("first_name", &User::firstName,  not_null()),
            make_column("last_name",  &User::lastName,   not_null()),
            make_column("birth_date", &User::birthDate,  not_null()),
            make_column("image_url",  &User::imageUrl),
            make_column("type_id",    &User::typeId,     not_null())),
        make_table("user_types",
            make_column("id",      &UserType::id,      primary_key().autoincrement()),
            make_column("name",    &UserType::name,    not_null(), default_value("name_placeholder")),
            make_column("comment", &UserType::comment, not_null(), default_value("user")))
    );

    storage.sync_schema();

    // 後續可用 storage.insert(), storage.get_all<User>() 等 CRUD API

    return 0;
}
```

## 參考資料

[^github-readme]: fnc12. (n.d.). sqlite_orm — SQLite ORM light header only library for modern C++. Retrieved 2026-09-25, from https://github.com/fnc12/sqlite_orm
[^wiki-storage]: fnc12. (n.d.). Making a storage. GitHub Wiki. Retrieved 2026-09-25, from https://github.com/fnc12/sqlite_orm/wiki/Making-a-storage
[^wiki-make-column]: fnc12. (n.d.). make_column. GitHub Wiki. Retrieved 2026-09-25, from https://github.com/fnc12/sqlite_orm/wiki/make_column
[^wiki-constraints]: DeepWiki. (n.d.). sqlite_orm 3.2 — Constraints. Retrieved 2026-09-25, from https://deepwiki.com/fnc12/sqlite_orm/3.2-constraints
[^example-fk]: fnc12. (n.d.). foreign_key.cpp example. GitHub. Retrieved 2026-09-25, from https://github.com/fnc12/sqlite_orm/blob/master/examples/foreign_key.cpp