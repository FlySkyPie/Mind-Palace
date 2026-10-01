# 在地端服務使用 AWS 服務時的鑑權處理方式

AWS 明確不建議在地端（on-premises）服務中使用長期有效的 IAM Access Key / Secret Key，因為這類長期憑證一旦外洩將造成持續性的安全風險[^sec-blog]。本文整理數種官方推薦的替代方案，並說明其適用場景與優缺點。

## AWS 不鼓勵長期 API Key 的原因

長期 IAM 使用者 Access Key 的本質是「永不過期」（除非手動輪換），這帶來幾個問題：

- 憑證外洩的影響視窗極長，攻擊者可長期使用
- 難以稽核——無法區分同一組 Key 被誰、何時使用
- 缺乏自動輪換機制，容易因人員離職或疏漏而忘記撤銷
- 無法利用 IAM Role 的 Session Policy 或 Condition Key 做細緻權限控制

AWS 官方安全最佳實務建議：**對於任何非人類（automated workload）的存取，應優先使用暫時性憑證，而非長期 Access Key**[^iam-best]。

## 方案一：IAM Roles Anywhere（最推薦）

### 運作原理

IAM Roles Anywhere 允許在地端或混合雲的工作負載使用 **X.509 憑證**（由既有的公開金鑰基礎設施 PKI 發行）向 AWS 請求暫時性的安全憑證[^ra-what]。它透過 `CreateSession` API 運作，過程類似 STS 的 `AssumeRole`——將憑證簽章交換為標準的 SigV4 相容 Session Credential[^ra-auth]。

### 核心元件

- **Trust Anchor**（信任錨點）：對憑證核發 CA（可為 AWS Private CA 或外部 CA）的參考，建立 IAM Roles Anywhere 與 PKI 之間的信任關係[^ra-concepts]。
- **Profile**：連結 Trust Anchor 與 IAM Role，可額外套用 Session Policy 限制權限。
- **IAM Role**：需有信任政策允許 `rolesanywhere.amazonaws.com` 呼叫 `sts:AssumeRole`、`sts:TagSession`、`sts:SetSourceIdentity`。
- **Credential Helper（`aws_signing_helper`）**：一個輕量級 Go 二進位檔，處理憑證簽章與憑證索取，支援三種運作模式：
  - `credential-process`：與 AWS SDK 的 `credential_process` 設定配合，自動更新。
  - `serve`：透過本地 IMDSv2 相容端點提供憑證。
  - `update`：直接寫入 `~/.aws/credentials` 檔案。

### 驗證流程

1. CA 簽發 X.509 憑證給在地端主機。
2. 主機透過 credential helper 將憑證提交給 IAM Roles Anywhere。
3. IAM Roles Anywhere 根據 Trust Anchor 驗證憑證有效性。
4. 通過驗證後，回傳暫時性 AWS 憑證（預設最長 12 小時）。
5. 主機用該憑證呼叫任何 AWS API。

### 支援的金鑰儲存方式

Plain files、OS 憑證儲存區（Windows/Mac）、PKCS#11 Token/HSM、TPM（Trusted Platform Module）包裝金鑰[^ra-key]。

### 優點

- ✅ 完全消除長期 IAM Access Key
- ✅ 憑證可綁定既有企業 PKI，重用心基建置
- ✅ 支援 Role Chaining，可跨帳號存取
- ✅ 與所有支援 `credential_process` 的 AWS SDK 相容
- ✅ 支援憑證撤銷清單（CRL）
- ✅ 可在 IAM Role 信任政策中限制憑證屬性（Subject、Issuer）
- ✅ 支援 HSM、TPM 等安全金鑰儲存
- ✅ CloudTrail 記錄 `SourceIdentity` 與憑證序號

### 缺點

- ❌ 需要 PKI 基礎設施（外部 CA 或 AWS Private CA，後者會產生費用）
- ❌ 需要管理憑證生命週期（輪換、撤銷、分發）
- ❌ 屬於區域性服務，需逐一區域設定
- ❌ 跨帳號的 Role Chaining 需要額外設定

### 適合場景

**在地端伺服器或應用程式需要程式化存取 AWS API** 時的首選方案。例如：資料庫備份至 S3、日誌匯入、CI/CD Runner、監控工具等。特別適合已擁有企業 PKI 的組織。

## 方案二：AWS Systems Manager Hybrid Activation

### 運作原理

AWS Systems Manager（原 SSM）允許將非 EC2 機器（在地端伺服器、VM、邊緣裝置）註冊為「受管節點」（Managed Node）。透過建立 Hybrid Activation 取得 Activation Code 與 ID，在機器上安裝 SSM Agent 後完成註冊，Agent 會自動管理後續的憑證輪換[^ssm-hybrid]。

### 核心特性

- 兩種註冊方式：**Hybrid Activation**（個別機器）與 **Cloud Connector**（Azure VM 大規模註冊）
- 機器獲得 `mi-` 前綴的 ID（EC2 則為 `i-`）
- 註冊後支援 Run Command、Session Manager、Patch Manager、Inventory 等功能
- 自 2026 年 6 月 30 日起：Advanced Instances Tier 已移除，不再有 1,000 台限制，按用量計費[^ssm-tier]

### 優點

- ✅ 同時提供 **管理功能**（修補、盤點、遠端指令）**與** AWS API 存取能力
- ✅ 無需 PKI 或憑證管理——SSM Agent 自動處理憑證輪換
- ✅ Session Manager 提供無 SSH Key 的安全 Shell 存取
- ✅ EC2 與在地端機器可在同一個管理介面中統一管理
- ✅ 管理操作可透過 IAM 進行精細權限控制

### 缺點

- ❌ 每台機器都必須安裝並運行 SSM Agent
- ❌ 主要設計用於 **管理與維運**，不是為了讓任意應用程式存取 AWS API
- ❌ 機器會成為「受管節點」，對某些單純只需呼叫 AWS API 的工作負載來說可能過重
- ❌ 需要對外連線至 AWS 端點

### 適合場景

當你已使用或打算使用 SSM 管理在地端伺服器（修補、盤點、遠端指令），且這些伺服器同時需要存取 AWS API。若已導入 SSM 管理，這是自然的延伸。

## 方案三：SAML 2.0 Federation

### 運作原理

設定企業身分提供者（IdP，如 ADFS、Okta、Ping、Shibboleth）產生 SAML Assertion。應用程式呼叫 `AssumeRoleWithSAML` 將 SAML Assertion 提交給 AWS STS，換回暫時性憑證[^saml]。

### 驗證流程

1. 使用者/服務向企業 IdP 驗證身分。
2. IdP 產生包含身分屬性的 SAML Assertion。
3. 應用程式呼叫 `AssumeRoleWithSAML`（非簽章 API 呼叫）攜帶 Assertion。
4. AWS 根據 IAM 中設定的 SAML Identity Provider 驗證 Assertion。
5. 回傳暫時性憑證。

### 關鍵設定

- 建立 **IAM SAML Identity Provider**——上傳 IdP 的中繼資料文件
- 建立 **IAM Role**——信任政策允許該 SAML Provider 呼叫 `sts:AssumeRoleWithSAML`
- 在 IdP 端設定 SAML 屬性對應，指定要 Assume 的 Role

### 優點

- ✅ 無長期 AWS 憑證
- ✅ 利用既有企業身分基礎設施（AD、ADFS）
- ✅ 支援透過 SAML 屬性進行細緻的屬性式存取控制
- ✅ 支援加密 SAML Assertion
- ✅ 標準協定，多數 IdP 支援

### 缺點

- ❌ 需要 SAML 2.0 相容的 IdP 基礎設施
- ❌ 服務/機器需能向 IdP 驗證（需連線至 IdP 網路）
- ❌ `AssumeRoleWithSAML` 呼叫是非簽章的——若經由不可信中介傳輸，政策可能被竄改
- ❌ 設定複雜度較高（中繼資料交換、憑證管理、Assertion 設定）
- ❌ Session 持續時間通常限制為 1 小時

### 適合場景

主要用於 **人類使用者 SSO（AWS Console 登入）**，或在地端服務 **已整合企業 IdP 驗證**（如 AD 整合應用）時。較不適合純自動化的伺服器對伺服器工作負載。

## 方案四：OIDC Federation

### 運作原理

類似 SAML 但使用 **OpenID Connect (OIDC)** 與 JWT 取代 SAML Assertion。應用程式將來自 OIDC Provider（如 Okta、Azure AD、GitHub Actions 或自訂 IdP）的 JWT Token 提交給 AWS STS，驗證後回傳暫時性憑證。

### 優點

- ✅ 無長期 AWS 憑證
- ✅ 較 SAML 輕量（JSON 格式）
- ✅ 現代標準，多數雲端 IdP 支援
- ✅ 很適合 CI/CD 場景（如 GitHub Actions OIDC）

### 缺點

- ❌ 需要 OIDC 相容的 IdP
- ❌ 機器到 IdP 的連線需求與 SAML 相同
- ❌ 需管理 JWT Token 的生命週期

### 適合場景

**CI/CD Runner**（GitHub Actions、GitLab、Jenkins 搭配 OIDC）或當企業 IdP 原生支援 OIDC 時。

## 方案五：STS AssumeRole 搭配 Bootstrap IAM User

### 運作原理

一種「過渡性」做法：在地端服務使用 **權限極小** 的 IAM User Credential 呼叫 `AssumeRole`，Assume 到一個權限較大的 Role 後使用暫時性憑證。Bootstrap IAM User 僅有 `sts:AssumeRole` 權限。

### 優點

- ✅ 實作簡單——任何 AWS SDK 皆可呼叫
- ✅ 暫時性憑證自動輪換（短生命週期）
- ✅ 可 Assume 不同 Role 來實現不同功能

### 缺點

- ❌ 機器上**仍然存在長期 IAM Access Key**（用於 Bootstrap 呼叫）
- ❌ Bootstrap Key 是單點風險——若外洩仍可 Assume Role
- ❌ 不完全符合「不使用長期憑證」的官方建議

### 適合場景

當其他方案因技術限制無法導入時，可作為**過渡或備用方案**。不建議作為主要架構。

## 方案六：AWS IAM Identity Center（原 AWS SSO）

### 運作原理

主要設計用於 **人員身分管理**（跨 AWS 帳號與應用程式的 SSO）。可透過 SAML 或 SCIM 與外部 IdP 整合。對自動化工作負載的適用性較低，但可透過 API 程式化取得暫時性憑證[^sso]。

### 優點

- ✅ 集中化管理多帳號權限
- ✅ 適合人類使用者與開發者工作站
- ✅ 可與外部 IdP 整合（Okta、Azure AD、Google Workspace）

### 缺點

- ❌ 主要設計為 **人員/使用者** 使用，非伺服器工作負載
- ❌ 較少用於自動化的無頭服務（Headless Service）

### 適合場景

**開發者工作站與人員存取**（例如開發者需從筆電 CLI 存取 AWS）。不適合自動化伺服器工作負載。

## 比較總表

| 方案 | 長期 Key？ | 需 PKI？ | 需 IdP？ | 需 Agent？ | 最適合 |
|---|---|---|---|---|---|
| **IAM Roles Anywhere** | ❌ 無 | ✅ 是 | ❌ 否 | ❌ 否（僅二進位檔） | **伺服器工作負載**的程式化存取 |
| **SSM Hybrid Activation** | ❌ 無 | ❌ 否 | ❌ 否 | ✅ SSM Agent | **伺服器管理** + AWS 存取 |
| **SAML Federation** | ❌ 無 | ❌ 否 | ✅ SAML IdP | ❌ 否 | **人類 SSO** 或 IdP 整合服務 |
| **OIDC Federation** | ❌ 無 | ❌ 否 | ✅ OIDC IdP | ❌ 否 | **CI/CD**、Web 應用 |
| **STS AssumeRole（Bootstrap）** | ⚠️ 仍有 Bootstrap Key | ❌ 否 | ❌ 否 | ❌ 否 | **過渡方案** |
| **IAM Identity Center** | ❌ 無 | ❌ 否 | ✅ 選擇性 | ❌ 否 | **人員/工作站**存取 |

## AWS 官方建議摘要

綜合 AWS 官方文件與安全部落格的指引[^sec-blog][^iam-best]：

1. **首選方案**：**IAM Roles Anywhere**——這是 AWS 為在地端自動化工作負載存取 AWS API 這一場景量身打造的解方。
2. **管理需求**：**Systems Manager Hybrid Activation**——當在地端伺服器除了存取 AWS 之外還需要維運管理（修補、遠端指令）時。
3. **人員存取**：**IAM Identity Center** 搭配 SAML/OIDC Federation，從企業 IdP 單一登入。
4. **不建議**：在地端伺服器上使用長期 IAM Access Key。若萬不得已，應使用 STS `AssumeRole` 搭配最小權限的 Bootstrap IAM User 作為最後手段。

AWS Security Blog 中〈Planning for your IAM Roles Anywhere deployment〉建議的最佳實務包括：
- 為每個應用程式建立專屬的 IAM Role（最小權限原則）
- 每個工作負載實例使用獨一無二的憑證
- 使用短效期的 End-Entity Certificate
- 啟用 CRL 撤銷機制
- 多數組織建議採用集中式 Trust Anchor 模式

## 參考資料

[^ra-what]: Amazon Web Services. (n.d.). AWS IAM Roles Anywhere. Retrieved 2026-09-27, from https://aws.amazon.com/iam/roles-anywhere/
[^ra-auth]: Amazon Web Services. (n.d.). Authentication in IAM Roles Anywhere. Retrieved 2026-09-27, from https://docs.aws.amazon.com/rolesanywhere/latest/userguide/authentication.html
[^ra-concepts]: Amazon Web Services. (n.d.). Concepts in IAM Roles Anywhere. Retrieved 2026-09-27, from https://docs.aws.amazon.com/rolesanywhere/latest/userguide/getting-started.html
[^ra-key]: Amazon Web Services. (n.d.). Credential helper for IAM Roles Anywhere. Retrieved 2026-09-27, from https://docs.aws.amazon.com/rolesanywhere/latest/userguide/credential-helper.html
[^sec-blog]: Bose, T. (2024). Planning for your IAM Roles Anywhere deployment. AWS Security Blog. Retrieved 2026-09-27, from https://aws.amazon.com/blogs/security/planning-for-your-iam-roles-anywhere-deployment/
[^ssm-hybrid]: Amazon Web Services. (n.d.). Creating a hybrid activation. AWS Systems Manager User Guide. Retrieved 2026-09-27, from https://docs.aws.amazon.com/systems-manager/latest/userguide/activations.html
[^ssm-tier]: Amazon Web Services. (n.d.). Systems Manager hybrid and multicloud environments. AWS Systems Manager User Guide. Retrieved 2026-09-27, from https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-hybrid-multicloud.html
[^saml]: Amazon Web Services. (n.d.). SAML 2.0 federation. AWS IAM User Guide. Retrieved 2026-09-27, from https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_saml.html
[^iam-best]: Amazon Web Services. (n.d.). Best practices for managing AWS access keys. AWS IAM User Guide. Retrieved 2026-09-27, from https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_request.html
[^sso]: Amazon Web Services. (n.d.). AWS IAM Identity Center. Retrieved 2026-09-27, from https://aws.amazon.com/iam/identity-center/