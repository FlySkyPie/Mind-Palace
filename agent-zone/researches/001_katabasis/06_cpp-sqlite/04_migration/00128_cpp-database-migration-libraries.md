# C++ 資料庫 Schema Migration 函式庫調查報告

## 摘要

本報告調查 C++ 生態系統中可用於資料庫 Schema Migration（結構遷移／版本化資料庫更新）的函式庫。結果顯示 C++ 領域**不存在如同 Flyway、Liquibase、Alembic 那樣廣為人知的獨立 Migration 函式庫**。現有方案主要分為三類：ORM 內建的自動 Schema 同步、框架捆綁的 Migration 模組，以及語言無關的命令列工具。

## 背景

DataBase Schema Migration（資料庫遷移）是指以版本控制的方式管理資料庫結構變更（新增表格、修改欄位、建立索引等），讓開發團隊能像管理程式碼一樣管理資料庫演進的流程[^medium]。典型的 Migration 工具支援：

- 版本化的 `up()` 與 `down()` SQL 腳本
- 自動追蹤已套用的遷移
- 支援回滾（Rollback）

## 調查方法

使用 GitHub Search 與 Web Search 進行廣泛搜尋，關鍵字包括："database migration C++ library"、"C++ schema migration"、"cpp migrator"、"C++ Flyway alternative"、"C++ Liquibase" 等。逐一檢視每個項目的 GitHub Star 數、維護狀態、支援的資料庫類型以及 Migration 功能完整度。

## 調查結果

### 類別一：ORM 內建 Migration 功能

#### 1. Oat++ ORM（SchemaMigration）

| 項目 | 內容 |
|------|------|
| **GitHub** | [oatpp/oatpp](https://github.com/oatpp/oatpp)[^oatpp] |
| **Stars** | ~8,700 |
| **支援資料庫** | SQLite (oatpp-sqlite)、PostgreSQL (oatpp-postgresql)、MySQL/MariaDB (社群擴充) |
| **Migration 方式** | 版本編號 Migration Script：透過 `addText(version, script)` 或 `addFile(version, filename)` 註冊腳本，再呼叫 `migrate()`。套用記錄寫入 Schema 版本控制表。 |
| **回滾支援** | **無** — 需自行撰寫撤銷腳本 |
| **維護狀態** | 非常活躍，最新穩定版 1.3.0，1.4.0 開發中 |

Oat++ 是一個全功能的 C++ Web 框架，其 ORM 模組內建 SchemaMigration 功能，支援多種資料庫後端。但 Migration 是透過 C++ 程式碼執行，而非純 SQL 檔案管理[^oatpp-migration]。

#### 2. sqlite_orm（sync_schema）

| 項目 | 內容 |
|------|------|
| **GitHub** | [fnc12/sqlite_orm](https://github.com/fnc12/sqlite_orm)[^sqlite_orm] |
| **Stars** | ~2,700 |
| **支援資料庫** | SQLite 限定 |
| **Migration 方式** | `sync_schema()` — 自動比對 C++ 結構定義與實際資料庫 Schema，自動執行 `ALTER TABLE` 或重建表格。**非版本化 Migration**，而是宣告式 Schema 同步。 |
| **回滾支援** | **無** |
| **維護狀態** | 非常活躍，header-only，C++14/17/20 支援 |

易用性極高 — 在 C++ 中定義結構並呼叫一個函式即可自動同步。但官方已明確指出**不保證資料能夠保留**[^sqlite_orm-docs]，不適合正式環境的部署流程。

#### 3. ODB（Code Synthesis）

| 項目 | 內容 |
|------|------|
| **GitHub** | [codesynthesis-com/odb](https://github.com/codesynthesis-com/odb)[^odb] |
| **Stars** | ~63（主要存儲庫） |
| **支援資料庫** | SQLite、MySQL、PostgreSQL、Oracle、SQL Server |
| **Migration 方式** | `schema_catalog::migrate_schema()` — 基於版本編號的 Schema 演進機制，支援增量 DDL 變更。從 C++ Header 透過程式碼產生器自動產生 Schema。 |
| **回滾支援** | 有限（透過版本遞增/遞減） |
| **維護狀態** | 商業化支援，開源版本可用 |

ODB 是 C++ ORM 中最專業的 Schema 演進方案，具備 `schema_catalog` 和正式 Migration API。但需要 GCC Plugin 進行程式碼產生，且有商業授權選擇[^odb-docs]。

#### 4. hiberlite

| 項目 | 內容 |
|------|------|
| **GitHub** | [paulftw/hiberlite](https://github.com/paulftw/hiberlite)[^hiberlite] |
| **Stars** | ~722 |
| **支援資料庫** | SQLite 限定 |
| **Migration 方式** | 自動從 C++ 類別定義建立/修改表格 — 新增欄位到 struct 後，啟動時自動送出 `ALTER TABLE`。 |
| **維護狀態** | 低度活躍，最後更新為數年前 |

#### 5. ormpp

| 項目 | 內容 |
|------|------|
| **GitHub** | [qicosmos/ormpp](https://github.com/qicosmos/ormpp)[^ormpp] |
| **Stars** | ~1,500 |
| **支援資料庫** | MySQL、PostgreSQL、SQLite |
| **Migration 支援** | **無** — 僅提供 CRUD、鏈式查詢、連線池等功能 |
| **維護狀態** | 活躍，header-only |

### 類別二：無 Migration 功能的資料庫存取函式庫

此類函式庫提供資料庫連線與查詢能力，但**完全沒有 Migration 機制**，使用者需自行處理 Schema 管理。

| 函式庫 | Stars | 支援資料庫 | 說明 |
|--------|-------|-----------|------|
| **SQLiteCpp**[^sqlitecpp] | ~2,800 | SQLite | 現代 C++ RAII 包裝 SQLite C API |
| **sqlpp11**[^sqlpp11] | ~2,600 | MySQL、MariaDB、SQLite、PostgreSQL | 編譯期型別安全 SQL EDSL |
| **SOCI**[^soci] | ~1,600 | 7+ 資料庫 | 成熟的多後端資料庫存取層 |
| **libpqxx**[^libpqxx] | ~1,400 | PostgreSQL | PostgreSQL 官方 C++ 用戶端 API |

### 類別三：語言無關的 Migration 工具（可搭配 C++ 使用）

#### dbmate

| 項目 | 內容 |
|------|------|
| **GitHub** | [amacneil/dbmate](https://github.com/amacneil/dbmate)[^dbmate] |
| **Stars** | ~7,400 |
| **語言** | Go（編譯為單一二進位檔） |
| **支援資料庫** | MySQL、MariaDB、PostgreSQL、SQLite、ClickHouse、BigQuery、Spanner |
| **Migration 方式** | 純 SQL 檔案搭配 `-- migrate:up` / `-- migrate:down` 區段，時間戳版本化。支援 Rollback、Schema Dump、資料庫等待。 |
| **回滾支援** | **有** |
| **維護狀態** | 非常活躍（最新 v2.28.0） |

dbmate 是最完整的 Migration 解決方案，可從 C++ 專案的建置腳本或 CI/CD Pipeline 中呼叫。缺點是無法從 C++ 程式碼直接呼叫 Migration 函式[^dbmate-docs]。

### 類別四：專屬 C++ Migration 函式庫（低知名度）

#### dodbm

| 項目 | 內容 |
|------|------|
| **GitHub** | [wopss/dodbm](https://github.com/wopss/dodbm)[^dodbm] |
| **Stars** | ~2 |
| **支援資料庫** | MySQL、MariaDB |
| **Migration 方式** | 專為 Migration 設計的 C++ 函式庫，支援 Provider 架構 |
| **維護狀態** | 已停止維護（最後更新 2018 年） |

dodbm 是本次調查中 **唯一專注於 Database Migration 的 C++ 函式庫**，但僅有 2 顆星且已停止維護**。**

### 類別五：未找到的專案

以下名義的專案經查證**不存在於 GitHub**：
- **cpp-migrator** — 無此專案
- **libpqmigrate** — 無此專案
- **migratepp** — 無此專案
- **poro_migrate** (ngghh) — 使用者存在但無公開儲存庫[^poro_migrate]
- **redaxmedia/media-library** — 組織/使用者不存在[^media_library]
- **Vix.cpp** — 有文件網站但無公開 GitHub Star 數可參考

## 綜合比較表

| 函式庫 | Stars | 資料庫 | Migration | Up/Down | Rollback | 語言 | 狀態 |
|--------|-------|--------|-----------|---------|----------|------|------|
| **Oat++ ORM** | ~8.7k | SQLite, PG, MySQL | ✅ 版本編號腳本 | ✅ | ❌ | C++ | 活躍 |
| **dbmate** | ~7.4k | 7+ 資料庫 | ✅ 完整版本化 | ✅ | ✅ | Go (CLI) | 活躍 |
| **SQLiteCpp** | ~2.8k | SQLite | ❌ 無 | ❌ | ❌ | C++ | 活躍 |
| **sqlite_orm** | ~2.7k | SQLite | ⚠️ 自動同步 | ❌ | ❌ | C++ | 活躍 |
| **sqlpp11** | ~2.6k | 4 種資料庫 | ❌ 無 | ❌ | ❌ | C++ | 活躍 |
| **SOCI** | ~1.6k | 7+ 資料庫 | ❌ 無 | ❌ | ❌ | C++ | 活躍 |
| **ormpp** | ~1.5k | MySQL, PG, SQLite | ❌ 無 | ❌ | ❌ | C++ | 活躍 |
| **libpqxx** | ~1.4k | PostgreSQL | ❌ 無 | ❌ | ❌ | C++ | 活躍 |
| **hiberlite** | ~722 | SQLite | ⚠️ 自動產生 | ❌ | ❌ | C++ | 低度 |
| **ODB** | ~63 | 5 種資料庫 | ✅ schema_catalog | ✅ | 有限 | C++ | 商業化 |
| **dodbm** | ~2 | MySQL, MariaDB | ✅ 專屬 Migration | ✅ | ❌ | C++ | 已停 |

## 結論與建議

### 核心發現

C++ 生態系統中**缺乏一個廣受歡迎、獨立且功能完整的 Database Schema Migration 函式庫**。這與 Java（Flyway、Liquibase）、Python（Alembic）、Go（Goose、golang-migrate）、Rust（diesel migration）等語言形成明顯對比。

### 使用情境建議

| 使用情境 | 最佳選擇 | 理由 |
|----------|---------|------|
| 需要完整 up/down migration + rollback | **dbmate**（CLI 工具） | 功能最完整、Stars 數高、活躍維護、支援多種資料庫 |
| SQLite-only，想要自動 Schema 同步 | **sqlite_orm** | 一行程式碼即可同步 C++ 結構與資料庫 |
| 正在開發 C++ Web 服務（含 REST API） | **Oat++ ORM** | 內建 SchemaMigration、連線池、Swagger-UI |
| 需要多資料庫支援（不含 Migration） | **SOCI + dbmate** | SOCI 提供存取層，dbmate 負責 Migration |
| 需要純 C++ Migration API | **自建 + SQLiteCpp/SOCI** | 因無成熟獨立方案，建議自行封裝簡易 Migration 邏輯 |

### 最終建議

對於多數 C++ 專案，**dbmate** 是最務實的 Migration 解決方案：單一二進位檔、無依賴、支援 Rollback、活躍維護。若要純 C++ 方案，**Oat++ ORM 的 SchemaMigration** 是最接近的選擇，但缺乏 Rollback。SQLite-only 專案則可考慮 **sqlite_orm** 的自動同步功能，但需理解其資料保留不保證的限制。

## 參考資料

[^medium]: TheTechDude. (2024). *Database Migrations 101*. Retrieved 2026-09-25, from https://medium.com/@TheTechDude/database-migrations-101-a24bdf18582d

[^oatpp]: Oat++. (n.d.). *Oat++ GitHub Repository*. Retrieved 2026-09-25, from https://github.com/oatpp/oatpp

[^oatpp-migration]: Oat++. (n.d.). *SchemaMigration — Oat++ API Reference*. Retrieved 2026-09-25, from https://oatpp.io/api/latest/oatpp/orm/SchemaMigration/

[^sqlite_orm]: fnc12. (n.d.). *sqlite_orm GitHub Repository*. Retrieved 2026-09-25, from https://github.com/fnc12/sqlite_orm

[^sqlite_orm-docs]: fnc12. (n.d.). *sqlite_orm — sync_schema*. Retrieved 2026-09-25, from https://sqliteorm.com/

[^odb]: Code Synthesis. (n.d.). *ODB GitHub Repository*. Retrieved 2026-09-25, from https://github.com/codesynthesis-com/odb

[^odb-docs]: Code Synthesis. (n.d.). *ODB — Schema Evolution*. Retrieved 2026-09-25, from https://codesynthesis.com/products/odb/

[^hiberlite]: paulftw. (n.d.). *hiberlite GitHub Repository*. Retrieved 2026-09-25, from https://github.com/paulftw/hiberlite

[^ormpp]: qicosmos. (n.d.). *ormpp GitHub Repository*. Retrieved 2026-09-25, from https://github.com/qicosmos/ormpp

[^sqlitecpp]: SRombauts. (n.d.). *SQLiteCpp GitHub Repository*. Retrieved 2026-09-25, from https://github.com/SRombauts/SQLiteCpp

[^sqlpp11]: rbock. (n.d.). *sqlpp11 GitHub Repository*. Retrieved 2026-09-25, from https://github.com/rbock/sqlpp11

[^soci]: SOCI. (n.d.). *SOCI GitHub Repository*. Retrieved 2026-09-25, from https://github.com/SOCI/soci

[^libpqxx]: jtv. (n.d.). *libpqxx GitHub Repository*. Retrieved 2026-09-25, from https://github.com/jtv/libpqxx

[^dbmate]: amacneil. (n.d.). *dbmate GitHub Repository*. Retrieved 2026-09-25, from https://github.com/amacneil/dbmate

[^dbmate-docs]: amacneil. (n.d.). *dbmate — Usage*. Retrieved 2026-09-25, from https://github.com/amacneil/dbmate

[^dodbm]: wopss. (n.d.). *dodbm GitHub Repository*. Retrieved 2026-09-25, from https://github.com/wopss/dodbm

[^poro_migrate]: ngghh. (n.d.). *GitHub User Page*. Retrieved 2026-09-25, from https://github.com/ngghh

[^media_library]: redaxmedia. (n.d.). *GitHub 404*. Retrieved 2026-09-25, from https://github.com/redaxmedia/media-library