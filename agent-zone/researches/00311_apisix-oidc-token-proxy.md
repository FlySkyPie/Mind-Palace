# Apache APISIX 作為 OIDC Token 代理：替內部微服務剝離認證

## 摘要

本研究探討 Apache APISIX 能否作為 OIDC (OpenID Connect) 反向代理，處理完整的 OIDC 認證流程（含 token rotation/refresh），並在驗證後剝除認證資訊，將內部位 API 以無保護的普通 HTTP 請求轉發給內部微服務。結論是 APISIX 的 `openid-connect` 插件完全支援此模式。

## 1. 問題定義

一個受 OIDC 保護的 API（需要 Bearer token 或完整授權碼流程），在內部微服務架構中可能不希望每個服務都處理 OIDC 協定。TLS termination proxy 可以將 HTTPS "轉換" 為 HTTP，同理，我們需要一個 **認證代理** 將受 OIDC 保護的 API "轉換" 為無保護的內部 API。

```
                 ┌─────────────┐
  (Bearer JWT)   │             │   (plain HTTP, 無 auth header)
   Client ──────▶│  APISIX     │──────▶ Internal Microservice
                 │  (OIDC      │
                 │   Proxy)    │
                 └─────────────┘
```

## 2. Apache APISIX 的 OIDC/OAuth2 插件

APISIX 有兩個主要相關插件[^oidc-plugin]：

### 2.1 `openid-connect`（主要插件）

基於 `lua-resty-openidc` 函式庫，支援：
- **授權碼流程** (Authorization Code) — 含 PKCE、PAR、DPoP
- **用戶端憑證流程** (Client Credentials) — JWT 驗證或 Token Introspection
- **密碼授權流程** (Password Grant)
- **更新 Token 流程** (Refresh Token Grant)
- **Bearer-only 模式** — 僅驗證傳入的 Bearer token，不啟動瀏覽器重新導向

### 2.2 `authz-keycloak`（Keycloak 專用）

整合 Keycloak 的 UMA 2.0 授權服務，支援資源層級的細粒度授權決策，但**僅限 Keycloak** IdP[^authz-keycloak]。

## 3. 關鍵能力：驗證後剝離認證資訊

`openid-connect` 插件會**消費並移除**傳入請求的 `Authorization: Bearer <token>` 標頭，**預設不會**將其轉發至上游。透過以下旗艦控制哪些資訊傳遞給上游[^oidc-config]：

| 配置旗艦 | 預設值 | 上游接收的標頭 | 用途 |
|---------|-------|---------------|------|
| `set_access_token_header` | `true` | `X-Access-Token` | 將 access token 傳給上游 |
| `set_id_token_header` | `true` | `X-ID-Token` | 將 base64 編碼的 ID token claims 傳給上游 |
| `set_userinfo_header` | `true` | `X-Userinfo` | 將 userinfo 資料傳給上游 |
| `set_refresh_token_header` | `false` | `X-Refresh-Token` | 將 refresh token 傳給上游 |
| `access_token_in_authorization_header` | `false` | `Authorization` | 將 token 放回 Authorization 標頭 |

**要達到完全清除認證、轉發無保護請求**，將所有旗艦設為 `false`：

```json
{
  "openid-connect": {
    "set_access_token_header": false,
    "set_id_token_header": false,
    "set_userinfo_header": false,
    "set_refresh_token_header": false
  }
}
```

這就是使用者描述的「將受 OIDC 保護的 API 轉換為無保護的正常 API」模式。

## 4. Token Rotation / Refresh 處理

### 4.1 授權碼流程（session-based，自動 refresh）

當 `bearer_only: false` 時，APISIX 在 session 中儲存 access token 與 refresh token。透過以下配置可自動處理 Token Rotation[^oidc-tutorial]：

```json
{
  "openid-connect": {
    "renew_access_token_on_expiry": true,
    "access_token_expires_leeway": 30,
    "session": {
      "secret": "your-session-secret-min-16-chars",
      "absolute_timeout": 86400
    }
  }
}
```

- `renew_access_token_on_expiry: true` — 預設即啟用。當 access token 過期但 refresh token 仍有效時，自動呼叫 IdP 的 `/token` endpoint，使用 `grant_type=refresh_token` 取得新 token。
- `access_token_expires_leeway: <seconds>` — 在 token 真正過期前觸發更新，避免競爭條件。

### 4.2 Bearer-only 模式（M2M，refresh 有限制）

**重要限制**：在 `bearer_only: true` 模式下，`renew_access_token_on_expiry` 的自動 refresh 功能**有已知問題**。當 access token 已過期，插件在看到 `exp` claim 時即拒絕請求，不會先嘗試 refresh[^apisix-issue-12020]。

解決方案：
- 使用 **Token Introspection**（每次請求向 IdP 驗證，不受本地 expiry 限制）
- 使用短效期 access token 搭配 introspection
- 用戶端自行管理 token refresh（非 APISIX 處理）

## 5. 完整架構方案

### 5.1 瀏覽器 SSO 場景（授權碼流程）

```
Browser ──▶ APISIX ──（無 session）──▶ 302 Redirect to IdP
Browser ──▶ IdP Login Page ──▶ Auth Code Callback
Browser ──▶ APISIX ──（exchange code for tokens）──▶ upstream (clean)
```

APISIX 自動處理 token refresh，上游永遠收到無 auth 的請求。

### 5.2 M2M 服務對服務場景（Bearer-only）

```
Microservice A ──（Bearer JWT）──▶ APISIX
   ├── JWT 驗證（JWKS/公鑰）或 Introspection
   ├── 移除 Authorization 標頭
   └──▶ Microservice B（無 auth 標頭）
```

### 5.3 未認證請求的處理

透過 `unauth_action` 配置可決定未認證請求的行為[^oidc-config]：

| 值 | 行為 |
|----|------|
| `"auth"` | 預設，要求認證 |
| `"pass"` | 放行未認證請求（可用於混合安全/公開路由） |
| `"deny"` | 拒絕（返回 401） |

## 6. 替代方案比較

| 方案 | 成本 | OIDC 支援 | Token Rotation | 剝離認證 |
|------|------|-----------|---------------|---------|
| **APISIX openid-connect** | 免費開源 | ✅ 完整（授權碼/用戶端憑證/introspection） | ✅ session 模式自動 refresh | ✅ 可完全清除 |
| Kong OIDC | Enterprise 版付費 | ✅ 完整 | ❌ 用戶端驅動，非自動 | ✅ |
| lua-resty-openidc (原始 NGINX) | 免費開源 | ✅ 完整（APISIX 底層即此函式庫） | ✅ 同 APISIX 能力 | ✅ 手動 Lua 控制 |
| Ory Oathkeeper | 免費開源 | ✅ | ✅ | ✅ |
| Keycloak Gatekeeper | 已棄用 | ✅ | ✅ | ✅ |

## 7. 建議

**APISIX 完全能勝任此角色**。具體建議：

1. **對瀏覽器 SSO**：使用授權碼流程（`bearer_only: false`），啟用 `renew_access_token_on_expiry`，APISIX 自動處理 token rotation。上游服務收到完全乾淨的請求。
2. **對 M2M 服務間通訊**：使用 bearer-only 模式搭配 **Token Introspection**（非僅 JWT 驗證），以規避自動 refresh 的限制。或將 token 生命週期管理留給用戶端。
3. **如需上游取得使用者身份**：僅啟用 `set_userinfo_header: true` 或 `set_id_token_header: true`，傳遞使用者資訊但不轉發原始 bearer token。
4. **考慮原始 lua-resty-openidc**：如需更大的靈活性（如自訂 refresh logic），可直接在 NGINX/OpenResty 上使用 `lua-resty-openidc`，APISIX 的底層即此函式庫。

---

[^oidc-plugin]: Apache APISIX. (n.d.). openid-connect plugin. Retrieved 2026-10-01, from https://apisix.apache.org/docs/apisix/plugins/openid-connect/
[^authz-keycloak]: Apache APISIX. (n.d.). authz-keycloak plugin. Retrieved 2026-10-01, from https://apisix.apache.org/docs/apisix/plugins/authz-keycloak/
[^oidc-config]: API7.ai. (n.d.). openid-connect configuration. Retrieved 2026-10-01, from https://docs.api7.ai/hub/openid-connect/configuration
[^oidc-tutorial]: Apache APISIX. (n.d.). Keycloak OIDC Tutorial. Retrieved 2026-10-01, from https://apisix.apache.org/docs/apisix/tutorials/keycloak-oidc/
[^apisix-issue-12020]: Apache APISIX. (n.d.). GitHub Issue #12020 — bearer_only renew_access_token_on_expiry. Retrieved 2026-10-01, from https://github.com/apache/apisix/issues/12020
[^lua-resty-openidc]: zmartzone. (n.d.). lua-resty-openidc. Retrieved 2026-10-01, from https://github.com/zmartzone/lua-resty-openidc