# OLAP（Online Analytical Processing）與傳統關聯式資料庫（如 MySQL）之差異

## 引言

OLAP（Online Analytical Processing，線上分析處理）與 OLTP（Online Transaction Processing，線上交易處理）是兩種本質上截然不同的資料處理模式。OLAP 專為**分析型工作負載**設計——支援複雜查詢、聚合運算以及跨大量歷史資料集的多維度分析；而 OLTP（如 MySQL、PostgreSQL 等傳統關聯式資料庫）則專為**交易型工作負載**設計——處理大量即時的資料新增、更新與刪除操作[^wikipedia]。

「OLAP」一詞由關聯式資料庫之父 Edgar F. Codd 於 1993 年刻意作為「OLTP」的對應而創立[^wikipedia]。

---

## 1. 資料模型：星狀綱要 vs 正規化

| 面向 | OLAP（分析型） | OLTP（交易型，如 MySQL） |
|---|---|---|
| **綱要設計** | **星狀綱要（Star Schema）** 或**雪花綱要（Snowflake Schema）**——中心為**事實表**（存放度量值與外部鍵），周圍環繞**維度表**（存放描述性屬性） | **高度正規化（3NF+）**——資料拆分至多個關聯表，消除重複 |
| **正規化程度** | **反正規化**——刻意保留重複以換取查詢速度 | **完全正規化**——確保 ACID 合規，消除更新異常 |
| **結構** | 多維「OLAP Cube」，度量值位於維度（時間、地區、產品等）的交集 | 列與行的關聯式表格，透過主鍵/外部鍵建立關聯 |

關聯式 OLAP（ROLAP）的 Cube 中繼資料通常由星狀綱要或雪花綱要產出，度量值來自事實表的記錄，維度則來自維度表[^wikipedia]。

---

## 2. 查詢模式：聚合運算 vs CRUD

| 面向 | OLAP | OLTP |
|---|---|---|
| **查詢類型** | **複雜、讀取密集的分析查詢**——SUM、COUNT、AVG 聚合、GROUP BY 跨越數百萬筆資料、多維度鑽取/彙總/切片/切塊 | **簡易 CRUD 操作**——INSERT、UPDATE、DELETE、SELECT 單筆或少數記錄 |
| **操作方式** | Roll-up（向上彙總）、Drill-down（向下鑽取）、Slicing（切片過濾維度）、Dicing（從不同角度檢視） | 原子讀寫交易，確保 ACID 特性 |
| **並發程度** | 使用者較少，但每個查詢耗費大量資源 | 數千名使用者同時操作，每筆操作小而快 |

OLAP 三種核心操作定義：**Consolidation（彙總）**——沿維度向上聚合；**Drill-down（鑽取）**——深入更細節的層級；**Slicing & Dicing（切片與切塊）**——取出特定子集並從不同視角檢視[^wikipedia]。

> OLAP 系統查詢涉及大量記錄的複雜分析，而 OLTP 系統的理想用途是對資料庫進行簡易的更新、插入與刪除，查詢通常僅涉及一筆或少數記錄[^ibm]。

---

## 3. 儲存架構：欄位導向 vs 列導向

| 面向 | OLAP | OLTP |
|---|---|---|
| **儲存格式** | **欄位導向（Column-oriented）**——同一欄位的數百萬筆值連續儲存在一起 | **列導向（Row-oriented）**——同一筆記錄的所有欄位值儲存在一起 |
| **原因** | 分析查詢通常掃描**少量欄位但跨越大量資料列**——欄位儲存只需讀取需要的欄位，實現大幅壓縮與更快的掃描 | 交易查詢通常讀寫**單筆記錄的所有欄位**——列儲存讓這成為單一次連續讀取 |

### 3.1 為何欄位儲存對分析更快

**I/O 縮減：** 在一個 50 欄位的表格中，查詢僅選取 3 個欄位時，欄位儲存讀取約 24 MB，而列儲存讀取約 400 MB——壓縮前的 I/O 差距即達 16 倍以上。若考慮壓縮（5–10 倍），實際讀取的位元組可縮減至列儲存的 1% 以下[^clickhouse]。

**資料跳過（Data Skipping）：** 每個欄位區塊儲存預先計算好的中繼資料——最小值/最大值（Zone Map）、Bloom Filter、集合索引——查詢規劃器可在**解壓縮前就跳過整個區塊**。按 `(tenant_id, timestamp)` 排序的時間序列資料，查詢近期活動時可跳過 99% 以上的區塊[^chistadata]。

**向量化執行：** 欄位引擎一次處理 1024–4096 筆值，而非逐筆處理，讓 CPU 快取保持緊湊的工作集，編譯器可發出 SIMD 指令（SSE、AVX2、AVX-512）在單一 CPU 週期內處理 8–16 筆值[^clickhouse]。

**延遲實體化（Late Materialization）：** 欄位組裝延至過濾與聚合**之後**才進行——先讀取謂詞欄位、評估過濾條件，再僅對**倖存的資料列 ID**擷取 SELECT 欄位。1:1000 的選擇性過濾下，下游欄位工作量減少約 1000 倍[^clickhouse]。

### 3.2 壓縮技術

| 技術 | 原理 | 適用場景 |
|---|---|---|
| **字典編碼** | 將重複值（國家名、產品類別）以緊湊整數 token 取代 | 低基數欄位 |
| **RLE（Run-Length Encoding）** | 將連續相同值以 `(值, 次數)` 表示 | 排序或低基數欄位 |
| **Delta Encoding** | 儲存連續值的差值而非絕對值 | 時間戳、單調遞增 ID |
| **Gorilla（時間序列）** | delta-of-delta 編碼 + 可變位元 XOR 壓縮 | 浮點數時間序列指標 |

欄位儲存相鄰資料型別一致（皆為整數、字串等），壓縮比可達 **5–10 倍（典型），低基數欄位最高 30 倍**；列儲存因型別混雜，壓縮比僅 1.5–3 倍[^airbyte]。

---

## 4. 效能特性

| 指標 | OLAP | OLTP |
|---|---|---|
| **回應時間** | 秒到分鐘（TB–PB 級資料集的複雜查詢） | 毫秒或以下（簡易操作） |
| **資料量** | **TB 到 PB**——來自多個來源的歷史彙總資料 | **GB 到低 TB**——僅當前的營運資料 |
| **更新頻率** | **週期性批次載入**（每日/每週/每月，透過 ETL） | **持續即時更新**——每筆交易立即反映 |
| **讀寫比例** | **高度讀取優化**——讀取遠多於寫入 | **讀寫平衡**——兩者皆頻繁 |

> 對於複雜查詢，OLAP Cube 可在 OLTP 關聯式資料所需時間的約 **0.1%** 內產出答案[^wikipedia]。

---

## 5. 寫入效能的權衡

| 面向 | 列儲存（OLTP） | 欄位儲存（OLAP） |
|---|---|---|
| **單筆插入** | O(log n) 經 B-tree——極快 | 昂貴——必須附加到每個欄位檔案 |
| **單筆更新** | O(log n)——重寫一個頁面 | 非常昂貴——可能需重寫整個欄位區塊 |
| **大量插入** | 每秒 1K–10K 筆（索引 + WAL 開銷） | **每秒數百萬筆**——附加排序後的欄位區塊，背景合併 |
| **並發寫入** | 每秒數千筆交易/節點 | 較差——寫入鎖在欄位區塊層級 |

欄位資料庫透過以下方式最佳化寫入：將寫入批次化為大區塊後再刷新；使用不可變區塊搭配背景合併（LSM-tree 風格，如 ClickHouse 的 MergeTree）；以及 Delta Store——近期寫入的熱資料以列導向緩衝，定期轉換為欄位格式[^clickhouse][^airbyte]。

---

## 6. 應用場景

| OLAP（分析型） | OLTP（交易型） |
|---|---|
| 商業智慧與報表 | 電子商務訂單處理 |
| 財務預測與預算編制 | 銀行/ATM 交易 |
| 銷售趨勢分析 | 航空/飯店訂位系統 |
| 客戶行為分析（如 Netflix 推薦、Spotify 個人化） | 庫存管理 |
| 供應鏈最佳化 | 客戶帳戶管理 |
| 高階儀表板與 KPI 追蹤 | POS 系統 |

OLAP 的典型應用包括銷售、行銷、管理報表、業務流程管理、預算與預測、財務報表等領域[^wikipedia]。

---

## 7. 代表性技術

| 類別 | OLAP 系統 | OLTP 系統 |
|---|---|---|
| **開源** | ClickHouse, Apache Druid, Apache Pinot, DuckDB, MonetDB | MySQL, MariaDB, PostgreSQL, SQLite |
| **商用/雲端** | Amazon Redshift, Google BigQuery, Snowflake, Microsoft Analysis Services | Oracle DB, Microsoft SQL Server, IBM Db2, Amazon RDS |
| **查詢語言** | SQL 搭配 OLAP 延伸（CUBE, ROLLUP, 視窗函數）、MDX | 標準 SQL |

---

## 8. 優缺權衡總結

**OLAP 優勢：**
- 對巨量資料集的複雜聚合查詢極快
- 多維度分析能力（鑽取、彙總、切片）
- 優異的壓縮比（5–30 倍）
- 預先計算的聚合值可比 OLTP 快 1000 倍

**OLAP 劣勢：**
- 資料新鮮度有延遲（批次載入）
- 儲存與運算成本高
- ETL 管線複雜
- 不適合交易型工作負載
- 預先聚合可能導致資料爆炸

**OLTP 優勢：**
- 即時資料準確性
- ACID 合規確保資料完整性
- 處理高並發
- 易於部署與查詢
- 生態工具成熟

**OLTP 劣勢：**
- 對大規模資料的分析查詢效能極差
- 正規化綱要需要昂貴的 JOIN
- 列導向儲存對聚合運算效率低
- 歷史資料保留有限
- 大量分析查詢可能拖垮系統

---

## 結論

> **兩者並非誰比較快，而是取決於查詢類型。** 對主鍵點查詢而言，列儲存比欄位儲存快 10–100 倍；對寬表格聚合運算而言，欄位儲存比列儲存快 10–1000 倍[^clickhouse]。

在實務上，多數組織**同時使用兩者**：OLTP 處理日常營運，OLAP 則分析這些營運產生的資料，從而支援決策。近年出現的 HTAP（Hybrid Transactional/Analytical Processing）架構試圖整合兩者，代表系統包括 TiDB、SingleStore 等[^wikipedia]。

---

## 參考文獻

[^wikipedia]: Wikipedia. (2024). *Online analytical processing*. Retrieved 2026-09-25, from https://en.wikipedia.org/wiki/Online_analytical_processing
[^aws]: Amazon Web Services. (n.d.). *OLTP vs OLAP: Difference Between Data Processing Systems*. Retrieved 2026-09-25, from https://aws.amazon.com/tw/compare/the-difference-between-olap-and-oltp/
[^ibm]: IBM. (n.d.). *OLAP vs. OLTP: What's the Difference?*. Retrieved 2026-09-25, from https://www.ibm.com/think/topics/olap-vs-oltp
[^clickhouse]: ClickHouse. (n.d.). *Why Columnar Databases Are Fast*. Retrieved 2026-09-25, from https://clickhouse.com/resources/engineering/why-columnar-databases-are-fast
[^chistadata]: ChistaDATA. (n.d.). *Compression Techniques in Column-Oriented Databases*. Retrieved 2026-09-25, from https://chistadata.com/compression-techniques-column-oriented-databases/
[^airbyte]: Airbyte. (n.d.). *Columnar Storage: Everything You Need to Know*. Retrieved 2026-09-25, from https://airbyte.com/data-engineering-resources/columnar-storage