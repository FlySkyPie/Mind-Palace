# C++ 專案中使用 SQL 語法的最佳實踐

## 前言

C++ 是一門系統級程式語言，與資料庫互動時有多種方式可選：從直接使用 SQL 字串搭配 C API，到完整的 ORM 程式碼生成工具。每種方式在型別安全、編譯期檢查、可維護性與執行效能之間有不同取捨。本文將系統性地探討在 C++ 專案中使用 SQL 的主流方案與對應的最佳實踐。

## 一、主流 C++ SQL 函式庫

C++ 生態系中的 SQL 函式庫可按抽象層級分為三大類：原始 SQL 封裝、型別安全查詢建構器（Query Builder）、以及全功能 ORM。[^lib-comparison]

| 函式庫 | 類別 | 支援資料庫 | SQL 輸入方式 | 型別安全 | 最低 C++ 標準 |
|--------|------|-----------|-------------|---------|---------------|
| **SQLiteCpp** | 輕量封裝 | SQLite 專用 | 原始 SQL 字串 | ❌ 無 | C++17 |
| **libpqxx** | 輕量封裝 | PostgreSQL 專用 | 原始 SQL 字串 | ⚠️ 結果綁定型別安全 | C++20 |
| **SOCI** | 中間抽象層 | 7+ 種（含 MySQL、PG、SQLite、Oracle 等） | 原始 SQL 字串 + 型別綁定 | ⚠️ 綁定層級 | C++14 |
| **sqlpp11** | 型別安全查詢建構器 | MySQL、PG、SQLite、SQLCipher | C++ 表達式（無原始 SQL） | ✅ 編譯期 | C++11 |
| **ODB** | 全功能 ORM | SQLite、PG、MySQL、Oracle、MSSQL | 從 C++ 類別自動生成 | ✅ 編譯期 | C++11 |
| **sqlite_orm** | 標頭檔唯 ORM | SQLite 專用 | C++ 表達式（無原始 SQL） | ✅ 編譯期 | C++14 |
| **MySQL Connector/C++** | JDBC 風格封裝 | MySQL/MariaDB 專用 | 原始 SQL 字串 | ❌ 無 | C++11 |

[^lib-comparison]: PythonLib. (n.d.). ORM for C++: Comparison and Examples. Retrieved 2026-09-25, from https://pythonlib.ru/en/library-theme142

### 1.1 原始 SQL 封裝：SQLiteCpp

SQLiteCpp 是 SQLite3 C API 的 C++17 RAII 包裝，使用原始 SQL 字串，但提供現代 C++ 的例外處理與資源管理。[^sqlitecpp]

```cpp
#include <SQLiteCpp/Database.h>
#include <SQLiteCpp/Statement.h>

SQLite::Database db("example.db3", SQLite::OPEN_READWRITE | SQLite::OPEN_CREATE);
db.exec("CREATE TABLE IF NOT EXISTS person (id INTEGER PRIMARY KEY, name TEXT, age INT)");

// 預備陳述式（Prepared Statement）+ 參數綁定
SQLite::Statement insert(db, "INSERT INTO person (name, age) VALUES (?, ?)");
insert.bind(1, "Bob");
insert.bind(2, 25);
insert.exec();
```

[^sqlitecpp]: SRombauts. (n.d.). SQLiteCpp GitHub Repository. Retrieved 2026-09-25, from https://github.com/SRombauts/SQLiteCpp

### 1.2 原始 SQL 搭配型別安全擷取：libpqxx

libpqxx 是 PostgreSQL 官方 libpq 的 C++ 封裝，支援 tuple-based 型別安全的結果提取（C++20 structured bindings）。[^libpqxx]

```cpp
#include <pqxx/pqxx>

pqxx::connection cx;
pqxx::work tx{cx};

// 型別安全的結果提取（編譯期檢查型別轉換）
for (auto [name, salary] : tx.query<std::string, int>(
    "SELECT name, salary FROM employee ORDER BY name"))
{
    std::cout << name << " earns " << salary << "\n";
}
tx.commit();
```

[^libpqxx]: Tv, J. (n.d.). libpqxx GitHub Repository. Retrieved 2026-09-25, from https://github.com/jtv/libpqxx

### 1.3 多資料庫抽象層：SOCI

SOCI 由 CERN 開發，提供統一的 SQL 存取介面，底層使用可插拔的 backend 模組。支援流（stream）風格的參數綁定。[^soci]

```cpp
#include <soci/soci.h>
#include <soci/sqlite3/soci-sqlite3.h>

soci::session sql(soci::sqlite3, "database.db");

int count;
sql << "SELECT count(*) FROM people", soci::into(count);

std::string name = "John";
int age;
sql << "SELECT age FROM people WHERE name = :name",
       soci::use(name), soci::into(age);
```

[^soci]: SOCI Project. (n.d.). SOCI - The C++ Database Access Library. Retrieved 2026-09-25, from https://github.com/SOCI/soci

### 1.4 型別安全查詢建構器：sqlpp11

sqlpp11 的核心設計理念是「在編譯期捕捉 SQL 語法錯誤」。它透過 C++ 模板表達式（EDSL）來建構查詢，編譯器會檢查資料表名稱、欄位名稱與型別。[^sqlpp11]

```cpp
// 需先透過 DDL-to-C++ 程式碼生成定義表格型別
TabFoo foo;
Db db(...);

// 編譯期檢查的查詢：若 foo.name 不存在或型別不符，編譯失敗
for (const auto& row : db(select(foo.name, foo.hasFun)
                          .from(foo)
                          .where(foo.id > 17 and foo.name.like("%bar%"))))
{
    std::string name = row.name;
    bool hasFun = row.hasFun;
}
```

[^sqlpp11]: R Bock. (n.d.). sqlpp11 GitHub Repository. Retrieved 2026-09-25, from https://github.com/rbock/sqlpp11

### 1.5 全功能 ORM：ODB

ODB 是 C++ 最成熟的 ORM，使用 GCC 外掛編譯器從標註過的 C++ 類別自動產生持久化程式碼。[^odb]

```cpp
// 以 pragma 標註的類別
#pragma db object
class Person {
public:
    Person(const std::string& name, int age) : name_(name), age_(age) {}
private:
    friend class odb::access;
    Person() = default;

    #pragma db id auto
    unsigned long id_;
    std::string name_;
    int age_;
};

// 使用 ORM API
odb::sqlite::database db("people.db");
Person p("Alice", 30);
db.persist(p);  // 自動產生 INSERT

auto result = db.query<Person>(odb::query<Person>::age > 25);
// 自動產生 SELECT ... WHERE age > 25
```

[^odb]: Code Synthesis. (n.d.). ODB - Object-Relational Mapping for C++. Retrieved 2026-09-25, from https://www.codesynthesis.com/products/odb/

## 二、SQL 注入防護

**核心原則：永遠不要將使用者輸入直接串接到 SQL 字串中。** 所有現代 C++ SQL 函式庫都支援參數綁定（Parameter Binding），這是防範 SQL 注入的第一道防線。[^owasp-sqli]

### 2.1 使用預備陳述式與參數綁定

```cpp
// ❌ 危險：字串串接
std::string query = "SELECT * FROM users WHERE name = '" + userInput + "'";

// ✅ 安全：參數綁定
// SQLiteCpp
SQLite::Statement query(db, "SELECT * FROM users WHERE name = ?");
query.bind(1, userInput);

// libpqxx
auto result = txn.exec_params(
    "SELECT * FROM users WHERE name = $1", userInput);

// SOCI
sql << "SELECT * FROM users WHERE name = :name", soci::use(userInput);

// sqlpp11（編譯器即防止注入，無原始字串）
auto query = select(all_of(users)).from(users).where(users.name == userInput);
```

### 2.2 使用 Query Builder 或 ORM 避免原始 SQL

sqlpp11、sqlite_orm 和 ODB 這類函式庫完全消除了原始 SQL 字串的拼接需求，從架構層面根除注入漏洞。編譯器會在編譯期檢查型別，進一步防護型別不符造成的安全問題。[^sqlpp11-safety]

### 2.3 常見反模式

- **字串串接**：`"WHERE id = " + std::to_string(id)` — 應使用參數綁定
- **跳脫函式庫**：依賴自製的跳脫函式（如 `escape_string`）容易有遺漏，應使用參數綁定
- **動態 Table/Column 名稱**：參數綁定不適用於資料表或欄位名稱，需白名單驗證

[^owasp-sqli]: OWASP. (n.d.). SQL Injection Prevention Cheat Sheet. Retrieved 2026-09-25, from https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
[^sqlpp11-safety]: R Bock. (n.d.). sqlpp11 Documentation - Safety. Retrieved 2026-09-25, from https://github.com/rbock/sqlpp11

## 三、資料庫連線管理

### 3.1 RAII 模式

C++ 的核心資源管理哲學 RAII（Resource Acquisition Is Initialization）完美適用於資料庫連線。連線在建構時獲取，解構時自動釋放。[^raii]

```cpp
// libpqxx - RAII：解構時自動關閉連線
try {
    pqxx::connection conn("host=localhost dbname=mydb user=app");
    // conn 超出作用域時自動清理
} catch (const pqxx::sql_error& e) {
    std::cerr << "SQL error: " << e.what() << "\n";
}
```

[^raii]: cppreference.com. (n.d.). RAII - Resource Acquisition Is Initialization. Retrieved 2026-09-25, from https://en.cppreference.com/w/cpp/language/raii

### 3.2 連線池（Connection Pool）

對於多執行緒或高併發場景，連線池避免每次請求都建立新連線的開銷。

```cpp
// SOCI 連線池
soci::connection_pool pool(10);  // 10 個連線
soci::session sql(pool);         // 從池中獲取（RAII 自動歸還）

// sqlpp11 連線池
auto pool = sqlpp::mysql::connection_pool::create(
    sqlpp::mysql::connection_config{"host", "user", "pass", "db"});
pool->initial_size(10);
auto db = pool->get(sqlpp::connection_check::ping);  // RAII
```

各函式庫的連線池支援：[^pool-comparison]

| 函式庫 | 連線池支援 | 執行緒安全 |
|--------|-----------|-----------|
| SOCI | `soci::connection_pool` | ✅ 執行緒安全 |
| sqlpp11 | 各 connector 內建 | ✅ 執行緒安全 |
| libpqxx | 無內建（建議使用 PgBouncer） | N/A |
| SQLiteCpp | 不需要（單一檔案） | ❌ 單寫入者 |
| Boost.MySQL | `boost::mysql::connection_pool`（非同步） | ✅ 執行緒安全 |

[^pool-comparison]: Pi Stack. (2026-06-22). Self-Hosted C++ Database Client Libraries 2026. Retrieved 2026-09-25, from https://www.pistack.xyz/posts/2026-06-22-cpp-database-client-libraries-libpqxx-soci-redisplusplus-mongocxx/

## 四、交易（Transaction）處理

### 4.1 RAII 交易守護

交易也應使用 RAII 模式管理：建構時 BEGIN，解構時若未提交則自動 ROLLBACK。[^soci-transaction]

```cpp
// SOCI - RAII 交易
{
    soci::transaction tr(sql);
    sql << "INSERT INTO accounts (id, balance) VALUES (1, 100)";
    sql << "INSERT INTO accounts (id, balance) VALUES (2, 200)";
    tr.commit();  // 明確提交
}  // 若例外發生或未呼叫 commit()，解構時自動 ROLLBACK

// sqlite_orm - lambda 交易
storage.transaction([&]() -> bool {
    auto user = storage.get<User>(2);
    user.typeId = 1;
    storage.update(user);
    return true;  // true = commit, false = rollback
});

// sqlite_orm - RAII guard
auto guard = storage.transaction_guard();
user.name = "Paul";
// 若此處拋出例外，guard 解構時自動 ROLLBACK
guard.commit();
```

### 4.2 交易最佳實踐

- **保持交易簡短**：鎖定資源的時間越長，競爭越嚴重
- **選擇正確的隔離等級**：一般使用 READ COMMITTED，財務等級資料考慮 SERIALIZABLE
- **注意巢狀交易**：SQLite 不支援巢狀交易，需使用 SAVEPOINT

[^soci-transaction]: SOCI Project. (n.d.). SOCI Transactions Documentation. Retrieved 2026-09-25, from https://github.com/SOCI/soci/blob/master/docs/transactions.md

## 五、效能最佳化

### 5.1 預備陳述式（Prepared Statements）

對於執行多次的同一個查詢，預備陳述式可避免重複解析與最佳化 SQL。[^libpqxx-perf]

```cpp
// sqlpp11 - 預備陳述式
auto preparedInsert = storage.prepare(
    insert_into(foo).set(foo.id = parameter(foo.id),
                         foo.name = parameter(foo.name))
);
// 重複使用，只綁定新值
get<0>(preparedInsert) = 1;
get<1>(preparedInsert) = "Alice";
storage.execute(preparedInsert);
```

### 5.2 批次操作

```cpp
// libpqxx COPY 協定（大量資料載入最佳方案）
pqxx::stream_to stream(conn, "users", {"id", "name", "email"});
stream << 1 << "Alice" << "alice@example.com" << pqxx::endrow;
stream << 2 << "Bob" << "bob@example.com" << pqxx::endrow;
stream.complete();

// libpqxx pipeline 模式（每秒 10 萬+ 查詢）
pqxx::pipeline pipe(conn);
pipe.insert("INSERT INTO logs (msg) VALUES ('entry1')");
pipe.insert("INSERT INTO logs (msg) VALUES ('entry2')");
pipe.complete();

// SOCI 批次插入
std::vector<int> ids = {1, 2, 3};
std::vector<std::string> names = {"Alice", "Bob", "Charlie"};
sql << "INSERT INTO users (id, name) VALUES (:id, :name)",
       soci::use(ids), soci::use(names);
```

### 5.3 其他效能原則

- 使用連線池減少連線建立開銷
- 善用 LIMIT/OFFSET 限制結果集大小
- 為 WHERE、JOIN 與 ORDER BY 涉及的欄位建立索引
- 對於大型結果集，使用 libpqxx 的 `stream<T>()` 進行串流讀取而非一次性載入

[^libpqxx-perf]: Tv, J. (n.d.). libpqxx Documentation - Performance. Retrieved 2026-09-25, from https://github.com/jtv/libpqxx

## 六、程式碼組織模式

### 6.1 Repository 模式

將所有資料庫邏輯封裝在 Repository 介面後方，隔離 SQL 實作細節，便於測試與替換。[^repo-pattern]

```cpp
class UserRepository {
public:
    virtual ~UserRepository() = default;
    virtual std::optional<User> findById(int id) = 0;
    virtual std::vector<User> findByName(const std::string& name) = 0;
    virtual void save(const User& user) = 0;
};

class SqliteUserRepository : public UserRepository {
    SQLite::Database& db_;
public:
    std::optional<User> findById(int id) override {
        SQLite::Statement query(db_, "SELECT * FROM users WHERE id = ?");
        query.bind(1, id);
        if (query.executeStep()) {
            return User{/* ... */};
        }
        return std::nullopt;
    }
};
```

### 6.2 SQL 分離策略

```text
專案結構範例（將 SQL 集中管理）：
project/
├── sql/
│   ├── migrations/           # 資料庫遷移腳本
│   │   ├── 001_create_users.sql
│   │   └── 002_add_posts.sql
│   ├── queries/              # 命名查詢
│   │   ├── users/get_active.sql
│   │   └── posts/by_user.sql
│   └── schema.sql
├── src/
│   ├── database/
│   │   ├── ConnectionManager.cpp
│   │   ├── QueryLoader.cpp   # 載入 .sql 檔案
│   │   └── migrations.cpp
│   └── models/
│       ├── User.cpp
│       └── Post.cpp
```

### 6.3 遷移（Migration）管理

資料庫 Schema 變更應透過版本化遷移管理，而非手動修改資料庫：

```cpp
// TinyORM 遷移（命令列）
"tom make:migration create_users_table"

// sqlite_orm - 自動 Schema 同步
storage.sync_schema();  // 自動比對並更新 Schema

// ODB - 從模型標頭檔產生資料庫程式碼
"odb -d mysql person.hxx"
```

### 6.4 選型決策樹

取決於專案規模與需求：

| 專案類型 | 建議方案 | 原因 |
|----------|---------|------|
| 小型（< 10 張表） | sqlite_orm 或 SQLiteCpp | 輕量、設定簡單 |
| 中型（10-50 張表） | Repository 模式 + sqlpp11 或 SOCI | 兼顧型別安全與靈活性 |
| 大型（> 50 張表） | ODB 或 TinyORM + 獨立遷移檔案 | 結構化管理、團隊協作 |
| 既有資料庫 | sqlpp11（DDL-to-C++ 腳本）或原始 SQL + 預備陳述式 | 不需修改 Schema |
| 多資料庫相容 | SOCI 或 sqlpp11 | 一次撰寫，多處執行 |

[^repo-pattern]: Microsoft. (n.d.). Repository Pattern. In .NET Architecture Guides. Retrieved 2026-09-25, from https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design

## 結論

在 C++ 專案中使用 SQL 的最佳實踐可歸納為以下核心原則：

1. **選擇合適的抽象層級**：輕量專案用 sqlite_orm/SQLiteCpp，中型專案用 sqlpp11/SOCI，大型專案用 ODB
2. **永遠使用參數綁定**：防止 SQL 注入的第一道防線，所有主流函式庫都支援
3. **採用 RAII 管理資源**：連線、交易、預備陳述式都應透過 RAII 模式自動管理生命週期
4. **使用 Repository 模式**：隔離資料庫邏輯，提升可測試性與可維護性
5. **管理資料庫遷移**：使用版本化的遷移腳本，避免 Schema 漂移
6. **優先使用預備陳述式與批次操作**：重複利用查詢計畫，大幅提升效能
7. **連線池應對高併發**：減少連線建立開銷，善用 SOCI、sqlpp11 或 Boost.MySQL 的內建支援