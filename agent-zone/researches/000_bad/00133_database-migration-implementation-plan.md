# 簡易資料庫遷移（Database Migration）實作計劃

> [!WARNING] 對齊失敗
> 需要更具體的實作計畫，而不是抽象且範圍廣泛的實作。

## 概述

本文探討如何在**不依賴第三方函式庫**的前提下，自行實作一套簡易的資料庫遷移系統。資料庫遷移是一種將資料庫綱要（Schema）之變更以版本控制方式管理的做法，使每一次綱要改動皆可追溯、可重現、可復原。以下將從追蹤機制、檔案組織、核心演算法、交易處理、向上/向下遷移、常見陷阱等面向逐一說明。

---

## 1. 追蹤遷移狀態（Tracking Migration State）

所有自製遷移系統的核心概念是一個專門用來記錄「哪些遷移已被執行」的追蹤表，常見命名為 `schema_migrations`[^shesh]。

### 最小設計

```sql
CREATE TABLE IF NOT EXISTS schema_migrations (
    version     INTEGER   PRIMARY KEY,
    name        TEXT      NOT NULL,
    migrated_at TIMESTAMP NOT NULL DEFAULT NOW()
);
```

- `version`：遷移檔案的版本編號（唯一）
- `name`：遷移檔案名稱（便於除錯）
- `migrated_at`：執行時間

### 更強固的設計

若需要支援檢查碼、執行時間量測、狀態追蹤，可擴充為以下結構[^sqlcheat]：

```sql
CREATE TABLE schema_migrations (
    version          VARCHAR(255) PRIMARY KEY,
    applied_at       TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    execution_time_ms INT,
    checksum         VARCHAR(64),
    description      TEXT,
    type             VARCHAR(20),   -- 'up' 或 'down'
    status           VARCHAR(20) DEFAULT 'pending'
);
```

### 防止並發執行的鎖定機制

在分散式部署環境中，可能多個實例同時嘗試執行遷移。可額外使用「鎖定表」或資料庫建議鎖（advisory lock）來防止衝突[^oneuptime]：

```sql
CREATE TABLE IF NOT EXISTS schema_migrations_lock (
    id        INTEGER PRIMARY KEY DEFAULT 1,
    locked_at TIMESTAMP WITH TIME ZONE,
    locked_by VARCHAR(255),
    CONSTRAINT single_row CHECK (id = 1)
);
```

---

## 2. 遷移檔案組織（Migration File Organization）

### 命名慣例

| 方式 | 範例 | 優點 | 缺點 |
|------|------|------|------|
| 序號前綴 | `001_create_users.sql` | 簡單直觀 | 多人協作易衝突 |
| 時間戳前綴 | `20240315120000_create_users.sql` | 避免衝突 | 檔名較長 |

### 向上/向下檔案模式

每筆遷移應包含「向上（up，套用變更）」與「向下（down，復原變更）」兩種操作[^sqlcheat]：

```
migrations/
├── 001_create_users_table.up.sql
├── 001_create_users_table.down.sql
├── 002_add_email_to_users.up.sql
├── 002_add_email_to_users.down.sql
└── ...
```

### 排序原則

檔案需以**字典序（lexicographic order）**排序，以確保執行順序正確[^shesh]。序號或時間戳格式需保證排序結果即為期望的執行順序。

---

## 3. 偵測未套用遷移（Detecting Pending Migrations）

核心演算法如下[^shesh]：

1. 掃描遷移目錄，解析所有檔案及其版本編號
2. 查詢 `schema_migrations` 表，取得已執行的版本集合
3. 兩者對比，篩選出「尚未執行」的遷移
4. 依版本排序後依序執行

```python
# 步驟 1：取得所有遷移檔案
migrations = []
for path in Path('./migrations').iterdir():
    migration = {}
    migration["name"] = path.name
    migration["version"] = int(path.name.split("_")[0])

# 步驟 2：從資料庫查詢已執行版本
query = "SELECT version FROM schema_migrations ORDER BY version ASC"
records = await connection.fetch(query)
applied_versions = [r["version"] for r in records]

# 步驟 3：篩選未執行者
pending = [m for m in migrations
           if m["version"] not in applied_versions]

# 步驟 4：排序後依序執行
pending = sorted(pending, key=lambda m: m['version'])
```

### 檢查碼（Checksum）驗證

為確保遷移檔案在套用後未被修改，可在執行時計算檔案內容的 SHA-256 雜湊值並存入追蹤表[^sqlcheat]。每次啟動時比對檔案與記錄的檢查碼，若有差異即發出警示。

---

## 4. 向上/向下遷移（Up / Down Operations）

### 向上遷移（Up）

執行尚未套用過的遷移，依版本順序將其 SQL 內容逐一執行，並在成功後寫入 `schema_migrations` 記錄。

### 向下遷移（Down / Rollback）

復原最近 N 筆已套用的遷移，反序執行其 `.down.sql`[^oneuptime]：

```python
# 取得已執行的遷移清單（最新者在前）
executed = get_executed_migrations()
to_rollback = reversed(executed)[:N]  # 最近 N 筆

for migration in to_rollback:
    begin_transaction()
    execute(migration.down_sql)         # 執行向下 SQL
    delete_from_schema_migrations(migration.version)  # 移除記錄
    commit_transaction()
```

### 實務建議：向前修復優先於向後復原

在正式環境中，「向前修復（roll forward）」通常比「向後復原（roll back）」更安全[^dev_article]。原因是：若某個綱要變更（如新增欄位）已被應用，且應用程式已寫入資料，此時復原綱變更將導致資料遺失。較穩健的做法是撰寫一筆新的「修正性向前遷移」來取代直接復原。

---

## 5. 交易處理（Transaction Handling）

### PostgreSQL vs MySQL 的重大差異

**PostgreSQL** 的 DDL（Data Definition Language，如 `CREATE TABLE`、`ALTER TABLE`）是**交易性**的[^zero_downtime]：

```sql
BEGIN;
ALTER TABLE users ADD COLUMN status text DEFAULT 'active';
ALTER TABLE users ADD COLUMN role text;
-- 若第二行失敗，第一行也會被 ROLLBACK 復原
COMMIT;
```

**MySQL 8.0 以前**的 DDL 會**隱含提交（implicit commit）**，亦即每一條 DDL 都會立即提交。將 DDL 包在 `BEGIN...ROLLBACK` 中是無效的[^zero_downtime]。

### 建議的交易模式

每筆遷移應在單一交易內完成「執行 SQL」與「寫入追蹤記錄」兩個動作[^shesh]：

```python
with connection.transaction():
    for migration in pending_migrations:
        execute(migration.sql_content)
        execute(
            "INSERT INTO schema_migrations (version, name) VALUES ($1, $2)",
            migration["version"], migration["name"]
        )
```

這樣可以確保綱要變更與記錄寫入是原子性的。

### 鎖定逾時

為避免遷移因等待鎖而無限期阻塞，應設定鎖定逾時[^postgresai]：

```sql
SET LOCAL lock_timeout = '3s';   -- PostgreSQL
-- 或
SET lock_wait_timeout = 3;       -- MySQL
```

---

## 6. 實作計劃摘要

以下為實作一套簡易遷移系統的步驟建議：

| 步驟 | 說明 |
|------|------|
| **1. 建立追蹤表** | 在目標資料庫建立 `schema_migrations` 表 |
| **2. 定義檔案格式** | 採用 `{version}_{description}.up.sql` / `.down.sql` 命名 |
| **3. 實作掃描器** | 讀取遷移目錄、解析版本編號、依字典序排序 |
| **4. 實作差異比對** | 比對檔案版本與資料庫記錄，找出待執行遷移 |
| **5. 實作執行器** | 每筆遷移在交易內執行 SQL + 寫入記錄 |
| **6. 實作向下復原** | 反序執行最近 N 筆遷移的 `.down.sql` |
| **7. 加入鎖定防護** | 防止並發執行 |
| **8. 加入檢查碼驗證** | 偵測已套用遷移被修改的情況 |

---

## 7. 常見陷阱與注意事項

1. **在單一步驟中新增 `NOT NULL` 欄位**：在 PostgreSQL 11 以前，這會觸發整張表的獨佔鎖定與重寫，對大型表格可能耗時數小時。應先新增可為 NULL 的欄位，填充資料後再單獨加上 `NOT NULL` 限制[^postgresai]。

2. **在交易內執行 `CREATE INDEX CONCURRENTLY`**：PostgreSQL 的並行索引建立**不能**在交易內執行，若失敗會留下 `INVALID` 狀態的索引。

3. **修改已套用的遷移**：已套用至任一環境的遷移應視為不可變，任何修正都應以新的遷移檔案進行[^dev_article]。

4. **將綱變更與資料遷移放在同一筆遷移中**：長時間執行的資料回填若與 DDL 在同一交易內，會長時間鎖定表格，應拆分為不同步驟[^postgresai]。

5. **未在與正式環境規模相當的資料庫上測試**：在開發環境 2 秒完成的遷移，可能在數 TB 的正式表格上鎖定數小時。

6. **在 MySQL 上未使用冪等語法（idempotent guards）**：由於 MySQL DDL 會隱含提交，部分失敗後重試可能遇到「欄位已存在」等錯誤，應使用 `IF NOT EXISTS` 或查詢 `information_schema` 保護[^zero_downtime]。

---

## 參考資料

[^shesh]: Shesh Babu. (2022). Demystifying Postgres Schema Migrations. Retrieved 2026-09-25, from https://www.sheshbabu.com/posts/demystifying-postgres-schema-migrations/
[^dev_article]: Rhuturaj Takle. (2025). Database Migrations: Managing Schema Changes as Version-Controlled Code. Retrieved 2026-09-25, from https://dev.to/rhuturaj_takle/database-migrations-managing-schema-changes-as-version-controlled-code-1o18
[^sqlcheat]: SQL Cheat. (n.d.). SQL Database Migrations. Retrieved 2026-09-25, from https://sqlcheat.com/tutorials/sql-database-migrations/
[^oneuptime]: OneUptime. (2026-01-22). Node.js Database Migration System. Retrieved 2026-09-25, from https://oneuptime.com/blog/post/2026-01-22-nodejs-database-migration-system/view
[^zero_downtime]: Zero-Downtime Schema. (n.d.). Transactional vs Non-Transactional Databases. Retrieved 2026-09-25, from https://www.zero-downtime-schema.com/database-migration-fundamentals-tool-selection/transactional-vs-non-transactional-dbs/
[^postgresai]: Postgres.ai. (2022-05-25). Common DB Schema Change Mistakes. Retrieved 2026-09-25, from https://postgres.ai/blog/20220525-common-db-schema-change-mistakes