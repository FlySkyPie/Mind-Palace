# Amazon Bedrock Application Inference Profiles 是否支援 OpenAI-Compatible API？

## 概述

本報告調查 Amazon Bedrock 的 **Application Inference Profiles（應用推論設定檔）** 是否支援 OpenAI-Compatible API 端點。結論是：**不支援**——Application Inference Profiles 僅作用於 Bedrock 原生 API（`InvokeModel` 與 `Converse`），在 OpenAI-Compatible API（Responses API 與 Chat Completions API）上使用會直接回傳 400 錯誤。

## 什麼是 Application Inference Profiles？

Application Inference Profiles 是 Amazon Bedrock 提供的一種使用者自訂資源，用於包裝特定基礎模型（Foundation Model）並附加成本歸屬標籤（cost allocation tags），例如 `team:alice-team`、`project:my-app`。建立設定檔後，使用者可用設定檔的 ARN（Amazon Resource Name）取代模型 ID 來呼叫 API，使 AWS Cost Explorer 與 Cost and Usage Reports 能夠按團隊或專案拆分成本。[^aws-aip-doc]

主要用途：
- 成本追蹤與歸屬
- 用量監控
- 多租戶（multi-tenant）場景下的費用拆分

## Amazon Bedrock 的 OpenAI-Compatible API 端點

Amazon Bedrock 確實提供了 OpenAI-Compatible API，包含兩組端點：[^aws-endpoints]

| 端點名稱 | 範例 URL | 說明 |
|---|---|---|
| `bedrock-runtime` | `https://bedrock-runtime.{region}.amazonaws.com/openai/v1` | 建議新應用使用 |
| `bedrock-mantle` | `https://bedrock-mantle.{region}.api.aws/v1` | 相容性端點 |

兩者皆支援 **OpenAI Responses API** 與 **OpenAI Chat Completions API**，使既有 OpenAI SDK 程式碼只需更換 base URL 與 API key 即可直接使用。

## 支援性對照

| API / 端點 | 是否支援 AIP？ |
|---|---|
| `InvokeModel` / `InvokeModelWithResponseStream` | ✅ 是 |
| `Converse` / `ConverseStream` | ✅ 是 |
| OpenAI Responses API（`bedrock-runtime`） | ❌ 否（400 錯誤） |
| OpenAI Responses API（`bedrock-mantle`） | ❌ 否（400 錯誤） |
| OpenAI Chat Completions API（`bedrock-runtime`） | ❌ 否（400 錯誤） |
| OpenAI Chat Completions API（`bedrock-mantle`） | ❌ 否（400 錯誤） |

官方文件明確指出：[^aws-responses-api]

> "Application inference profiles aren't supported by the Responses and Chat Completions APIs, on either endpoint. A request to those APIs that names an application inference profile as its inference target is rejected with a 400 error."

此外，在 `bedrock-runtime` 端點上，Responses API 僅能依 IAM principal 歸屬用量，**不支援 per-request metadata tagging 與 application inference profiles**。[^aws-responses-api]

## 替代方案

若需要在 OpenAI-Compatible API 上進行成本歸屬，Amazon 建議：[^aws-responses-api]

- **Projects**（僅 `bedrock-mantle` 端點支援的 OpenAI-compatible projects）
- **IAM principal attribution**（依 IAM 身份歸屬）
- **Per-request metadata tagging**

## 結論

Amazon Bedrock 的 Application Inference Profiles **不支援 OpenAI-Compatible API 端點**。此為官方明確限制：任何在 Responses API 或 Chat Completions API 上使用 Application Inference Profiles 的請求都會被拒絕並回傳 400 錯誤。若需要使用 OpenAI-Compatible API 並同時進行成本追蹤，應改用 Projects、IAM 歸屬或 per-request metadata 等方式。

---

[^aws-aip-doc]: Amazon Web Services. (n.d.). Set up a model invocation resource using inference profiles. Retrieved 2026-10-01, from https://docs.aws.amazon.com/bedrock/latest/userguide/inference-profiles.html

[^aws-endpoints]: Amazon Web Services. (n.d.). Endpoints supported by Amazon Bedrock. Retrieved 2026-10-01, from https://docs.aws.amazon.com/bedrock/latest/userguide/endpoints.html

[^aws-responses-api]: Amazon Web Services. (n.d.). Responses API — Amazon Bedrock User Guide. Retrieved 2026-10-01, from https://docs.aws.amazon.com/bedrock/latest/userguide/inference-responses-api.html