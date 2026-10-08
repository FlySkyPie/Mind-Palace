# 將 OIDC 保護的遠端 API 轉換為本地無保護 API 的 FOSS 解決方案

本研究調查能作為「OIDC Token Proxy / Auth Relay」的開放原始碼解決方案——即代理伺服器處理 OIDC 交握（取得與刷新 access token），讓內部客戶端以無認證方式呼叫，而代理負責向外送請求注入有效的 Bearer Token。

## 核心需求模式

```
內部服務（無認證）→ 代理（處理 OIDC 交握）→ 外部 API（收到有效的 Bearer Token）
```

這與常見的「API Gateway 幫 API 加上 OAuth2.0 保護」完全相反，目標是**卸除** OIDC 複雜性。

---

## 最佳解決方案

### 1. gogatekeeper/gatekeeper（向前簽署代理模式）— 最適合

- **倉庫**：[github.com/gogatekeeper/gatekeeper](https://github.com/gogatekeeper/gatekeeper)
- **授權**：Apache 2.0
- **語言**：Go

原本是 Keycloak Gatekeeper / Louketo Proxy，後來由社群接手改名為 Gogatekeeper[^gogatekeeper]。其 **forward-signing proxy mode** 專為此場景設計：

> 「提供服務間使用 IdP 頒發的 token 進行認證授權的機制。在此模式下，代理會自動取得 access token（為你處理 refresh 或 login），並在出請求加入 `Authorization` header。」[^forward-signing]

支援 **Password Grant** 與 **Client Credentials Grant** 兩種 token 取得方式。

**Client Credentials 範例設定：**
```yaml
--enable-forwarding=true
--forwarding-domains=projecta.svc.cluster.local
--client-id=xxxxxx
--client-secret=xxxx
--discovery-url=http://keycloak:8080/realms/master
--forwarding-grant-type=client_credentials
```

代理將針對指定的 `forwarding-domains` 發出請求，並自動注入 Bearer Token。適用於機器對機器（M2M）場景。

---

### 2. Apache APISIX（openid-connect 套件 + Client Credentials Flow）

- **網站**：[apisix.apache.org](https://apisix.apache.org/)
- **授權**：Apache 2.0
- **語言**：Lua（基於 OpenResty）

APISIX 的 `openid-connect` 套件支援 **Client Credential Flow**[^apisix-ccf]，專為機器對機器場景設計，代理負責向 IdP 取得 token。

**重要設定選項：**
- **`bearer_only: true`** — 不進行 redirect 式認證，只檢查 Bearer Token
- **`access_token_in_authorization_header: true`** — 將 access token 放入上游請求的 `Authorization: Bearer` header
- **`client_credentials` grant type** — 代理本身即為 OAuth2 confidential client
- **`unauth_action: pass`** — 允許未認證請求通過（適用於部分路由需 token 注入、部分不需的混合情境）

**設定範例：**
```json
{
  "openid-connect": {
    "client_id": "my-service",
    "client_secret": "secret",
    "discovery": "https://idp.example.com/.well-known/openid-configuration",
    "bearer_only": true,
    "access_token_in_authorization_header": true,
    "set_access_token_header": true,
    "scope": "openid api_access"
  }
}
```

APISIX 也支援 `renew_access_token_on_expiry`，可在 token 過期時自動刷新，無須手動干預[^apisix-renew]。

---

### 3. pezops/oidc-proxy（Egress 模式）— 輕量替代

- **倉庫**：[github.com/pezops/oidc-proxy](https://github.com/pezops/oidc-proxy)
- **授權**：Apache 2.0
- **語言**：Go
- **Stars**：~2（小但文件完整）

明確設計為「工作負載間（workload-to-workload / service-to-service）」認證。有兩種模式：**Egress**（主要）與 **Ingress**（次要）。

Egress 模式支援三種認證方式：
1. **Google Cloud Instance Identity** (`gcp`) — 使用 GCP VM/GKE workload identity
2. **Manual Key Signing** (`manual`) — 用 RSA/HMAC 金鑰簽署自訂 JWT identity token
3. **Static Key** (`static`) — 使用預先提供的 JWT token

**範例：**
```bash
oidc-proxy --target-url="https://external-api.example.com" \
  --audience=external-api \
  --egress-enabled \
  --egress-auth-type=manual \
  --egress-auth-manual-issuer="internal" \
  --egress-auth-manual-subject=my-service \
  --egress-auth-manual-signing-method=rs256 \
  --egress-auth-manual-signing-key="$(cat private.pem)"
```

因為味道較少、仍屬早期階段，但設計理念與您的需求高度一致。

---

## 次要／特定生態系解決方案

### 4. Spring Cloud Gateway（TokenRelay Filter）

- **文件**：[spring.io TokenRelay filter 文檔](https://docs.spring.io/spring-cloud-gateway/reference/spring-cloud-gateway-server-webmvc/filters/tokenrelay.html)
- **授權**：Apache 2.0
- **語言**：Java

TokenRelay filter 將 OAuth2 access token 轉遞至下游服務。設定 `spring.security.oauth2.client.*` 屬性能建立 `OAuth2AuthorizedClientManager` bean，使用 client credentials 取得 token 並注入上游請求。最適合已大量使用 Spring 技術棧的團隊。

### 5. lua-resty-openidc（直接對 NGINX 整合）

- **倉庫**：[github.com/zmartzone/lua-resty-openidc](https://github.com/zmartzone/lua-resty-openidc)
- **授權**：Apache 2.0
- **語言**：Lua（NGINX / OpenResty 模組）

APISIX 與 Kong 使用的底層函式庫。可直接嵌入 NGINX 設定。在認證成功後，用 `ngx.req.set_header` 將 token 注入上游請求。支援 `"pass"` 模式，允許未認證請求通過但仍為已認證請求注入 token。

### 6. OidcProxy.Net（.NET 生態系、BFF 模式）

- **倉庫**：[github.com/oidcproxydotnet/OidcProxy.Net](https://github.com/oidcproxydotnet/OidcProxy.Net)
- **授權**：LGPL-3.0
- **語言**：C#

基於 YARP 的身分感知反向代理。主要設計為 SPA/BFF 場景，但也支援 Confidential Client 模式。會 `Authorization: Bearer` 將 token 加入每一個發往下游的請求。適合 .NET 架構團隊。

---

## 對照表

| 工具 | 模式 | Token 取得方式 | Token 刷新 | 內部客戶端免認證？ | 成熟度 |
|------|------|----------------|-------------|-------------------|--------|
| **gogatekeeper/gatekeeper** | Forward-signing proxy | Password / Client Credentials | ✅ 內建 | ✅ 是 | 成熟（Go, 5.x） |
| **Apache APISIX** | API Gateway + OIDC plugin | Client Credentials / Auth Code / Introspection | ✅ `renew_access_token_on_expiry` | ✅ `unauth_action: pass` | 非常成熟 |
| **pezops/oidc-proxy** | Egress proxy | Manual signing / GCP / Static | ❌ 需手動更新 | ✅ 是 | 早期 |
| **Spring Cloud Gateway** | API Gateway + TokenRelay | Client Credentials | ✅ OAuth2 client 內建 | ✅ 是 | 成熟 |
| **lua-resty-openidc** | NGINX module | Auth Code / Bearer / Introspection | ✅ 內建 | ✅ `pass` 模式 | 成熟 |
| **OidcProxy.Net** | BFF proxy | Auth Code + PKCE | ✅ 內建 | ✅ 是（但偏使用者面向） | 成熟 |

---

## 不適用的工具（易混淆）

| 工具 | 原因 |
|------|------|
| **oauth2-proxy** | 為你的 App 加上 OIDC 保護，與本需求相反。雖有 `--pass-access-token` 只傳遞**使用者**的 token，非為內部服務取得**服務** token。 |
| **Kong（社群 OIDC plugin）** | 社群套件 `cuongntr/kong-openid-connect-plugin` 較新且維護強度不足。 |
| **Louketo Proxy（原版）** | 已於 2020/11 停止維護（EOL）。 |

---

## 建議

1. **最推薦：gogatekeeper/gatekeeper** — forward-signing proxy 模式完全對應「內部客戶端免認證，代理負責 OIDC 交握」的需求，成熟度高且專門為此設計。
2. **次推薦：Apache APISIX** — 如已使用或計劃使用 API Gateway 統一管理流量，APISIX 的 OIDC plugin 用 client credentials flow 就能達成，且功能遠不止 token proxy（rate limiting、routing、observability 等）。
3. **輕量選項：pezops/oidc-proxy** — 如果只需要一個專注單一職責的輕量二元檔，且可接受早期專案的風險。

---

[^gogatekeeper]: Gogatekeeper Community. (n.d.). *gatekeeper*. GitHub. Retrieved 2026-10-08, from https://github.com/gogatekeeper/gatekeeper
[^forward-signing]: Gogatekeeper Community. (n.d.). *gatekeeper User Guide — Forward Signing Proxy*. Retrieved 2026-10-08, from https://gogatekeeper.github.io/gatekeeper/#forward-signing-proxy
[^apisix-ccf]: Apache Software Foundation. (n.d.). *openid-connect — Client Credential Flow*. Apache APISIX Docs. Retrieved 2026-10-08, from https://apisix.apache.org/docs/apisix/plugins/openid-connect/#client-credential-flow
[^apisix-renew]: Apache Software Foundation. (n.d.). *openid-connect — Session and Token Management*. Apache APISIX Docs. Retrieved 2026-10-08, from https://apisix.apache.org/docs/apisix/plugins/openid-connect/#session-and-access-token-management