# C++ 資料庫遷移方法論 — 通用模式整理

## 概述

資料庫遷移（Database Migration）的機制是獨立於語言的。C++ 專案因缺乏統一的 ORM 生態系，反而更常採用純 SQL 的遷移方法。本報告整理通用的遷移方法與模式，而非特定的函式庫或工具。

## 1. Schema 版本管理（Schema Versioning）

遷移的核心是「將資料庫 schema 當作版本化程式碼管理」。有三種通用編號策略[^ver]：

| 策略 | 格式範例 | 特性 |
|---|---|---|
| 序號遞增 | `001`、`002`、`003` | 簡單易懂，但多人協作合併時容易衝突 |
| 時間戳記 | `20240315120000_add_user_table` | 避免合併衝突，提供時間線索 |
| 混合式 | `20240315_001_description` | 日期前綴 + 每日序號 |

版本資訊必須存放在一個**專用的追蹤表**（schema history table）中：

```sql
CREATE TABLE schema_migrations (
    version VARCHAR(255) PRIMARY KEY,
    applied_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    execution_time_ms INTEGER,
    checksum VARCHAR(64),
    description TEXT,
    status VARCHAR(20) DEFAULT 'pending'
);
```

對於 SQLite 這類嵌入資料庫，可用 `PRAGMA user_version` 作為輕量版本記號。**重點：**絕對不能寫入 `PRAGMA schema_version`，這是 SQLite 內部的追蹤值，竄改會造成毀損[^sqlite_pragma]。

## 2. 遷移腳本類別

遷移腳本分為兩大類，這個分類是語言無關的[^types]：

**版本化遷移（Versioned）**——只執行一次，依序套用：
- 命名如 `V1__create_users.sql`、`002_add_index.sql`
- 負責表格、欄位、索引、約束條件的結構變更
- 一旦在某個版本套用過，就永遠不會再執行

**可重複遷移（Repeatable）**——內容變更時重新套用：
- 命名如 `R__refresh_view.sql`
- 用於檢視表、預存程序、觸發器、種子資料
- 透過 checksum 比對來決定是否需要重新套用

## 3. 前向遷移與回退（Up / Down）

每組遷移應該包含前向（Up）與回退（Down）兩個方向[^expand]：

```sql
-- Up: 新增欄位
ALTER TABLE products ADD COLUMN discount_percent NUMERIC(5,2) NULL;

-- Down: 移除欄位
ALTER TABLE products DROP COLUMN discount_percent;
```

**關鍵洞察：** 實際上 Down 遷移經常遺失資料（DROP 欄位會丟棄該欄的所有資料）。因此業界普遍推薦**前向修正策略（Roll Forward）**而非回退：遇到有問題的遷移時，撰寫一個新的修正遷移去修復問題，而不是 revert 回舊的 schema。

### Expand / Contract 模式（零停機遷移的黃金標準）

此模式分為四個階段，所有階段都透過部署新遷移來完成，不依賴回退[^expand]：

1. **Expand（擴展）**——新增 schema 元素（新欄位、新表格），不刪除任何舊元素
2. **Migrate（遷移）**——部署同時寫入新舊兩處的應用程式碼，並回填既有資料
3. **Verify（驗證）**——確認所有應用實例都在執行雙寫程式碼，且資料一致
4. **Contract（收縮）**——部署最終遷移除舊 schema 元素（通常在後續版本）

## 4. 冪等性（Idempotency）

遷移必須保證**無論執行一次或 N 次，最終狀態一致**[^idem]。這在 CI/CD 管線因網路問題、逾時或重啟而造成重複執行時至關重要。

兩種實現機制：

| 機制 | 說明 | 範例 |
|---|---|---|
| **條件守衛** | DDL 執行前檢查目標是否存在 | `CREATE TABLE IF NOT EXISTS`、`ALTER TABLE ... ADD COLUMN IF NOT EXISTS`（PostgreSQL） |
| **版本帳本** | 追蹤表記錄已套用的遷移，執行器自動跳過 | `INSERT INTO migration_ledger ... ON CONFLICT DO NOTHING` |

**為何需要雙層保護：** 如果遷移執行器在 DDL 成功提交之後、寫入帳本之前崩潰或逾時，重試時若缺乏條件守衛就會重複執行 DDL。帳本負責正常情況，守衛負責邊界情況。

## 5. 遷移執行流程

通用的執行流程在 C++ 中與其他語言完全一致[^flow]：

1. 應用程式啟動時讀取版本追蹤表（或 `PRAGMA user_version`）中的目前版本
2. 比對程式碼中的最新遷移檔案
3. 依序套用每個待處理的遷移（在同一個交易中）
4. **只在遷移成功提交後**，才更新版本追蹤
5. 若任一遷移失敗，交易回滾，版本不變，保證可重複執行

## 6. C++ 專案中常見的遷移實作方式

C++ 不像 Rails 或 Django 有深度整合的 ORM，因此通常採用以下策略之一[^cpp_ways]：

### 方式 A：內嵌 SQLite + 手動版本檢查（最常見）

- 應用程式內嵌 SQLite，遷移目錄隨二進位檔一起發布
- 啟動時檢查 `PRAGMA user_version`，與編譯時的預期版本比較
- 將待處理的 SQL 遷移在 `BEGIN IMMEDIATE ... COMMIT` 交易中依序執行
- 每筆遷移完成後，在同一個交易中更新 `PRAGMA user_version`
- 整個遷移引擎通常是客製的輕量 C++ class，負責讀取 SQL 檔、追蹤版本、遷移後執行完整性驗證

### 方式 B：獨立 CLI 工具

- 使用 `dbmate`、`Flyway` 等語言無關的工具在 CI/CD 管線中執行
- 遷移腳本是純 SQL 檔案，由工具追蹤
- 好處是無需自訂遷移引擎，缺點是增加工具依賴

### 方式 C：ORM 輔助遷移

- ODB 是 C++ 生態中支援 schema evolution 的主要 ORM
- 透過 `#pragma db model version(base, current)` 在 C++ 標頭中宣告模型版本
- ODB 會自動產生遷移前（pre-migration）與遷移後（post-migration）的 SQL 配對檔案
- 遷移前「放鬆」schema（新增可空欄位、移除約束），遷移後「收緊」schema（移除舊欄位、加入 NOT NULL）

## 7. 最佳實踐摘要

| 實踐 | 說明 |
|---|---|
| **啟動時執行遷移，而非建置時** | 二進位檔開啟資料庫時自檢並升級 |
| **每筆遷移使用獨立交易** | 確保原子性，版本更新必須在同行交易中 |
| **遷移後執行完整性檢查** | SQLite：`PRAGMA integrity_check` 與 `PRAGMA foreign_key_check` |
| **破壞性遷移前先備份** | 使用 SQLite 的線上備份 API 或冷備份 |
| **遷移必須是確定性的** | 遷移交易中禁止網路呼叫 |
| **schema 版本是相容性合約** | 舊二進位檔必須拒絕開啟比它理解的版本更新的資料庫 |

## 結論

資料庫遷移的模式是語言無關的：Schema 版本化、Up/Down 遷移、Expand/Contract、冪等性設計。C++ 專案的差異在於其更常採用「純 SQL 腳本 + 啟動時自動套用」的輕量方式，而非依賴 ORM 產生的遷移程式碼。

---

[^ver]: Schema versioning strategies — timestamp-based avoids merge conflicts; sequential numbering is simpler but collision-prone in teams. (n.d.). Database Migration 101. Retrieved 2026-09-25, from https://dev.to/rhuturaj_takle/database-migrations-managing-schema-changes-as-version-controlled-code-1o18

[^types]: Two migration categories: versioned (run once, sequential, structural) and repeatable (re-applied on content change, for views/functions/seed data). (n.d.). Idempotent Script Design. Retrieved 2026-09-25, from https://www.zero-downtime-schema.com/database-migration-fundamentals-tool-selection/idempotent-script-design/

[^expand]: Expand-Contract migration pattern: Expand (add new schema), Migrate (dual-write code), Verify (consistency check), Contract (remove old schema). (n.d.). Zero-Downtime Schema Migration Patterns. Retrieved 2026-09-25, from https://danielleackerman.github.io/vibe-coding-iceberg/patterns/05-schema-migration-patterns/

[^idem]: Two-layer idempotency: conditional DDL guards (IF NOT EXISTS) for crash-recovery edge case, plus versioned ledger for the normal execution path. See idempotency patterns at https://www.zero-downtime-schema.com/database-migration-fundamentals-tool-selection/idempotent-script-design/

[^sqlite_pragma]: SQLite defines `PRAGMA user_version` (safe for applications) as distinct from `PRAGMA schema_version` (internal, writing it causes corruption). See SQLite official documentation on pragma usage. (n.d.). Retrieved 2026-09-25, from https://stackoverflow.com/questions/989558/best-practices-for-in-app-database-migration-for-sqlite

[^flow]: Migration execution flow: read current version → compare against latest → apply pending migrations in order within transactions → update version after commit. (n.d.). Database Schema Migrations in C++. Retrieved 2026-09-25, from https://stackoverflow.com/questions/34321372/c-database-schema-using-migration

[^cpp_ways]: Common C++ migration approaches: (A) embedded SQLite with PRAGMA user_version and custom class, (B) standalone CLI tools like Flyway/dbmate, (C) ODB ORM with auto-generated pre/post migration SQL. (n.d.). Retrieved 2026-09-25, from https://stackoverflow.com/questions/13632093/handling-class-changes-when-using-orm-such-as-odb