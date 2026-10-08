# 資料孤島匯集至 Data Lake / Data Warehouse 時的 AuthN/AuthZ 保護方案

## 問題概述

在 Data Project 或 ETL 領域中，將分散在不同資料孤島（data silos）的資料匯集至集中式 Data Lake 或 Data Warehouse 時，各孤島原有的認證（Authentication, AuthN）與授權（Authorization, AuthZ）規則往往無法直接被帶到目標平台。這造成：

- 資料從來源移動到目標後，誰可以看什麼、誰不能看什麼的規則被破壞
- 必須在目標平台重新實作各來源的存取控制，但各孤島可能使用完全不同的授權模型（RBAC、ABAC、自訂 ACL 等）
- 跨來源的資料合併（join）可能揭露比任一來源單獨揭露更多的敏感資訊

以下整理目前業界的主要工具、平台與解決方案。

---

## 工具與解決方案

### 1. 中央化存取控制平台（Commercial）

這類平台專為此問題設計，提供 **單一控制面（single control plane）** 來管理跨平台的授權策略。

| 工具 | 核心機制 | 支援平台 |
|------|---------|---------|
| **Immuta** | 通用 Data Lakehouse 存取控制，策略定義一次，跨 Snowflake、Databricks、Trino、Redshift Spectrum 等強制執行 | Snowflake、Databricks、Starburst、Trino、Redshift Spectrum、BigQuery、Azure Synapse |
| **Commvault Data Access Manager (原 Satori Cyber)** | Proxy-based 強制 + 原生 policy pushdown 到 Snowflake Horizon、Databricks Unity Catalog；支援動態遮罩、RLS、CLS | Snowflake、Databricks、Redshift、BigQuery |
| **Privacera** | Apache Ranger 的商業延伸，使用 **PolicySync** 機制將中央策略翻譯為各目標平台（Snowflake、Databricks 等）的原生存取控制建構 | Snowflake、Databricks、Starburst、Dremio、AWS、Azure、GCP |
| **OvalEdge** | 將治理嵌入 ETL 流程本身，支援 170+ 預建連接器，在擷取、轉換、載入階段即施予存取控制 | 各種異質來源系統 |

### 2. 開源授權框架

| 工具 | 描述 | 授權模型 |
|------|------|---------|
| **Apache Ranger** | Hadoop/Data Lake 生態系的中央化授權框架，透過 **Plugin 模型** 在每個資料服務（Hive、Spark、HBase 等）內部執行策略決策 | RBAC、ABAC、Tag-Based (TBAC) |
| **Open Policy Agent (OPA)** | 通用策略引擎，使用 Rego 宣告語言定義存取規則，可嵌入 API Gateway、ETL Job、查詢代理 | 屬性導向（ABAC），任意自訂規則 |
| **PACE (Policy As Code Engine)** | Kotlin 開源專案，將資料策略程式化建立並套用到 Snowflake、Databricks、BigQuery | Policy-as-Code |

### 3. 元資料治理框架

| 工具 | 角色 | 整合 |
|------|------|------|
| **Apache Atlas** | 元資料管理與治理框架，提供資料分類（classification）層，不直接執行授權，而是驅動 Ranger 的 Tag-Based 策略 | 分類 → Apache Ranger 執行 |
| **OpenMetadata** | 混合 RBAC + ABAC 模型，對元資料資產本身進行存取控制 | RBAC + ABAC |

### 4. 雲端原生平台

#### AWS Lake Formation

提供 **LF-Tag ABAC（Tag-Based Attribute-Based Access Control）**，是 AWS 上解決此問題的核心工具：

- **標籤繼承**：`classification=restricted` 等標籤從 Database → Table → Column 自動繼承
- **資料列安全（Row-Level Security）**：對查詢透明注入 SQL `WHERE` 條件
- **資料行安全（Column-Level Security）**：透過 `ColumnWildcard.ExcludedColumnNames` 排除特定欄位
- **跨帳戶共用**：結合 AWS RAM（Resource Access Manager）支援聯邦查詢
- **Terraform 支援**：`aws_lakeformation_permissions` resource 可將治理流程版本控制化

#### Databricks Unity Catalog

提供多層存取控制：

- **ABAC 策略**：透過 Governed Tags 動態控制的標籤驅動授權
- **資料列過濾策略（Row Filter Policies）**：基於使用者身份/群組的 SQL UDF 過濾
- **資料行遮罩策略（Column Mask Policies）**：條件式遮罩（例如 SSN 只對 HR 部門顯示）
- **資料分類**：自動掃描偵測 PII 等敏感資料

#### Snowflake Tag-Based ABAC

- 建立標籤（如 `pii`、`ai_governance`）並指派遮罩/列過濾策略
- **標籤繼承**：設定在 Database/Schema 層級的標籤自動套用至所有 Table
- **標籤傳播**：當資料移動或下游物件依賴已標記物件時，標籤（及其策略）自動跟隨
- 可整合外部身份提供者（Microsoft Entra ID、Okta）

#### Microsoft Fabric (OneLake)

- 角色基礎系統決定 OneLake 資料存取
- **資料行層級安全（CLS）**：透過 `GRANT` 限制特定欄位
- **資料列層級安全（RLS）**：使用 Mapping Table、Inline TVF 與 `CREATE SECURITY POLICY`
- **動態資料遮罩（DDM）**：內建遮罩函數

### 5. Data Mesh / 資料合約架構

**Data Mesh** 的第四原則——**聯合計算治理（Federated Computational Governance）**——直接處理此問題：

- 各領域團隊定義資料產品的存取規則，編碼為策略
- 在 API Gateway / Event Broker 層自動執行
- 使用 **資料合約（Data Contracts）** 將 Schema、品質規則、SLA、存取策略捆綁為一個框架

**LakeLogic** 專案提供了資料合約在 Microsoft Fabric 與 Snowflake 上的參考實作，包含 Bronze/Silver/Gold 管線、列隔離、PII 遮罩與 lineage 追蹤。

### 6. Policy-as-Code 方法

多個工具趨向「策略即程式碼」的範式：

| 工具 | 策略語言 | 執行點 |
|------|---------|-------|
| Open Policy Agent (OPA) | Rego | API Gateway、ETL Job、資料存取 Proxy |
| Immuta | CLI/YAML | 中央控制面 |
| AWS Lake Formation | LF-Tags + Terraform | 原生在 Glue/Athena/Redshift |
| PACE | Kotlin DSL | 原生在 Snowflake/Databricks/BigQuery |
| dbt + Snowflake | dbt Macros | 模型具體化後自動套用策略 |

---

## 常見實作模式

### 1. Plugin-Based 強制（Decentralized Enforcement）

Apache Ranger 首創此模式：**策略決策集中化（Ranger Admin）**，但 **策略強制分散化**——輕量 Plugin 在各資料服務內部執行，快取策略以降低延遲。[^ranger-plugin]

### 2. PolicySync / 翻譯模式

Privacera 的做法：將中央 ABAC/TBAC 策略**翻譯為各目標平台的原生存取控制建構**，無需在現代雲端平台（Snowflake、Databricks）上安裝 Plugin。[^privacera-policysync]

### 3. 標籤/分類驅動 ABAC

- 建立 Tag（如 `classification=confidential`）
- 將 Tag 與策略關聯（如「Senior Analyst 可讀取 `classification=confidential`」的資料）
- 將 Tag 套用到物件（Database、Table、Column）
- 標籤繼承與傳播使策略自動涵蓋新匯入的資料

Snowflake、Databricks Unity Catalog、Apache Atlas 都支援此模式。[^tag-abac]

### 4. 聯合存取模式（Federated Access）

當查詢跨越多個自主來源時：

- **分離策略決策與策略強制**：中央層決定允許與否；各來源連接器證明能強制所需控制
- **身份傳傳播模型**：根據敏感度選擇 End-User Delegation（來源看得到真實使用者）或 Brokered Credentials
- **衍生資料比輸入更敏感**：跨來源 Join 可能揭露比任一來源更多的資訊；轉換後應重新分類並重新套用控制
- **Fail Closed**：若某來源無法強制所需控制，拒絕請求而非默默放寬權限[^federated-access]

### 5. ACL 表 + dbt 自動化

在 Reporting Layer 維護中央 ACL 表：

- 列層級 ACL 表：`user_group → region_code`
- 行層級 ACL 表：`user_group → column_name → can_read`
- 透過 dbt Macros 自動將 ACL 邏輯注入每個 View[^acl-dbt]

---

## 各情境建議方案

| 情境 | 建議方案 |
|------|---------|
| **跨平台 Lakehouse（Snowflake + Databricks + Redshift）** | Immuta 或 Satori——中央策略面 |
| **Hadoop/On-Prem Data Lake** | Apache Ranger + Apache Atlas |
| **AWS 原生 Data Lake** | AWS Lake Formation LF-Tag ABAC + Terraform |
| **開源可擴展棧** | OPA + Rego + OpenMetadata |
| **ETL 中心治理** | OvalEdge 或 dbt + 資料合約 |
| **新創/小團隊** | OPA + 資料合約——輕量、程式驅動 |
| **高度合規需求（GDPR、HIPAA、SOC 2）** | Immuta 或 Satori——內建稽核軌跡 |

---

## 核心結論

現代方法的核心原則是：**集中定義策略 + 分散式、情境感知的強制執行（Centralized Policy Definition + Decentralized Context-Aware Enforcement）**。[^summary]

策略在一個地方撰寫（或從分類推導），然後翻譯或原生地套用到各目標平台，並在所有來源上一致稽核。沒有任何平台試圖讓所有資料來源使用相同的存取控制模型——而是建立抽象層，將全域治理規則映射到各來源已有的原生機制上。

---

[^ranger-plugin]: Apache Ranger. (n.d.). Apache Ranger — Centralized Fine-Grained Authorization. Retrieved 2026-10-03, from https://ranger.apache.org/

[^privacera-policysync]: Privacera. (n.d.). Our Tribute to Apache Ranger by Extending Its Greatness to the Cloud. Retrieved 2026-10-03, from https://privacera.com/blog/our-tribute-to-apache-ranger-by-extending-its-greatness-to-the-cloud/

[^tag-abac]: Snowflake. (n.d.). Tag-Based Policies. Retrieved 2026-10-03, from https://docs.snowflake.com/en/user-guide/tag-based-policies

[^federated-access]: InfiniSynapse. (n.d.). Federated Access. Retrieved 2026-10-03, from https://infinisynapse.com/en/blog/federated-access

[^acl-dbt]: Scalefree. (n.d.). Row & Column Level Security in the Reporting Layer. Retrieved 2026-10-03, from https://www.scalefree.com/knowledge/webinars/data-vault-friday/row-column-level-security-in-the-reporting-layer/

[^summary]: Datasops. (n.d.). Lake Formation Data Governance. Retrieved 2026-10-03, from https://www.datasops.com/blog/lakeformation-data-governance