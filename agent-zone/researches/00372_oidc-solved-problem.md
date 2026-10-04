# OpenID Connect (OIDC) 解決了什麼問題？

## 問題背景：網頁應用的身分驗證困境

在 OIDC 誕生之前，需要「使用者登入」功能的應用程式面臨兩種選擇，各有其痛點：

**方案一：自建認證系統**。應用程式必須自己維護使用者資料庫、儲存雜湊密碼、處理密碼重設與帳號鎖定，並承擔憑證外洩的完整安全責任。這對小型開發團隊而言是沉重且高風險的負擔。

**方案二：使用第三方「Login with X」按鈕**。但 Facebook、Google、Twitter 等各平台使用私有 API，缺乏互通標準。先前的身分標準 OpenID 2.0 使用 XML 協議和自訂簽章演算法，實作困難且互通性問題不斷[^oidc-faq]。

更深層的問題在於：**OAuth 2.0（2012 年發布）是授權框架，不是認證協議**[^auth0-oauth]。OAuth 的 Access Token 只是存取資源的「鑰匙」，本身不承載標準化、可驗證的身分資訊。開發者用 OAuth 拼湊「登入」功能，但缺乏安全的標準做法，導致 Confused Deputy Attack、Session Fixation 等安全漏洞頻傳。

此外，既有的 SAML（企業級身分聯合標準）僅支援瀏覽器環境，無法服務原生行動應用與單頁應用（SPA）[^connect2id]。

## OIDC 解決的核心問題

**身分聯合驗證（Federated Identity Verification）**：讓應用程式安全地回答「誰正在使用這個瀏覽器或行動裝置？」而不必自己管理密碼。如 OpenID Foundation 所述：「OIDC 為『誰正在使用這個瀏覽器或行動裝置？』提供了安全且可驗證的答案，最佳的是它移除了設定、儲存、管理密碼的責任。」[^oidc-how]

具體而言，OIDC 解決了以下問題：

### 1. 標準化的身分斷言（Identity Assertion）

OIDC 引入 **ID Token**，這是一個經 Identity Provider（IdP/OP）簽署的 JWT（JSON Web Token），內含使用者經驗證的聲明：
- `sub` — 使用者的唯一識別碼（Subject Identifier）
- `iss` — 簽發者（Issuer，即 IdP）
- `aud` — 預期接收者（Audience，即應用程式）
- `exp` / `iat` — 有效期限與簽發時間
- `auth_time` — 認證發生的時間
- `nonce` — 防止 Token Replay 攻擊
- 可選：name、email、picture 等

ID Token 的概念等同於「標準 JWT 格式的數位身分證」，由 OpenID Provider 簽署，應用程式不需額外來回查詢即可驗證使用者身分[^connect2id]。

### 2. 聯合認證與密碼管理委派

應用程式（Relying Party, RP）將使用者認證委託給受信任的 IdP（如 Google、Microsoft 或企業 IdP）。IdP 負責：憑證驗證、MFA 多因子驗證、密碼政策管理。應用程式只需信任簽署後的 ID Token，無需接觸任何密碼[^oidc-how]。

### 3. 單一登入（Single Sign-On, SSO）

使用者一旦在 IdP 完成認證，IdP 維持其工作階段。任何啟用 OIDC 的應用程式重新導向至該 IdP 進行認證時，皆可辨識現有階段，免除重複登入。如 Microsoft 所述：「你可以使用 OIDC 透過 ID Token 在 OAuth 啟用的應用程式之間實現 SSO。」[^microsoft-oidc]

### 4. 跨平台支援

OIDC 從設計之初就支援：網頁瀏覽器、原生行動應用（iOS/Android）、桌面應用、JavaScript 單頁應用（SPA）。相比之下，SAML 僅支援瀏覽器重新導向模式[^oidc-faq]。

## OIDC 與 OAuth 2.0 的關係與區別

OIDC **不是取代 OAuth 2.0**，而是在 OAuth 2.0 之上疊加了一個身分層（Identity Layer）[^auth0-oidc]。

| 面向 | OAuth 2.0 | OpenID Connect (OIDC) |
|---|---|---|
| **目的** | **授權（Authorization）** — 授予 API 資源的存取權 | **認證（Authentication）** — 驗證使用者身分 |
| **核心 Token** | Access Token（不透明或 JWT）— 存取資源的「鑰匙」 | ID Token（必定為 JWT）— 可驗證的「身分證」 |
| **回答的問題** | 「這個應用可以存取哪些資源？」 | 「這個使用者是誰？」 |
| **標準聲明** | 無標準規範 | 標準聲明集：sub、name、email 等 |
| **使用者資訊** | 無標準端點 | **UserInfo Endpoint** — 標準化的使用者資料端點 |

## 為何建立 OIDC？（歷史脈絡）

OIDC 於 2014 年發布，填補了 OAuth 2.0 的缺口，並解決了其前身 OpenID 2.0 的不足：

1. **OAuth 2.0 不處理認證** — 開發者用 OAuth 拼湊登入功能，缺乏標準化導致安全漏洞[^auth0-oauth]。
2. **OpenID 2.0 過於複雜** — 使用 XML、自訂簽章、互通性問題頻傳：「OpenID 2.0 的實作有時會莫名其妙地無法互通。」[^oidc-faq]
3. **現代化協議需求** — OIDC 基於 JSON（非 XML）、JWT（RFC 7519）、TLS/HTTPS、RESTful API。實作難度大幅降低，互通性顯著提升[^oidc-faq]。
4. **行動應用與原生客戶端需求** — 相較於僅支援瀏覽器的 SAML，OIDC 從零開始就設計支援行動與桌面應用。

## 總結

OpenID Connect 解決了**跨平台、標準化的身分聯合驗證問題**。它在 OAuth 2.0 授權框架之上增加了可驗證的身分層，讓應用程式透過一個經簽署的 ID Token 安全地回答「這個使用者是誰？」，無需自行管理密碼、不需面對早期身分協議的複雜性，並原生支援網頁、行動與單頁應用。這是現代 SSO 與「Login with Google/Microsoft/Apple」按鈕的技術骨幹。

---

[^oidc-faq]: OpenID Foundation. (n.d.). *OpenID Connect FAQ*. Retrieved 2026-10-01, from https://openid.net/connect/faq/
[^oidc-how]: OpenID Foundation. (n.d.). *How OpenID Connect Works*. Retrieved 2026-10-01, from https://openid.net/developers/how-connect-works/
[^auth0-oauth]: Auth0. (n.d.). *OAuth 2.0 Authorization Framework*. Retrieved 2026-10-01, from https://auth0.com/docs/authenticate/protocols/oauth
[^auth0-oidc]: Auth0. (n.d.). *OpenID Connect Protocol Documentation*. Retrieved 2026-10-01, from https://auth0.com/docs/protocols/openid-connect-protocol
[^connect2id]: Connect2id. (n.d.). *OpenID Connect explained*. Retrieved 2026-10-01, from https://connect2id.com/learn/openid-connect
[^microsoft-oidc]: Microsoft. (n.d.). *OpenID Connect on the Microsoft identity platform*. Retrieved 2026-10-01, from https://learn.microsoft.com/en-us/entra/identity-platform/v2-protocols-oidc