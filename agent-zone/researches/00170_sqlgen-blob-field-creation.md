# sqlgen 中建立 BLOB 欄位的方法

## 背景

sqlgen 是 getml 開發的一套基於 C++20 的反射式 ORM 與 SQL 查詢產生器，類似 Python 的 SQLAlchemy 或 Rust 的 Diesel[^sqlgen-repo]。sqlgen 本身**並未內建 BLOB（二進位大型物件）資料型別**——其型別系統中沒有 `Blob` 或 `Binary` 結構體，且 SQLite 後端的 `type_to_sql()` 對應表中沒有任何型別對應到 SQLite 的 `BLOB` 型別[^sqlite-to-sql]。

## 使用 Dynamic 機制建立 BLOB 欄位

sqlgen 提供 `sqlgen::Dynamic` 擴充機制，允許使用者定義自訂 SQL 型別。透過特化 `sqlgen::parsing::Parser<T>` 樣板，並在 `to_type()` 中回傳 `Dynamic{"BLOB"}`，即可為任意自訂的二元資料型別（如 `std::vector<uint8_t>`、`std::array<uint8_t, N>` 或自訂結構）建立 BLOB 欄位[^sqlgen-dynamic]。

```cpp
#include <sqlgen/dynamic/types.hpp>
#include <sqlgen/parsing/Parser.hpp>

namespace sqlgen::parsing {

template <>
struct Parser<std::vector<uint8_t>> {
  using Type = std::vector<uint8_t>;

  static Result<std::vector<uint8_t>> read(
      const std::optional<std::string>& dbValue) {
    if (!dbValue) {
      return error("BLOB cannot be NULL.");
    }
    // 從資料庫傳回的十六進位字串（或 base64 編碼）轉換為二元資料
    return hex_to_bytes(*dbValue);  // 需自行實作轉換
  }

  static std::optional<std::string> write(
      const std::vector<uint8_t>& value) {
    // 將二元資料轉換為資料庫可接受的十六進位字串
    return bytes_to_hex(value);  // 需自行實作轉換
  }

  static dynamic::Type to_type() noexcept {
    return sqlgen::dynamic::types::Dynamic{"BLOB"};
  }
};

}  // namespace sqlgen::parsing
```

### `Dynamic` 結構說明

`sqlgen::dynamic::types::Dynamic` 包含兩個成員[^types-header]：

- `type_name`：SQL 型別名稱字串（此處設定為 `"BLOB"`）
- `properties`：欄位屬性（如 `primary`、`nullable`、`unique`、`auto_incr`、`foreign_key_reference`）

`type_name` 的值會直接傳遞至資料庫，不對應任何內建型別對應——這正是它能產生任意自訂 SQL 型別的關鍵。當 `type_name` 設為 `"BLOB"` 時，sqlgen 產生的 `CREATE TABLE` 語句中該欄位型別即為 `BLOB`。

### Parser 特化的三個必要方法

1. **`read`**：將資料庫字串（或 `std::nullopt` 代表 SQL NULL）轉換為 C++ 型別 `T`
2. **`write`**：將 C++ 型別 `T` 轉換為資料庫可接受的字串（回傳 `std::nullopt` 代表寫入 SQL NULL）
3. **`to_type`**：回傳 `Dynamic{"BLOB"}` 以告知 sqlgen 此型別對應的 SQL 型別名稱

### DuckDB 專用特化

若使用 DuckDB 作為後端，則需要額外特化 `sqlgen::duckdb::parsing::Parser<T>`，其介面與通用 Parser 不同——`read` 接受 `const duckdb_string_t*`，`write` 接受 `duckdb_appender` 參數[^sqlgen-dynamic]。

### 在結構體中使用 BLOB 型別

一旦完成 Parser 特化，即可在結構體中使用該自訂型別：

```cpp
struct Document {
  sqlgen::PrimaryKey<int> id;
  std::string name;
  std::vector<uint8_t> content;  // 對應 SQL BLOB
  std::optional<std::vector<uint8_t>> thumbnail;  // 可為 NULL 的 BLOB
};
```

### 跨資料庫的對應建議

- **SQLite**：`Dynamic{"BLOB"}`
- **PostgreSQL**：`Dynamic{"BYTEA"}`
- **MySQL**：`Dynamic{"BLOB"}` 或 `Dynamic{"LONGBLOB"}`
- **DuckDB**：`Dynamic{"BLOB"}`

## 與 `rfl::Binary` 的區別

reflect-cpp（sqlgen 的依賴庫）中有一個 `rfl::Binary<T>`，但它並非 SQL 二元型別——它實際上包裝了一個無號整數，將其序列化為二元字串表示（如 `"101010"`）用於旗標/位元模式用途，其 `ReflectionType` 為 `std::string`，是純文字表示，不適合儲存任意二元資料[^reflect-binary]。

## 已知限制

- sqlgen 內建的 `type_to_sql` 對應表中沒有 `BLOB` 型別，因此無法透過內建型別直接使用
- `column_or_value_to_sql()` 中對 `dynamic::String` 的處理會將值以單引號包裝並跳脫單引號——如果您的二元編碼格式包含單引號需特別注意
- sqlgen 目前沒有 BLOB 相關的測試案例或使用範例

## 結論

要在 sqlgen 中建立 BLOB 欄位，需使用 `sqlgen::Dynamic` 擴充機制，為您的二元資料型別特化 `sqlgen::parsing::Parser<T>`，並在 `to_type()` 中回傳 `Dynamic{"BLOB"}`。此機制可產生對應的 SQL `BLOB` 型別，並與 sqlgen 的所有操作（`create_table`、`insert`、`select`、`update`、`delete`）相容。

## 參考資料

[^sqlgen-repo]: getml. (n.d.). sqlgen: A reflection-based ORM and SQL query generator for C++20. Retrieved 2026-09-25, from https://github.com/getml/sqlgen
[^sqlgen-dynamic]: getml. (n.d.). sqlgen::Dynamic — Custom SQL types. Retrieved 2026-09-25, from https://github.com/getml/sqlgen/blob/4515fc6f8984d82e5d6351c92f5ebe20cb31f6fb/docs/content/dynamic.md
[^sqlite-to-sql]: getml. (n.d.). src/sqlgen/sqlite/to_sql.cpp — SQLite type mapping. Retrieved 2026-09-25, from https://raw.githubusercontent.com/getml/sqlgen/main/src/sqlgen/sqlite/to_sql.cpp
[^types-header]: getml. (n.d.). include/sqlgen/dynamic/types.hpp — Dynamic type definitions. Retrieved 2026-09-25, from https://raw.githubusercontent.com/getml/sqlgen/main/include/sqlgen/dynamic/types.hpp
[^reflect-binary]: getml. (n.d.). rfl::Binary — reflect-cpp binary representation. Retrieved 2026-09-25, from https://github.com/getml/reflect-cpp/tree/main/include/rfl/Binary.hpp