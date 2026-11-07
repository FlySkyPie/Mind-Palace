# DevLake 生態系中「不接真實 Jira 也能測試」的假資料方案調查

## 摘要

本報告調查 Apache DevLake（以下簡稱 DevLake）生態系中，是否存在用於「播種（seed）／偽造（fake）／模擬（mock）」資料、以便在不串接真實 Jira 實例的情況下進行測試的現成方案。

結論是：**DevLake 官方並未提供獨立的「Mock Jira 伺服器」或通用的假資料產生器**，但專案內建了一套成熟且被大量使用的替代機制，核心是 `e2ehelper.DataFlowTester` 搭配 CSV 形式的原始資料（raw tables）與快照資料（snapshot tables）。這套機制讓 Jira 外掛的 extractor 與 convertor 子任務可以在完全不發出任何 HTTP 請求的情況下被完整測試[^dft][^devmanual]。此外，單元測試層使用 mockery 產生的介面 mock 與 `unithelper` 的 Dummy 輔助物件[^mockery][^unithelper]，伺服器層則有 `backend/test/e2e` 的「假外掛或無外掛」測試與 `backend/test/helper` 的 Go 測試客戶端[^testreadme][^devmanual]。

---

## 一、背景：DevLake 的三層測試策略

DevLake 官方開發手冊明確將測試分為三類[^devmanual]：

| 測試類型 | 說明 | 是否需真實資料源 |
|---|---|---|
| 單元測試（Unit-tests） | 元件層級測試，以 mock 取代依賴，通常與被測原始碼放在同一目錄 | 否 |
| E2E 測試（E2E-tests） | 以「偽造資料」模擬資料收集結果，測試外掛的 extractor 與 convertor 子任務 | 否 |
| 整合測試（Integration-tests） | 測試整個 DevLake 伺服器，透過 Go Client 連線既有伺服器或啟動記憶體內實例 | 是（多數情況） |

其中第二類正是本報告關注的「不接真實 Jira」路徑。手冊原文指出 E2E 測試「使用偽造資料來模擬資料收集的結果」[^devmanual]。

```mermaid
flowchart LR
    A["真實 Jira REST API"] -.->|"正常生產流程"| B["_raw_jira_api_* 原始表"]
    C["CSV 假資料<br/>raw_tables/*.csv"] -->|"ImportCsvIntoRawTable"| B
    B -->|"Extractor 子任務"| D["_tool_jira_* 工具層表"]
    D -->|"Convertor 子任務"| E["domain layer 表<br/>issues / board_issues ..."]
    D -->|"VerifyTable"| F["snapshot_tables/*.csv<br/>期望值比對"]
    E -->|"VerifyTable"| F
```

上圖說明：測試時以 CSV 直接寫入原始表，繞過整個 collector（也就是真正呼叫 Jira API 的階段），因此不需要任何 Jira 連線[^dft]。

---

## 二、核心方案：`e2ehelper.DataFlowTester`

### 2.1 定位與檔案位置

`DataFlowTester` 位於 `backend/helpers/e2ehelper/data_flow_tester.go`，是 DevLake 用來驗證外掛各階段資料完整性的通用工具[^dft]。其原始碼註解描述的工作流程為[^dft]：

1. 清空指定資料表，並透過 `ImportCsv` 方法從 CSV 檔匯入資料
2. 透過 `Subtask` 方法執行指定子任務
3. 將資料庫實際資料與另一份 CSV 的期望資料比對
4. 重複步驟 2 與 3

### 2.2 關鍵方法

| 方法 | 用途 |
|---|---|
| `NewDataFlowTester(t, pluginName, pluginMeta)` | 建立測試器，需 `E2E_DB_URL` 環境變數指向測試資料庫[^dft] |
| `ImportCsvIntoRawTable(csvRelPath, rawTableName)` | 將 CSV 匯入原始資料表（模擬 API 回應）[^dft] |
| `ImportCsvIntoTabler(csvRelPath, dst)` | 將 CSV 匯入工具層模型表，空字串視為 NULL[^dft] |
| `ImportNullableCsvIntoTabler(...)` | 同上，但僅字面值 `NULL` 視為 NULL[^dft] |
| `FlushTabler(dst)` / `FlushRawTable(name)` | 刪除並重建資料表，確保乾淨狀態[^dft] |
| `Subtask(subtaskMeta, taskData)` | 執行外掛子任務[^dft] |
| `VerifyTable(dst, csvRelPath, targetFields)` | 依主鍵逐筆比對資料庫與 CSV[^dft] |
| `VerifyTableWithOptions(dst, opts)` | 進階比對，支援 `IgnoreFields`、`IgnoreTypes`、`Nullable`[^dft] |
| `CreateSnapshot(dst, opts)` | 由資料庫現況反向產生 CSV 快照（期望檔不存在時自動呼叫）[^dft] |
| `ExportRawTable(rawTableName, csvRelPath)` | 將原始表匯出成 CSV[^dft] |
| `UpdateCSVs(csvDir, updater)` | 批次改寫目錄下所有 CSV[^dft] |

值得注意的是 `VerifyTableWithOptions` 的行為：若指定的 CSV 不存在，會直接呼叫 `CreateSnapshot` 產生該檔並返回[^dft]。這代表開發者可以先寫測試、跑一次、讓工具自動生成期望值，再人工檢視後提交。

### 2.3 執行前提

`NewDataFlowTester` 會讀取 `E2E_DB_URL`，若未設定則 panic，錯誤訊息為「e2e can only run with E2E_DB_URL」[^dft]。官方 `docker-compose-dev-mysql.yml` 已為 devlake 服務同時設定 `DB_URL`（`lake` 資料庫）與 `E2E_DB_URL`（`lake_test` 資料庫）[^compose]。測試執行方式為[^testreadme]：

```bash
export E2E_DB_URL='mysql://merico:merico@127.0.0.1:3306/lake_test?charset=utf8mb4&parseTime=True'
cd backend
go run ./test/init.go
make e2e-test
```

官方 Readme 特別註明資料庫 URL 必須使用 `127.0.0.1` 而非 `localhost`，因為部分 Python 依賴無法正確解析 `localhost`[^testreadme]。

---

## 三、Jira 專屬的假資料：CSV 形式的「偽 Jira API 回應」

### 3.1 目錄結構

Jira 外掛的 E2E 測試位於 `backend/plugins/jira/e2e/`，內含兩個資料目錄[^jirae2e]：

- `raw_tables/`：模擬 Jira REST API 回應的原始資料
- `snapshot_tables/`：處理後的期望輸出

`raw_tables/` 下共有 14 個 CSV 檔[^jiraraw]，包含：

| 檔案 | 大小（位元組） |
|---|---|
| `_raw_jira_api_issues.csv` | 1,178,310 |
| `_raw_jira_api_development_panels.csv` | 945,384 |
| `_raw_jira_api_issue_changelogs.csv` | 441,939 |
| `_raw_jira_api_issue_relationships.csv` | 233,100 |
| `_raw_jira_api_remotelinks.csv` | 99,378 |
| `_raw_jira_api_worklogs.csv` | 74,753 |
| `_raw_jira_api_issues_for_due_date.csv` | 54,921 |
| `_raw_jira_api_users.csv` | 49,162 |
| `_raw_jira_external_epics.csv` | 36,344 |
| `_raw_jira_api_status.csv` | 29,303 |
| `_raw_jira_api_sprints.csv` | 14,863 |
| `_raw_jira_api_projects.csv` | 12,205 |
| `_raw_jira_api_issue_types.csv` | 3,583 |
| `_raw_jira_api_issue_fields.csv` | 882 |

### 3.2 CSV 的欄位結構

原始表 CSV 的欄位固定為 `id, params, data, url, input, created_at`[^jiraraw]。其中 `data` 欄位存放的是**完整的 Jira API JSON 回應字串**，`params` 存放連線與看板資訊。以 `_raw_jira_api_issue_types.csv` 為例，實際內容如下[^jiraraw]：

```csv
id,params,data,url,input,created_at
20,"{""ConnectionId"":2,""BoardId"":8}","{""self"":""https://merico.atlassian.net/rest/api/3/issuetype/10001"",""id"":""10001"",""description"":""表述为用户目标的功能。"",""name"":""故事"",""untranslatedName"":""Story"",""subtask"":false,""avatarId"":10315,""hierarchyLevel"":0}",https://merico.atlassian.net/rest/api/3/issuetype,null,2022-07-16 06:27:15.411
```

可以看到 `data` 欄位就是 Jira 的 `/rest/api/3/issuetype` 回應本體，因此 extractor 讀取時與真實情境完全一致，只是資料來源從 HTTP 換成了資料庫中的原始表[^jiraraw][^dft]。

### 3.3 測試案例：`TestIssueDataFlow`

`backend/plugins/jira/e2e/issue_test.go` 中的 `TestIssueDataFlow` 是最具代表性的範例[^issuetest]。其流程為：

1. 建立 `DataFlowTester`
2. 以 `ImportCsvIntoRawTable` 匯入 `_raw_jira_api_issues.csv` 與 `_raw_jira_api_issue_types.csv`
3. 清空 `JiraIssue`、`JiraBoardIssue`、`JiraSprintIssue` 等工具層表
4. 執行 `ExtractIssueTypesMeta` 與 `ExtractIssuesMeta` 子任務
5. 以 `VerifyTable` 比對 `_tool_jira_issue_types.csv`、`_tool_jira_issues.csv` 等快照
6. 清空 `ticket.Issue`、`ticket.BoardIssue`、`ticket.IssueAssignee` 等領域層表
7. 執行 `ConvertIssuesMeta` 子任務
8. 以 `VerifyTable` 比對 `issues.csv`、`board_issues.csv`、`issue_assignees.csv`

同一目錄下另有 `board_test.go`、`sprint_test.go`、`changelog_test.go`、`epic_test.go`、`worklog_test.go`、`status_test.go`、`label_test.go`、`project_test.go`、`account_test.go`、`remotelink_test.go`、`issue_relationship_test.go`、`issue_repo_commit_test.go`、`migration_schema_test.go` 等 19 個以上的測試檔[^jirae2e]。

### 3.4 測試資料的來源與真實性

這些 CSV 並非隨機生成的假資料，而是從真實 Jira 實例（`merico.atlassian.net`）擷取後落地的回應[^jiraraw]。因此它們保留了真實世界的欄位型態、多語系字串（例如中文的「故事」「子任務」）、以及 Xray 等外掛自訂的 issue type（如 `Test Execution`）[^jiraraw]。這使得測試能覆蓋真實 API 的邊界情況，而不只是理想化的資料。

---

## 四、單元測試層的 mock 機制

### 4.1 mockery 產生的介面 mock

DevLake 使用 mockery v3 產生核心介面的 mock，設定檔為 `backend/.mockery.core.yml` 與 `backend/.mockery.helpers.yml`，輸出至 `backend/mocks/`[^mockery]。設定檔註解說明其取代了舊版 v2 的命令列參數，並刻意將 core 與 helpers 兩份設定分開，因為 `helpers/unithelper` 會匯入 `mocks/core/...`，core mock 必須先存在才能型別檢查[^mockery]。

產生的 mock 涵蓋 `core/context`、`core/dal`、`core/log`、`core/plugin` 等套件，底層使用 `testify/mock`[^mockery]。

### 4.2 `unithelper` 的 Dummy 輔助物件

`backend/helpers/unithelper/` 提供三個便利函式[^unithelper]：

- `DummyBasicRes(callback)`：建立帶有 mock DAL 與 mock Logger 的 `BasicRes`[^dummybasicres]
- `DummySubTaskContext(db)`：建立假的子任務 context
- `DummyLogger()`：建立 mock logger

`DummyBasicRes` 的實作會先建立 `mockcontext.BasicRes`、`DummyLogger()` 與 `mockdal.Dal`，執行 callback 讓呼叫端設定 DAL 期望值，再掛上 `GetDal`、`GetLogger`、`GetConfig`、`AllTables` 等預設回傳[^dummybasicres]。

### 4.3 直接建構 `http.Response`（不啟動 mock 伺服器）

Jira 的 collector 單元測試採取另一種策略：**直接以字串建構 `*http.Response` 物件**，完全不啟動 HTTP 伺服器。`issue_collector_test.go` 中的 `Test_parseIssuesResponse` 定義了輔助函式[^issuecollectortest]：

```go
makeResponse := func(body string) *http.Response {
    return &http.Response{
        Body:    io.NopCloser(strings.NewReader(body)),
        Request: &http.Request{URL: &url.URL{Scheme: "https", Host: "jira.example.com", Path: "/rest/api/2/search"}},
    }
}
```

測試案例涵蓋：正常 JSON、代理回傳的 HTML 錯誤頁（502 Bad Gateway）、被截斷的 JSON、以及空回應[^issuecollectortest]。這證實了 DevLake 在 Jira 測試中**並未使用 `httptest.NewServer`**，而是繞過 HTTP 層。

同一檔案還包含 `Test_buildJQL`、`Test_buildFilterJQL`、`Test_renderExtraJQL` 等純函式測試，以及使用 `testing/quick` 的屬性測試（property-based testing）[^issuecollectortest]。

---

## 五、伺服器層的 E2E 與整合測試

### 5.1 `backend/test/e2e`：假外掛或無外掛

`backend/test/Readme.md` 明確區分兩個目錄[^testreadme]：

- `e2e/`：包含與「假外掛或完全沒有外掛」互動的 DevLake 伺服器測試
- `integration/`：針對「含有測試資料的真實資料源」撰寫的伺服器測試

`backend/test/e2e/` 下分為 `manual/`、`migrationhelper/`、`remote/`、`services/` 四個子目錄[^teste2e]。其中 `services/` 包含 `pipeline_runner_test.go` 與 `server_startup_test.go`[^testservices]。

### 5.2 `backend/test/helper`：Go 測試客戶端

`backend/test/helper/` 提供 `api.go`、`bootstrap.go`、`client.go`、`client_factory.go`、`db_selector.go`、`json_helper.go`、`models.go`、`utils.go` 等檔案[^testhelper]。開發手冊說明此客戶端可「與既有 DevLake 伺服器互動，或為測試目的啟動記憶體內的 DevLake 實例」[^devmanual]。

手冊中的範例 `ConnectLocalServer` 可指定 `CreateServer: true` 啟動記憶體內伺服器、`TruncateDb` 清空資料、`DropDb` 刪除資料表，並以 `Plugins` 清單指定要編譯進來的外掛[^devmanual]。

### 5.3 `manual/`：需真實憑證的手動測試

`backend/test/e2e/manual/Readme.md` 說明此目錄的測試「不供 CI/CD 自動執行，而是由開發者手動執行」，且「通常會連接真實資料源，因此任何資料源特定的資料／憑證都需由外部提供」[^manualreadme]。目前包含 `argocd`、`azuredevops`、`gitlab`、`pagerduty`、`testmo` 五個子目錄[^manualdir]，**沒有 Jira 的手動整合測試目錄**。

---

## 六、其他相關機制

### 6.1 `kiro` 外掛：以介面 mock 取代真實 S3

`kiro` 外掛（收集 AWS Kiro 的 AI 使用紀錄）示範了另一種模式：將資料源客戶端抽象成介面，測試時注入假實作。`discovery_test.go` 定義了 `prefixMockS3` 結構，實作 `ListObjectsV2` 與 `GetObject`，以預先寫死的 prefix 樹回應查詢[^kirotest]：

```go
type prefixMockS3 struct {
    commonPrefixes map[string][]string
    objects        map[string][]string
    seenDelimiter  []string
    seenPrefix     []string
}
```

這是「以介面 mock 模擬外部 API」的範例，但它是針對 S3 而非 Jira，且屬於該外掛自身的測試實作，並非通用工具[^kirotest]。

### 6.2 `webhook` 外掛：可注入資料，但非 Jira 格式

DevLake 有 `webhook` 外掛，可透過 HTTP 推送資料進入系統[^plugins]。然而其資料格式為 DevLake 內部的領域模型，而非 Jira API 格式，因此無法直接當作「假 Jira」使用。

### 6.3 外掛清單中無 mock／fake 外掛

檢視 `backend/plugins/` 目錄，現有外掛包括 `ae`、`argocd`、`asana`、`azuredevops_go`、`bamboo`、`bitbucket`、`circleci`、`claude_code`、`clickup`、`cursor`、`customize`、`dora`、`feishu`、`gh-copilot`、`gitee`、`gitextractor`、`github`、`github_graphql`、`gitlab`、`grafana_irm`、`icla`、`incidentio`、`issue_trace`、`jenkins`、`jira`、`kiro`、`linear`、`linker`、`monorepo`、`opsgenie`、`org`、`pagerduty`、`refdiff`、`rootly`、`schema_e2e`、`slack`、`sonarqube`、`starrocks`、`taiga`、`tapd`、`teambition`、`tempo`、`testmo`、`trello`、`webhook`、`youtrack`、`zentao` 等[^plugins]，**其中沒有任何名為 `mock` 或 `fake` 的外掛**。

---

## 七、綜合對照表

| 方案 | 位置 | 是否需真實 Jira | 適用層級 |
|---|---|---|---|
| `DataFlowTester` + CSV 原始表 | `backend/helpers/e2ehelper/`、`backend/plugins/jira/e2e/raw_tables/` | 否 | 外掛 extractor／convertor |
| CSV 快照比對 | `backend/plugins/jira/e2e/snapshot_tables/` | 否 | 期望值驗證 |
| mockery 介面 mock | `backend/mocks/`、`.mockery.*.yml` | 否 | 單元測試 |
| `unithelper` Dummy 物件 | `backend/helpers/unithelper/` | 否 | 單元測試 |
| 直接建構 `http.Response` | `backend/plugins/jira/tasks/*_test.go` | 否 | collector 純函式 |
| 假外掛／無外掛伺服器測試 | `backend/test/e2e/` | 否 | 伺服器層 |
| Go 測試客戶端 | `backend/test/helper/` | 視設定 | 整合測試 |
| 手動整合測試 | `backend/test/e2e/manual/` | 是 | 端到端（無 Jira 目錄） |
| 獨立 Mock Jira 伺服器 | 不存在 | — | — |
| 通用假資料產生器 | 不存在 | — | — |

---

## 八、批判性反思

### 8.1 來源權威性

本報告主要依據 DevLake 官方儲存庫的原始碼與官方文件（`DevelopmentManual.md`、`backend/test/Readme.md`、`backend/test/e2e/manual/Readme.md`），這些屬於第一手權威來源。GitHub API 回傳的目錄清單與檔案大小為即時資料，可信度高[^plugins][^jiraraw]。

### 8.2 時效性

DevLake 儲存庫的 `pushed_at` 為 2026-11-06，`updated_at` 為 2026-11-06，顯示專案仍在活躍維護[^repo]。本報告引用的檔案路徑與內容皆取自 `main` 分支，可能隨時間變動。特別值得注意的是 `kiro`、`claude_code`、`cursor`、`gh-copilot` 等 AI 工具相關外掛的出現，反映專案正在擴展資料源範圍[^plugins]。

### 8.3 潛在偏誤

官方文件傾向呈現「測試策略完整」的正面形象，但實際上：

- E2E 測試仍**需要一個真實的 MySQL 或 Postgres 資料庫**（`E2E_DB_URL`），並非完全無依賴[^dft]
- CSV 假資料是從真實 Jira 擷取的快照，若 Jira API 格式變更，這些 CSV 不會自動更新，需人工維護
- 手動整合測試目錄中沒有 Jira，意味著 Jira 外掛的「完整端到端流程」（含 collector 實際呼叫 API）缺乏官方自動化覆蓋

### 8.4 佐證與交叉驗證

本報告的結論在三個獨立來源中一致：(1) 官方 `DevelopmentManual.md` 的測試策略描述[^devmanual]；(2) `backend/test/Readme.md` 對 `e2e/` 與 `integration/` 的區分[^testreadme]；(3) 實際原始碼中 `DataFlowTester` 的實作與 Jira E2E 測試的用法[^dft][^issuetest]。三者互相印證，可信度高。

### 8.5 未找到的部分

本報告未能找到任何第三方或社群工具專門為 DevLake 模擬 Jira。若讀者需要「以 HTTP 層模擬 Jira」的方案，DevLake 生態系內並無現成工具，需自行以 `httptest` 或 WireMock 等通用工具建構，或改用上述 CSV 原始表方案繞過 HTTP 層。

---

## 參考文獻

[^dft]: Apache Software Foundation. (n.d.). *data_flow_tester.go*. Retrieved 2026-11-07, from https://github.com/apache/devlake/blob/main/backend/helpers/e2ehelper/data_flow_tester.go

[^devmanual]: Apache Software Foundation. (n.d.). *DevelopmentManual.md*. Retrieved 2026-11-07, from https://github.com/apache/devlake/blob/main/backend/DevelopmentManual.md

[^testreadme]: Apache Software Foundation. (n.d.). *backend/test/Readme.md*. Retrieved 2026-11-07, from https://github.com/apache/devlake/blob/main/backend/test/Readme.md

[^manualreadme]: Apache Software Foundation. (n.d.). *backend/test/e2e/manual/Readme.md*. Retrieved 2026-11-07, from https://github.com/apache/devlake/blob/main/backend/test/e2e/manual/Readme.md

[^jirae2e]: Apache Software Foundation. (n.d.). *backend/plugins/jira/e2e directory listing*. Retrieved 2026-11-07, from https://github.com/apache/devlake/tree/main/backend/plugins/jira/e2e

[^jiraraw]: Apache Software Foundation. (n.d.). *backend/plugins/jira/e2e/raw_tables directory listing and _raw_jira_api_issue_types.csv*. Retrieved 2026-11-07, from https://github.com/apache/devlake/tree/main/backend/plugins/jira/e2e/raw_tables

[^issuetest]: Apache Software Foundation. (n.d.). *issue_test.go*. Retrieved 2026-11-07, from https://github.com/apache/devlake/blob/main/backend/plugins/jira/e2e/issue_test.go

[^issuecollectortest]: Apache Software Foundation. (n.d.). *issue_collector_test.go*. Retrieved 2026-11-07, from https://github.com/apache/devlake/blob/main/backend/plugins/jira/tasks/issue_collector_test.go

[^mockery]: Apache Software Foundation. (n.d.). *.mockery.core.yml*. Retrieved 2026-11-07, from https://github.com/apache/devlake/blob/main/backend/.mockery.core.yml

[^unithelper]: Apache Software Foundation. (n.d.). *backend/helpers/unithelper directory listing*. Retrieved 2026-11-07, from https://github.com/apache/devlake/tree/main/backend/helpers/unithelper

[^dummybasicres]: Apache Software Foundation. (n.d.). *dummy_basicres.go*. Retrieved 2026-11-07, from https://github.com/apache/devlake/blob/main/backend/helpers/unithelper/dummy_basicres.go

[^teste2e]: Apache Software Foundation. (n.d.). *backend/test/e2e directory listing*. Retrieved 2026-11-07, from https://github.com/apache/devlake/tree/main/backend/test/e2e

[^testservices]: Apache Software Foundation. (n.d.). *backend/test/e2e/services directory listing*. Retrieved 2026-11-07, from https://github.com/apache/devlake/tree/main/backend/test/e2e/services

[^testhelper]: Apache Software Foundation. (n.d.). *backend/test/helper directory listing*. Retrieved 2026-11-07, from https://github.com/apache/devlake/tree/main/backend/test/helper

[^manualdir]: Apache Software Foundation. (n.d.). *backend/test/e2e/manual directory listing*. Retrieved 2026-11-07, from https://github.com/apache/devlake/tree/main/backend/test/e2e/manual

[^kirotest]: Apache Software Foundation. (n.d.). *discovery_test.go*. Retrieved 2026-11-07, from https://github.com/apache/devlake/blob/main/backend/plugins/kiro/tasks/discovery_test.go

[^plugins]: Apache Software Foundation. (n.d.). *backend/plugins directory listing*. Retrieved 2026-11-07, from https://github.com/apache/devlake/tree/main/backend/plugins

[^compose]: Apache Software Foundation. (n.d.). *docker-compose-dev-mysql.yml*. Retrieved 2026-11-07, from https://github.com/apache/devlake/blob/main/docker-compose-dev-mysql.yml

[^repo]: Apache Software Foundation. (n.d.). *apache/devlake repository metadata*. Retrieved 2026-11-07, from https://api.github.com/repos/apache/devlake
