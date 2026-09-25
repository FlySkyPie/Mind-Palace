# C++ 資料庫遷移函式庫調查

## 概述

本報告調查可用於 C++ 專案的資料庫結構遷移（database migration / schema migration）函式庫與工具。資料庫遷移是一種管理資料庫結構版本變更的技術，類似於版本控制系統對程式碼的管理[^mig101]。

> **注意：** Medium 文章《Database Migrations 101》[^medurl]無法存取（回傳 403），以下調查內容主要來自 GitHub 與官方文件。

---

## 方案比較

| 方案 | 類型 | 資料庫支援 | 遷移格式 | 回滾 | 狀態 |
|---|---|---|---|---|---|
| **TinyORM / tom** | C++ 函式庫 + CLI | MySQL, SQLite, PostgreSQL, MariaDB | C++ 類別（編譯式） | ✅ | 活躍，成熟 |
| **Vix.cpp DB** | C++ 函式庫 + CLI | SQLite, MySQL | SQL 檔案 + C++ 類別 | ✅ | 活躍，成長中 |
| **dbmate** | 獨立 CLI | PostgreSQL, MySQL, SQLite, ClickHouse 等 | SQL 檔案 | ✅ | 活躍（7.4k ⭐） |
| **Atlas** | 獨立 CLI | 多種資料庫 | HCL / SQL | ✅ | 活躍 |
| **QSqlMigrator** | Qt C++ 函式庫 | PostgreSQL, MySQL, SQLite | C++ 類別 | ✅ | **已封存** |
| **dodbm** | C++ 函式庫 | MySQL, MariaDB | C++ 程式碼 | ❓ | 不活躍 |
| **Nuclex.ThinOrm.Native** | C++ 函式庫 | 規劃中 | C++ 類別 | ✅ | 早期設計階段 |

---

## 1. TinyORM / tom

**GitHub：** https://github.com/silverqx/TinyORM[^tinyorm]
**文件：** https://www.tinyorm.org/database/migrations[^tinyormdocs]

現代 C++20 ORM 函式庫，內建**編譯式遷移系統**與專用 CLI 工具 `tom`，受 Laravel 的 Eloquent 啟發。這是目前最完整的 C++ 遷移解決方案。

**主要特點：**
- **編譯式遷移** — 遷移以 C++ 類別撰寫，編譯進 `tom` CLI 二進位檔（新增遷移後需重新編譯）
- **Schema Builder** — C++ API 用於建立/修改資料表、欄位、索引、外鍵（資料庫無關）
- **`tom` CLI 指令：** `make:migration`、`migrate`、`fresh`、`refresh`、`rollback`、`reset`、`status`、`install`、`uninstall`
- 支援資料填充（seeding）
- 支援 vcpkg 與 CMake FetchContent
- 3,378 個單元/功能測試

---

## 2. Vix.cpp DB

**GitHub：** https://github.com/vixcpp/db[^vixdb]
**文件：** https://docs.vixcpp.com/guides/database/migrations[^vixdocs]

Vix.cpp 生態系的資料庫抽象層，同時支援**檔案式 SQL 遷移**與**程式碼式 C++ 遷移類別**。

**主要特點：**
- 檔案式遷移：成對的 `.up.sql` / `.down.sql` 檔案，使用時間戳命名
- 程式碼式遷移：實作 `Migration` 介面，包含 `up()` 和 `down()` 方法
- CLI 指令：`vix db migrate`、`rollback`、`status`、`makemigrations`
- 遷移追蹤表 `schema_migrations`，記錄 `checksum` 與 `applied_at`
- 支援客製化遷移資料表名稱

---

## 3. dbmate

**GitHub：** https://github.com/amacneil/dbmate[^dbmate]

輕量級、**語言無關的 CLI 工具**，可與任何程式語言搭配使用（含 C++）。以 Go 撰寫，單一二進位檔發行。

**主要特點：**
- 純 SQL 遷移檔案，時間戳版本命名
- 支援 PostgreSQL、MySQL、MariaDB、SQLite、ClickHouse、BigQuery、Spanner
- 指令：`new`、`up`、`create`、`drop`、`migrate`、`rollback`、`down`、`status`、`dump`、`load`、`wait`
- 支援 `DATABASE_URL` 環境變數與 `.env` 檔案
- 遷移在交易內原子執行

---

## 4. Atlas

**網站：** https://atlasgo.io/[^atlas]
**GitHub：** https://github.com/ariga/atlas[^atlasgh]

現代、語言無關的結構遷移工具，可用於 C++ 專案。將資料庫結構以程式碼管理。

**主要特點：**
- **宣告式工作流程** — 定義目標結構（HCL / SQL），Atlas 自動生成遷移計畫
- **版本式工作流程** — 傳統的循序遷移
- 結構檢查（linting）與 CI/CD 整合
- 自動遷移計畫生成

---

## 5. QSqlMigrator（已封存）

**GitHub：** https://github.com/hicknhack-software/QSqlMigrator[^qsqlmigrator]

Qt 基礎的 C++ 資料庫遷移函式庫，受 Rails ActiveRecord 遷移啟發。已於 2025-05-11 封存，**不再維護**。

---

## 6. dodbm

**GitHub：** https://github.com/WopsS/dodbm[^dodbm]

小型 C++11 遷移函式庫，聚焦 MySQL/MariaDB。僅 2 顆星，長期未更新。

---

## 7. Nuclex.ThinOrm.Native

**GitHub：** https://github.com/nuclex-shared-cpp/Nuclex.ThinOrm.Native[^nuclex]

C++20 微 ORM，受 .NET 的 FluentMigrator 啟發。仍處於早期設計階段，0 顆星。

---

## 建議

- **需要完整的 C++ 整合方案：** **TinyORM** 搭配 `tom` CLI 是最成熟完善的選擇，提供編譯式遷移、Schema Builder、資料填充與 3,300+ 測試。
- **偏好輕量 C++ 方案與純 SQL：** **Vix.cpp DB** 同時支援檔案式 SQL 與程式碼式 C++ 遷移，API 簡潔。
- **語言無關 CLI 工具：** **dbmate** 或 **Atlas** 是優秀的獨立工具，可從建置腳本呼叫。
- **Qt 專案：** 原本的 **QSqlMigrator** 已封存，建議改用 **TinyORM** 或 **dbmate**。

---

[^mig101]: The TechDude. (n.d.). Database Migrations 101. Retrieved 2026-09-25, from https://medium.com/@TheTechDude/database-migrations-101-a24bdf18582d
[^medurl]: 同上，原文回傳 403 無法存取。
[^tinyorm]: silverqx. (n.d.). TinyORM. GitHub. Retrieved 2026-09-25, from https://github.com/silverqx/TinyORM
[^tinyormdocs]: TinyORM. (n.d.). Database Migrations. Retrieved 2026-09-25, from https://www.tinyorm.org/database/migrations
[^vixdb]: vixcpp. (n.d.). db. GitHub. Retrieved 2026-09-25, from https://github.com/vixcpp/db
[^vixdocs]: Vix.cpp. (n.d.). Database Migrations. Retrieved 2026-09-25, from https://docs.vixcpp.com/guides/database/migrations
[^dbmate]: amacneil. (n.d.). dbmate. GitHub. Retrieved 2026-09-25, from https://github.com/amacneil/dbmate
[^atlas]: Ariga. (n.d.). Atlas. Retrieved 2026-09-25, from https://atlasgo.io/
[^atlasgh]: ariga. (n.d.). atlas. GitHub. Retrieved 2026-09-25, from https://github.com/ariga/atlas
[^qsqlmigrator]: hicknhack-software. (n.d.). QSqlMigrator. GitHub. Retrieved 2026-09-25, from https://github.com/hicknhack-software/QSqlMigrator
[^dodbm]: WopsS. (n.d.). dodbm. GitHub. Retrieved 2026-09-25, from https://github.com/WopsS/dodbm
[^nuclex]: nuclex-shared-cpp. (n.d.). Nuclex.ThinOrm.Native. GitHub. Retrieved 2026-09-25, from https://github.com/nuclex-shared-cpp/Nuclex.ThinOrm.Native