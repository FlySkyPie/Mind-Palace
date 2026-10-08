# Upstream OAuth Proxy (IDC Sidecar / Outbound OAuth Proxy / OAuth Client Credentials Proxy) FOSS 解決方案研究

## 概述

本報告探討微服務架構中，內部服務需要呼叫外部 API 時，由一個 **sidecar 代理** 統一處理 OAuth2 Client Credentials Grant 流程（又稱 Outbound OAuth / Upstream OAuth / Client Credentials Proxy）的開源解決方案。此代理位於內部微服務與外部 API 之間，負責取得、快取、自動更新 access token，並注入至上游請求中，使內部服務完全無需管理 token、refresh token 或 client secret。

## 研究發現

以下為經由 Web Search 及 GitHub 調查所得之相關 FOSS（Free and Open Source）專案。

### 1. `oauth2-client-credentials-api-sidecar` ✅ **最直接符合**

| 項目 | 內容 |
|---|---|
| 名稱 | OAuth2 Client Credentials API Sidecar Container |
| 倉儲 | [surfkansas/oauth2-client-credentials-api-sidecar](https://github.com/surfkansas/oauth2-client-credentials-api-sidecar) |
| 語言 | Go |
| 授權 | MIT |
| GitHub Stars | 7 |
| 最後更新 | 2023 |

**說明**：專為 OAuth2 Client Credentials Grant 設計的 sidecar 代理容器。啟動時從 OAuth2 token endpoint 取得 access token，並在背景每 5 分鐘（到期前）自動更新。當客戶端應用程式向此代理發出 HTTP 請求時，代理會將請求轉送至真實 HTTPS API，並注入 `Authorization: Bearer <token>`（可選 `x-api-key` header）。

**關鍵特性**：
- 純 OAuth2 Client Credentials 流程（outbound）
- 背景自動更新 token（到期前 5 分鐘）
- 環境變數組態，不需修改應用程式程式碼
- 設計為 Docker sidecar 容器
- 簡單架構：client app → (未認證 HTTP) → sidecar proxy → (Bearer Token) → 外部 API

---

### 2. `openshift/oauth-proxy` ⚠️ **部分相關**

| 項目 | 內容 |
|---|---|
| 名稱 | OpenShift OAuth Proxy |
| 倉儲 | [openshift/oauth-proxy](https://github.com/openshift/oauth-proxy) |
| 語言 | Go |
| 授權 | MIT |
| GitHub Stars | 286 |
| 最後更新 | 2024 |

**說明**：一個反向代理與靜態檔案伺服器，透過 OpenShift OAuth / Kubernetes service account 提供認證。主要作為 Kubernetes Pod 中的 sidecar 容器。主要用途是 **inbound** 認證（保護應用免受未授權使用者存取），但具備 `--pass-user-bearer-token` 及 `--openshift-delegate-urls` 等 bearer token 委派功能，可將 client 的 bearer token 轉傳至 upstream。

**關鍵特性**：
- Kubernetes Pod sidecar 模式
- 零組態 OAuth（OpenShift 環境）
- RBAC 授權檢查
- 多 upstream 支援
- HMAC 請求簽章

> ⚠️ 主要為 *inbound* 認證代理，但具備 token 轉寄功能，可部分滿足 outbound 需求。

---

### 3. `kong-plugin-upstream-oauth2` (enioka) ✅ **直接符合（Kong 2.x）**

| 項目 | 內容 |
|---|---|
| 名稱 | kong-plugin-upstream-oauth2 |
| 倉儲 | [enioka-Haute-Couture/kong-plugin-upstream-oauth2](https://github.com/enioka-Haute-Couture/kong-plugin-upstream-oauth2) |
| 語言 | Lua |
| 授權 | MIT |
| GitHub Stars | 5 |
| 相容性 | Kong 2.x 限定 |

**說明**：Kong API Gateway 的社群 plugin，專門與上游服務協商 OAuth2 Client Credentials token。取得 token 後快取並於到期前自動更新，注入至 upstream 請求。

**關鍵特性**：
- Client Credentials Grant 流程
- Token 快取與自動更新
- LuaRocks 可安裝

> ⚠️ 僅相容於 Kong 2.x，未更新至 Kong 3.x。

---

### 4. `kong-oauth2-client-plugin` (KleberMotta) ✅ **直接符合（Kong 3.x）**

| 項目 | 內容 |
|---|---|
| 名稱 | Kong OAuth2 Client Plugin |
| 倉儲 | [KleberMotta/kong-oauth2-client-plugin](https://github.com/KleberMotta/kong-oauth2-client-plugin) |
| 語言 | Lua |
| 授權 | 未指定 |
| GitHub Stars | 0 |
| 相容性 | Kong 3.4+ |

**說明**：為 Kong 3.4 重新撰寫的 plugin，取得 OAuth2 access token（Client Credentials）後注入至 upstream 請求。具備完整的 token 快取、到期前更新、收到 upstream 401 後清除快取等機制。支援 `client_secret_post` 與 `client_secret_basic` 兩種認證方式。

**關鍵特性**：
- Kong 3.4 相容
- Token 快取（可設定 TTL 與到期緩衝時間）
- 收到 upstream 401 自動清除快取並重試
- 可設定的 upstream header 名稱
- 完整的 docker-compose 測試環境

---

### 5. `oauth2-proxy/oauth2-proxy` ⚠️ **部分相關（成熟專案）**

| 項目 | 內容 |
|---|---|
| 名稱 | OAuth2 Proxy |
| 倉儲 | [oauth2-proxy/oauth2-proxy](https://github.com/oauth2-proxy/oauth2-proxy) |
| 語言 | Go |
| 授權 | MIT |
| GitHub Stars | 15,100 |
| 專案狀態 | CNCF Sandbox，極活躍 |

**說明**：一個成熟、通用的反向代理，主要處理 **inbound** OAuth2 / OIDC 認證流程（Authorization Code Grant）。可部署為 Kubernetes sidecar。雖然主要用途非 outbound CC proxy，但支援 `--pass-access-token`、`--pass-authorization-header`、`--skip-jwt-bearer-tokens` 等功能，可在 sidecar 模式下轉送 token 至 upstream。

**關鍵特性**：
- 支援 Google、Azure AD、GitHub、OIDC 等多種 provider
- Kubernetes sidecar 部署模式
- Redis-backed session store
- Nginx `auth_request` 整合
- JWT bearer token 驗證
- 夜間 Docker image

> ⚠️ 主要為 inbound 認證，但 sidecar 模式加 token 轉寄功能可部分滿足需求。如需同時處理 inbound auth 與 token relay，此為最佳選擇。

---

### 6. Envoy OAuth2 Filter ⚠️ **部分相關**

| 項目 | 內容 |
|---|---|
| 名稱 | Envoy OAuth2 Authentication Filter |
| 文件 | [Envoy OAuth2 Filter Docs](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/oauth2_filter) |
| 語言 | C++ |
| 授權 | Apache 2.0 |
| 附屬於 | Envoy Proxy（~25k stars） |

**說明**：Envoy Proxy 內建的 OAuth2 filter，實作 Authorization Code flow。具備 `forward_bearer_token` 設定，可將 bearer token 轉送至 upstream。服務 mesh 環境（如 Istio）中可使用 `EnvoyFilter` 整合。

**關鍵特性**：
- Envoy 原生內建，不需外部依賴
- `forward_bearer_token` 支援 token 轉送
- HMAC-signed session cookie
- Istio 整合

> ⚠️ 同樣主要為 inbound 認證，但其 token forwarding 功能在 service mesh 場景中可扮演 token relay 角色。

---

### 比較表格

| 專案 | 類型 | 語言 | 授權 | Stars | 直接符合？ |
|---|---|---|---|---|---|
| **oauth2-client-credentials-api-sidecar** | 純 Outbound CC sidecar 代理 | Go | MIT | 7 | ✅ **最直接符合** |
| openshift/oauth-proxy | Inbound auth sidecar | Go | MIT | 286 | ⚠️ 部分（bearer 委派） |
| kong-plugin-upstream-oauth2 | Kong plugin (Kong 2.x) | Lua | MIT | 5 | ✅ 直接（僅 2.x） |
| kong-oauth2-client-plugin | Kong plugin (Kong 3.4+) | Lua | — | 0 | ✅ 直接 |
| oauth2-proxy/oauth2-proxy | 通用 inbound auth proxy | Go | MIT | 15.1k | ⚠️ 部分（sidecar + token 轉寄） |
| Kong Upstream OAuth (Enterprise) | Kong Enterprise plugin | Lua | **Proprietary** | N/A | ❌ 非 FOSS |
| Envoy OAuth2 Filter | Envoy 內建 filter | C++ | Apache 2.0 | ~25k | ⚠️ 部分（token 轉寄） |

---

### 其他搜尋結果

以下名稱未找到對應專案：
- **ORO (OAuth Resource Owner proxy)** — 無對應專案
- **token-proxy** — 無相關專案
- **sidecar-oauth** — 無相關專案
- **outbound-oauth-proxy** — 無以此為名的獨立產品

---

## 結論

針對 **Outbound OAuth Proxy / OAuth Client Credentials Proxy** 場景，最直接的 FOSS 解決方案為：

1. **`surfkansas/oauth2-client-credentials-api-sidecar`**（獨立 sidecar，輕量 Go，MIT 授權，純 Client Credentials 流程）— 適合不需要 API Gateway 的架構。
2. **`KleberMotta/kong-oauth2-client-plugin`**（Kong 3.4+ 的社群 plugin）— 適合已使用 Kong API Gateway 的架構。
3. **`oauth2-proxy/oauth2-proxy`**（成熟專案，sidecar 模式 + token 轉寄）— 適合需要同時處理 inbound auth 與 token relay 的複雜場景。

若架構中已使用 **Envoy** 或 **Kong**，則優先考慮其對應的整合方案；若僅需一個專注於 OAuth2 Client Credentials 的輕量 sidecar，`oauth2-client-credentials-api-sidecar` 是最佳選擇。

---

## 參考文獻

- surfkansas. (n.d.). OAuth2 Client Credentials API Sidecar Container. Retrieved 2026-10-03, from https://github.com/surfkansas/oauth2-client-credentials-api-sidecar [^sc]
- OpenShift. (n.d.). OpenShift OAuth Proxy. Retrieved 2026-10-03, from https://github.com/openshift/oauth-proxy [^oscp]
- enioka-Haute-Couture. (n.d.). kong-plugin-upstream-oauth2. Retrieved 2026-10-03, from https://github.com/enioka-Haute-Couture/kong-plugin-upstream-oauth2 [^ehc]
- KleberMotta. (n.d.). Kong OAuth2 Client Plugin. Retrieved 2026-10-03, from https://github.com/KleberMotta/kong-oauth2-client-plugin [^km]
- OAuth2 Proxy. (n.d.). oauth2-proxy/oauth2-proxy. Retrieved 2026-10-03, from https://github.com/oauth2-proxy/oauth2-proxy [^op]
- Kong Inc. (n.d.). Kong Upstream OAuth Plugin. Retrieved 2026-10-03, from https://developer.konghq.com/plugins/upstream-oauth/ [^kong]
- Envoy Project. (n.d.). Envoy OAuth2 Authentication Filter. Retrieved 2026-10-03, from https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/oauth2_filter [^envoy]

[^sc]: surfkansas. (n.d.). OAuth2 Client Credentials API Sidecar Container. Retrieved 2026-10-03, from https://github.com/surfkansas/oauth2-client-credentials-api-sidecar
[^oscp]: OpenShift. (n.d.). OpenShift OAuth Proxy. Retrieved 2026-10-03, from https://github.com/openshift/oauth-proxy
[^ehc]: enioka-Haute-Couture. (n.d.). kong-plugin-upstream-oauth2. Retrieved 2026-10-03, from https://github.com/enioka-Haute-Couture/kong-plugin-upstream-oauth2
[^km]: KleberMotta. (n.d.). Kong OAuth2 Client Plugin. Retrieved 2026-10-03, from https://github.com/KleberMotta/kong-oauth2-client-plugin
[^op]: OAuth2 Proxy. (n.d.). oauth2-proxy/oauth2-proxy. Retrieved 2026-10-03, from https://github.com/oauth2-proxy/oauth2-proxy
[^kong]: Kong Inc. (n.d.). Kong Upstream OAuth Plugin. Retrieved 2026-10-03, from https://developer.konghq.com/plugins/upstream-oauth/
[^envoy]: Envoy Project. (n.d.). Envoy OAuth2 Authentication Filter. Retrieved 2026-10-03, from https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/oauth2_filter