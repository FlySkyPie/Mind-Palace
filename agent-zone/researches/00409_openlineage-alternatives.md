# OpenLineage 替代方案與競爭對手調查

## 概要

OpenLineage 是 LF AI & Data 基金會下的一套開放標準，用於跨工具收集與傳遞資料沿革（data lineage）中繼資料。本報告調查 OpenLineage 的開源替代方案、競爭專案以及互補工具，涵蓋純線索蒐集伺服器、完整中繼資料平台、以及特定領域的線索分析工具。

本文件假設讀者對 OpenLineage 已有基礎認識，旨在提供「若我不使用 OpenLineage，還有哪些選擇」的技術評估。對於每個替代方案，均說明其定位、關鍵功能、與 OpenLineage 的比較、以及授權類型。

## 目錄

1.  純線索儲存與視覺化
2.  完整中繼資料平台
3.  特定領域工具
4.  總結比較

## 第一部分：純線索儲存與視覺化

### 1. Marquez（OpenLineage 參考實作）

- **定位：** OpenLineage 標準的參考實作伺服器。由 WeWork 開發，現為社群維護。
- **關鍵功能：**
  - 原生接收 OpenLineage 事件（無需轉譯層）
  - 支援欄位級別（column-level）線索（當整合端提供時）
  - 執行生命週期追蹤（start / complete / fail）
  - REST API 與網頁 UI 視覺化線索圖
  - PostgreSQL 儲存後端，Docker 二容器即可部署
- **與 OpenLineage 的關係：** Marquez 是 OpenLineage 標準的**實作端**。OpenLineage 定義格式，Marquez 儲存、查詢、視覺化。如果生態相容，這是最直接的替代選擇。
- **授權：** Apache 2.0 [^marquez]

### 2. SQLLineage

- **定位：** 輕量級 Python 函式庫，純從 SQL 文字解析出來源/目標表格與欄位依賴，**不需連接資料庫**。
- **關鍵功能：**
  - 表格級別與欄位級別線索（純 SQL 解析）
  - 支援多種 SQL 方言（sqlfluff、sqlparse 等可插拔解析器）
  - CLI 與 Python API
  - 圖形視覺化
- **與 OpenLineage 比較：** SQLLineage 是**函式庫/工具**，用於離線分析 SQL 文字；OpenLineage 是**標準**，用於即時跨工具管道觀測。兩者互補而非直接競爭。
- **授權：** Apache 2.0 [^sqllineage]

## 第二部分：完整中繼資料平台

### 3. DataHub（LinkedIn）

- **定位：** 功能完整的中繼資料平台，涵蓋資料發現、治理、資料品質監控、商業詞彙與政策執行。LF AI & Data 專案，GitHub 11,800+ stars。
- **關鍵功能：**
  - 欄位級別線索（來自 dbt、Airflow、Spark、Snowflake 等）
  - GraphQL API 用於自訂線索應用
  - Elasticsearch 全文搜尋
  - RBAC / ABAC 政策引擎
  - 連接資料集、儀表板、管道、ML 資產的中繼資料圖
- **與 OpenLineage 比較：** DataHub 是**完整中繼資料平台**，OpenLineage 是線索**標準**。DataHub 可消費 OpenLineage 事件，但自有更廣的中繼資料模型。部署重量級（需 Kafka、Elasticsearch、MySQL、多服務）。
- **授權：** Apache 2.0 [^datahub]

### 4. OpenMetadata

- **定位：** 統一資料目錄平台，整合資料發現、剖析、治理、可觀測性、資料品質與線索追蹤。核心團隊來自 Uber Databook。
- **關鍵功能：**
  - 自動化表格級別與欄位級別線索
  - 40+ 連接器（資料庫、BI 工具、調度引擎、ML 服務）
  - 手動拖放線索編輯
  - 資料品質與剖析
  - 基於 W3C PROV-O 本體
  - Elasticsearch 搜尋
- **與 OpenLineage 比較：** OpenMetadata 是**更廣泛的平台**，線索是功能之一；OpenLineage 是**專注的標準**。OpenMetadata 可消費 OpenLineage 事件。適合想要一站式中繼資料解決方案的團隊。
- **授權：** Apache 2.0 [^openmetadata]

### 5. Apache Atlas

- **定位：** Hadoop 生態系的中繼資料管理與治理框架，提供資料分類、線索追蹤與安全政策執行。
- **關鍵功能：**
  - 深度整合 Hive、HBase、Sqoop、Storm、Kafka
  - 自訂型別系統用於領域特定中繼資料
  - PII/PCI/PHI 分類自動沿線索傳播
  - Ranger 整合細粒度存取控制
  - Hook 式自動擷取中繼資料
- **與 OpenLineage 比較：** Atlas 是治理優先的平台，深度綁定 Hadoop 生態；OpenLineage 是供應商中立的標準。Atlas 學習曲線陡峭，對現代雲端架構不合適。
- **授權：** Apache 2.0 [^atlas]

### 6. Egeria

- **定位：** 開放中繼資料與治理框架（LF AI & Data），定義 600+ 型別的開放中繼資料標準，實現供應商中立的中繼資料交換。
- **關鍵功能：**
  - 開放中繼資料標準用於互操作性
  - Hub-and-spoke 治理架構
  - 使用 OpenLineage 進行動態線索擷取
  - Kafka 事件驅動線索攝取
  - 資料目錄、線索映射、治理政策
- **與 OpenLineage 比較：** Egeria 與 OpenLineage 是**姊妹專案**，Egeria 使用 OpenLineage 做為其線索擷取機制。Egeria 涵蓋治理與中繼資料標準，OpenLineage 是線索規格。相輔相成而非直接競爭。
- **授權：** Apache 2.0 [^egeria]

### 7. TrueDat（Bluetab / IBM）

- **定位：** 開源資料治理與目錄平台（被 IBM 收購），包含資料目錄、線索視覺化、品質規則管理、商業詞彙與 RBAC。
- **關鍵功能：**
  - 視覺化線索與時間點可視性
  - 變更影響分析
  - CSV 線索圖匯出
  - 可設定的治理工作流程
  - 資料品質規則管理
- **與 OpenLineage 比較：** TrueDat 是治理平台，線索為功能之一；OpenLineage 是線索標準。部署較重，適合 IBM 導向的企業。
- **授權：** Apache 2.0 [^truedat]

## 第三部分：特定領域工具

### 8. Tokern

- **定位：** 從雲端資料倉儲查詢歷史（Snowflake、Redshift、PostgreSQL、BigQuery、Athena）中自動提取欄位級別線索的專業工具。
- **關鍵功能：**
  - 零設定欄位級別線索（從查詢歷史）
  - PII/PHI 偵測（PIICatcher 使用 regex + NLP）
  - 互動式圖形視覺化（NetworkX + Kedro-Viz）
  - Python API 與 SDK
- **與 OpenLineage 比較：** Tokern 範圍狹窄，從查詢日誌逆向提取線索，不需儀器化管道；OpenLineage 是即時跨工具線索標準。Tokern 對 Snowflake/Redshift 使用者極佳，但限於特定平台。
- **授權：** BSL 1.1（社群版可用）[^tokern]

### 9. Spline（AbsaOSS）

- **定位：** 從 Apache Spark 工作中自動擷取線索的專業工具，在 Spark 執行時建立轉換視覺化。
- **關鍵功能：**
  - 自動 Spark 線索擷取（無需手動文件）
  - 轉換追蹤與依賴分析
  - 線索視覺化
  - Spark 工作執行可視性
- **與 OpenLineage 比較：** Spline 是 Spark 專用工具；OpenLineage 是跨工具標準。Spline 適合重度 Spark 團隊。
- **授權：** Apache 2.0 [^spline]

### 10. Pachyderm

- **定位：** 開源框架，提供版控且自動化的端到端資料管道。Git 式資料版控搭配不可變更線索追蹤，主要用於 ML 與資料科學。
- **關鍵功能：**
  - 資料版控（如 Git for data）
  - 不可變更稽核軌跡
  - 語言/框架無關管道定義
  - 自動 DAG 線索視覺化
  - 資料管道 CI/CD
  - 任意時間點管道重現
- **與 OpenLineage 比較：** Pachyderm 是管道引擎 + 資料版控系統，線索是版控的副產品。提供更強保證（不可變性、可重現性），但範圍集中於 ML/資料科學。
- **授權：** Apache 2.0 [^pachyderm]

### 11. Dagster

- **定位：** 資料資產開發、生產與可觀測的調度平台，內建線索/可觀測性功能。
- **關鍵功能：**
  - 軟體定義資產（追蹤資料輸入/輸出）
  - Dagster+ 資產線索視覺化
  - OpenLineage 整合
  - 豐富中繼資料/可觀測性
  - 16,000+ GitHub stars
- **與 OpenLineage 比較：** Dagster 是管道調度器，可發出 OpenLineage 事件。使用 Dagster 即獲得線索作為調度的一部分；OpenLineage 提供跨工具線索。
- **授權：** Apache 2.0 [^dagster]

### 12. Apache Hamilton

- **定位：** 用於定義資料流（DAG）的微框架，將線索、追蹤與中繼資料編碼為一級關注點。
- **關鍵功能：**
  - 在資料流定義中編碼線索/追蹤
  - 在 Python 可執行處皆可運作
  - 文件化與視覺化資料依賴
  - 2,600+ GitHub stars
- **與 OpenLineage 比較：** Hamilton 在**程式碼層級**編碼線索（什麼轉換什麼）；OpenLineage 在**執行層級**捕捉線索（實際運行了什麼、何時、用什麼資料）。兩者解決互補的問題。
- **授權：** Apache 2.0 [^hamilton]

## 總結比較

| 工具 | 類型 | 範疇 | 欄位級別 | 生態深度 | 操作複雜度 |
|------|------|------|:-------:|:--------:|:---------:|
| OpenLineage | 標準 | 線索收集 | ✅ | 現代資料棧 | 低（僅規格） |
| Marquez | 伺服器 | 線索儲存與視覺化 | ✅ | OpenLineage 生態 | 低（2 容器） |
| DataHub | 平台 | 中繼資料 + 線索 + 治理 | ✅ | 廣泛（40+ 整合） | 高 |
| Apache Atlas | 平台 | Hadoop 治理 + 線索 | ✅ | Hadoop 生態 | 高 |
| OpenMetadata | 平台 | 目錄 + 治理 + 線索 | ✅ | 廣泛（40+ 連接器） | 中 |
| Egeria | 框架 | 中繼資料標準 + 治理 | ✅ | 企業/治理 | 高 |
| Tokern | 工具 | 查詢日誌欄位級線索 | ✅ | Snowflake/Redshift/BigQuery | 低 |
| SQLLineage | 函式庫 | SQL 分析 | ✅ | SQL 方言 | 極低 |
| Pachyderm | 平台 | ML 管道版控 + 線索 | ❌ 表格 | ML/資料科學 | 中 |
| TrueDat | 平台 | 資料治理 + 線索 | ✅ | IBM/企業生態 | 中 |
| Spline | 工具 | Spark 專用擷取 | ✅ | Apache Spark | 低 |
| Dagster | 調度器 | 管道可觀測 + 線索 | ✅ | 現代資料棧 | 中 |
| Apache Hamilton | 框架 | 程式碼層級線索 | ✅ | Python 生態 | 低 |

## 快速選擇指南

- **純粹輕量線索收集 + 視覺化** → Marquez（OpenLineage 本身的參考實作）
- **需要完整中繼資料平台** → DataHub 或 OpenMetadata
- **Hadoop / 企業環境** → Apache Atlas
- **Snowflake / Redshift 查詢歷史欄位級線索** → Tokern
- **不需儀器化的 SQL 線索分析** → SQLLineage
- **ML 管道版控 + 可重現性** → Pachyderm
- **跨系統中繼資料標準 / 互通性** → Egeria

[^marquez]: Marquez Project. (n.d.). Marquez: OpenLineage Reference Implementation. Retrieved 2026-10-03, from https://marquezproject.ai/
[^sqllineage]: reata. (n.d.). SQLLineage: SQL Lineage Analysis. Retrieved 2026-10-03, from https://github.com/reata/sqllineage
[^datahub]: DataHub Project. (n.d.). DataHub: A Metadata Platform for the Modern Data Stack. Retrieved 2026-10-03, from https://github.com/datahub-project/datahub
[^openmetadata]: OpenMetadata. (n.d.). OpenMetadata: Unified Data Catalog. Retrieved 2026-10-03, from https://github.com/open-metadata/OpenMetadata
[^atlas]: Apache Software Foundation. (n.d.). Apache Atlas: Metadata Management for Hadoop. Retrieved 2026-10-03, from https://github.com/apache/atlas
[^egeria]: ODPi. (n.d.). Egeria: Open Metadata and Governance Framework. Retrieved 2026-10-03, from https://github.com/odpi/egeria
[^truedat]: Bluetab. (n.d.). TrueDat: Data Governance Platform. Retrieved 2026-10-03, from https://github.com/Bluetab
[^tokern]: Borneo. (n.d.). Tokern: Column-Level Lineage from Query History. Retrieved 2026-10-03, from https://pypi.org/project/data-lineage/
[^spline]: AbsaOSS. (n.d.). Spline: Spark Lineage. Retrieved 2026-10-03, from https://github.com/AbsaOSS/spline
[^pachyderm]: Pachyderm Project. (n.d.). Pachyderm: Data Versioning and Lineage for ML. Retrieved 2026-10-03, from https://github.com/pachyderm/pachyderm
[^dagster]: Dagster Labs. (n.d.). Dagster: Orchestration for Data Assets. Retrieved 2026-10-03, from https://github.com/dagster-io/dagster
[^hamilton]: Apache Software Foundation. (n.d.). Apache Hamilton: Dataflow Microframework. Retrieved 2026-10-03, from https://github.com/apache/hamilton