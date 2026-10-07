# ETL 專案在資料匯集過程中處理來源 AuthN/AuthZ 毀損問題的工具與模式

## 問題背景

在 Data Project / ETL 領域中，來自多個資料孤島（data silos）的資料被彙集到 Data Lake 或 Data Warehouse。每個資料孤島原本各有獨立的 AuthN（Authentication，身份驗證）和 AuthZ（Authorization，授權）規則。ETL 程序使用服務帳戶（service account）或 API token 連接到來源端，以高權限提取資料，這會造成：

- 資料提取時，來源系統的原始使用者身分不會跟著資料走
- 來源系統的 ACL／RBAC 規則並未嵌入在資料酬載（payload）中
- 資料到達目標倉儲後，原有的授權脈絡已完全斷裂

此報告不討論「對資料重新施行新的 ACM（Access Control Model）」這類方案，而是聚焦於 ETL 工具與專案實務上如何處理這個問題。

## 主要發現：沒有 ETL 工具保留了來源的 AuthN/AuthZ 脈絡

目前市場上沒有任何主流 ETL 工具能夠：

- 在提取資料時同時捕獲來源端的使用者身分
- 將來源端的 ACL／權限規則轉譯為目標端政策
- 在 ETL 管道中自動保留和轉換授權脈絡

以下逐一檢視各工具與模式的做法。

## 各工具如何處理（或不處理）此問題

### Apache Airflow

Airflow 是工作流程調度器（DAG scheduler），本身不具備資料層級的授權處理能力[^airflow]。Airflow operators 使用 Airflow 連線後端儲存的服務帳戶憑證連接到來源端，operator 以該憑證的權限執行，而非觸發 DAG 的使用者。Airflow 的 RBAC 僅控制 Airflow UI 層面的存取，並不影響通過管道移動的資料。

**結論**: Airflow 對此問題完全無關——它不處理資料層級的授權轉移，所有遮罩／過濾邏輯必須由 DAG 作者在 transformation tasks 中自行實作。

### dbt

dbt（data build tool）在 ELT 流程中負責 T（轉換）階段，資料已存在倉儲中[^dbt]。dbt 不從來源提取授權資訊，但它是**在目標端以程式碼方式部署安全性原則**的主要工具：

- dbt 可透過 YAML 設定檔定義 Snowflake 的 `row_access_policies`、`masking_policies` 和 `tag_based_masking_policies`，並在 CI/CD 管道中部署
- dbt Mesh / dbt Data Contracts 提供版本化的結構契約（關聯欄位名稱、型別、更新頻率），但這是有關 schema 治理的契約，並非授權規則的契約[^dbt_mesh]
- dbt 本身無法從來源捕獲 AuthZ 規則，但可讓資料工程師用程式碼管理目標端的安全性原則

### Apache NiFi

NiFi 專注於**保護 ETL 管道基礎設施本身**，而非保留來源的授權脈絡[^nifi]：

- 支援多租戶授權——控制哪些使用者／群組能存取 NiFi resources（processors、queues、flow）
- 支援 LDAP、Kerberos、OpenID Connect、SAML、Apache Knox、JWT 等認證
- processor 使用各來源／目標已設定的憑證執行——不會模擬（impersonate）原始使用者
- NiFi 可在資料流中套用轉換邏輯（`ReplaceText`、`JoltTransformJSON` 等）進行遮罩或假名化處理，但這是人工設定的，並非從來源授權自動推導

### Informatica

Informatica Cloud Data Integration（現由 Salesforce 管理）不處理來源授權轉移[^informatica]。其做法是在目標端透過**資料治理（Data Governance）**模組施行原則：

- CLAIRE AI 助理用於管道生成
- 支援在管道中設定資料遮罩（masking）和資料品質規則
- Data Marketplace 功能用於受治理的資料分享
- 這些規則由資料管理員（data stewards）設定，非源自來源系統

### Talend（Qlik Talend Cloud）

Talend 採取類似 Informatica 的做法——使用 AI agents 產生管道並整合治理步驟[^talend]。治理規則在目標端套用，而非從來源端繼承。

### Fivetran 與 Stitch

Fivetran 是資料複製（ELT 導向）工具[^fivetran]：

- 在來源端僅要求唯讀權限
- 所有連線使用 SSL/TLS 加密
- API connectors 以服務帳戶（OAuth tokens、API keys）認證並提取該帳戶可看見的全部資料
- 不提取或保留來源端的存取控制規則

Stitch（Talend 旗下）的做法相同。

## 現行實務中解決此問題的四種模式

儘管 ETL 工具本身不保留來源授權，業界發展出以下四種處理模式：

### 模式一：在目標倉儲重新實作存取原則（最常見）

這是目前最主要的做法。步驟為：

1. 將來源系統的**使用者目錄**（透過 Okta、Azure AD、LDAP 等）同步或 SSO 佈建到目標倉儲的身分系統
2. 使用倉儲原生政策語言**重新實作**存取規則：

   **Snowflake Row Access Policies（列層級安全）**[^sf_row]：
   ```sql
   CREATE ROW ACCESS POLICY rap_region AS (region varchar) RETURNS BOOLEON ->
     CASE
       WHEN CURRENT_ROLE() = 'admin' THEN true
       ELSE false
     END;
   ```

   **Snowflake Dynamic Data Masking（欄位層級安全）**[^sf_column]：
   ```sql
   CREATE MASKING POLICY ssn_mask AS (val string) RETURNS STRING ->
     CASE
       WHEN CURRENT_ROLE() IN ('PAYROLL') THEN val
       ELSE '***-**-****'
     END;
   ```

3. 政策的部署透過 dbt 或其他 IaC（Infrastructure as Code）工具進行版本控制

### 模式二：外部代幣化（External Tokenization）

Snowflake 支援在資料載入前先代幣化（tokenize），並在查詢時為授權角色還原（detokenize）[^sf_token]：

- 資料在載入 Snowflake 前由第三方代幣化服務處理
- Masking policies 搭配外部函數在查詢時為授權角色還原
- 儲存層的資料不暴露原始值
- 需要 Enterprise Edition 並整合外部代幣化服務

**注意**: 代幣化保留了資料的保護層，但授權規則仍由 Snowflake 的 masking policy 條件重新實作——並非從來源繼承。

### 模式三：標籤式遮罩原則（Tag-Based Masking）——屬性導向做法

Snowflake 的標籤式遮罩原則允許將遮罩原則套用於**標籤（tag）**而非個別欄位[^sf_tag]：

```sql
SYSTEM$GET_TAG_ON_CURRENT_COLUMN('tags.pii_col_string') = 'visible'
```

- 設定一個標籤在 database／schema 層級，所有符合的欄位自動受保護
- 可基於標籤字串值進行動態判斷
- 這是最接近屬性導向存取控制（ABAC）的做法，但仍需人力設定標籤和原則

### 模式四：資料契約（Data Contracts）——新興模式

dbt Mesh 的資料契約定義了 model producers 和 consumers 之間的結構約定[^dbt_mesh]。雖然這不是授權契約，但代表了**資料治理的契約式思維**。部分團隊在其契約中擴展了敏感度標記或存取要求：

```yaml
model: customer_orders
columns:
  - name: email
    sensitivity: PII
  - name: ssn
    sensitivity: PII
authorized_consumers:
  - team: billing
retention_days: 90
```

此模式仍在發展初期，尚未成為授權保留的主流解決方案。

## ABAC（屬性導向存取控制）在資料擷取的應用

ABAC 在倉儲環境中的實作依賴以下機制[^sf_access]：

- Policy body 中的 context functions：`CURRENT_ROLE()`、`INVOKER_ROLE()`、`CURRENT_IP()`、`CURRENT_TIMESTAMP()`
- Mapping tables：集中定義屬性和存取對應關係的表格，在 policy body 中被 JOIN
- Tag-based decisions：表格／欄位的標籤文字值充當查詢時計算的屬性

**關鍵認知**: ABAC 極少在 ETL 擷取階段套用——它是在查詢時間（query time）針對倉儲自己的使用者目錄和角色階層來判定。這表示來源的使用者目錄必須佈建到倉儲，且來源的存取規則必須手動以倉儲政策語言重新實作。

## Apache Atlas 與 Apache Ranger 的定位

Apache Ranger 是 Hadoop 時代的細粒度授權專案（源自 Cloudera），提供 HDFS、Hive、Impala 的角色和屬性授權[^ranger]。Apache Atlas 為其後繼者，整合了更精細的 ABAC 支援。

**現狀**: 這兩個專案已實質上被淘汰。隨著企業從 Hadoop 遷移至雲端資料倉儲（Snowflake、Databricks、BigQuery），Hadoop 安全堆疊（Sentry → Ranger → Atlas）已被放棄。現代的等位方案為 Snowflake Row Access Policies + Masking Policies、Databricks Unity Catalog policies、AWS Glue / Lake Formation permissions[^legacy]。

## OpenLineage 的輔助角色

OpenLineage 提供跨 ETL 管道的資料血緣追蹤（lineage tracking）[^openlineage]。雖非授權工具，血緣是了解資料來源和轉換歷程的前提條件——任何有意義的授權保留都需要先知道資料從哪裡來、經過哪些轉換。

## 結論與現實建議

| 問題 | 答案 |
|---|---|
| 有 ETL 工具可保留來源 AuthN/AuthZ 嗎？ | **沒有**——主流工具皆不支援。 |
| 業界實際做什麼？ | 在目標端使用倉儲原生安全特性重新實作存取原則。 |
| 最相關的工具？ | Snowflake（Row Access + Masking + Tokenization）、dbt（以程式碼部署原則）。 |
| Apache Atlas/Ranger 可用嗎？ | **不**——它們是 Hadoop 時代的遺留工具，不適用現代雲端倉儲。 |
| 資料契約有幫助嗎？ | 新興的 schema 治理模式，但尚未廣泛用於授權。 |
| ABAC 可用於 ETL 階段嗎？ | ABAC 在查詢時間而非 ETL 階段執行。 |
| **業界最佳實踐** | 1) 將來源使用者目錄複製到倉儲<br>2) 以程式碼（dbt）部署倉儲的 Row Access + Masking Policies<br>3) 對 PII 使用 External Tokenization 進行載入前保護 |

**現實**：這是 data engineering 中一個公認但尚未有工具解決的缺口。來源端的授權脈絡在 ETL 提取過程中必然斷裂，目前只能依賴目標端的重新實作和治理流程來填補。

[^airflow]: Apache Airflow. (n.d.). Apache Airflow Documentation. Retrieved 2025-10-10, from https://airflow.apache.org/docs/
[^dbt]: dbt Labs. (n.d.). dbt Documentation. Retrieved 2025-10-10, from https://docs.getdbt.com/
[^dbt_mesh]: dbt Labs. (n.d.). dbt Mesh / About Mesh. Retrieved 2025-10-10, from https://docs.getdbt.com/docs/mesh/about-mesh
[^nifi]: Apache NiFi. (n.d.). Apache NiFi Administration Guide. Retrieved 2025-10-10, from https://nifi.apache.org/docs/nifi-docs/html/administration-guide.html
[^informatica]: Informatica. (n.d.). Informatica Cloud Data Integration. Retrieved 2025-10-10, from https://www.informatica.com/products/data-integration.html
[^talend]: Talend / Qlik. (n.d.). Talend Data Integration. Retrieved 2025-10-10, from https://www.talend.com/products/data-integration/
[^fivetran]: Fivetran. (n.d.). Fivetran Security and Privacy: Security. Retrieved 2025-10-10, from https://fivetran.com/docs/security-and-privacy/security
[^sf_row]: Snowflake. (n.d.). Row Access Policies. Retrieved 2025-10-10, from https://docs.snowflake.com/en/user-guide/security-row-intro
[^sf_column]: Snowflake. (n.d.). Column-Level Security (Dynamic Data Masking + External Tokenization). Retrieved 2025-10-10, from https://docs.snowflake.com/en/user-guide/security-column-intro
[^sf_token]: Snowflake. (n.d.). External Tokenization. Retrieved 2025-10-10, from https://docs.snowflake.com/en/user-guide/security-column-ext-token-intro
[^sf_tag]: Snowflake. (n.d.). Tag-Based Masking Policies. Retrieved 2025-10-10, from https://docs.snowflake.com/en/user-guide/tag-based-masking-policies
[^sf_access]: Snowflake. (n.d.). Snowflake Access Control Framework. Retrieved 2025-10-10, from https://docs.snowflake.com/en/user-guide/security-access-control-overview
[^ranger]: Cloudera / Apache. (n.d.). Apache Ranger (Fine-Grained Authorization). Retrieved 2025-10-10, from https://ranger.apache.org/
[^legacy]: Databricks. (n.d.). Data Lakehouse Glossary. Retrieved 2025-10-10, from https://www.databricks.com/glossary/data-lakehouse
[^openlineage]: OpenLineage. (n.d.). OpenLineage — Data Lineage for Data Pipelines. Retrieved 2025-10-10, from https://openlineage.io/