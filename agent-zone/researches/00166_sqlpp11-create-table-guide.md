# sqlpp11 建立表格（Table）完整指南

## 概述

sqlpp11 是一套型別安全的 C++ SQL 模板函式庫，採用嵌入式領域特定語言（EDSL）設計，讓開發者能以 C++ 結構體和函式操作 SQL 查詢，並在編譯期進行型別檢查。[^sqlpp11]

**核心概念：sqlpp11 不會自動從 C++ 定義建立資料庫表格。** 使用者必須自行透過原始 SQL 在資料庫中建立表格，然後將表格結構以 C++ 結構體的方式定義（或自動產生）供 sqlpp11 使用。

## 流程總覽

```mermaid
flowchart LR
    A[撰寫 DDL<br/>CREATE TABLE ...] --> B[執行 ddl2cpp<br/>產生 C++ Header]
    B --> C[在 C++ 中使用<br/>db.execute 建立實際表格]
    C --> D[使用產生的結構體<br/>進行型別安全查詢]
```

## 步驟一：撰寫 DDL（資料定義語言）

首先撰寫標準 SQL 的 `CREATE TABLE` 陳述式，例如：

```sql
CREATE TABLE tab_sample (
    alpha bigint(20) DEFAULT NULL,
    beta tinyint(1) DEFAULT NULL,
    gamma varchar(255) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```

## 步驟二：使用 ddl2cpp 產生 C++ 定義

sqlpp11 官方提供一個 Python 腳本 `scripts/ddl2cpp`，能自動將 DDL 解析並產生對應的 C++ 結構體。[^ddl2cpp]

```bash
python scripts/ddl2cpp TabSample.ddl ~/output/TabSample MyNamespace
```

### 支援的命令列參數

| 參數 | 說明 |
|------|------|
| `-no-timestamp-warning` | 隱藏日期/時間類型的時區警告 |
| `-auto-id` | 假設 `id` 欄位具有自動遞增值（如 SQLite ROWID） |
| `-identity-naming` | 保留原始表格/欄位名稱（預設轉為 CamelCase） |
| `-split-tables` | 每個表格產生獨立的 Header 檔案 |
| `--datatype-file=<path.csv>` | 提供 CSV 檔案對應不支援的資料型別 |
| `--help` | 顯示幫助資訊 |
| `--test` | 執行腳本自我測試 |

### 支援的 SQL 資料型別對應

| SQL 型別 | ddl2cpp 輸出 | sqlpp11 C++ 型別 |
|-----------|-------------|------------------|
| `BOOL`, `BOOLEAN` | `boolean` | `sqlpp::boolean` |
| `INT`, `INTEGER`, `BIGINT`, `SMALLINT`, `TINYINT`, `MEDIUMINT` | `integer` | `sqlpp::integer`, `sqlpp::bigint`, `sqlpp::smallint` 等 |
| `BIGSERIAL`, `SERIAL`, `SERIAL2` 等 | `integer`（含 `hasAutoValue`） | 同上，自動標記 `must_not_insert` |
| `FLOAT`, `DOUBLE`, `DECIMAL`, `NUMERIC`, `REAL` | `floating_point` | `sqlpp::floating_point` |
| `CHAR`, `VARCHAR`, `TEXT`, `CLOB`, `ENUM`, `JSON`, `JSONB` | `text` | `sqlpp::varchar` |
| `BLOB`, `BYTEA`, `BINARY`, `VARBINARY` | `blob` | `sqlpp::blob` |
| `DATE` | `day_point` | `sqlpp::day_point` |
| `DATETIME`, `TIMESTAMP`, `TIMESTAMPTZ` | `time_point` | `sqlpp::time_point` |
| `TIME` | `time_of_day` | `sqlpp::time_of_day` |
| `INT UNSIGNED` 等 | `integer_unsigned` | `sqlpp::integer_unsigned`, `sqlpp::bigint_unsigned` |

不支援的型別會標記為 `UNKNOWN`，可透過 `--datatype-file` 提供 CSV 映射檔解決。[^datatypefile]

## 步驟三：產生的 C++ 結構體解析

以下為 ddl2cpp 產生的輸出範例，展示 C++ 表格定義的完整結構：

```cpp
#pragma once

#include <sqlpp11/table.h>
#include <sqlpp11/data_types.h>
#include <sqlpp11/char_sequence.h>

namespace MyNamespace
{
  // ===== 欄位定義命名空間 =====
  // 命名規則：<TableName>_
  namespace TabSample_
  {
    // --- 每個欄位是一個結構體 ---
    struct Alpha
    {
      // _alias_t 定義 SQL 欄位名稱與型別對應
      struct _alias_t
      {
        static constexpr const char _literal[] = "alpha";
        using _name_t = sqlpp::make_char_sequence<sizeof(_literal), _literal>;

        // _member_t 提供查詢結果的存取介面
        template <typename T>
        struct _member_t
        {
          T alpha;  // camelCase 版本
          T& operator()() { return alpha; }
          const T& operator()() const { return alpha; }
        };
      };

      // _traits 定義資料型別與約束標籤
      using _traits = ::sqlpp::make_traits<
          ::sqlpp::bigint,
          ::sqlpp::tag::can_be_null
      >;
    };

    struct Beta
    {
      struct _alias_t
      {
        static constexpr const char _literal[] = "beta";
        using _name_t = sqlpp::make_char_sequence<sizeof(_literal), _literal>;
        template <typename T>
        struct _member_t
        {
          T beta;
          T& operator()() { return beta; }
          const T& operator()() const { return beta; }
        };
      };
      using _traits = ::sqlpp::make_traits<::sqlpp::varchar, ::sqlpp::tag::can_be_null>;
    };

    struct Gamma
    {
      struct _alias_t
      {
        static constexpr const char _literal[] = "gamma";
        using _name_t = sqlpp::make_char_sequence<sizeof(_literal), _literal>;
        template <typename T>
        struct _member_t
        {
          T gamma;
          T& operator()() { return gamma; }
          const T& operator()() const { return gamma; }
        };
      };
      using _traits = ::sqlpp::make_traits<::sqlpp::boolean>;
    };
  } // namespace TabSample_

  // ===== 表格結構體 =====
  // 繼承 sqlpp::table_t，第一個模板參數是自身，其餘是各欄位結構體
  struct TabSample
      : sqlpp::table_t<TabSample,
                       TabSample_::Alpha,
                       TabSample_::Beta,
                       TabSample_::Gamma>
  {
    struct _alias_t
    {
      static constexpr const char _literal[] = "tab_sample";
      using _name_t = sqlpp::make_char_sequence<sizeof(_literal), _literal>;
      template <typename T>
      struct _member_t
      {
        T tabSample;
        T& operator()() { return tabSample; }
        const T& operator()() const { return tabSample; }
      };
    };
  };
} // namespace MyNamespace
```

### 結構體元件說明

#### `_alias_t`（每個結構體都必須有）

提供 SQL 名稱與 C++ 型別系統的連結：

- `_literal`：靜態字串陣列，存 SQL 的原始名稱
- `_name_t`：透過 `sqlpp::make_char_sequence` 將字串轉為編譯期型別
- `_member_t<T>`：模板結構體，提供查詢結果的存取成員

#### `_traits`（僅欄位結構體需要）

透過 `sqlpp::make_traits<>` 定義欄位的資料型別與約束：

| 標籤（Tag） | 說明 |
|------------|------|
| `sqlpp::tag::can_be_null` | 欄位可為 NULL |
| `sqlpp::tag::must_not_insert` | INSERT 時跳過此欄位（如自動遞增主鍵） |
| `sqlpp::tag::must_not_update` | UPDATE 時跳過此欄位 |
| `sqlpp::tag::require_insert` | INSERT 時必須提供此欄位值 |

標籤邏輯規則：

- 若有 `hasAutoValue`（SERIAL / AUTO_INCREMENT）或 `autoId` 啟用且欄位名為 `id` → 加上 `must_not_insert` + `must_not_update`
- 若非 `NOT NULL` 且非主鍵 → 加上 `can_be_null`
- 若無預設值、無自動值、非 nullable → 加上 `require_insert`[^ddl2cpp_gen]

## 步驟四：在資料庫中建立實際表格

sqlpp11 的 C++ 結構體僅供編譯期型別檢查與查詢建構使用，**實際的 SQL 表格必須另外建立**。常見做法：

### 方法 A：使用 db.execute()

```cpp
#include <sqlpp11/sqlite3/sqlite3.h>
#include <sqlpp11/sqlpp11.h>
#include "TabSample.h"

namespace sql = sqlpp::sqlite3;

int main() {
    // 1. 建立連線
    sql::connection_config config;
    config.path_to_database = ":memory:";
    config.flags = SQLITE_OPEN_READWRITE | SQLITE_OPEN_CREATE;
    sql::connection db(config);

    // 2. 使用原始 SQL 建立表格
    db.execute(R"(CREATE TABLE tab_sample (
        alpha INTEGER PRIMARY KEY,
        beta varchar(255) DEFAULT NULL,
        gamma bool DEFAULT NULL
    ))");

    // 3. 實例化 C++ 表格結構體，開始型別安全操作
    const auto tab = TabSample{};

    // 插入
    db(insert_into(tab).set(tab.alpha = 1, tab.beta = "hello", tab.gamma = true));

    // 查詢
    for (const auto& row : db(select(all_of(tab)).from(tab).unconditionally())) {
        std::cout << row.alpha << ", " << row.beta << ", " << row.gamma << std::endl;
    }

    // 更新
    db(update(tab).set(tab.gamma = false).where(tab.alpha == 1));

    // 刪除
    db(remove_from(tab).where(tab.alpha == tab.alpha + 3));
}
```

### 方法 B：使用外部資料庫遷移工具

在正式專案中，建議搭配資料庫遷移工具（如 Flyway、Liquibase 或自訂腳本）管理 DDL，而非在程式中直接執行 `CREATE TABLE`，以確保資料庫結構的版本控制。[^sample_cpp]

## 手動定義表格（不建議）

雖然可以直接手寫 C++ 結構體，但官方強烈建議使用 ddl2cpp 自動產生，原因如下：

1. 結構體模式繁複，每個欄位都需要 `_alias_t`、`_member_t`、`_traits` 三個內部結構
2. camelCase 轉換、標籤邏輯容易出錯
3. 使用腳本能確保 DDL 與 C++ 定義一致[^tables_doc]

## 常見問題

### Q: ddl2cpp 顯示 "Error: datatype of xxx.yyy is not supported"

解法一：在 sqlpp11 中實作該資料型別（參考 `include/sqlpp11/data_types`）。

解法二：使用 `--datatype-file` 提供 CSV 映射：

```csv
Boolean, one_or_zero
Text, url, uuid
```

解法三：修改 ddl2cpp 腳本中的型別清單。

### Q: 日期時間型別出現時區警告

sqlpp11 的 `day_point` 和 `time_point` 不處理時區資訊。如果資料庫使用 `TIMESTAMP WITH TIME ZONE`，需在應用層自行處理時區轉換。可使用 `-no-timestamp-warning` 隱藏警告。

## 總結

sqlpp11 建立表格的標準流程：

1. **撰寫 DDL** → 標準 SQL `CREATE TABLE`
2. **執行 `ddl2cpp`** → 自動產生 C++ 結構體定義
3. **`db.execute("CREATE TABLE ...")`** → 在資料庫中建立實際表格
4. **使用結構體** → 進行型別安全的 CRUD 操作

sqlpp11 的設計哲學是**定義與建立分離**：C++ 結構體僅負責查詢的型別安全，實際的資料庫結構管理仍回歸標準 SQL 工具。

[^sqlpp11]: rbock. (n.d.). sqlpp11: A type safe SQL template library for C++. Retrieved 2026-09-25, from https://github.com/rbock/sqlpp11

[^ddl2cpp]: rbock. (n.d.). ddl2cpp script. Retrieved 2026-09-25, from https://github.com/rbock/sqlpp11/blob/main/scripts/ddl2cpp

[^datatypefile]: rbock. (n.d.). sqlpp11 README — Unsupported column types. Retrieved 2026-09-25, from https://github.com/rbock/sqlpp11?tab=readme-ov-file#unsupported-column-types

[^tables_doc]: rbock. (n.d.). sqlpp11 Tables documentation. Retrieved 2026-09-25, from https://github.com/rbock/sqlpp11/blob/main/docs/Tables.md

[^sample_cpp]: rbock. (n.d.). sqlpp11 SQLite3 usage sample. Retrieved 2026-09-25, from https://github.com/rbock/sqlpp11/blob/main/tests/sqlite3/usage/Sample.cpp

[^ddl2cpp_gen]: rbock. (n.d.). ddl2cpp — createHeader function (code generation logic). Retrieved 2026-09-25, from https://github.com/rbock/sqlpp11/blob/main/scripts/ddl2cpp

[^sample_header]: rbock. (n.d.). sqlpp11 core usage sample header. Retrieved 2026-09-25, from https://github.com/rbock/sqlpp11/blob/main/tests/core/usage/Sample.h