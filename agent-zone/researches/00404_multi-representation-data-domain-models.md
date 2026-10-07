# 同一資料多型態多處存在問題：Data Engineering / ETL 領域之領域模型與解決方案調查

## 問題陳述

在實務企業環境中，同一份資料或檔案經常以多種型態（ODS、CSV、JSON、Parquet）、副本、變體的方式存在於複數個地點或系統（SQL DB、NoSQL DB、物件儲存、資料倉儲、資料湖泊）。此現象帶來治理、一致性、可追溯性與變更管理上的根本挑戰。

本報告調查 Data Project 與 ETL 領域中對此問題進行建模、描述與解決的 Domain Model 與 Solution Pattern。

---

## 1. 正規多模型資料管理（Multi-Model Data Management）

**多模型資料庫（Multi-model Database）** 是設計用來在同一資料庫管理系統中支援多種資料模型的 DBMS（關聯式、文件、圖形、鍵值、寬欄位），與「多語言持久化」（Polyglot Persistence）需要拼湊多個不同資料庫產品不同。[^multimodel-db]

近年學術界提出以 **範疇論（Category Theory）** 統一的數學語言來建模與轉換多模型資料。Svoboda、Čontoš 與 Holubová（2021-2022）提出了 **Schema Category** 與 **Instance Category**，能在關聯式、鍵值、文件、寬欄位、圖形五種模型之間進行形式化變換，透過 Wrapper 橋接原生格式與範疇表示。[^category-theory]

此路線提供了一個 **統一形式化基礎**，讓同一份邏輯資料在其多種物理表示之間存在可證的轉換關係。

---

## 2. 多語言持久化（Polyglot Persistence）

由 Scott Leberknight 提出、Martin Fowler（2011）推廣的 **Polyglot Persistence** 描述了在單一應用中使用多種資料庫技術的實踐，每一種資料庫因其處理特定資料型態與存取模式的優勢而被選用。[^polyglot]

其相關的 Domain Pattern 目錄（Software Patterns Lexicon 收錄 30+ 種模式）包括：[^polyglot-patterns]

| 模式 | 說明 |
|---|---|
| **Best Fit Storage** | 依使用案例選擇最適當的資料庫 |
| **API Layer Abstraction** | 以穩定合約隱藏儲存後端差異 |
| **CQRS** | 讀取／寫入使用不同的模型／儲存 |
| **Data Federation** | 將多個來源合成為單一虛擬資料庫 |
| **Event-Driven Data Propagation** | 以領域事件同步不同儲存 |
| **Cross-Store Joins** | 跨異質資料庫進行 Join |
| **Database per Service** | 每個微服務擁有自己的資料庫 |
| **Sagas for Distributed Transactions** | 協調跨儲存交易 |
| **Data Synchronization** | 確保跨儲存一致性 |
| **Materialized Views Across Data Stores** | 跨來源預先計算的檢視 |

這些模式從架構層面為「同一資料在不同儲存中以不同形式存在」提供了可重複運用的解決方案。

---

## 3. 邏輯資料模型（Logical Data Model）作為抽象層

傳統的 **三層綱要架構**（ANSI/X3/SPARC）將資料模型分為：[^three-schema] [^ldm-vs-pdm]

| 層級 | 目的 | 抽象程度 |
|---|---|---|
| **概念模型（Conceptual）** | 商業實體、業務詞彙、高階關係 | 最抽象 |
| **邏輯模型（Logical）** | 與平台無關的結構、屬性、關係、約束 | 中層 |
| **物理模型（Physical）** | 特定 DBMS 的實作（資料表、欄位、索引） | 最具體 |

**邏輯資料模型（LDM）** 的關鍵在於它以平台無關的方式定義資料——不依賴於底層是 SQL（關聯式）、NoSQL（文件、圖形、鍵值）還是檔案（Parquet、CSV、JSON）。這使得 LDM 成為跨格式、跨系統統一的**抽象合約**，實務上從「資料治理」的角度回答「這些不同格式的檔案本質上是同一份資料嗎？」這個核心問題。

現代湖倉架構（Lakehouse）中的 **Bronze/Silver/Gold 分層** 可以視為此模型在巨量資料場景下的具象化：Bronze 是原始物理攝入、Silver 是統一的邏輯層（商業鍵、正規化關聯）、Gold 是概念層（商業實體、已備消費）。[^medallion]

---

## 4. 資料虛擬化與聯邦式資料庫（Data Virtualization / Federated Database）

**聯邦式資料庫系統（FDBS）** 建立一個統一的虛擬資料庫，即時查詢多個自主、異質的來源資料庫而不移動／複製資料。[^federated]

其 **五層綱要架構** 提供了解決此問題的正式抽象框架：[^federated-five]

| 層級 | 說明 |
|---|---|
| **Local Schema** | 各來源資料庫自身的綱要 |
| **Component Schema** | 轉換成通用資料模型後的綱要 |
| **Export Schema** | 各來源授權對外曝光的子集 |
| **Federated Schema** | 整合後的統一綱要 |
| **External Schema** | 給特定消費者使用的檢視 |

**Data Federation** vs **Data Consolidation** 的根本取捨：[^federation-vs-consolidation]

- **Data Federation（虛擬化）**：不移動資料，即時查詢，無重複儲存，但面臨跨源 Join 複雜查詢的效能瓶頸
- **Data Consolidation（實體彙整）**：透過 ETL 將資料物理移動至單一儲存（資料倉儲／湖泊），獲得效能與治理優勢，但產生儲存重複與延遲

實務上多數企業**同時採用兩者**——Data Virtualization 層用於探索性查詢，Data Warehouse 用於治理的 BI 報表。

---

## 5. 資料網格（Data Mesh）

Zhamak Dehghani 提出的 **Data Mesh** 是一種去中心化架構，其中領域團隊將其資料作為產品來擁有與管理。四大核心原則：[^data-mesh]

1. **領域所有權（Domain Ownership）**——團隊對其資料的完整生命週期負責
2. **資料作為產品（Data as a Product）**——資料被發佈、版本化、以產品級嚴謹度管理
3. **自助式基礎設施（Self-Serve Infrastructure）**——領域團隊建立可互操作的資料產品
4. **聯邦式計算治理（Federated Computational Governance）**——政策由集體定義，一致地執行

相比之下，**Data Fabric**（Gartner，2019）是基於元資料驅動的集中式自動化層，解決的是**技術碎片化**問題。兩者互補而非競爭。[^mesh-vs-fabric]

在 Data Mesh 中，一個商業實體可能在不同領域以 CSV 匯出、JSON API、SQL Table、NoSQL Document 等形式存在，但**資料合約（Data Contract）**——形式化、版本化、機器可讀的產出者—消費者的協議——提供了語意映射與轉換規則，使其可被發現與信任。[^data-contracts]

---

## 6. 資料合約（Data Contract）與資料產品（Data Product）

**資料合約** 是資料產出者與消費者之間的正式協議，包含：[^data-contracts-practice]

- **綱要定義**：欄位名稱、型別、可空性
- **語意定義**：商業定義與計算邏輯
- **SLA**：資料新鮮度、延遲要求
- **資料品質規則**：驗證條件
- **擁有權與變更管理**：負責人、版本、變更通知

成熟度層級：[^data-contracts-maturity]

| 層級 | 描述 |
|---|---|
| 1 | 無記錄 |
| 2 | 有記錄但未強制 |
| 3 | 強制執行，自動化驗證 |
| 4 | 整合至目錄 |
| 5 | 持續最佳化 |

Data Contract 回答的不是「資料在哪裡？」而是「你承諾這個資料集是什麼？」，為多格式存在的資料提供了**信任基礎**。

---

## 7. 資料版本化與分支（Data Versioning / Branching）

如同一份原始碼在不同分支中以不同版本共存，資料版本化工具讓同一份資料集在不同時間點、不同實驗脈絡中以不同版本存在。工具光譜：[^data-versioning]

| 類別 | 代表性工具 | 粒度 |
|---|---|---|
| Git-native 檔案 | DVC、Git LFS | 檔案層級 |
| 物件儲存層 | lakeFS、Pachyderm | 物件層級（branch/commit/merge） |
| Table format + Catalog | Nessie on Iceberg/Delta | 快照層級 |
| In-database 完整 Git | Dolt、XetHub | 資料列／單元格層級 |

DVC 在 Git 中儲存指標、在雲端儲存實際資料，搭配 Pipeline DAG 追蹤，主要用於 ML 可再現性。[^dvc]

lakeFS 在物件儲存（S3 相容）上提供 Git 般的分支、提交、合併、回滾等零複製元資料操作，支援 ETL 測試中的隔離 Dev/Test Branch 與 Write-Audit-Publish 模式。[^lakefs]

Nessie 為 Iceberg/Delta Lake 提供目錄層級的 Git 式分支，跨 Table 進行 branch/merge。[^nessie]

---

## 8. Event Sourcing 與 CQRS

Martin Fowler 於 2005/2011 年分別提出的 **Event Sourcing** 與 **CQRS**，從架構層面直接模擬了「同一資料以多種形式存在」這個命題。

**CQRS（Command Query Responsibility Segregation）**[^cqrs]：
- **Command Model（寫入端）**：處理更新、驗證、商業邏輯，產生事件
- **Query Model（讀取端）**：為展示／查詢最佳化，可使用完全不同的資料儲存、綱要或格式
- 這在架構上**明確接受**同一份底層資料同時存在寫入最佳化模型與一個或多個讀取最佳化模型（materialized views、報表資料庫、快取）

**Event Sourcing**[^eventsourcing]：
- 所有狀態變更儲存為事件序列
- 可捨棄應用狀態並透過重播事件來重建
- 可查詢任一時點的狀態（= 多時間線 = 分支）
- 可修正過去事件並重新計算狀態

二者結合提供了嚴謹的基礎：Command 產生事件（唯一事實源），**多個 Read Model**（不同表示）訂閱事件，每個 Read Model 可針對不同使用案例最佳化（SQL 報表、NoSQL 即時儀表板、全文搜尋索引）。

直接對應企業 ETL 場景：事件記錄 = ODS / Data Lake 原始資料，各 Read Model = CSV 匯出、JSON API、SQL Data Warehouse、Parquet for ML。

---

## 9. Data Vault 2.0

Dan Linstedt（2013）提出的 **Data Vault 2.0** 是一種專門為「混亂、不一致、多來源」資料設計的形式化建模方法論，目前已納入 NoSQL、非結構化、半結構化資料整合。[^datavault]

核心結構：

| 元件 | 功能 | 應對問題 |
|---|---|---|
| **Hub** | 儲存唯一商業鍵，作為跨來源的整合點 | 不同系統對同一實體不同識別方式 |
| **Link** | 捕捉 Hub 之間的關係 | 不同格式間的關聯線索 |
| **Satellite** | 儲存描述性屬性，append-only 歷史 | 同一實體在不同系統中有不同屬性集合 |
| **Raw Vault** | 來源驅動的整合層，保留顆粒度與可稽核歷史 | 保留原始格式痕跡 |
| **Business Vault** | 衍生層，套用商業規則 | 為消費端提供統一的邏輯視角 |

其設計特點：新增來源**不需要重構現有架構**（Non-Invasive），結構性資訊與描述性資訊分離，使對變更具有韌性（Resilience）。每一列承載 `Record Source` 與 `Load Date` 屬性，實現完整稽核性。

Data Vault 的核心哲學正是承認「同一資料必然以多種型態從多處來源進入」，並從建模層面追容這個事實而非強迫統一。

---

## 10. 資料目錄（Data Catalog）作為跨系統發現機制

前述各種模式解決了「為什麼有多種型式」和「如何管理」的問題，而 **Data Catalog** 解決「有哪些型式、分佈在哪裡」的發現問題。

主要的開源 Data Catalog 比較：[^data-catalog-comparison]

| 功能 | DataHub | OpenMetadata | Apache Atlas |
|---|---|---|---|
| 起源 | LinkedIn | 社群 | Apache Hadoop 生態 |
| 行(lineage)追蹤 | 欄位層級（Airflow、dbt） | 欄位層級 | 實體＋欄位層級 |
| 治理 | 細粒度 ACL、PII 標記、GDPR | Domain-based、DQ tests | 標籤分類 |
| 現狀（2026） | 活躍（Acryl 支援） | 非常活躍 | 穩定 |

Data Catalog 做為「跨系統資料的單一事實參照點」（Single Source of Truth for Metadata），讓團隊能回答「這份客戶資料在哪些系統中、以什麼格式存在、誰擁有它」。

---

## 11. 形式化資料溯源模型（Data Lineage / Provenance）

兩個主要標準：

**W3C PROV**（PROV-DM、PROV-O）[^prov]：語意豐富的本體論，定義三種核心類型（Entity、Activity、Agent）及關係（wasDerivedFrom、wasGeneratedBy、wasAttributedTo）。擅長：
- 語意豐富性與法規查核準備
- 跨領域互操作性（作為溯源領域的通用語言）
- 以 SPARQL 作為知識圖譜查詢

**OpenLineage**[^openlineage]：Linux Foundation 標準（Marquez 專案），提供具體、可運行的規範，從 Apache Spark、Airflow、dbt 等工具自動收集行(lineage)。關注：
- 操作性的即時行(lineage)追蹤
- 預先建置的整合與最低侵入性
- 根源分析（Root-Cause Analysis）
- 欄位層級行(lineage)

二者的差異在於 PROV 側重語意模型與知識表示，OpenLineage 側重操作互通性——PROV 回答「X 源自 Y，因為活動 Z」，OpenLineage 回答「這份 Parquet 檔案由哪個 Airflow DAG 的哪一步產生」。

---

## 12. 多表示資料處理架構（Multi-Representation Data Processing）

學術界對「同一邏輯資料同時以多種表示存在」的正式研究：

- **Multi-representation based data processing architecture for IoT**（IEEE，2017）——資料同時以列、欄、圖形三種方式儲存，以支援多樣化的查詢工作負載。[^multirep-iot]
- **Multiple Representation Modeling for Spatial Data**（Springer）——對同一真實世界資料提供多種表示的概念資料模型，支援不同視角下的一致管理。[^multirep-spatial]
- **Multi-Model Data Modeling and Representation**（ACM，2021）——運用範疇論處理多格式多來源資料的建模技術調查。[^multimodel-acm]

這些研究提供了理論基礎：多表示（Multi-Representation）不僅是工程權衡，而可以被正式建模為一個有轉換規則的系統。

---

## 綜合比較

| 解決方案 | 側重面向 | 核心機制 | 適合場景 |
|---|---|---|---|
| 多模型資料庫 | 統一儲存 | 單一引擎支援多模型 | 新專案，需多模型查詢 |
| Polyglot Persistence | 模式目錄 | 最佳工具解決最佳問題 | 微服務、已有異質系統 |
| 邏輯資料模型 | 抽象層 | 平台無關的定義 | 治理、跨系統統一定義 |
| Data Federation | 虛擬整合 | 即時查詢包裝層 | 不移動資料的即時整合 |
| Data Mesh | 組織治理 | 領域擁有、資料產品化 | 大型組織去中心化治理 |
| Data Contract | 產消者約定 | 版本化、機器可讀協議 | 跨團隊資料分享 |
| Data Vault 2.0 | 建模方法 | Hub-Link-Satellite | 多來源資料倉儲 |
| CQRS/Event Sourcing | 架構模式 | 寫入事件、多讀取模型 | 同一資料需多種表示之系統設計 |
| 資料版本化 | 變更管理 | 分支、提交、合併 | ML 實驗、ETL 開發測試 |
| Data Catalog | 發現與治理 | 元資料集中管理 | 跨系統盤點與發現 |
| 溯源模型（PROV/OpenLineage） | 可追溯性 | 產生活動關係的記錄 | 法規合規、故障根本原因分析 |

---

## 結論

業界與學界已發展出一系列互補的 Domain Model 與 Solution Pattern 來應對「同一資料以多種型態、副本、變體存在於複數個地點或系統」此一根本性問題。從形式化的範疇論多模型轉換（Category Theory）、架構級的 CQRS/Event Sourcing、治理級的 Data Mesh/Contract、建模級的 Data Vault 2.0，到抽象層的邏輯資料模型與聯邦式五層綱要，再到操作層的資料版本化與 Data Catalog——這些解決方案並非互斥，而是從不同抽象層次與視角切入：

- **形式層**（Category Theory）提供可證的轉換關係
- **架構層**（CQRS、Federation）提供系統設計模式
- **治理層**（Data Mesh、Data Contract）提供組織管理框架
- **建模層**（Data Vault、三層綱要）提供資料結構方法論
- **操作層**（Versioning、Catalog、Lineage）提供日常工具

實務企業部署時通常需要**多層次的組合策略**，而非單一萬能方案。

---

## References

[^multimodel-db]: Wikipedia. Multi-model database. Retrieved 2026-10-03, from https://en.wikipedia.org/wiki/Multi-model_database

[^category-theory]: Svoboda, M., Čontoš, P., Holubová, M. (2022). A unified representation and transformation of multi-model data using category theory. *Springer*. Retrieved 2026-10-03, from https://link.springer.com/article/10.1186/s40237-022-00613-3

[^polyglot]: Fowler, M. (2011). Polyglot Persistence. Retrieved 2026-10-03, from https://martinfowler.com/bliki/PolyglotPersistence.html

[^polyglot-patterns]: Software Patterns Lexicon. Polyglot Persistence Patterns. Retrieved 2026-10-03, from https://softwarepatternslexicon.com/data-modeling/polyglot-persistence-patterns/

[^three-schema]: ThoughtSpot. Conceptual vs Logical vs Physical Data Models. Retrieved 2026-10-03, from https://www.thoughtspot.com/data-trends/data-modeling/conceptual-vs-logical-vs-physical-data-models

[^ldm-vs-pdm]: AWS. The difference between logical and physical data model. Retrieved 2026-10-03, from https://aws.amazon.com/compare/the-difference-between-logical-and-physical-data-model/

[^medallion]: TDWI. Data Architecture Patterns. Retrieved 2026-10-03, from https://tdwi.org/blogs/data-101/2026/05/data-architecture-patterns.aspx

[^federated]: Wikipedia. Federated database system. Retrieved 2026-10-03, from https://en.wikipedia.org/wiki/Federated_database_system

[^federated-five]: Wikipedia. Federated database system § Five level schema architecture for FDBSs. Retrieved 2026-10-03, from https://en.wikipedia.org/wiki/Federated_database_system#Five_level_schema_architecture_for_FDBSs

[^federation-vs-consolidation]: Precisely. Data Consolidation vs Data Federation. Retrieved 2026-10-03, from https://www.precisely.com/blog/big-data/data-consolidation-vs-data-federation/

[^data-mesh]: DataBricks. Data Mesh vs Data Fabric. Retrieved 2026-10-03, from https://www.databricks.com/blog/data-mesh-vs-data-fabric

[^mesh-vs-fabric]: Promethium. Data Fabric vs Data Mesh Architecture Comparison. Retrieved 2026-10-03, from https://www.promethium.ai/guides/data-fabric-vs-data-mesh-architecture-comparison-2026/

[^data-contracts]: The Data Governor. Data Contracts. Retrieved 2026-10-03, from https://thedatagovernor.com/data-contracts/

[^data-contracts-practice]: Medium / Reliable Data Engineering. Data Contracts in Practice: What 50 Production Implementations Actually Look Like. Retrieved 2026-10-03, from https://medium.com/@reliabledataengineering/data-contracts-in-practice-what-50-production-implementations-actually-look-like-f1c953336bf2

[^data-contracts-maturity]: DataCamp. Data Contracts. Retrieved 2026-10-03, from https://datacamp.com/blog/data-contracts

[^data-versioning]: MatrixOrigin. Git4Data Landscape. Retrieved 2026-10-03, from https://www.matrixorigin.io/blog/git4data-part4-landscape

[^dvc]: Wikipedia. Data Version Control (software). Retrieved 2026-10-03, from https://en.wikipedia.org/wiki/Data_Version_Control_(software)

[^lakefs]: lakeFS. Version Data. Retrieved 2026-10-03, from https://docs.lakefs.io/guides/version-data/

[^nessie]: Project Nessie. Git-like branching for data lake catalogs. Retrieved 2026-10-03, from https://projectnessie.org/

[^cqrs]: Fowler, M. (2011). CQRS. Retrieved 2026-10-03, from https://martinfowler.com/bliki/CQRS.html

[^eventsourcing]: Fowler, M. (2005). Event Sourcing. Retrieved 2026-10-03, from https://martinfowler.com/eaaDev/EventSourcing.html

[^datavault]: Wikipedia. Data Vault Modeling. Retrieved 2026-10-03, from https://en.wikipedia.org/wiki/Data_Vault_Modeling

[^data-catalog-comparison]: PiStack (2026). Self-hosted Data Mesh Platforms: OpenMetadata, Atlas, DataHub Guide. Retrieved 2026-10-03, from https://www.pistack.xyz/posts/2026-05-17-self-hosted-data-mesh-platforms-openmetadata-atlas-datahub-guide/

[^prov]: W3C. PROV-DM: The PROV Data Model. Retrieved 2026-10-03, from https://www.w3.org/TR/prov-dm/

[^openlineage]: Linux Foundation. OpenLineage. Retrieved 2026-10-03, from https://openlineage.io/

[^multirep-iot]: IEEE (2017). Multi-representation based data processing architecture for IoT. Retrieved 2026-10-03, from https://ieeexplore.ieee.org/document/7980175

[^multirep-spatial]: Springer. Multiple Representation Modeling for Spatial Data. Retrieved 2026-10-03, from https://link.springer.com/rwe/10.1007/978-1-4614-8265-9_237

[^multimodel-acm]: ACM (2021). Multi-Model Data Modeling and Representation. Retrieved 2026-10-03, from https://dl.acm.org/doi/10.1145/3472163.3472267