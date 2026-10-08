# 反向 OAuth Proxy：為本機應用代理上游 OIDC 保護 API 的解決方案調查

## 概述

傳統 OAuth Proxy（如 oauth2-proxy）作為**正向代理**，位於使用者客戶端與本機應用之間，負責處理 OIDC 認證以**保護後端應用**。而**反向 OAuth Proxy**（Reverse OAuth Proxy）則翻轉方向：代理位於**本機應用與遠端 API 之間**，由代理負責與遠端 API 的 IdP（Identity Provider）進行 OIDC 通訊，取得並注入存取權杖，使本機應用無需實作 OIDC 即可呼叫受保護的遠端 API。

此模式亦被稱為 **Upstream OAuth Proxy**、**OIDC Sidecar**、**Outbound OAuth Proxy** 或 **OAuth Client Credentials Proxy**。

## 架構模式

### 模式 A：API Gateway Plugin（Kong、Tyk、Zuplo）

```
本機客戶端 → [API Gateway + Upstream OAuth Plugin] → 遠端 OIDC 保護 API
                                                   ↕
                                            IdP Token Endpoint
```

Gateway 向 IdP 取得機器對機器（client credentials）權杖，注入請求後轉發至上游 API。適合**伺服器對伺服器**場景。

### 模式 B：Sidecar / BFF Proxy（Wonderwall、OidcProxy.Net、lua-resty-openidc）

```
瀏覽器 → [Sidecar Proxy] → 本機應用 → (選擇性) 遠端 API
           ↕
         IdP
```

Sidecar 認證使用者，然後將使用者委託的存取權杖（access token）注入流向遠端 API 的請求。適合**使用者委託**場景，即上游 API 預期攜帶使用者身份權杖。

### 模式 C：Client Credentials Sidecar（純粹的反向 OAuth Proxy）

```
本機應用 → [Sidecar Proxy] → 遠端 OIDC 保護 API
               ↕
            IdP Token Endpoint
```

輕量代理部署於本機應用旁，向 IdP 取得機器對機器權杖，附加至對上游 API 的呼叫中。這是最純粹的「反向 OAuth Proxy」模式[^purest]。

## 現有解決方案

### 1. Kong API Gateway — Upstream OAuth Plugin（Enterprise）

Kong 的 **Upstream OAuth** Plugin 作為 OAuth 2.0 client，向遠端 IdP 取得 Client Credentials Grant 權杖並注入上游請求。若無快取權杖，Kong 會向 IdP Token Endpoint 請求新權杖，快取後透過可設定標頭（預設 `Authorization`）傳遞給上游 API。

- 支援 client_credentials、password grant
- 支援 client_secret_basic、client_secret_post、client_secret_jwt 認證
- 快取可選記憶體或 Redis，依 `expires_in` 決定有效期

配置範例[^kong-upstream]：

```yaml
plugins:
  - name: upstream-oauth
    config:
      oauth:
        token_endpoint: https://idp.example.com/oauth2/token
        grant_type: client_credentials
        client_id: $CLIENT_ID
        client_secret: $CLIENT_SECRET
        scopes: [openid, profile]
      behavior:
        upstream_access_token_header_name: X-Custom-Auth
```

### 2. Tyk API Gateway — Upstream OAuth 2.0 Authentication

Tyk 支援 Upstream OAuth，使用 **client credentials** 或 **password grant** 向認證伺服器取得權杖，注入至 `Authorization` 標頭傳遞上游[^tyk-upstream]。

```json
{
  "upstream": {
    "url": "https://upstream-api.example.com",
    "authentication": {
      "enabled": true,
      "oauth": {
        "enabled": true,
        "allowedAuthorizeTypes": ["clientCredentials"],
        "clientCredentials": {
          "tokenUrl": "http://auth-server/token",
          "clientId": "client123",
          "clientSecret": "secret123",
          "scopes": ["scope1"]
        }
      }
    }
  }
}
```

### 3. Apache APISIX — OpenID Connect Plugin

APISIX 的 `openid-connect` plugin 基於 `lua-resty-openidc` 建置，可設定 upstream 權杖注入行爲[^apisix-oidc]：

- `set_access_token_header`（預設 `true`）— 以 `X-Access-Token` 標頭轉發 Access Token
- `access_token_in_authorization_header`（預設 `false`）— 若設為 `true`，Access Token 將以 `Authorization: Bearer` 傳遞
- `set_id_token_header` — 轉發 ID Token 為 `X-ID-Token`
- `renew_access_token_on_expiry`（預設 `true`）— 使用 Refresh Token 自動更新
- 支援所有 OIDC flow：authorization code、client credentials、password、introspection、refresh token

此為目前**功能最完整的 OIDC 上游代理 Plugin**之一。

### 4. Envoy — OAuth2 Filter + JWT Auth Filter

Envoy 內建 OAuth2 HTTP filter 和 JWT authentication filter[^envoy-oauth2]：

- `forward_bearer_token` — 若為 `true`，access token 以 `BearerToken` cookie 傳遞，`Authorization` 標頭同步填入
- `forward_payload_header` — 已驗證的 JWT payload 以可設定標頭傳遞至 upstream
- `use_refresh_token` — 支援 refresh token 自動更新
- Envoy Gateway（EG）提供 Kubernetes-native CRD（`SecurityPolicy`）用於設定 OIDC

### 5. Zuplo — UpstreamOAuthClientCredentialsInboundPolicy

Zuplo 具備專用策略，支援自訂 token URL、client ID/secret、scope、audience、權杖快取與自動更新、自訂標頭名稱與 scheme、權杖取得失敗重試[^zuplo-upstream]。

### 6. lua-resty-openidc（NGINX / OpenResty Library）

NGINX 的 Lua 函式庫，可將 NGINX 變為 OIDC Relying Party。認證後取得 `res.access_token`、`res.id_token`、`res.user`，使用 `ngx.req.set_header()` 自訂注入行爲[^lua-resty-openidc]：

```lua
ngx.req.set_header("Authorization", "Bearer " .. res.access_token)
ngx.req.set_header("X-USER", res.id_token.sub)
```

最靈活但需撰寫 Lua 配置。

### 7. oauth2-proxy（部分相關）

經典正向 OAuth Proxy，主要用於保護後端應用。但其上游權杖傳遞功能可部分滿足反向需求[^oauth2-proxy-behaviour]：

- `--pass-access-token` — 以 `X-Forwarded-Access-Token` 標頭傳遞 Access Token 至 upstream
- `--pass-authorization-header` — 以 `Authorization: Bearer` 傳遞 ID Token 至 upstream
- `--set-authorization-header=true` — 設定 `Authorization` 標頭

限制：Access Token 無法原生注入 `Authorization` 標頭（僅 ID Token 可）。相關 Issue #843 要求支援 `pass_access_token_as_bearer` 尚未合併。

### 8. Wonderwall（nais/wonderwall）

以 Kubernetes sidecar 模式運作的 OIDC Relying Party，支援 Authorization Code Flow with PKCE、DPoP、RP-Initiated Logout。認證後將使用者 access token 以 `Authorization: Bearer` 附加至代理請求[^wonderwall]。

### 9. OidcProxy.Net

.NET 為基礎的 BFF（Backend for Frontend）Identity-Aware Reverse Proxy，實作 Token Handler Pattern，將 `Authorization: Bearer` 注入所有 downstream（upstream）請求[^oidcproxy-net]。

### 10. Forward Networks token-gateway

輕量 Go-based 代理，純粹實作 OAuth2 client_credentials token exchange。接受 Basic Auth 換取 OAuth2 token，再以 `Authorization: Bearer` 轉發至 upstream。記憶體內快取並支援 TTL[^token-gateway]。

## 比較總表

| 方案 | 架構 | 權杖轉發方式 | 權杖更新 | 授權條款 | 備註 |
|------|------|-------------|---------|---------|------|
| **Kong Upstream OAuth** | API Gateway Plugin | 自訂標頭（預設 `Authorization`） | 依 `expires_in` 快取更新 | Enterprise | 功能完整但為付費方案 |
| **Tyk Upstream OAuth** | API Gateway Plugin | `Authorization: Bearer` | 依 `expires_in` 更新 | Enterprise | 同上 |
| **APISIX OIDC** | API Gateway Plugin | `X-Access-Token` 或 `Authorization: Bearer`（可設定） | 自動（Refresh Token） | Apache 2.0 | **最完整開放原始碼方案** |
| **Envoy OAuth2** | Edge Proxy Filter | `Authorization` + `BearerToken` cookie | Refresh Token | Apache 2.0 | 底層設定複雜，EG 提供 CRD |
| **Zuplo Upstream OAuth** | API Gateway 策略 | `Authorization: Bearer` | 自動（快取更新） | Commercial | SaaS 形式 |
| **lua-resty-openidc** | NGINX/OpenResty Lib | 完全自訂（`ngx.req.set_header`） | Refresh Token | MIT | 最靈活，需 Lua 腳本 |
| **oauth2-proxy** | 獨立 Reverse Proxy | Access Token 在自訂標頭，ID Token 在 Authorization | Cookie Refresh | MIT | 正向代理為主要用途 |
| **Wonderwall** | Kubernetes Sidecar | `Authorization: Bearer` | Refresh Token | MIT | K8s 生態系 |
| **OidcProxy.Net** | BFF Framework | `Authorization: Bearer` | Refresh Token | LGPL-3.0 | .NET 生態系 |
| **token-gateway** | 輕量獨立 Proxy | `Authorization: Bearer` | 記憶體快取 TTL | MIT | 僅 client_credentials |

## 關鍵議題

### TLS Termination

所有上述方案均支援 TLS Termination——代理終止 HTTPS 連線，以 HTTP 轉發至本機應用，由代理統一處理憑證管理。此為典型 API Gateway / Reverse Proxy 基本功能。

### M2M（Machine-to-Machine）vs 使用者委託

反向 OAuth Proxy 的兩種主要使用情境：

- **M2M（Client Credentials）**：代理以自身身份向 IdP 請求權杖，適用於後端服務呼叫上游 API 的場景（模式 A/C）。Kong、Tyk、Zuplo 原生支援。
- **使用者委託（Authorization Code + PKCE）**：代理協助使用者完成 OIDC flow，再將使用者權杖注入上游請求（模式 B）。Wonderwall、OidcProxy.Net 原生支援。

### 權杖快取與更新

所有成熟方案均實作權杖快取機制，避免每次請求都向 IdP 重新取得權杖。APISIX 與 lua-resty-openidc 支援透過 Refresh Token 在權杖過期前自動更新，完全不影響用戶端與上游服務。

## 結論

「反向 OAuth Proxy」作為概念無單一標準名詞，但其模式在各 API Gateway 與代理方案中廣泛實作。根據需求選擇：

1. **若需要開放原始碼、功能最完整的 API Gateway 解決方案** → **Apache APISIX** 的 `openid-connect` plugin 支援最完整的 OIDC upstream 轉發能力與自動更新
2. **若已使用 NGINX 且需高度自訂** → **lua-resty-openidc** 提供完全控制
3. **若為 K8s 原生 sidecar 需求** → **Wonderwall**
4. **若偏向簡潔輕量的專用代理** → **token-gateway** 或自建小型 Go/Python 代理處理 client credentials flow

所有方案均滿足 TLS Termination 需求，使本機應用不需處理 HTTPS 亦不需實作 OIDC handler。

---

[^purest]: Reverse OAuth Proxy 的核心模式：代理負責 OIDC 通訊，本機應用僅需普通 HTTP 請求即可呼叫受 OIDC 保護的遠端 API。
[^kong-upstream]: Kong Inc. (n.d.). Upstream OAuth. Retrieved 2026-10-03, from https://developer.konghq.com/plugins/upstream-oauth/
[^tyk-upstream]: Tyk Technologies. (n.d.). Upstream Authentication — OAuth. Retrieved 2026-10-03, from https://tyk.io/docs/api-management/upstream-authentication/oauth
[^apisix-oidc]: Apache Software Foundation. (n.d.). APISIX — OpenID Connect Plugin. Retrieved 2026-10-03, from https://apisix.apache.org/docs/apisix/plugins/openid-connect/
[^envoy-oauth2]: Envoy Project. (n.d.). OAuth2 Filter. Retrieved 2026-10-03, from https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/oauth2_filter
[^zuplo-upstream]: Zuplo. (n.d.). Upstream OAuth Client Credentials Inbound Policy. Retrieved 2026-10-03, from https://zuplo.com/docs/policies/upstream-oauth-client-credentials-inbound
[^lua-resty-openidc]: Zmartzone. (n.d.). lua-resty-openidc — OpenID Connect Relying Party implementation for NGINX / OpenResty. Retrieved 2026-10-03, from https://github.com/zmartzone/lua-resty-openidc
[^oauth2-proxy-behaviour]: OAuth2 Proxy Project. (n.d.). Behaviour — OAuth2 Proxy. Retrieved 2026-10-03, from https://oauth2-proxy.github.io/oauth2-proxy/behaviour/
[^wonderwall]: Norwegian Labour and Welfare Administration (NAIS). (n.d.). wonderwall — OIDC Relying Party Sidecar. Retrieved 2026-10-03, from https://github.com/nais/wonderwall
[^oidcproxy-net]: OidcProxy.Net Contributors. (n.d.). OidcProxy.Net — Identity-Aware Reverse Proxy. Retrieved 2026-10-03, from https://github.com/oidcproxydotnet/OidcProxy.Net
[^token-gateway]: Forward Networks. (n.d.). token-gateway — Lightweight Proxy for Upstream API Authentication. Retrieved 2026-10-03, from https://github.com/forwardnetworks/token-gateway