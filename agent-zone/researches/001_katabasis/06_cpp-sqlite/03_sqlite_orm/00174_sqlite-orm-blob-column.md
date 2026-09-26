# sqlite_orm 中建立 BLOB 欄位的方法

## 概述

sqlite_orm 是一個 header-only 的 C++ ORM 函式庫，支援將 C++ 結構體映射到 SQLite 資料表。BLOB（Binary Large Object）欄位用於儲存二進位資料，如圖片、雜湊值、序列化物件等。本報告說明如何在 sqlite_orm 中定義、寫入與讀取 BLOB 欄位。

## 內建 BLOB 型別：`std::vector<char>`

sqlite_orm 唯一內建支援的 BLOB 型別是 `std::vector<char>`[^builtin-type]。當結構體成員宣告為此型別時，函式庫會自動將其映射為 SQLite 的 `BLOB` 型別，無需任何額外標記。

```cpp
#include <sqlite_orm/sqlite_orm.h>
#include <vector>
#include <string>

struct User {
    int id;
    std::string name;
    std::vector<char> hash;  // 自動映射為 BLOB
};
```

### 建立表格

使用 `make_storage` 與 `make_table` 建立包含 BLOB 欄位的表格，語法與一般欄位完全相同[^make-column]：

```cpp
using namespace sqlite_orm;

auto storage = make_storage("blob.sqlite",
    make_table("users",
        make_column("id", &User::id, primary_key()),
        make_column("name", &User::name),
        make_column("hash", &User::hash)  // BLOB 欄位，無特殊語法
    )
);

storage.sync_schema();  // 建立或更新表格
```

### 寫入 BLOB 資料

直接使用 `std::vector<char>` 填入資料，再呼叫 `insert` 或 `replace` 即可[^blob-example]：

```cpp
User alex{0, "Alex", {0x10, 0x20, 0x30, 0x40}};
alex.id = storage.insert(alex);
```

### 讀取 BLOB 資料

取得記錄後，BLOB 欄位會自動反序列化為 `std::vector<char>`[^blob-example]：

```cpp
auto user = storage.get<User>(alex.id);
auto hash = user.hash;
// hash.size() == 4
// hash[0] == 0x10, hash[1] == 0x20, etc.
```

## 使用 `type_()` 明確指定型別

若想明確宣告欄位為 BLOB（即使 C++ 型別不同），可使用 `type_()` 輔助函式[^type-helper]：

```cpp
make_column("avatar", &UserProfile::avatar, type_("BLOB"))
```

不過對於 `std::vector<char>` 來說這是多餘的，因為自動映射已是 BLOB。

## 自訂型別映射為 BLOB

若要將自訂 C++ 結構體（非 `std::vector<char>`）儲存為 BLOB 欄位，需要在 `sqlite_orm` 命名空間下特化三個模板[^blob-binding]：

### 1. `type_printer` — 宣告 SQL 型別為 BLOB

```cpp
namespace sqlite_orm {
    template<>
    struct type_printer<Rect> : public blob_printer {};
}
```

繼承 `blob_printer` 即可，它會回傳字串 `"BLOB"`[^blob-printer]。

### 2. `statement_binder` — 將自訂型別序列化為位元組以綁定

```cpp
namespace sqlite_orm {
    template<>
    struct statement_binder<Rect> {
        int bind(sqlite3_stmt* stmt, int index, const Rect& value) {
            std::vector<char> blobValue;
            blobValue.reserve(16);
            // 將 Rect 的欄位編碼進 blobValue...
            return statement_binder<std::vector<char>>().bind(
                stmt, index, blobValue);
        }
    };
}
```

### 3. `row_extractor` — 從 SQLite 讀取時反序列化

```cpp
namespace sqlite_orm {
    template<>
    struct row_extractor<Rect> {
        Rect extract(sqlite3_stmt* stmt, int columnIndex) const {
            auto blobPointer = sqlite3_column_blob(stmt, columnIndex);
            auto size = sqlite3_column_bytes(stmt, columnIndex);
            // 從位元組解碼為 Rect...
            return value;
        }
    };
}
```

### 4. （選用）`field_printer` — 提供除錯輸出

```cpp
namespace sqlite_orm {
    template<>
    struct field_printer<Rect> {
        std::string operator()(const Rect& value) const {
            std::stringstream ss;
            ss << "{ x = " << value.x << ", y = " << value.y
               << ", w = " << value.w << ", h = " << value.h << " }";
            return ss.str();
        }
    };
}
```

完成上述特化後，即可像一般型別一樣使用自訂結構體：

```cpp
struct Zone {
    int id;
    Rect rect;  // 自訂型別，經由上述特化映射為 BLOB
};

auto storage = make_storage("zones.db",
    make_table("zones",
        make_column("id", &Zone::id, primary_key()),
        make_column("rect", &Zone::rect)
    )
);
```

## C++ 型別與 SQLite 型別映射對照

sqlite_orm 的內建型別映射規則如下[^type-mapping]：

| C++ 型別 | SQLite 型別 |
|---|---|
| `int`, `char`, `short`, `long`, `bool` 等整數型別 | `INTEGER` |
| `float`, `double` | `REAL` |
| `std::string`, `const char*` | `TEXT` |
| **`std::vector<char>`** | **`BLOB`** |
| `std::unique_ptr<T>`, `std::shared_ptr<T>` | 繼承 T 的型別（可為 NULL） |

注意 `std::string` 映射為 `TEXT` 而非 `BLOB`，不可用於儲存二進位資料。`std::vector<uint8_t>` 沒有內建支援，需使用自訂型別特化方式處理。

## 注意事項

1. **空 BLOB**：空的 `std::vector<char>` 會綁定為空 BLOB（長度為 0）[^builtin-binder]。
2. **記憶體**：BLOB 資料在讀取時會完整載入記憶體，大型 BLOB（如數 MB 圖片）需注意記憶體用量。
3. **可空 BLOB**：若要允許 BLOB 欄位為 `NULL`，可將型別宣告為 `std::unique_ptr<std::vector<char>>` 或 `std::shared_ptr<std::vector<char>>`[^nullable]。
4. **約束條件**：BLOB 欄位同樣可加上 `primary_key()`、`default_value(...)`、`unique()` 等約束[^constraints]。
5. **型別親和性 (Type Affinity)**：即使明確標記為 `BLOB`，SQLite 的型別親和性機制仍可能接受其他型別的資料，但 ORM 層會確保正確的序列化與反序列化[^type-affinity]。

## 參考資料

[^builtin-type]: sqlite_orm 原始碼 — `type_printer<std::vector<char>>` 特化。GitHub. (n.d.). Retrieved 2026-09-25, from https://github.com/fnc12/sqlite_orm/blob/dev/include/sqlite_orm/type_printer.h
[^blob-example]: sqlite_orm 範例 — `blob.cpp`。GitHub. (n.d.). Retrieved 2026-09-25, from https://github.com/fnc12/sqlite_orm/blob/master/examples/blob.cpp
[^blob-binding]: sqlite_orm 範例 — `blob_binding.cpp`（自訂型別 BLOB 綁定）。GitHub. (n.d.). Retrieved 2026-09-25, from https://github.com/fnc12/sqlite_orm/blob/master/examples/blob_binding.cpp
[^blob-printer]: sqlite_orm 原始碼 — `blob_printer` 定義。GitHub. (n.d.). Retrieved 2026-09-25, from https://github.com/fnc12/sqlite_orm/blob/dev/include/sqlite_orm/type_printer.h
[^make-column]: sqlite_orm Wiki — `make_column`。GitHub. (n.d.). Retrieved 2026-09-25, from https://github.com/fnc12/sqlite_orm/wiki/make_column
[^type-helper]: sqlite_orm 文件 — `type_()` 輔助函式用於明確指定欄位型別。Volcengine. (n.d.). Retrieved 2026-09-25, from https://www.volcengine.com/article/305033
[^builtin-binder]: sqlite_orm 原始碼 — `statement_binder<std::vector<char>>` 使用 `sqlite3_bind_blob` 並帶 `SQLITE_TRANSIENT` 旗標。GitHub. (n.d.). Retrieved 2026-09-25, from https://github.com/fnc12/sqlite_orm/blob/dev/include/sqlite_orm/statement_binder.h
[^nullable]: sqlite_orm 原始碼 — 可空型別（`std::unique_ptr`、`std::shared_ptr`）支援。GitHub. (n.d.). Retrieved 2026-09-25, from https://github.com/fnc12/sqlite_orm/blob/dev/include/sqlite_orm/type_printer.h
[^constraints]: sqlite_orm Wiki — `make_column` 支援的約束條件。GitHub. (n.d.). Retrieved 2026-09-25, from https://github.com/fnc12/sqlite_orm/wiki/make_column
[^type-mapping]: sqlite_orm README — BLOB 支援與型別映射說明。GitHub. (n.d.). Retrieved 2026-09-25, from https://github.com/fnc12/sqlite_orm/blob/master/README.md
[^type-affinity]: SQLite 官方文件 — 型別親和性 (Type Affinity)。SQLite. (n.d.). Retrieved 2026-09-25, from https://www.sqlite.org/datatype3.html