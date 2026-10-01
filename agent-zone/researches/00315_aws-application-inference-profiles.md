# AWS Application Inference Profiles 是什麼？

## 概述

**Application Inference Profile (AIP)** 是 Amazon Bedrock 提供的一種**成本歸屬（Cost Attribution）機制**，讓使用者可以按照應用程式、團隊、租戶（Tenant）或工作負載來區分 Bedrock 模型推論的費用[^aip-main]。

AIP 本質上是一個指向特定基礎模型（Foundation Model）的**邏輯端點**（Logical Endpoint），並可綁定自訂的**成本分配標籤**（Cost Allocation Tags）。應用程式在呼叫 Bedrock API 時，將原本的 Model ID 換成 Profile 的 ARN，後續的費用記錄就會帶上這些標籤，流入 AWS Cost Explorer 和 Cost & Usage Reports (CUR)，從而實現精確的費用分攤[^aip-guide]。

AIP 本身不收費，僅按模型調用的 Token 計費[^aip-guide]。

## AWS Bedrock 的兩種 Inference Profile

AWS Bedrock 有兩種類型的 Inference Profile，用途截然不同[^aip-overview]：

| 特性 | System-Defined (Cross-Region) Inference Profile | Application Inference Profile (AIP) |
|------|--------------------------------------------------|--------------------------------------|
| **建立者** | AWS 預先定義 | 使用者自行建立 |
| **主要目的** | **跨區域路由** — 將請求分散到多個區域以提高可用性和吞吐量 | **成本歸屬** — 加上標籤來追蹤哪個團隊／專案花了多少錢 |
| **路由行為** | 自動在不同 AWS 區域間分配請求 | 無特殊路由行為，僅是加上標籤層 |
| **可加標籤** | 否 | 是（可加自訂 Cost Allocation Tags） |
| **可管理性** | 只能使用，無法修改或刪除 | 可建立、修改標籤、刪除 |
| **ARN 格式** | `arn:aws:bedrock:region::inference-profile/...` | `arn:aws:bedrock:region:account-id:application-inference-profile/...` |

一句話總結：System-defined Profile 是**路由層**（Routing Layer），Application Inference Profile 是**歸屬層**（Attribution Layer）[^aip-overview]。

## 核心用途

AIP 的主要價值在於**成本歸屬與分攤**（Chargeback）[^aip-guide]：

- **成本歸屬**：多個團隊或租戶共用同一個基礎模型時，區分各自的花費[^aip-guide]
- **成本分配標籤**：每個 AIP 可綁定自訂 key-value 標籤（如 `dept=claims`、`team=alpha`、`tenantID=customerA`），這些標籤會流入 AWS 帳單工具[^aip-tag]
- **跨部門 Chargeback**：財務部門根據 Profile 標籤進行內部計費[^aip-blog]
- **多租戶成本管理**：為每個租戶建立獨立的 Profile，實現 SaaS 情境下的成本追蹤[^aip-blog]
- **預算控制**：可搭配 AWS Budgets 設定各 Profile 的預算告警
- **異常偵測**：可搭配 AWS Cost Anomaly Detection 偵測各 Profile 的異常支出
- **用量監控**：可搭配 CloudWatch 監控每個 Profile 的 Token 用量

## 使用情境

### 情境一：多部門成本分攤

公司內 HR、會計、IT 三個部門都透過 Bedrock 使用 Claude 模型。若直接用 Model ID 呼叫，帳單只有一筆總費用。為每個部門建立一個 AIP（指向同一個 Claude 模型），分別加上 `dept=HR`、`dept=Accounting`、`dept=IT` 標籤，Cost Explorer 就能按部門分組檢視花費[^aip-guide]。

### 情境二：多租戶 SaaS 成本追蹤

SaaS 平台為每個客戶租戶建立一個 AIP，標籤如 `tenantID=acme-corp`。即使所有租戶使用同一個模型，也能精確計算每個租戶的推論成本，用於內部計費或轉嫁成本[^aip-blog]。

### 情境三：跨區域成本歸屬

AIP 可以從 System-defined Cross-Region Inference Profile 建立，同時獲得跨區域路由和成本歸屬的能力，即使請求被路由到不同 AWS 區域，費用仍歸屬到同一 Profile[^aip-create]。

### 情境四：強制使用 AIP 治理

透過 Service Control Policies (SCP) 強制要求所有 Bedrock 調用必須使用已標籤的 AIP，否則拒絕請求。這確保了沒有未歸屬的費用產生[^aip-blog]。

## 建立與管理

### 建立 AIP

```bash
aws bedrock create-inference-profile \
  --inference-profile-name "HR-Team-Profile" \
  --model-source '{"copyFrom": "arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-3-sonnet-20240229-v1:0"}' \
  --tags '[{"key": "dept", "value": "HR"}]'
```

也可以從跨區域 System-defined Profile 建立，同時獲得跨區域路由與成本歸屬[^aip-create]。

### 檢視 AIP

```bash
aws bedrock list-inference-profiles --type-equals APPLICATION
aws bedrock get-inference-profile \
  --inference-profile-identifier arn:aws:bedrock:us-east-1:123456789012:application-inference-profile/abc123
```

### 修改標籤

```bash
aws bedrock tag-resource \
  --resource-arn arn:aws:bedrock:us-east-1:123456789012:application-inference-profile/abc123 \
  --tags '[{"key": "dept", "value": "HR"}]'
```

### 刪除 AIP

```bash
aws bedrock delete-inference-profile \
  --inference-profile-identifier arn:aws:bedrock:us-east-1:123456789012:application-inference-profile/abc123
```

刪除後使用該 Profile ARN 的應用程式會立即受影響[^aip-delete]。

### 啟用成本分配標籤

建立 AIP 並加上標籤後，需到 AWS Billing Console → **Cost allocation tags** 找到使用的標籤並點 **Activate**。啟用後最多需 24-48 小時資料才會出現在 Cost Explorer，且不回溯（non-retroactive）[^aip-tag]。

### 在應用程式中使用

將 Profile ARN 傳入 `modelId` 參數即可取代原本的 Model ID[^aip-use]：

```python
import boto3

client = boto3.client("bedrock-runtime")

response = client.converse(
    modelId="arn:aws:bedrock:us-east-1:123456789012:application-inference-profile/hr-team-profile",
    messages=[{"role": "user", "content": [{"text": "Hello"}]}]
)
```

### IAM 權限設定

使用 AIP 時，IAM Policy 需要同時允許 Profile 和底層的 Foundation Model[^aip-use]：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["bedrock:InvokeModel", "bedrock:InvokeModelWithResponseStream"],
      "Resource": [
        "arn:aws:bedrock:us-east-1:111122223333:application-inference-profile/*",
        "arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-3-sonnet-20240229-v1:0"
      ]
    }
  ]
}
```

## 使用限制與注意事項

- **支援的 API**：僅限 `InvokeModel`/`InvokeModelWithResponseStream`、`Converse`/`ConverseStream`。不支援較新的 `Responses` 和 `Chat Completions` API（回傳 400 錯誤）。後者需改用 IAM Principal Attribution 或 Per-request Metadata Tagging[^aip-support]
- **可建立的 Region**：AIP 僅能在特定 AWS Region 建立，非所有 Bedrock 可用區域都支援[^aip-support]
- **成本粒度**：提供的是聚合後的美元金額（按天、按用量類型），並非每筆請求的成本。若需每筆請求的 Token 明細，請使用 Per-request metadata tagging 搭配模型調用日誌[^aip-guide]
- **標籤上限**：每個 Profile 最多 50 個標籤[^aip-guide]
- **Profile 擴散問題**：每個 AIP 綁定一個特定模型加團隊組合。若組織有大量團隊和模型版本，Profile 數量會快速增長。AWS 建議改用 **Projects** 或 **IAM Principal Attribution** 以獲得更大彈性[^aip-blog]
- **無額外費用**：使用 AIP 本身不收費，僅按模型調用的 Token 計費[^aip-guide]
- **Console 可視性**：AIP 在 Bedrock Console 中不可見，僅 System-defined Profile 出現在 Console 中。需透過 API 或 CLI 管理[^aip-cli]

## 結論

Application Inference Profile 是 AWS Bedrock 的**成本歸屬機制**，透過為 API 呼叫加上一層自訂標籤，讓多團隊、多租戶或多應用程式共用同一個基礎模型進行推論時，能夠在 AWS Cost Explorer 中按標籤分組檢視各自的花費，實現精確的內部計費與預算管控。它不改變模型行為或請求路由，純粹是帳務與治理層面的功能。

---

[^aip-main]: Amazon Web Services. (n.d.). Application inference profiles. *Amazon Bedrock User Guide*. Retrieved 2026-10-01, from https://docs.aws.amazon.com/bedrock/latest/userguide/cost-mgmt-application-inference-profiles.html

[^aip-guide]: Amazon Web Services. (n.d.). Application inference profiles. *Amazon Bedrock User Guide*. Retrieved 2026-10-01, from https://docs.aws.amazon.com/bedrock/latest/userguide/cost-mgmt-application-inference-profiles.html

[^aip-overview]: Amazon Web Services. (n.d.). Set up a model invocation resource using inference profiles. *Amazon Bedrock User Guide*. Retrieved 2026-10-01, from https://docs.aws.amazon.com/bedrock/latest/userguide/inference-profiles.html

[^aip-create]: Amazon Web Services. (n.d.). Create an application inference profile. *Amazon Bedrock User Guide*. Retrieved 2026-10-01, from https://docs.aws.amazon.com/bedrock/latest/userguide/inference-profiles-create.html

[^aip-use]: Amazon Web Services. (n.d.). Use an inference profile in model invocation. *Amazon Bedrock User Guide*. Retrieved 2026-10-01, from https://docs.aws.amazon.com/bedrock/latest/userguide/inference-profiles-use.html

[^aip-support]: Amazon Web Services. (n.d.). Inference profiles support. *Amazon Bedrock User Guide*. Retrieved 2026-10-01, from https://docs.aws.amazon.com/bedrock/latest/userguide/inference-profiles-support.html

[^aip-tag]: Amazon Web Services. (2024). CreateInferenceProfile API Reference. *Amazon Bedrock API Reference*. Retrieved 2026-10-01, from https://docs.aws.amazon.com/bedrock/latest/APIReference/API_CreateInferenceProfile.html

[^aip-delete]: Amazon Web Services. (2024). DeleteInferenceProfile API Reference. *Amazon Bedrock API Reference*. Retrieved 2026-10-01, from https://docs.aws.amazon.com/bedrock/latest/APIReference/API_DeleteInferenceProfile.html

[^aip-blog]: Amazon Web Services. (2025). Manage multi-tenant Amazon Bedrock costs using application inference profiles. *AWS Machine Learning Blog*. Retrieved 2026-10-01, from https://aws.amazon.com/blogs/machine-learning/manage-multi-tenant-amazon-bedrock-costs-using-application-inference-profiles/

[^aip-cli]: Amazon Web Services. (n.d.). list-inference-profiles CLI Reference. *AWS CLI Documentation*. Retrieved 2026-10-01, from https://awscli.amazonaws.com/v2/documentation/api/2.18.18/reference/bedrock/list-inference-profiles.html