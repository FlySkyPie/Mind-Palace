# AWS Bedrock Application Inference Profile 是什麼？

## 概述

**Application Inference Profile (AIP)** 是 Amazon Bedrock 中一種用來為應用程式、團隊或工作負載**歸屬（Attribute）Bedrock 使用成本**的資源概念。它本質上是一個帶有成本分配標籤（Cost Allocation Tags）的模型包裝層。透過建立一個指向特定模型的 Profile 並綁定自訂標籤，後續在 API 呼叫中使用該 Profile 的 ARN 來取代原本的 Model ID，AWS Cost Explorer 和 Cost & Usage Reports (CUR) 就可以根據標籤來區分不同團隊或專案的費用[^aip-guide]。

AIP 本身不收費，僅按模型調用的 Token 計費[^aip-guide]。

## Application Inference Profile 與 System-Defined Inference Profile 的差別

AWS Bedrock 有兩種 Inference Profile，目的完全不同：

| 特性 | System-Defined (Cross-Region) Inference Profile | Application Inference Profile (AIP) |
|------|--------------------------------------------------|--------------------------------------|
| **建立者** | AWS 預先定義 | 使用者自行建立 |
| **主要目的** | **跨區域路由** — 將請求分散到多個區域以提高吞吐量和可用性 | **成本歸屬** — 加上標籤來追蹤哪個團隊／專案花了多少錢 |
| **路由行為** | 會自動在不同區域間分配請求 | **無路由行為差異** — 僅是加上標籤層，不改變模型行為 |
| **可加標籤** | 否 | 是（可加自訂 Cost Allocation Tags） |
| **可用 API** | InvokeModel、Converse、Responses、Chat Completions 皆可 | 僅限 InvokeModel 和 Converse（Responses／Chat Completions 會回傳 400 錯誤） |
| **ARN 格式** | `arn:aws:bedrock:region::inference-profile/...` | `arn:aws:bedrock:region:account-id:application-inference-profile/...` |
| **管理方式** | 只能使用，無法修改或刪除 | 可建立、修改標籤、刪除 |

一句話總結差異：System-defined Profile 是**路由層**（Routing Layer），Application Inference Profile 是**歸屬層**（Attribution Layer）[^aip-diff]。

## 核心用途

AIP 的主要價值在於**成本歸屬與分攤**（Chargeback）。它解決了多團隊共用同一個基礎模型時，帳單上只顯示一條總費用而無法區分各團隊花費的問題[^aip-guide]。

具體用途包括：

- **成本歸屬**：讓多個團隊共用同一個基礎模型時，仍能區分誰花多少錢[^aip-guide]。
- **成本分配標籤**：每個 AIP 可綁定自訂標籤（如 `Team=HR`），這些標籤會流入 AWS Cost Explorer 和 CUR[^aip-tag]。
- **跨部門 Chargeback**：財務部門可以根據 Profile 標籤進行內部計費[^aip-arch]。
- **預算控制**：可搭配 AWS Budgets 設定各 Profile 的預算告警[^aip-arch]。
- **異常偵測**：可搭配 AWS Cost Anomaly Detection 偵測各 Profile 的異常支出[^aip-arch]。
- **用量監控**：可搭配 CloudWatch 監控每個 Profile 的 Token 用量（需搭配模型調用日誌）[^aip-arch]。

## 使用情境

### 情境一：多團隊共用模型、各自分攤成本

公司內 HR、會計、IT 三個部門都透過 Bedrock 的 Converse API 使用 Claude 模型。若直接用 Model ID 呼叫，帳單上只會看到一條「Claude 總費用」，無法區分哪個部門花了多少。

解法是為每個部門建立一個 AIP（都指向同一個 Claude 模型），分別加上 `Team=HR`、`Team=Accounting`、`Team=IT` 標籤。應用程式層根據使用者所屬部門，選擇對應的 Profile ARN 來呼叫。Cost Explorer 就能用 `Team` 標籤分組，清楚顯示各部門的花費[^aip-guide][^aip-arch]。

### 情境二：使用 SCP 強制執行 AIP 使用

組織可以透過 Service Control Policies (SCP) 強制要求所有 Bedrock 調用都必須透過已標籤的 AIP，否則拒絕請求，以達到「不用 Profile 就不能呼叫模型」的治理效果[^aip-arch]。

### 情境三：跨區域成本追蹤

AIP 可以從跨區域（cross-Region / system-defined）的 Inference Profile 建立，這樣即使請求被路由到不同 AWS 區域，成本仍然會歸屬到同一個 Profile 下[^aip-create]。

## 建立與管理方式

### 建立 AIP

可以透過 AWS CLI、API 或 Console 建立：

```bash
aws bedrock create-inference-profile \
  --inference-profile-name "HR-Team-Profile" \
  --model-source '{"copyFrom": "arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-3-sonnet-20240229-v1:0"}' \
  --tags '[{"key": "Team", "value": "HR"}]'
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
  --tags '[{"key": "Team", "value": "HR"}]'
```

### 刪除 AIP

```bash
aws bedrock delete-inference-profile \
  --inference-profile-identifier arn:aws:bedrock:us-east-1:123456789012:application-inference-profile/abc123
```

刪除後使用該 Profile ARN 的應用程式會立即受影響，需重新建立（會得到新的 ARN）[^aip-delete]。

### 啟用成本分配標籤

建立並標記好 AIP 後，需到 AWS Billing and Cost Management Console → **Cost allocation tags** 找到使用的標籤（如 `Team`）並點 **Activate**，啟用後最多需 24-48 小時資料才會出現在 Cost Explorer[^aip-tag]。

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

使用 AIP 時，IAM Policy 需要同時允許 Profile 本身和底層的 Foundation Model[^aip-use]：

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

- **支援的 API**：僅限 `InvokeModel`、`InvokeModelWithResponseStream`、`Converse`、`ConverseStream`。不支援 Responses 和 Chat Completions API（會回傳 400 錯誤）[^aip-support]。
- **可建立的 Region**：AIP 僅能在特定 AWS Region 建立，非所有 Bedrock 可用區域都支援[^aip-support]。
- **成本粒度**：提供的是聚合後的美元金額（按天、按用量類型），並非每筆請求的成本。若需每筆請求的 Token 明細，請使用 Per-request metadata tagging[^aip-guide]。
- **標籤上限**：每個 Profile 最多 50 個標籤[^aip-guide]。
- **Profile 擴散問題**：每個 Profile 綁定一個特定模型加團隊組合。若組織有大量團隊和模型版本，Profile 數量會快速增長。建議用 Projects 來替代以獲得更大彈性[^aip-arch]。
- **無額外費用**：使用 AIP 本身不收費，僅按模型調用的 Token 計費[^aip-guide]。

## 結論

Application Inference Profile 是 AWS Bedrock 提供的**成本歸屬機制**，並非用來改變模型行為或路由請求。它為 Bedrock API 呼叫加上一層標籤來區分成本歸屬對象，讓多團隊共用同一個基礎模型進行推論時，能夠在 Cost Explorer 中按團隊檢視各自的花費，實現精確的內部計費與預算管控。

---

[^aip-guide]: Amazon Web Services. (n.d.). Application inference profiles. *Amazon Bedrock User Guide*. Retrieved 2026-09-27, from https://docs.aws.amazon.com/bedrock/latest/userguide/cost-mgmt-application-inference-profiles.html

[^aip-diff]: Amazon Web Services. (n.d.). Set up a model invocation resource using inference profiles. *Amazon Bedrock User Guide*. Retrieved 2026-09-27, from https://docs.aws.amazon.com/bedrock/latest/userguide/inference-profiles.html

[^aip-create]: Amazon Web Services. (n.d.). Create an application inference profile. *Amazon Bedrock User Guide*. Retrieved 2026-09-27, from https://docs.aws.amazon.com/bedrock/latest/userguide/inference-profiles-create.html

[^aip-use]: Amazon Web Services. (n.d.). Use an inference profile in model invocation. *Amazon Bedrock User Guide*. Retrieved 2026-09-27, from https://docs.aws.amazon.com/bedrock/latest/userguide/inference-profiles-use.html

[^aip-support]: Amazon Web Services. (n.d.). Inference profiles support. *Amazon Bedrock User Guide*. Retrieved 2026-09-27, from https://docs.aws.amazon.com/bedrock/latest/userguide/inference-profiles-support.html

[^aip-tag]: Amazon Web Services. (2024). CreateInferenceProfile API Reference. *Amazon Bedrock API Reference*. Retrieved 2026-09-27, from https://docs.aws.amazon.com/bedrock/latest/APIReference/API_CreateInferenceProfile.html

[^aip-arch]: Amazon Web Services. (2024). Track generative AI costs with Amazon Bedrock inference profiles. *AWS Architecture Blog*. Retrieved 2026-09-27, from https://aws.amazon.com/blogs/architecture/track-generative-ai-costs-with-amazon-bedrock-inference-profiles/

[^aip-delete]: Amazon Web Services. (2024). DeleteInferenceProfile API Reference. *Amazon Bedrock API Reference*. Retrieved 2026-09-27, from https://docs.aws.amazon.com/bedrock/latest/APIReference/API_DeleteInferenceProfile.html