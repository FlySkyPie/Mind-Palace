# Amazon Bedrock API 純 HTTP 請求指南（無 SDK）

## 概述

Amazon Bedrock 是一項 AWS 服務，提供透過 API 存取多種基礎模型（如 Anthropic Claude、Meta Llama、Amazon Titan 等）的能力。所有請求均需使用 **AWS Signature Version 4 (SigV4)** 進行簽署認證，無法透過傳統 API Key 直接呼叫[^aws-sigv4]。

本文件說明如何在**不依賴任何 AWS SDK** 的情況下，使用純 HTTP 請求搭配原始 HMAC-SHA256 簽署程序來呼叫 Bedrock API。

## 認證機制：AWS Signature V4

SigV4 是一種對稱式簽署協定，使用 AWS 憑證（Access Key ID + Secret Access Key）來證明身分並確保請求完整性[^aws-sigv4-procedure]。

核心原則：

- Secret Access Key **絕不直接傳送**——而是用它推導出簽署金鑰後計算簽章
- 簽章範圍限定於**特定服務 + 區域 + 日期**，無法跨域重用
- 請求時間戳與伺服器時間差距不得超過 5 分鐘（anti-replay）
- Bedrock 使用 `bedrock`（控制層）或 `bedrock-runtime`（推論）作為服務代碼[^bedrock-api-ref]

所需憑證：

- `AWS_ACCESS_KEY_ID` — 存取金鑰 ID
- `AWS_SECRET_ACCESS_KEY` — 私密存取金鑰
- `AWS_SESSION_TOKEN` — 僅在使用臨時憑證（STS/SSO）時需要

## API 端點

### 推論端點（Runtime）

基本主機名稱格式：

```
https://bedrock-runtime.{region}.amazonaws.com
```

| 操作 | HTTP 方法 | 路徑 | 說明 |
|------|-----------|------|------|
| InvokeModel | POST | `/model/{modelId}/invoke` | 單次提示推論（文字、圖片、嵌入向量） |
| InvokeModelWithResponseStream | POST | `/model/{modelId}/invoke-with-response-stream` | 串流推論 |
| Converse | POST | `/model/{modelId}/converse` | 統一對話 API（LLM 推薦） |
| ConverseStream | POST | `/model/{modelId}/converse-stream` | 串流版 Converse[^bedrock-converse] |

### 控制層端點

```
https://bedrock.{region}.amazonaws.com
```

用於 `ListFoundationModels`、`GetFoundationModel` 等操作。

### 模型 ID 格式

`{provider}.{model-name}[:{version}]`

範例[^bedrock-model-ids]：

- `anthropic.claude-3-5-sonnet-20240620-v1:0`
- `amazon.titan-text-premier-v1:0`
- `amazon.titan-embed-text-v2:0`
- `meta.llama3-8b-instruct-v1:0`
- `mistral.mistral-large-2407-v1:0`

## AWS Signature V4 簽署程序（6 步驟）

### 步驟 1：建立正規化請求（Canonical Request）

將以下元素以換行符串接：

```
<HTTPMethod>
<CanonicalURI>
<CanonicalQueryString>
<CanonicalHeaders>
<SignedHeaders>
<HashedPayload>
```

各元素說明[^aws-sigv4-elements]：

- **HTTPMethod** — 如 `POST`
- **CanonicalURI** — URI 編碼的絕對路徑（以 `/` 開頭）
- **CanonicalQueryString** — 若無查詢字串則為空字串 `""`
- **CanonicalHeaders** — 小寫標頭名稱，依字母排序，結尾各加 `\n`，值前後空白需移除
- **SignedHeaders** — 以分號分隔的標頭名稱清單（與上述相同），依字母排序
- **HashedPayload** — 請求主體的小寫十六進位 SHA-256 雜湊

### 步驟 2：雜湊正規化請求

```
HashedCanonicalRequest = Hex(SHA256Hash(CanonicalRequest))
```

### 步驟 3：建立簽署字串（String to Sign）

```
<Algorithm>\n
<RequestDateTime>\n
<CredentialScope>\n
<HashedCanonicalRequest>
```

其中[^aws-sigv4-create]：

- **Algorithm** — `AWS4-HMAC-SHA256`
- **RequestDateTime** — `X-Amz-Date` 的值（如 `20241001T120000Z`）
- **CredentialScope** — `YYYYMMDD/region/service/aws4_request`

### 步驟 4：推導簽署金鑰

一系列 HMAC-SHA256 操作：

```
kDate     = HMAC-SHA256("AWS4" + SecretAccessKey, YYYYMMDD)
kRegion   = HMAC-SHA256(kDate, region)
kService  = HMAC-SHA256(kRegion, service)
kSigning  = HMAC-SHA256(kService, "aws4_request")
```

### 步驟 5：計算簽章

```
signature = Hex(HMAC-SHA256(kSigning, StringToSign))
```

### 步驟 6：建構 Authorization 標頭

```
Authorization: AWS4-HMAC-SHA256 Credential=AKIA.../20241001/us-east-1/bedrock-runtime/aws4_request, SignedHeaders=host;x-amz-content-sha256;x-amz-date, Signature=<簽章>
```

注意：`AWS4-HMAC-SHA256` 與 `Credential=` 之間有空格[^aws-sigv4-header]。

## 請求標頭

### 必要標頭

| 標頭 | 說明 |
|------|------|
| `Host` | 端點主機名稱（如 `bedrock-runtime.us-east-1.amazonaws.com`） |
| `Content-Type` | `application/json` |
| `X-Amz-Date` | 目前 UTC 時間，ISO 8601 格式 `YYYYMMDDTHHMMSSZ` |
| `X-Amz-Content-Sha256` | 請求主體的小寫十六進位 SHA-256 |
| `Authorization` | 上述 SigV4 授權標頭 |

### 選用標頭

| 標頭 | 說明 |
|------|------|
| `X-Amz-Security-Token` | 臨時憑證階段性權杖 |
| `X-Amzn-Bedrock-Trace` | `ENABLED` / `DISABLED` / `ENABLED_FULL` |
| `X-Amzn-Bedrock-PerformanceConfig-Latency` | `standard` / `optimized` |
| `X-Amzn-Bedrock-Service-Tier` | `priority` / `default` / `flex` / `reserved`[^bedrock-opt-headers] |

## 各模型請求主體格式

不同模型供應商對 `InvokeModel` 有**不同的原生請求格式**[^bedrock-invoke-api]：

| 供應商 | 範例模型 | 請求主體格式 |
|--------|---------|-------------|
| Anthropic Claude | `anthropic.claude-3-5-sonnet-...` | `{"anthropic_version": "bedrock-2023-05-31", "max_tokens": 1024, "messages": [...]}` |
| Amazon Titan Text | `amazon.titan-text-premier-v1:0` | `{"inputText": "...", "textGenerationConfig": {...}}` |
| Amazon Titan Embeddings | `amazon.titan-embed-text-v2:0` | `{"inputText": "...", "embeddingTypes": ["float"]}` |
| Meta Llama | `meta.llama3-8b-instruct-v1:0` | `{"prompt": "...", "temperature": 0.5, "max_gen_len": 512}` |
| Mistral | `mistral.mistral-large-2407-v1:0` | `{"prompt": "...", "max_tokens": 500, "temperature": 0.2}` |
| Cohere | `cohere.command-text-v14` | `{"prompt": "...", "max_tokens": 200}` |
| AI21 Jamba | `ai21.jamba-instruct-v1:0` | `{"messages": [...], "max_tokens": 200}` |
| Stability AI | `stability.stable-diffusion-xl-v1` | `{"text_prompts": [{"text": "..."}], "cfg_scale": 10, "steps": 50}` |

使用 **Converse API** 時，請求格式在所有模型間統一，使用 `messages`、`system`、`inferenceConfig` 欄位[^bedrock-converse]。

## 完整範例

### Claude 3.5 Sonnet 文字生成（InvokeModel）

**請求**：

```http
POST /model/anthropic.claude-3-5-sonnet-20240620-v1:0/invoke HTTP/1.1
Host: bedrock-runtime.us-east-1.amazonaws.com
Content-Type: application/json
X-Amz-Date: 20241001T120000Z
X-Amz-Content-Sha256: 9c5b8b1e7c9c0b3d8f2e4c6a1b3d8f2e4c6a1b3d8f2e4c6a1b3d8f2e4c6a1b
Authorization: AWS4-HMAC-SHA256 Credential=AKIA.../20241001/us-east-1/bedrock-runtime/aws4_request, SignedHeaders=host;x-amz-content-sha256;x-amz-date, Signature=cf7a9b3c5d1e...

{
    "anthropic_version": "bedrock-2023-05-31",
    "max_tokens": 1024,
    "messages": [
        {"role": "user", "content": "Hello, how are you?"}
    ]
}
```

**回應**（200 OK）：

```json
{
    "id": "msg_01ABC...",
    "type": "message",
    "role": "assistant",
    "content": [{"type": "text", "text": "Hello! I'm doing well..."}],
    "model": "claude-3-5-sonnet-20240620",
    "stop_reason": "end_turn",
    "usage": {"input_tokens": 13, "output_tokens": 23}
}
```

### Titan 嵌入向量（Embeddings）

**請求主體**：

```json
{
    "inputText": "What are the different services that you offer?",
    "embeddingTypes": ["float"]
}
```

**端點**：`POST /model/amazon.titan-embed-text-v2:0/invoke`

**回應**：

```json
{
    "embedding": [0.025, -0.017, 0.038, ...],
    "inputTextTokenCount": 8,
    "embeddingsByType": {"float": [0.025, -0.017, 0.038, ...]}
}
```

### Converse API（LLM 推薦）

```http
POST /model/anthropic.claude-3-5-sonnet-20240620-v1:0/converse HTTP/1.1
Host: bedrock-runtime.us-east-1.amazonaws.com
Content-Type: application/json
X-Amz-Date: 20241001T120000Z
X-Amz-Content-Sha256: <sha256-hex>
Authorization: <sigv4-header>

{
    "messages": [
        {"role": "user", "content": [{"text": "Write an article about impact of high inflation to GDP"}]}
    ],
    "system": [{"text": "You are an economist"}],
    "inferenceConfig": {"maxTokens": 1000, "temperature": 0.5}
}
```

### cURL 內建 SigV4（cURL >= 7.75）

```bash
curl --user "${AWS_ACCESS_KEY_ID}:${AWS_SECRET_ACCESS_KEY}" \
     --aws-sigv4 "aws:amz:${region}:bedrock" \
     --header "content-type: application/json" \
     --data '{"inputText":"love"}' \
     "https://bedrock-runtime.${region}.amazonaws.com/model/amazon.titan-embed-text-v1/invoke"
```

**注意**：cURL 的 `--aws-sigv4` 使用服務代碼 `bedrock` 而非 `bedrock-runtime`，因為 cURL 透過主機名稱判斷實際服務。但若模型 ID 包含冒號（如 `...-v1:0`），cURL 的 SigV4 實作可能失敗，建議使用不含冒號的模型 ID[^curl-sigv4]。

## 完整 Python 實作（無 SDK）

以下為不依賴任何 AWS SDK 的完整 Python 實作，使用 `requests` 函式庫：

```python
import datetime
import hashlib
import hmac
import json
import requests

# AWS 憑證
access_key = "YOUR_ACCESS_KEY"
secret_key = "YOUR_SECRET_ACCESS_KEY"
session_token = None  # 臨時憑證時設定

# 請求設定
method = "POST"
service = "bedrock-runtime"
region = "us-east-1"
model_id = "anthropic.claude-3-5-sonnet-20240620-v1:0"
host = f"bedrock-runtime.{region}.amazonaws.com"
endpoint = f"/model/{model_id}/invoke"

# 請求主體
body = json.dumps({
    "anthropic_version": "bedrock-2023-05-31",
    "max_tokens": 1024,
    "messages": [{"role": "user", "content": "Hello!"}]
})

# 步驟 1：建立時間戳
t = datetime.datetime.utcnow()
amz_date = t.strftime("%Y%m%dT%H%M%SZ")
date_stamp = t.strftime("%Y%m%d")

# 步驟 2：雜湊主體
payload_hash = hashlib.sha256(body.encode("utf-8")).hexdigest()

# 步驟 3：建立正規化標頭（字母排序）
canonical_headers = (
    f"host:{host}\n"
    f"x-amz-content-sha256:{payload_hash}\n"
    f"x-amz-date:{amz_date}\n"
)
signed_headers = "host;x-amz-content-sha256;x-amz-date"
if session_token:
    canonical_headers += f"x-amz-security-token:{session_token}\n"
    signed_headers += ";x-amz-security-token"

# 步驟 4：建立正規化請求
canonical_request = (
    f"{method}\n"
    f"{endpoint}\n"
    f"\n"  # 空查詢字串
    f"{canonical_headers}\n"
    f"{signed_headers}\n"
    f"{payload_hash}"
)

# 步驟 5：雜湊正規化請求
hashed_canonical_request = hashlib.sha256(
    canonical_request.encode("utf-8")
).hexdigest()

# 步驟 6：建立簽署字串
algorithm = "AWS4-HMAC-SHA256"
credential_scope = f"{date_stamp}/{region}/{service}/aws4_request"
string_to_sign = (
    f"{algorithm}\n{amz_date}\n{credential_scope}\n"
    f"{hashed_canonical_request}"
)

# 步驟 7：推導簽署金鑰
def sign(key, msg):
    return hmac.new(key, msg.encode("utf-8"), hashlib.sha256).digest()

k_date = sign(f"AWS4{secret_key}".encode("utf-8"), date_stamp)
k_region = sign(k_date, region)
k_service = sign(k_region, service)
k_signing = sign(k_service, "aws4_request")

# 步驟 8：計算簽章
signature = hmac.new(
    k_signing, string_to_sign.encode("utf-8"), hashlib.sha256
).hexdigest()

# 步驟 9：建構 Authorization 標頭
authorization_header = (
    f"{algorithm} "
    f"Credential={access_key}/{credential_scope}, "
    f"SignedHeaders={signed_headers}, "
    f"Signature={signature}"
)

# 步驟 10：發送請求
headers = {
    "Host": host,
    "Content-Type": "application/json",
    "X-Amz-Date": amz_date,
    "X-Amz-Content-Sha256": payload_hash,
    "Authorization": authorization_header,
}
if session_token:
    headers["X-Amz-Security-Token"] = session_token

url = f"https://{host}{endpoint}"
response = requests.post(url, headers=headers, data=body)
print(json.dumps(response.json(), indent=2))
```

## 注意事項

1. **SigV4 時效性**：請求時間戳必須在 AWS 伺服器時間的 5 分鐘內，否則請求被拒絕
2. **cURL 冒號問題**：cURL 的 `--aws-sigv4` 對含有冒號的模型 ID 有已知錯誤，應使用不帶冒號的版本 ID 或改用其他語言實作
3. **Content-Type**：部分模型可能需要 `Accept: application/json` 標頭
4. **串流**：`invoke-with-response-stream` 回傳 SSE（Server-Sent Events）串流，每個 chunk 包含 base64 編碼的 JSON 位元組，需解碼後提取文字
5. **Converse API 優勢**：Converse 提供跨模型統一的請求格式，建議新實作優先使用
6. **憑證安全**：絕不能將 Secret Access Key 寫死在原始碼中，應透過環境變數或安全的憑證管理系統取得

## 參考資料

[^aws-sigv4]: Amazon Web Services. (n.d.). AWS Signature Version 4 for API requests. Retrieved 2026-10-01, from https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_sigv.html
[^aws-sigv4-procedure]: Amazon Web Services. (n.d.). Signing AWS API Requests. Retrieved 2026-10-01, from https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_sigv-create-signed-request.html
[^aws-sigv4-elements]: Amazon Web Services. (n.d.). Elements of an AWS API Request Signature. Retrieved 2026-10-01, from https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_sigv-signing-elements.html
[^aws-sigv4-create]: Amazon Web Services. (n.d.). Signing AWS API Requests — Create the String to Sign. Retrieved 2026-10-01, from https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_sigv-create-signed-request.html
[^aws-sigv4-header]: Amazon Web Services. (n.d.). Signing AWS API Requests — Construct the Authorization Header. Retrieved 2026-10-01, from https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_sigv-create-signed-request.html
[^bedrock-api-ref]: Amazon Web Services. (n.d.). Bedrock API Reference — Runtime. Retrieved 2026-10-01, from https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_InvokeModel.html
[^bedrock-converse]: Amazon Web Services. (n.d.). Bedrock API Reference — Converse. Retrieved 2026-10-01, from https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_Converse.html
[^bedrock-invoke-api]: Amazon Web Services. (n.d.). Bedrock Inference API — Using the Invoke API. Retrieved 2026-10-01, from https://docs.aws.amazon.com/bedrock/latest/userguide/inference-api.html
[^bedrock-model-ids]: Amazon Web Services. (n.d.). Bedrock Model IDs. Retrieved 2026-10-01, from https://docs.aws.amazon.com/bedrock/latest/userguide/model-ids.html
[^bedrock-opt-headers]: Amazon Web Services. (n.d.). Bedrock Performance Configuration Headers. Retrieved 2026-10-01, from https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_InvokeModel.html
[^curl-sigv4]: JGalego. (n.d.). Curl and AWS SigV4 for Bedrock. Retrieved 2026-10-01, from https://gist.github.com/JGalego/3dd5b4bec19544453c3031ddc4a36a3b