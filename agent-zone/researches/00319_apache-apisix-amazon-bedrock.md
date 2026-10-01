# Apache APISIX 能否做為 Amazon Bedrock API 的閘道？

## 概述

本報告探討 Apache APISIX 是否能夠以 API 閘道（Gateway）的角色，代理（Proxy）Amazon Bedrock 的 API 請求。研究結果顯示，**自 APISIX 3.17.0 起，APISIX 已透過內建 `ai-proxy` 外掛原生支援 Amazon Bedrock**，涵蓋認證、串流（Streaming）、模型推論端點等關鍵需求。

## 1. AWS SigV4 認證支援

APISIX 可透過以下機制原生支援 AWS Signature Version 4（SigV4）認證：

- **`ai-proxy` 外掛（建議使用）**：當設定 `provider: "bedrock"` 時，外掛自動為每個上游請求簽署 AWS SigV4，使用 `auth.aws` 下設定的憑證（包含存取金鑰、秘密金鑰，以及選擇性的臨時權杖 `session_token`）。[^ai-proxy-docs]

- **`aws-lambda` 外掛**：同樣包含 SigV4 實作，惟該外掛專門用於代理 AWS Lambda／API Gateway，不適用於 Bedrock。[^aws-lambda-docs]

- **社群外掛 `apisix-plugin-aws-auth`**（WIP）：由 Lensual 在 GitHub 上開發，但對於 Bedrock 場景而言並非必要。[^community-plugin]

## 2. 現有外掛與設定

### `ai-proxy` 外掛（核心）

自 APISIX 3.17.0 起，`ai-proxy` 外掛即內建支援 Amazon Bedrock 做為提供者（Provider）。支援以下功能：[^ai-proxy-docs]

- **Converse API**（`/model/<model>/converse`）— 標準推論
- **ConverseStream API**（`/model/<model>/converse-stream`）— 串流推論
- 自動從 AWS 區域（region）和模型 ID 建構端點 URL
- SigV4 請求簽署
- AWS 推論設定檔（Inference Profile，透過 ARN 指定）
- 完整 AWS 臨時憑證支援（access key + secret key + session token）

### `ai-proxy-multi` 外掛（選用）

延伸 `ai-proxy`，加入負載平衡、重試機制、備援（Fallback）以及健康檢查功能，可在多個 Bedrock 模型或不同 LLM 提供者之間切換。[^ai-proxy-multi]

## 3. Bedrock 特定需求處理

### 串流（Streaming）

`ai-proxy` 外掛支援串流：僅需在請求主體中加入 `"stream": true`。APISIX 會將請求路由至 Bedrock 的 ConverseStream 端點，並將 AWS EventStream 框架（`application/vnd.amazon.eventstream`）原封不動地轉送至客戶端。[^ai-proxy-streaming]

```json
{ "stream": true, "messages": [...] }
```

### 模型推論端點

外掛完整支援 Bedrock Converse API 格式，包含：
- `messages`（角色 + 內容區塊）
- `system` 提示
- `inferenceConfig`（maxTokens、temperature、topP 等）
- `options.model` 模型選擇

APISIX 會自動建構正確的 Bedrock Runtime 端點 URL。[^ai-proxy-docs]

### 認證

請求簽署完全由外掛自動處理，不需手動實作：

```json
"auth": {
  "aws": {
    "access_key_id": "YOUR_AWS_ACCESS_KEY_ID",
    "secret_access_key": "YOUR_AWS_SECRET_ACCESS_KEY",
    "session_token": "OPTIONAL_STS_TOKEN"
  }
}
```

憑證以加密形式儲存於 etcd，並可透過 keyring 機制進一步加密。[^ai-proxy-docs]

## 4. 所需外掛彙整

| 外掛 | 必要性 | 用途 |
|------|--------|------|
| **`ai-proxy`** | **必要** | 核心外掛，設定 `provider: "bedrock"`，處理 SigV4 簽署、端點建構、請求和回應轉換、串流。 |
| **`ai-proxy-multi`** | 選用 | 加入負載平衡、重試、備援、健康檢查。 |
| **`ai-request-rewrite`** | 選用 | 代理前轉換請求主體格式。 |
| **`aws-lambda`** | **不建議** | 用於 Lambda／API Gateway，非 Bedrock。 |
| **標準 APISIX 外掛** | 選用 | `rate-limiting`（節流 Bedrock 成本）、`key-auth`／`jwt-auth`（保護閘道入口）、`prometheus`／`kafka-logger`（監控與日誌）。 |

## 5. 設定快速參考

```json
{
  "uri": "/bedrock/converse",
  "methods": ["POST"],
  "plugins": {
    "ai-proxy": {
      "provider": "bedrock",
      "auth": {
        "aws": {
          "access_key_id": "YOUR_AWS_ACCESS_KEY_ID",
          "secret_access_key": "YOUR_AWS_SECRET_ACCESS_KEY"
        }
      },
      "provider_conf": {
        "region": "us-east-1"
      },
      "options": {
        "model": "anthropic.claude-3-5-sonnet-20240620-v1:0"
      },
      "timeout": 60000
    }
  }
}
```

## 6. 參考資源

官方文件與原始碼：
- `ai-proxy` 外掛文件，內含 Bedrock 代理完整範例 [^ai-proxy-docs]
- APISIX 官方「Proxy Amazon Bedrock Requests」操作指南 [^howto-guide]
- `ai-proxy` 原始碼與文件（GitHub） [^github-source]

社群的 Bedrock 搭配 APISIX 的搜尋結果：
- 目前尚未發現已公開的 APISIX + Bedrock 生產環境案例研究，但 APISIX 本身已用於多家公司的生產環境（如 Naspers、IOL、RingCentral 等）。[^apisix-users]
- AWS 官方曾發表使用 Amazon API Gateway 建立 Bedrock AI 閘道的文章，此為 AWS 原生方案而非 APISIX。

## 結論

| 問題 | 答案 |
|------|------|
| AWS SigV4 原生支援？ | ✅ **有** — `ai-proxy` 外掛自動處理。 |
| 現有外掛與設定？ | ✅ **有** — `ai-proxy` 搭配 `provider: "bedrock"` 自 APISIX 3.17.0 起即為內建功能。 |
| 串流、推論、認證？ | ✅ **完整支援** — ConverseStream、Converse API、SigV4 簽署全部涵蓋。 |
| 所需外掛？ | ✅ **僅需 `ai-proxy`**，其餘皆為選用。 |
| 實例與文件？ | ✅ 官方文件有完整範例，惟尚未發現已公開的 APISIX + Bedrock 生產案例研究。 |

**結論：Apache APISIX 完全能夠做為 Amazon Bedrock API 的 API 閘道**，並提供開箱即用的 AWS SigV4 認證、串流支援，以及完整的 Converse API 代理能力。

---

[^ai-proxy-docs]: Apache APISIX. (n.d.). ai-proxy plugin. Retrieved 2026-10-01, from https://apisix.apache.org/docs/apisix/plugins/ai-proxy/
[^aws-lambda-docs]: Apache APISIX. (n.d.). aws-lambda plugin. Retrieved 2026-10-01, from https://apisix.apache.org/docs/apisix/plugins/aws-lambda/
[^community-plugin]: Lensual. (n.d.). apisix-plugin-aws-auth. GitHub. Retrieved 2026-10-01, from https://github.com/Lensual/apisix-plugin-aws-auth
[^ai-proxy-multi]: Apache APISIX. (n.d.). ai-proxy-multi plugin. Retrieved 2026-10-01, from https://apisix.apache.org/docs/apisix/plugins/ai-proxy-multi/
[^ai-proxy-streaming]: Apache APISIX. (n.d.). ai-proxy plugin — Streaming. Retrieved 2026-10-01, from https://apisix.apache.org/docs/apisix/plugins/ai-proxy/#streaming
[^howto-guide]: API7.ai. (n.d.). Proxy Amazon Bedrock Requests. Retrieved 2026-10-01, from https://docs.api7.ai/apisix/how-to-guide/ai-gateway/proxy-amazon-bedrock-requests
[^github-source]: Apache APISIX. (n.d.). ai-proxy.md. GitHub. Retrieved 2026-10-01, from https://github.com/apache/apisix/blob/master/docs/en/latest/plugins/ai-proxy.md
[^apisix-users]: Apache APISIX. (n.d.). Users. Retrieved 2026-10-01, from https://apisix.apache.org/