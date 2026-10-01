# Amazon Bedrock Per-Request Metadata Tagging

## 概述

Amazon Bedrock 的 **Per-Request Metadata Tagging**（每次請求中標記元數據）是一種可以讓你在每次 Bedrock 模型推論 API 呼叫時附加任意鍵值對（key-value pairs）的功能。與需要在 AWS 端預先建立的資源標籤不同，這些標籤是在**執行時**動態傳入的——每次 API 呼叫可以攜帶完全不同的標籤集合。元數據會記錄在模型呼叫日誌中，並以 `requestMetadata` 欄位呈現，以便後續進行查詢、過濾和彙總分析。[^aws-docs]

## 支援的 API

| API 端點 | 傳遞方式 |
|-----------|----------|
| `InvokeModel` | HTTP Header: `X-Amzn-Bedrock-Request-Metadata` |
| `InvokeModelWithResponseStream` | HTTP Header: `X-Amzn-Bedrock-Request-Metadata` |
| `Converse` | Request body 中的 `requestMetadata` 欄位 |
| `ConverseStream` | Request body 中的 `requestMetadata` 欄位 |

**不支援** `bedrock-mantle` 端點（即 OpenAI 相容端點）。[^aws-api-ref]

## 使用方法

### 1. InvokeModel — HTTP Header 方式

SigV4 簽署請求時**必須**將 `X-Amzn-Bedrock-Request-Metadata` 納入 `SignedHeaders`，否則會收到 `InvalidSignatureException`。AWS SDK 在暴露 metadata 參數時會自動處理此問題。[^aws-invoke]

```http
POST /model/anthropic.claude-3-haiku-20240307-v1:0/invoke HTTP/1.1
Content-Type: application/json
X-Amzn-Bedrock-Request-Metadata: {"team": "orchestrator", "environment": "preview-test"}

{
  "anthropic_version": "bedrock-2023-05-31",
  "max_tokens": 50,
  "messages": [{"role": "user", "content": "Say hello in one word."}]
}
```

### 2. InvokeModel — Boto3 Python SDK

自 2026 年 5 月起，Boto3 的 `invoke_model` 方法支援 `requestMetadata` 參數（型別為 `string`，需傳入 JSON 編碼的字串）：[^boto3-invoke]

```python
import boto3, json

client = boto3.client("bedrock-runtime")

response = client.invoke_model(
    modelId="anthropic.claude-3-haiku-20240307-v1:0",
    body=json.dumps({
        "anthropic_version": "bedrock-2023-05-31",
        "max_tokens": 50,
        "messages": [{"role": "user", "content": "Say hello in one word."}]
    }),
    contentType="application/json",
    accept="application/json",
    requestMetadata=json.dumps({
        "team": "orchestrator",
        "environment": "preview-test",
        "test_case": "invoke_model_sync"
    })
)
```

### 3. Converse — Boto3 Python SDK（更簡單的寫法）

對於 `converse`，`requestMetadata` 為 **dict** 參數（非字串）：[^boto3-converse]

```python
import boto3

client = boto3.client("bedrock-runtime")

response = client.converse(
    modelId="us.anthropic.claude-opus-4-8",
    messages=[{"role": "user", "content": [{"text": "Summarize this ticket."}]}],
    requestMetadata={
        "user": "alice@example.com",
        "team": "growth",
        "feature": "summarizer",
        "environment": "prod",
    },
)
```

### 4. AWS CLI

```bash
aws bedrock-runtime invoke-model \
    --model-id anthropic.claude-3-haiku-20240307-v1:0 \
    --body '{"anthropic_version": "bedrock-2023-05-31", "max_tokens": 50, "messages": [{"role": "user", "content": "Hi"}]}' \
    --content-type application/json \
    --accept application/json \
    --request-metadata '{"team":"orchestrator","environment":"preview-test"}'
```

### 5. 推薦架構模式

使用共享的客戶端包裝（client wrapper）統一注入 metadata，而非在每個呼叫點手動傳遞：[^yopa]

```python
from contextvars import ContextVar

_request_ctx: ContextVar[dict] = ContextVar("bedrock_ctx", default={})

class BedrockClient:
    def __init__(self, profile_arn: str):
        self._client = boto3.client("bedrock-runtime")
        self._profile_arn = profile_arn

    def invoke(self, messages: list, **kwargs) -> dict:
        ctx = _request_ctx.get()
        response = self._client.invoke_model(
            modelId=self._profile_arn,
            body=json.dumps({...}),
            contentType="application/json",
            accept="application/json",
            requestMetadata=json.dumps({
                "env": os.getenv("APP_ENV", "unknown"),
                **ctx,
            }),
        )
        return json.loads(response["body"].read())
```

在 FastAPI 等框架的中介層設定 context，每個請求邊界執行一次即可。

## 使用場景

| 場景 | 說明 |
|------|------|
| **團隊/組織成本歸屬** | 標記 `team`、`project`、`service` 以追蹤各單位用量 |
| **功能層級經濟效益** | 標記 `feature` 以計算某功能的每位活躍用戶成本 |
| **SaaS 多租戶計費** | 標記 `tenantId` 用於 SaaS 客戶的 LLM 用量精確計費 |
| **環境追蹤** | 標記 `env`（dev/staging/prod）區分測試與正式用量 |
| **實驗追蹤** | 標記 `experiment` 或 `prompt_version` 比較不同 prompt 的成本差異 |
| **可觀測性儀表板** | 使用 CloudWatch Logs Insights 或 Athena 對日誌資料進行任意維度的切片分析 |
| **Prompt 回歸測試** | 在 prompt 更新後確認輸入 token 數量是否發生變化 |
| **Agent 工作流程追蹤** | 標記 `task_id`、`session_id`、`workflow_step` 用於多步驟 agent 呼叫追蹤 |

## 限制與要求

### 限制條件

| 限制項 | 值 |
|--------|-----|
| 每次請求最大標籤數量 | **16** 個鍵值對 |
| Key 最大長度 | **256 字元** |
| Value 最大長度 | **256 字元** |
| 允許字元（key 與 value） | 字母數字、空格、以及 `+ - = . _ : / @` |
| Header 總長度（InvokeModel） | JSON 字串最大 **8,500 字元** |
| 超限行為 | 請求被拒絕，回傳 **validation error** |

### 必要條件

1. **必須啟用 Model Invocation Logging**（在進行呼叫的 AWS 區域中）。若日誌未開啟，請求仍會成功，但 metadata 會被無聲拋棄。[^aws-logging]
2. 日誌目的地可為 **S3**（長期低成本儲存 + Athena 查詢）和/或 **CloudWatch Logs**（即時 grep/Insights）。
3. 使用 InvokeModel 時，`X-Amzn-Bedrock-Request-Metadata` 必須包含在 SigV4 簽署的 `SignedHeaders` 中。SDK 會自動處理。
4. 建議在組織內推行**一致的 key schema 規範**，避免 key 散亂（如 `team` vs `Team` vs `team_name`）。

### 安全注意事項

- **不要在 metadata 中放入 PII 或憑證**——這些值會儲存在呼叫日誌中，並可能被任何可讀取日誌的系統存取。
- **Metadata 可偽造**——任何有 `InvokeModel` 權限的人都可以設定任意標籤。不能用於安全邊界。請使用 IAM 進行授權。
- **伺服端不強制要求**——不帶 metadata 的請求仍會成功。建議透過共用客戶端包裝來強制覆蓋率。

## 模型相容性

**Per-request metadata tagging 與模型無關**——它在 Bedrock runtime API 的 HTTP header / request body 層級運作，不特定於任何模型。支援所有透過 `bedrock-runtime` 端點提供的模型：[^aws-docs]

- Anthropic Claude（所有版本：Opus、Sonnet、Haiku）
- Meta Llama（所有版本）
- Mistral、Cohere、Amazon Titan、Stability AI、AI21 Labs
- 任何可透過 bedrock-runtime 端點使用的模型

唯一的端點限制：可在 **`bedrock-runtime`** 上使用，但**不可**在 **`bedrock-mantle`**（OpenAI 相容端點）上使用。

## 成本與計費影響

### 使用 metadata 是否有額外費用？

**沒有。** 傳遞 request metadata 不收任何額外費用——不計 API 呼叫費、token 費或附加費。[^aws-cost]

### 如何用 metadata 進行成本分析

由於 metadata 並不會直接流入 AWS 帳單，有兩種路徑可選：

| 路徑 | 精確度 | 延遲 |
|------|--------|------|
| **從日誌估算（Athena/CloudWatch）** | Token 數 × 公佈的模型費率 = **估算成本**（不反映折扣、承諾、批次價格、免費方案或預付佈署） | 近即時 |
| **將日誌與 CUR（Cost & Usage Report）結合** | 以模型/用量類型為粒度的**發票級精確成本** | 約 24 小時延遲 |

**CloudWatch Logs Insights 查詢範例：**[^aws-docs]

```
fields requestMetadata.user as user, modelId,
       input.inputTokenCount as inTokens,
       output.outputTokenCount as outTokens
| stats sum(inTokens) as totalInput,
        sum(outTokens) as totalOutput,
        count() as calls
        by user, modelId
| sort totalInput desc
```

**Athena SQL 查詢範例：**[^aws-docs]

```sql
SELECT
  request_metadata['team']     AS team,
  modelId,
  SUM(input.inputTokenCount)   AS input_tokens,
  SUM(output.outputTokenCount) AS output_tokens,
  SUM(input.inputTokenCount)  * 0.000015 AS est_input_cost,
  SUM(output.outputTokenCount) * 0.000075 AS est_output_cost
FROM bedrock_invocation_logs
GROUP BY request_metadata['team'], modelId
ORDER BY est_input_cost DESC;
```

### 與其他成本機制的搭配

官方文件和第三方顧問皆建議**結合使用多種機制**：[^binary-tech]

| 機制 | 粒度 | 回答問題 | 含計費金額？ |
|------|------|----------|-------------|
| **Application Inference Profiles** | 每個服務/應用 | "這個產品花費多少？" | ✅ 是（Cost Explorer） |
| **IAM Principal Attribution** | 每個 IAM 身份 | "這個呼叫者花費多少？" | ✅ 是（Cost Explorer） |
| **Per-Request Metadata** | 每個請求 | "這個團隊/功能/使用者花費多少？" | ❌ 僅 token 數（日誌） |

## 結論

1. **Per-request metadata** 提供了 Bedrock 中最細粒度的成本和用量歸屬能力——精確到每個 prompt。
2. 2026 年 5 月 AWS 擴展了對 `InvokeModel`/`InvokeModelWithResponseStream` 的支援（原先僅限 `Converse`/`ConverseStream`）。
3. **每次最多 16 個標籤**，每個 key/value 最多 256 字元，限定字元集——請謹慎設計 schema。
4. **免費使用**——除正常推論費用外無任何價格影響。
5. **與任何模型相容**（僅限 `bedrock-runtime` 端點）。
6. **必須啟用 Model Invocation Logging** 才能記錄 metadata。
7. **不可取代 IAM**（安全邊界）或 **Inference Profiles**（發票級計費）——三者應搭配使用。

## References

[^aws-docs]: Amazon Web Services. (n.d.). Per-request metadata tagging — Amazon Bedrock. Retrieved 2026-10-01, from https://docs.aws.amazon.com/bedrock/latest/userguide/cost-mgmt-request-metadata.html

[^aws-whatsnew]: Amazon Web Services. (2026-05). Amazon Bedrock expands support for request-level usage attribution. AWS What's New. Retrieved 2026-10-01, from https://aws.amazon.com/about-aws/whats-new/2026/05/amazon-bedrock-request-level-usage-attribution/

[^aws-api-ref]: Amazon Web Services. (n.d.). InvokeModel — Bedrock API Reference. Retrieved 2026-10-01, from https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_InvokeModel.html

[^aws-invoke]: Amazon Web Services. (n.d.). InvokeModel — AWS SDK for Python (Boto3) Reference. Retrieved 2026-10-01, from https://docs.aws.amazon.com/boto3/latest/reference/services/bedrock-runtime/client/invoke_model.html

[^boto3-invoke]: Amazon Web Services. (n.d.). invoke_model — Boto3 Documentation. Retrieved 2026-10-01, from https://docs.aws.amazon.com/boto3/latest/reference/services/bedrock-runtime/client/invoke_model.html

[^boto3-converse]: Amazon Web Services. (n.d.). converse — Boto3 Documentation. Retrieved 2026-10-01, from https://docs.aws.amazon.com/boto3/latest/reference/services/bedrock-runtime/client/converse.html

[^aws-logging]: Amazon Web Services. (n.d.). Monitor model invocation using CloudWatch Logs and Amazon S3. Retrieved 2026-10-01, from https://docs.aws.amazon.com/bedrock/latest/userguide/model-invocation-logging.html

[^aws-cost]: Amazon Web Services. (n.d.). Track usage and costs in Amazon Bedrock. Retrieved 2026-10-01, from https://docs.aws.amazon.com/bedrock/latest/userguide/cost-management.html

[^yopa]: yopa.page. (2026-06-01). Per-Request Cost Attribution on Amazon Bedrock — InvokeModel Metadata. Retrieved 2026-10-01, from https://www.yopa.page/blog/2026-06-01-bedrock-request-level-usage-attribution.html

[^binary-tech]: Binary Tech Lab. (n.d.). AWS Bedrock Cost Tracking with Application Inference Profiles and Request Metadata. Retrieved 2026-10-01, from https://binarytechlab.com/blog/aws-bedrock-cost-tracking-aip-request-metadata