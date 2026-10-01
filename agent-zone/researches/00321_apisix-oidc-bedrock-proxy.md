# Apache APISIX 作為 Amazon Bedrock API 閘道：OIDC Token 獲取 → 內部無保護 API 轉換可行性

## 研究背景

本報告探討以下企業架構：企業環境中使用外部 ID Provider（如 Okta、Azure AD、Keycloak），Amazon Bedrock API 須透過 OIDC（OpenID Connect）取得 access token 才能存取資源。目標是讓 APISIX 居中處理 OIDC Token 獲取與輪替，對後端微服務提供「無需認證」的內部 API。

---

## 1. 理解核心架構

用戶描述的場景並非「Bedrock 原生支援 OIDC」，而是企業在 Bedrock 前面架設了企業 IdP 認證層，需透過 OIDC 取得 token 後才能存取 Bedrock。

```
┌─────────────────┐     無需認證       ┌──────────────┐   OIDC Token    ┌──────────────┐
│   Microservice   │ ────────────────>  │  APISIX      │ ─────────────> │  Bedrock     │
│  (unprotected)   │                    │  Gateway     │                │  (OIDC IdP)  │
└─────────────────┘                    └──────────────┘                └──────────────┘
                                               │
                                               │  1. 向 IdP 取得 Token
                                               │  2. 管理 Token 輪替
                                               │  3. 將 Token 附加至請求
                                               ▼
                                        ┌──────────────┐
                                        │  Enterprise  │
                                        │  OIDC IdP    │
                                        │  (Okta/Azure │
                                        │   AD/Keycloak)│
                                        └──────────────┘
```

企業實現此架構的可能方式：

| 模式 | 說明 | 參考 |
|------|------|------|
| **AgentCore Identity (JWT Inbound Auth)** | AgentCore Runtime/Gateway 接受 JWT Bearer Token 取代 SigV4 | AWS AgentCore Runtime OAuth 文件[^runtime-oauth] |
| **IAM OIDC Provider + AssumeRoleWithWebIdentity** | OIDC Token 交由 STS 交換為臨時 AWS 憑證，再用 SigV4 呼叫 Bedrock | Open Bedrock Server 認證指南[^obs-auth] |
| **OIDC → AWS STS Proxy** | 企業自建代理：接受 OIDC Token，呼叫 STS 換取 AWS 臨時憑證後轉發 Bedrock | 通用 OIDC IdP 設定[^generic-oidc] |

[^runtime-oauth]: Amazon. (n.d.). AgentCore Runtime OAuth. Retrieved 2025-10-01, from https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/runtime-oauth.html
[^obs-auth]: Team Branch. (n.d.). Open Bedrock Server — AWS Authentication. Retrieved 2025-10-01, from https://open-bedrock-server.teabranch.dev/guides/aws-authentication.html
[^generic-oidc]: CDharma. (n.d.). Generic OIDC Setup for Claude with Bedrock. Retrieved 2025-10-01, from https://cdharma.github.io/claude-with-bedrock/providers/generic-oidc-setup/

---

## 2. Apache APISIX 的關鍵能力

### 2.1 `openid-connect` plugin — 驗證出入 Token

- 支援 Authorization Code Flow、Bearer Token Introspection、JWT（JWKS）驗證
- 設定 `bearer_only: true` 強制要求 Bearer Token[^oidc-plugin]
- **Token 輪替**：`renew_access_token_on_expiry`（預設 `true`）到期時自動用 refresh token 換新
- 可將 Token 轉發至 upstream：`set_access_token_header`、`access_token_in_authorization_header`

### 2.2 `ai-proxy` plugin (bedrock) — SigV4 簽章

當企業後端使用 AssumeRoleWithWebIdentity 模式（OIDC Token → STS → SigV4）時：
- `provider: "bedrock"` 自動對 Bedrock 進行 AWS SigV4 簽章
- 支援 `access_key_id`、`secret_access_key`、`session_token`[^ai-proxy]

[^oidc-plugin]: Apache. (n.d.). openid-connect plugin documentation. Retrieved 2025-10-01, from https://apisix.apache.org/docs/apisix/plugins/openid-connect/
[^ai-proxy]: Apache. (n.d.). ai-proxy plugin documentation. Retrieved 2025-10-01, from https://apisix.apache.org/docs/apisix/plugins/ai-proxy/

---

## 3. 關鍵限制：APISIX 原生不支援「主動獲取 OIDC Token」

**這是本架構的核心發現。** APISIX 的 `openid-connect` plugin **只驗證客戶端出示的 Token，不會主動向 IdP 請求 Token**。審查 plugin 原始碼[^oidc-source]與 `lua-resty-openidc` 函式庫[^resty-openidc]後確認：

| 能力 | APISIX 原生支援？ | 說明 |
|------|:---:|------|
| 驗證客戶端出示的 Bearer Token | ✅ `bearer_only: true` + 自選 introspection/JWKS |
| 將 token 轉發至 upstream | ✅ `set_access_token_header` / `access_token_in_authorization_header` |
| 處理 refresh token 輪替 | ✅ `renew_access_token_on_expiry: true` |
| 主動向 IdP Token Endpoint 請求 Token | ❌ **無內建支援** |
| 以 `client_credentials` grant type 獲取 Token | ❌ 需自訂實作 |
| 快取 Token 並管理生命週期 | ❌ 需自訂實作 |

[^oidc-source]: Apache. (n.d.). openid-connect.lua (plugin source code, lines 1106-1114, 1237). Retrieved 2025-10-01, from https://raw.githubusercontent.com/apache/apisix/release/3.19/apisix/plugins/openid-connect.lua
[^resty-openidc]: ZmartZone. (n.d.). lua-resty-openidc library. Retrieved 2025-10-01, from https://github.com/zmartzone/lua-resty-openidc

---

## 4. 可行方案

### 方案 A ─ 自訂 APISIX Plugin（推薦）

撰寫自訂 Lua Plugin，在 `access` phase 中：
1. 檢查快取中是否有未過期 Token
2. 若無，向 IdP Token Endpoint 以 `client_credentials` grant 取得 Token
3. 將 Token 以 TTL 快取於 `ngx.shared.DICT`
4. 到期前提前更新
5. 將 Token 注入 `Authorization` header

**所需 APISIX 模組**：

```
lua_shared_dict oidc_token_cache 10m;   # Token 快取
```

**偽代碼結構**：

```lua
-- 1. 檢查快取
local cached = oidc_token_cache:get(cache_key)
if cached then
    -- 注入快取的 Token
    core.request.set_header(ctx, "Authorization", "Bearer " .. cached)
    return
end

-- 2. 向 IdP 取得新 Token
local res = httpc:request_uri(token_endpoint, {
    method = "POST",
    body = "grant_type=client_credentials&client_id=...&client_secret=...&scope=...",
})

-- 3. 解析 response，取得 access_token 與 expires_in
local data = core.json.decode(res.body)

-- 4. 快取 Token（提前 30 秒過期以預留緩衝）
oidc_token_cache:set(cache_key, data.access_token, data.expires_in - 30)

-- 5. 注入 header
core.request.set_header(ctx, "Authorization", "Bearer " .. data.access_token)
```

**優點**：完全控制 Token 生命週期、可在同一 plugin 內處理 refresh logic  
**缺點**：需撰寫 Lua code（約 60-100 行）

### 方案 B ─ serverless-pre-function

使用 `serverless-pre-function` plugin 在 `rewrite` phase 執行 Lua 擷取 Token。

```json
{
  "plugins": {
    "serverless-pre-function": {
      "phase": "rewrite",
      "functions": ["return function(conf, ctx) ... end"]
    },
    "proxy-rewrite": {
      "headers": {
        "set": {}
      }
    }
  }
}
```

**限制**：Lua 字串被 JSON 編碼，不適合複雜邏輯；建議用於 PoC 而非生產。

### 方案 C ─ 外部 Token Service + forward-auth

建置一個輕量 Token Service（Node.js/Python/Go），負責：
1. 向 IdP 取得 OIDC Token（`client_credentials` grant）
2. 管理 Token 輪替與快取
3. 提供 `/get-token` endpoint

APISIX 使用 `forward-auth` plugin 呼叫此 Service：

```json
{
  "forward-auth": {
    "uri": "http://token-service:8000/get-token",
    "request_method": "GET",
    "upstream_headers": ["Authorization"],
    "client_headers": ["Authorization"]
  }
}
```

若 Token Service 在 response header 回傳 `Authorization: Bearer <token>`，`forward-auth` 可將此 header 傳遞至 upstream。[^forward-auth]

**優點**：不需自訂 APISIX Lua，使用通用語言開發  
**缺點**：多一跳網路延遲；Token Service 需高可用

[^forward-auth]: Apache. (n.d.). forward-auth plugin documentation. Retrieved 2025-10-01, from https://apisix.apache.org/docs/apisix/plugins/forward-auth/

### 方案 D ─ 企業 IdP Plugin 生態

特定 IdP 可能有專屬整合：

| IdP | APISIX 整合方式 | 備註 |
|-----|----------------|------|
| **Keycloak** | `authz-keycloak` plugin[^authz-keycloak] | 但主要用於 inbound，非 upstream token acquisition |
| **Okta** | 一般 OIDC `openid-connect` | 無專屬 plugin |
| **Azure AD** | 一般 OIDC `openid-connect` | 無專屬 plugin |
| **Auth0** | 一般 OIDC `openid-connect` | 無專屬 plugin |

[^authz-keycloak]: Apache. (n.d.). authz-keycloak plugin. Retrieved 2025-10-01, from https://apisix.apache.org/docs/apisix/plugins/authz-keycloak/

---

## 5. Token 輪替（Rotation）實作考量

若使用方案 A 自訂 Plugin，Token 輪替邏輯：

```lua
-- Token 到期前 60 秒即更新（leeway）
local leeway = 60
local expires_at = data.expires_in - leeway

-- 寫入快取
oidc_token_cache:set(cache_key, data.access_token, expires_at)

-- 非同步背景更新（可選）：提前續約
-- 可在每次 request 中檢查剩餘時間，若接近到期則非同步更新
if remaining_ttl < 30 then
    -- 背景發起 refresh 請求
end
```

若使用 `lua-resty-openidc` 的 `openidc_cache_set` 模式，快取支援 `discovery`、`jwks`、`introspection`、`jwt_verification` 四種 shared dict[^resty-openidc]，可在自訂 plugin 中重用。

---

## 6. 安全注意事項

| 風險 | 因應措施 |
|------|----------|
| Token 洩漏 | 使用 `bearer_only: true` 模式避免 Token 被記錄；限內部網路存取 |
| 憑證硬編碼 | 使用環境變數（`$ENV://...`）或 `secret/aws.lua` 從 AWS Secrets Manager 擷取 |
| IDP Token Endpoint 可用性 | 實作快取降級：快取過期但 IdP 不可用時，使用現有 Token |
| 微服務偽造身份 | 內部 Route 搭配 `ip-restriction` 僅允許內部 IP；或 mTLS |

---

## 7. 結論

**可以實現，但需自訂開發。** APISIX 原生不支援主動獲取 OIDC Token，但具備所有建構模塊：

| 需求 | 可行性 | 實作方式 |
|------|:------:|----------|
| 後端微服務無需認證 | ✅ | 內部 Route 搭配 IP whitelist |
| APISIX 向 IdP 取得 OIDC Token | ✅ 需自訂 | 自訂 Lua Plugin（方案 A）為最佳實踐 |
| Token 輪替（rotation） | ✅ 需自訂 | 自訂 Lua Plugin 管理 TTL + leeway |
| 轉發請求至 Bedrock | ✅ | `ai-proxy`（SigV4）或 `proxy-rewrite`（Bearer Token） |
| 無自訂程式碼 | ❌ | `openid-connect` 只驗證不出示；此場景必定需自訂 |

**推薦路徑**：方案 A（自訂 Lua Plugin，約 60-100 行）或方案 C（外部 Token Service，使用熟悉語言開發）。方案 A 效能最佳（零網路開銷），方案 C 開發門檻最低。

---

## 參考資料

- Apache. (n.d.). openid-connect plugin. Retrieved 2025-10-01, from https://apisix.apache.org/docs/apisix/plugins/openid-connect/
- Apache. (n.d.). ai-proxy plugin. Retrieved 2025-10-01, from https://apisix.apache.org/docs/apisix/plugins/ai-proxy/
- Apache. (n.d.). forward-auth plugin. Retrieved 2025-10-01, from https://apisix.apache.org/docs/apisix/plugins/forward-auth/
- Apache. (n.d.). serverless-pre-function plugin. Retrieved 2025-10-01, from https://apisix.apache.org/docs/apisix/plugins/serverless/
- Apache. (n.d.). Keycloak OIDC Tutorial — Client Credentials Grant. Retrieved 2025-10-01, from https://apisix.apache.org/docs/apisix/tutorials/keycloak-oidc/
- Apache. (n.d.). Plugin Developer Guide. Retrieved 2025-10-01, from https://apisix.apache.org/docs/apisix/plugin-develop/
- API7.ai. (n.d.). Create a Custom Plugin in Lua. Retrieved 2025-10-01, from https://docs.api7.ai/apisix/how-to-guide/custom-plugins/create-plugin-in-lua
- Apache. (n.d.). openid-connect.lua (source code). Retrieved 2025-10-01, from https://raw.githubusercontent.com/apache/apisix/release/3.19/apisix/plugins/openid-connect.lua
- ZmartZone. (n.d.). lua-resty-openidc library. Retrieved 2025-10-01, from https://github.com/zmartzone/lua-resty-openidc
- Amazon. (n.d.). AgentCore Identity — Runtime OAuth. Retrieved 2025-10-01, from https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/runtime-oauth.html
- Team Branch. (n.d.). Open Bedrock Server — AWS Authentication. Retrieved 2025-10-01, from https://open-bedrock-server.teabranch.dev/guides/aws-authentication.html
- CDharma. (n.d.). Generic OIDC Setup for Claude with Bedrock. Retrieved 2025-10-01, from https://cdharma.github.io/claude-with-bedrock/providers/generic-oidc-setup/