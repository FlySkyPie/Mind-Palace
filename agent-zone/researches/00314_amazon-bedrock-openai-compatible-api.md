# Amazon Bedrock OpenAI-Compatible API 與純 HTTP 請求研究

## 概述

本報告探討 Amazon Bedrock 是否支援 OpenAI-Compatible API，以及如何在不用 SDK 的情況下，透過純 HTTP(S) 請求呼叫 Amazon Bedrock 服務。答案為 **是**——AWS 已在 Bedrock 上原生提供 OpenAI 相容端點。

---

## 1. OpenAI-Compatible API 支援情況

Amazon Bedrock 現已原生提供 OpenAI-Compatible API，使用標準的 Chat Completions 格式與 Responses API 格式[^bedrock-endpoints]。

### 1.1 兩個端點

| 端點 | Base URL | 說明 |
|---|---|---|
| **`bedrock-runtime`** (推薦) | `https://bedrock-runtime.{region}.amazonaws.com` | 支援 OpenAI-Compatible API、Anthropic Messages API、以及 Bedrock 原生 Converse API |
| **`bedrock-mantle`** (相容層) | `https://bedrock-mantle.{region}.api.aws` | 僅支援 OpenAI-Compatible API 與 Anthropic Messages API，另有背景推論等特有功能 |

[^bedrock-endpoints]: Amazon Web Services. (n.d.). Bedrock endpoints. Retrieved 2026-10-01, from https://docs.aws.amazon.com/bedrock/latest/userguide/endpoints.html

### 1.2 OpenAI-Compatible API 路徑

| API | bedrock-runtime 路徑 | bedrock-mantle 路徑 |
|---|---|---|
| Chat Completions | `POST /openai/v1/chat/completions` | `POST /v1/chat/completions` |
| Responses API | `POST /openai/v1/responses` | `POST /v1/responses` |
| List Models | ❌ (使用 AWS `ListFoundationModels`) | `GET /v1/models` |

### 1.3 支援 OpenAI-Compatible API 的模型

許多模型支援 Chat Completions API，包括但不限於[^models-api-compat]：

- **OpenAI 系列**：GPT-5.6 Sol, GPT-6 Astra, GPT-6.1 Sol 等
- **DeepSeek**：V3.1, V3.2
- **Google**：Gemma 3
- **Mistral**：Devstral 2, Magistral, Ministral
- **xAI Grok**：4.3, 4.6, 4.7
- **Moonshot Kimi**：K3, K2.5
- **其他**：NVIDIA Nemotron, Qwen3, MiniMax M2/M2.5, Writer Palmyra, Z.AI GLM

**注意**：Anthropic Claude **不支援** OpenAI Chat Completions API，需使用 Anthropic Messages API 或 Bedrock 原生 Converse API。

[^models-api-compat]: Amazon Web Services. (n.d.). Models API compatibility. Retrieved 2026-10-01, from https://docs.aws.amazon.com/bedrock/latest/userguide/models-api-compatibility.html

---

## 2. 認證方式

呼叫 Bedrock API 有兩種認證方式：

### 2.1 方式 A：AWS Signature Version 4 (SigV4)

標準 AWS 簽章認證，使用 `AWS_ACCESS_KEY_ID` 與 `AWS_SECRET_ACCESS_KEY`。若使用臨時憑證（STS 或 assume-role），需額外加上 `x-amz-security-token` 標頭[^sigv4-examples]。

```bash
curl -X POST "https://bedrock-runtime.us-east-1.amazonaws.com/openai/v1/chat/completions" \
  -H "Content-Type: application/json" \
  --aws-sigv4 "aws:amz:us-east-1:bedrock" \
  --user "$AWS_ACCESS_KEY_ID:$AWS_SECRET_ACCESS_KEY" \
  -d '{
    "model": "openai.gpt-oss-120b-1:0",
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

[^sigv4-examples]: Amazon Web Services. (n.d.). SigV4 signing examples. Retrieved 2026-10-01, from https://github.com/aws-samples/sigv4-signing-examples

### 2.2 方式 B：Bedrock API Key (Bearer Token) — 較簡潔

直接在 AWS 管理主控臺產生 Bedrock API Key，以 Bearer Token 方式使用，無需 SigV4 簽署[^bedrock-api-keys]。

```bash
curl -X POST "https://bedrock-runtime.us-east-1.amazonaws.com/openai/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
    "model": "openai.gpt-oss-120b-1:0",
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

[^bedrock-api-keys]: Amazon Web Services. (n.d.). Bedrock API keys. Retrieved 2026-10-01, from https://docs.aws.amazon.com/bedrock/latest/userguide/api-keys.html

---

## 3. Chat Completions API 請求/回應格式

### 請求格式 (OpenAI 標準格式)

```json
{
  "model": "openai.gpt-oss-120b-1:0",
  "messages": [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Hello!"}
  ],
  "temperature": 0.7,
  "max_tokens": 1000,
  "stream": false
}
```

### 回應格式

```json
{
  "id": "chatcmpl-...",
  "object": "chat.completion",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Hello! How can I help you today?"
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 10,
    "completion_tokens": 8,
    "total_tokens": 18
  }
}
```

---

## 4. Bedrock 原生 Converse API

若不使用 OpenAI-Compatible API，也可使用 Bedrock 原生 Converse API（統一、model-agnostic 的多輪對話介面），同樣可透過純 HTTP 呼叫[^bedrock-converse]。

```bash
curl -X POST "https://bedrock-runtime.us-east-1.amazonaws.com/model/us.anthropic.claude-sonnet-4-6/converse" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $AWS_BEARER_TOKEN_BEDROCK" \
  -d '{
    "messages": [{"role": "user", "content": [{"text": "Hello"}]}]
  }'
```

[^bedrock-converse]: Amazon Web Services. (n.d.). Conversation inference (Converse API). Retrieved 2026-10-01, from https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference.html

---

## 5. deprecated 專案：bedrock-access-gateway

先前社群使用 `bedrock-access-gateway` 作為代理來提供 OpenAI 相容介面，但此專案現在已 **deprecated 且 archive（read-only）**[^bedrock-gateway]。

> This project is deprecated. Amazon Bedrock now serves OpenAI-compatible and Anthropic-compatible APIs natively.

[^bedrock-gateway]: Amazon Web Services. (n.d.). bedrock-access-gateway (deprecated). Retrieved 2026-10-01, from https://github.com/aws-samples/bedrock-access-gateway

---

## 6. 結論

1. **Amazon Bedrock 原生支援 OpenAI-Compatible API**，無需代理或閘道。
2. 使用 `bedrock-runtime` 端點（`/openai/v1/chat/completions`）為新專案最佳選擇。
3. 認證可選擇 **SigV4**（使用 AWS 憑證）或 **Bearer Token**（使用 Bedrock API Key），後者更簡潔。
4. 所有操作皆可透過純 curl 或任何 HTTP 客戶端完成，完全不需要 AWS SDK。
5. 若偏好 Bedrock 原生格式，可使用 Converse API，同樣支援純 HTTP 請求。