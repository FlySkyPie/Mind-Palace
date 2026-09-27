# DuckDB 與 SQLite 全面比較分析

## 引言

DuckDB 與 SQLite 皆為**嵌入式、無伺服器、單一檔案**的資料庫系統——資料庫引擎以函式庫形式嵌入宿主應用程式中執行，無需獨立伺服器行程，亦無網路延遲[^ddb-why][^sqlite-about]。然而二者服務的運算場景截然不同：SQLite 自 2000 年問世以來一直是**交易處理（OLTP）** 的黃金標準；DuckDB 則自 2019 年起專注於**分析處理（OLAP）**，提供向量化執行引擎與多核心平行處理能力[^ddb-why][^posthog]。

本報告從架構、儲存格式、查詢執行、並行模型、擴展生態系、效能基準與實務場景等面向進行系統性比較。

---

## 架構差異

| 維度 | SQLite | DuckDB |
|---|---|---|
| **目標工作負載** | OLTP（交易處理）—點查詢、小量插入/更新/刪除 | OLAP（分析處理）—彙總、JOIN、大量掃描 |
| **首次發布** | 2000 年 | 2019 年（荷蘭 CWI 研究中心） |
| **核心大小** | ~600 KB（C 語言 amalgamation） | ~5–20 MB（含擴充） |
| **授權** | Public Domain | MIT |
| **語言** | C | C++17 |
| **成熟度** | 25+ 年，數十億部署，航空級測試 | ~7 年，快速成長中 |
| **加密** | 無內建（需 SQLCipher 或付費擴充） | 內建 AES-256-GCM（v1.4.0 起）[^betterstack] |

兩者的本質差異來自儲存模型與執行引擎的取捨，而非嵌入式的定位。

---

## 儲存引擎與格式

### SQLite：B-Tree 列式儲存（Row Store）

- 資料以**列**為單位儲存，一列的所有欄位在磁碟上連續擺放
- 使用 B+Tree 結構，葉節點（leaf pages）包含完整列資料
- 預設頁大小 4 KB（可設 512–65536 bytes）[^sqlite-limits]
- **最大資料庫**：~281 TB（64 KB pages × 42.9 億頁）
- **最大列大小**：~1 GB
- **無壓縮**：原始位元組儲存，僅 VACUUM 可重整碎片
- **跨平臺性極佳**：32/64-bit 與 big/little-endian 間自由複製

分析查詢只需 3 個欄位時，仍須讀取整列所有欄位，造成巨量 I/O 浪費。

### DuckDB：行群分段欄位式儲存（Columnar Row-Group Store）

- 資料以**行群（row group）** 組織，預設每行群 122,880 列
- 每行群內，每個欄位獨立儲存為連續壓縮的**欄位段（column segment）**
- 每個欄位段包含**區域分佈圖（zone maps）**——記錄該段各欄位的最小/最大值，供述詞下推（predicate pushdown）跳過不相關段[^sysint-storage]
- 支援多種輕量壓縮：ALP（自適應無損浮點）、Chimp、Patas、字典編碼、RLE（行程編碼）、Bitpacking、Constant[^ddb-storage]
- **無實務限制**：已驗證 15 TB+ 檔案正常運作
- 直接讀寫 Parquet、CSV、JSON、Arrow 等多種格式，無需匯入[^ddb-why]

**關鍵意涵**：讀取 50 欄表格中 3 欄時，DuckDB 比 SQLite 少讀取約 94% 的資料。基準測試顯示，6M 列資料匯入後，DuckDB 僅佔 **17.6 MB**，SQLite 則需 **221.9 MB**——壓縮比達 **12.6 倍**[^markaicode]。

---

## 查詢執行模型

### SQLite：VDBE 位元組碼直譯器（逐列執行）

```
SQL → 剖析器 → VDBE 位元組碼 → 逐列解譯迴圈:
  列 1 → 列 2 → 列 3 → … → 列 N
```

- SQL 被編譯為自定義位元組碼，由 Virtual Database Engine（VDBE）逐條解譯
- 每個運算子（掃描、過濾、JOIN、彙總）對**每一列**執行一次函式呼叫
- 優化器為**基於規則（rule-based）**，保守且可預測——無成本模型、無自動 JOIN 重排[^sysint-compare]
- 單執行緒執行，無法利用多核心[^posthog]

### DuckDB：向量化執行引擎（批次處理）

```
SQL → 邏輯計畫 → 成本優化器 → 物理計畫 → 管線:
  向量 1（第 1–2048 列）→ 向量 2（第 2049–4096 列）→ …
```

- 每秒處理 **STANDARD_VECTOR_SIZE = 2048 筆值**的向量批次
- 成本優化器支援：JOIN 重排、述詞下推、子查詢展開（unnesting）
- 緊湊的型別陣列迴圈——無逐列函式呼叫開銷、CPU 快取友善、SIMD 友善[^ddb-why]
- **咀嚼式平行處理（morsel-driven parallelism）**：將行群子範圍（morsel）分配給各工作執行緒平行執行[^sysint-compare]
- 受 MonetDB/X100 論文啟發[^monetdb-paper]

---

## 並行模型

| 面向 | SQLite | DuckDB |
|---|---|---|
| **讀取並行** | 多讀取者（WAL 模式） | 多執行緒（同一行程內） |
| **寫入並行** | 單寫入者（WAL 模式仍僅 1 寫入者） | 單寫入者 |
| **單查詢平行化** | 無（單執行緒） | 多核心（morsel-driven） |
| **隔離等級** | Serializable（預設） | Serializable（Snapshot Isolation via MVCC） |
| **寫入吞量** | ~400K 列/秒（WAL 模式） | ~200K 列/秒 |
| **同時連線數** | 最多 1024（WAL 模式 62） | 單寫入 + 多唯讀行程 |
| **鎖定模型** | UNLOCKED / SHARED / RESERVED / PENDING / EXCLUSIVE | MVCC + 樂觀併發控制 |

SQLite 的 WAL（Write-Ahead Log）模式讓讀取者永不阻塞寫入者、寫入者永不阻塞讀取者，使用獨立的 `-wal` 檔案與 `-shm` 共享記憶體索引。DuckDB 採用 MVCC，基於 Neumann 等人的論文實作[^mvcc-paper]。

**取捨**：SQLite 對交易寫入較佳；DuckDB 對平行分析讀取較佳。

---

## 索引類型

| 功能 | SQLite | DuckDB |
|---|---|---|
| **主要索引** | B-tree（INTEGER PRIMARY KEY 自動建立） | ART（Adaptive Radix Tree） |
| **次要索引** | B-tree（CREATE INDEX） | ART |
| **區域分佈圖** | 無 | 內建（每欄位段自動維護 min/max） |
| **空間索引** | R-tree（擴充）[^sqlite-ext] | R-tree（Spatial 擴充） |
| **全文檢索** | FTS5（擴充） | FTS（擴充） |

SQLite 的 B-tree 針對**點查詢**高度最佳化（~0.01ms）。DuckDB 的 ART 索引針對主記憶體場景設計，單列查詢較慢（~0.1ms），但區域分佈圖讓它在掃描時常可免用索引[^sysint-compare]。

---

## 資料型別

### SQLite：鬆散型別（Type Affinity）

- 5 種儲存類別：NULL、INTEGER、REAL、TEXT、BLOB
- 型別**親和性（affinity）** 系統——欄位建議型別但不強制
- 可在 INTEGER 欄位存入 TEXT
- 無原生 boolean（用 INTEGER 0/1 代替）
- 無原生日期時間（用 TEXT/INTEGER/REAL 代替）[^sqlite-limits]

### DuckDB：豐富嚴格型別

- 完整數值：TINYINT、SMALLINT、INTEGER、BIGINT、HUGEINT、FLOAT、DOUBLE、DECIMAL
- 時間型別：DATE、TIME、TIMESTAMP、TIMESTAMPTZ、INTERVAL
- 巢狀型別：STRUCT、LIST、MAP、UNION
- 特化型別：UUID、JSON、ENUM、BITSTRING、POINT、GEOMETRY[^ddb-limits]
- 數百個內建分析函式（統計、字串、日期數學、視窗函式等）

---

## 擴充生態系

### SQLite 擴充

執行時期載入的共享函式庫（`.so` / `.dylib` / `.dll`），透過 `.load` 指令載入[^sqlite-ext]。知名擴充：

- **FTS5**：全文檢索引擎
- **JSON1**：JSON 函式（自 3.38.0 起內建）
- **R*Tree**：空間索引
- **ICU**：國際化元件
- **Math**：延伸數學函式
- **CSV**：CSV 虛擬表

無中心化簽署儲存庫——需自行編譯或尋找 `.so` 檔案。

### DuckDB 擴充

透過 SQL 指令 `INSTALL name; LOAD name;` 安裝**簽署版**擴充，DuckDB 在 LOAD 時驗證簽章防止篡改[^ddb-ext]。核心與社群擴充逾 60+，包括：

- **parquet**：Parquet 讀寫（核心）
- **httpfs**：HTTP/S3/GCS/Azure 檔案操作
- **postgres / mysql / sqlite**：外部資料庫直接查詢
- **iceberg / delta**：資料湖格式
- **spatial**：完整地理空間支援
- **vss**：向量相似度搜尋
- **excel**：Excel (.xlsx) 讀寫
- **ui**：本機 Web UI

社群擴充（第三方維護）含 BigQuery、ClickHouse、Prometheus、OpenAPI、圖形查詢（duckpgq）等，全部簽署驗證[^ddb-community-ext]。

---

## 外部資料與檔案格式支援

| 資料來源 | SQLite | DuckDB |
|---|---|---|
| **CSV** | 擴充（有限） | 原生，自動偵測 |
| **JSON** | 擴充函式 | 原生完整支援 |
| **Parquet** | 無 | 原生，含投影/濾波下推 |
| **Excel** | 無 | 擴充 (.xlsx) |
| **Arrow / Pandas** | 無（需手動匯入） | **零拷貝**直接查詢 |
| **S3 / HTTP** | 無 | httpfs 擴充 |
| **PostgreSQL / MySQL / SQLite** | 無 | 擴充直接查詢 |
| **Iceberg / Delta Lake** | 無 | 擴充支援 |

DuckDB 可直接查詢外部檔案與資料庫，無需 ETL[^ddb-why]。

---

## 效能基準

以下彙整多個獨立來源的實測數據。

### 基準一：百萬列電子商務資料集（DuckDB Lab，2026）

硬體：AMD EPYC 4 vCPU、8 GB RAM、NVMe SSD[^ddb-lab]

| 查詢類型 | DuckDB v1.5.2 | SQLite 3.45.1 | 加速倍率 |
|---|---|---|---|
| COUNT(*) | 0.004s | 0.035s | **8.7×** |
| SUM(price × quantity) | 0.005s | 0.408s | **81.6×** |
| GROUP BY category（6 類別） | 0.020s | 2.275s | **113.8×** |
| 日期範圍過濾 + 彙總 | 0.006s | 0.413s | **68.8×** |
| 多維度 GROUP BY（region × category） | 0.020s | 4.185s | **209.3×** |
| 視窗函式（月累計） | 0.094s | 1.555s | **16.5×** |
| TOP 10 產品 | 0.041s | 1.473s | **35.9×** |
| 地區平均訂單金額 | 0.010s | 1.687s | **168.7×** |
| HAVING 子句 | 0.056s | 2.323s | **41.5×** |
| 條件式彙總（CASE WHEN） | 0.043s | 1.303s | **30.3×** |

彙總查詢（SUM、GROUP BY）上 DuckDB 快 **80–200 倍**。多維度 GROUP BY 達 **209 倍**。

### 基準二：600 萬列 CSV（Markaicode，2026-09）

硬體：1 vCPU Linux、Intel Xeon 2.10 GHz、3.9 GB RAM、168 MB CSV[^markaicode]

| 階段 | DuckDB v1.5.5 | SQLite 3.51.1 | 比率 |
|---|---|---|---|
| 載入 CSV 至資料表 | 1.63s | 12.01s | **7.4×** DuckDB |
| 僅查詢（資料已載入） | 0.24s | 1.86s | **7.8×** DuckDB |
| 最快路徑 | 1.30s（直接查 CSV） | 13.87s（必先載入） | **10.7×** DuckDB |
| 磁碟大小（載入後） | **17.6 MB** | 221.9 MB | **12.6×** DuckDB 更小 |

### 基準三：3500 萬列 GTFS 運輸資料集（Lukas Barth）

硬體：2016 ThinkPad T460s、i5-6300U、20 GB RAM[^lukas-barth]

**有索引查詢 → SQLite 勝出**

| 查詢類型 | DuckDB | SQLite | 倍率 |
|---|---|---|---|
| 單列 PK 查詢 | 0.927ms | **0.063ms** | **14.7×** SQLite |
| 複合鍵查詢 | 9.68ms | **0.078ms** | **124.1×** SQLite |
| PK JOIN | 13.3ms | **0.118ms** | **112.7×** SQLite |

**無索引掃描 → DuckDB 勝出**

| 查詢類型 | DuckDB | SQLite | 倍率 |
|---|---|---|---|
| 非索引欄位過濾 | **7.7ms** | 1,822ms | **236×** DuckDB |
| 大範圍掃描 | **2.9ms** | 2,722ms | **938×** DuckDB |

此為兩者最極端的對比——DuckDB 在分析掃描上快 **938 倍**，SQLite 在索引查詢上快 **14–124 倍**。

### 基準四：1000 萬列銷售資料（Fastero，2026）

查詢：按月分地區統計 COUNT、SUM、AVG、GROUP BY、ORDER BY[^fastero]

| 引擎 | 時間 | 加速倍率 |
|---|---|---|
| SQLite | 14.2s | 基準 |
| DuckDB | **0.18s** | **~79×** |

100M 列時 SQLite 需 3 分鐘以上，DuckDB **不到 2 秒**。

---

## 限制

### SQLite 限制

| 限制 | 說明 |
|---|---|
| **分析效能** | 逐列執行使大規模掃描與彙總極慢 |
| **單執行緒查詢** | 無法平行化單一查詢 |
| **單寫入者**（即使 WAL 模式） | 同一時間僅一程序可寫入 |
| **無欄位修剪** | 即使只需少數欄位仍須讀取整列 |
| **有限 SQL 分析能力** | 無 GROUPING SETS、有限視窗函式、無 PIVOT |
| **有限外部資料** | 無 Parquet、無雲端儲存、無 DataFrame 直接查詢 |
| **弱型別** | 親和性系統易意外混入異質型別 |

### DuckDB 限制

| 限制 | 說明 |
|---|---|
| **單寫入者模型** | 僅一程序可寫入 `.duckdb` 檔案 |
| **無使用者/角色模型** | 無 GRANT/REVOKE、無列層級安全 |
| **不適合 OLTP** | 高頻小量插入與點更新效能不佳 |
| **單節點** | 無法水平擴展（MotherDuck 雲端服務除外） |
| **批次寫入導向** | 最佳化大量追加，非串流或高頻小交易 |
| **需設定記憶體** | 需明確設定 `memory_limit` 與 `temp_directory` |
| **溢出限制** | 部分彙總函式（`list()`、`string_agg()`）與 PIVOT 尚無法溢出至磁碟 |
| **較年輕生態系** | ~7 年 vs SQLite 25+ 年 |
| **無跨版本回溯相容** | 新版 DuckDB 寫入的檔案可能無法被舊版讀取 |

---

## 實務場景建議

### 使用 SQLite 時機

- 行動應用、桌面應用、IoT/嵌入式裝置的區域交易儲存
- 需要**快速點查詢**（使用者設定檔、工作階段、設定）
- **高頻小量寫入**與**並行寫入**需求
- 資源受限裝置（~600 KB 核心大小）
- 需要**極高可靠性**（航空級測試 25+ 年）
- 資料集 <10 GB 且主要為 OLTP 工作負載
- 需要**長期歸檔**（美國國會圖書館推薦格式）[^sqlite-about]

### 使用 DuckDB 時機

- **分析查詢**、彙總、報表
- 直接查詢 **Parquet/CSV/JSON 檔案**（尤其 S3/HTTP 上）
- **資料科學與探索分析**（Python、R、Jupyter）
- **ETL/ELT 管線**
- 處理**百萬至十億列**規模資料
- 需**向量化平行執行**充分利用多核心 CPU
- 需直接查詢 **外部資料庫**（PostgreSQL、MySQL、SQLite）無需遷移
- **嵌入式分析**於應用程式中

### 同時使用的黃金法則

兩者非競爭關係而是**互補**。DuckDB 可透過 `sqlite_scanner` 擴充直接查詢 SQLite 資料庫：

```sql
INSTALL sqlite;
LOAD sqlite;
SELECT department, AVG(salary)
FROM sqlite_scan('app.db', 'employees')
GROUP BY department;
```

典型架構：**SQLite 處理交易資料（OLTP）**，**DuckDB 執行分析報表（OLAP）**，無需遷移資料。

---

## 總結決策矩陣

| 需求 | 推薦引擎 | 原因 |
|---|---|---|
| 快速 PK 點查詢 | **SQLite** | B-tree 高度最佳化，快 DuckDB 15–124× |
| 複雜分析查詢 × 大資料集 | **DuckDB** | 向量化執行快 80–938× |
| 行動 / 嵌入式儲存 | **SQLite** | 600 KB、25 年可靠性 |
| 資料科學 / Python 分析 | **DuckDB** | 零拷貝 DataFrame、直接讀 Parquet |
| 多並行寫入者 | **兩者皆否** → PostgreSQL | 二者皆單寫入者模型 |
| 直接查詢 S3 Parquet | **DuckDB** | httpfs 擴充原生支援 |
| 最大交易耐久性與 ACID | **SQLite** | WAL 模式 + 25 年驗證 |
| 列層級安全與審計 | **兩者皆否** → PostgreSQL | 二者無使用者模型 |
| 超記憶體處理（out-of-core） | **DuckDB** | 支援溢出至磁碟 |
| CI/CD 即時 SQL 測試 | **DuckDB** | 秒級載入查詢、無需伺服器 |
| 長期歸檔格式 | **SQLite** | 正向相容 >20 年 |

---

[^ddb-why]: DuckDB. (n.d.). Why DuckDB. Retrieved 2026-09-25, from https://duckdb.org/why_duckdb
[^sqlite-about]: SQLite Consortium. (n.d.). About SQLite. Retrieved 2026-09-25, from https://www.sqlite.org/about.html
[^posthog]: PostHog. (n.d.). DuckDB vs SQLite: an in-depth architectural and performance comparison. Retrieved 2026-09-25, from https://posthog.com/blog/duckdb-vs-sqlite
[^sqlite-limits]: SQLite Consortium. (n.d.). SQLite Limits. Retrieved 2026-09-25, from https://www.sqlite.org/limits.html
[^sqlite-ext]: SQLite Consortium. (n.d.). Loadable Extensions. Retrieved 2026-09-25, from https://www.sqlite.org/loadext.html
[^ddb-limits]: DuckDB. (n.d.). Operations Manual — Limits. Retrieved 2026-09-25, from https://duckdb.org/docs/current/operations_manual/limits.html
[^ddb-ext]: DuckDB. (n.d.). Core Extensions Overview. Retrieved 2026-09-25, from https://duckdb.org/docs/current/core_extensions/overview.html
[^ddb-community-ext]: DuckDB. (n.d.). Community Extensions. Retrieved 2026-09-25, from https://duckdb.org/community_extensions/
[^ddb-storage]: SystemInternals. (n.d.). DuckDB Storage Engine. Retrieved 2026-09-25, from https://systeminternals.dev/duckdb/storage-format/
[^sysint-storage]: SystemInternals. (n.d.). DuckDB vs SQLite: Architecture, Storage, and Performance. Retrieved 2026-09-25, from https://systeminternals.dev/duckdb/vs-sqlite/
[^sysint-compare]: SystemInternals. (n.d.). DuckDB vs SQLite — Row Store vs Column Store, Bytecode vs Vectorized. Retrieved 2026-09-25, from https://systeminternals.dev/duckdb/vs-sqlite/
[^monetdb-paper]: Boncz, P., Zukowski, M., & Nes, N. (2005). MonetDB/X100: Hyper-Pipelining Query Execution. CIDR 2005. Retrieved 2026-09-25, from https://cidrdb.org/cidr2005/papers/P19.pdf
[^mvcc-paper]: Neumann, T., Mühlbauer, T., & Kemper, A. (2015). Fast Serializable Multi-Version Concurrency Control for Main-Memory Database Systems. SIGMOD 2015. Retrieved 2026-09-25, from https://db.in.tum.de/~muehlbau/papers/mvcc.pdf
[^betterstack]: Better Stack. (2026). DuckDB vs SQLite: Detailed Comparison with Performance Benchmarks. Retrieved 2026-09-25, from https://www.betterstack.com/community/guides/scaling-python/duckdb-vs-sqlite/
[^ddb-lab]: DuckDB Lab. (2026). DuckDB vs SQLite — 1M-row e-commerce benchmark. Retrieved 2026-09-25, from https://duckdblab.org/en/post/duckdb-vs-sqlite-benchmark/
[^markaicode]: Markaicode. (2026). DuckDB vs SQLite: Real Benchmarks for Analytics Workloads. Retrieved 2026-09-25, from https://markaicode.com/vs/duckdb-vs-sqlite/
[^lukas-barth]: Barth, L. (n.d.). SQLite vs DuckDB: Benchmarks on 35M-row GTFS dataset. Retrieved 2026-09-25, from https://www.lukas-barth.net/blog/sqlite-duckdb-benchmark/
[^fastero]: Fastero. (2026). DuckDB vs SQLite for Analytics: Benchmarks, Tradeoffs. Retrieved 2026-09-25, from https://fastero.com/blog/duckdb-vs-sqlite-for-analytics-2026
[^motherduck]: MotherDuck. (n.d.). DuckDB vs SQLite: Which Embedded Database Should You Use? Retrieved 2026-09-25, from https://motherduck.com/learn/duckdb-vs-sqlite-databases/
[^datacamp]: DataCamp. (n.d.). DuckDB vs SQLite: A Complete Database Comparison. Retrieved 2026-09-25, from https://www.datacamp.com/blog/duckdb-vs-sqlite-complete-database-comparison