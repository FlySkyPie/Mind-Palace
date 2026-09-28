# FOSS 自託管密碼管理解決方案調查

## 概述

本報告調查市面上可用的自由開源（FOSS）自託管密碼管理解決方案，涵蓋從輕量級個人工具到企業級團隊方案，協助使用者依據自身需求選擇合適的軟體。

## 解決方案總覽

### 1. Vaultwarden

Vaultwarden[^vaultwarden] 是以 Rust 語言實作的 Bitwarden 伺服器 API 非官方重新實作，完全相容所有 Bitwarden 官方客戶端（瀏覽器擴充功能、行動應用、桌面應用、CLI）。

- **授權條款**：AGPL-3.0
- **技術棧**：Rust，單一 Docker 容器，支援 SQLite / MySQL / PostgreSQL
- **最低需求**：閒置約 50 MB RAM，可運行於 Raspberry Pi 3B+
- **特色功能**：
  - 相容所有 Bitwarden 官方客戶端（瀏覽器擴充功能覆蓋 Chrome、Firefox、Edge、Safari、Opera）
  - 解鎖 Bitwarden Premium 功能（TOTP、附件、Bitwarden Send）無需付費
  - 支援組織（Organization）與集合（Collection）分享功能，無需訂閱
  - REST API 完整相容
  - 部署簡易：`docker compose up -d` 十分鐘內完成

**優點**：極輕量、零遷移成本（直接使用 Bitwarden 客戶端）、社群活躍（GitHub 約 68K+ stars）。

**缺點**：無廠商支援、無第三方安全審計報告、FIDO2/passkey 支援進度略落後官方版本、不適合 20 人以上規模。

> **適用對象**：個人、家庭、小型團隊（20 人以下），追求完整 Bitwarden 體驗但硬體資源有限者。

### 2. Bitwarden（官方自託管版）

Bitwarden[^bitwarden] 由 Bitwarden Inc. 開發的官方開源密碼管理方案，提供自託管部署選項。

- **授權條款**：AGPL-3.0（伺服器端）、GPL-3.0（客戶端）；部分企業功能為專屬授權
- **技術棧**：C#（.NET），Docker Compose 多服務架構，支援 SQL Server 或 PostgreSQL
- **最低需求**：約 2 GB RAM，Docker Compose 6+ 容器
- **特色功能**：
  - 完整企業功能：SSO（SAML 2.0、OIDC）、SCIM 帳號佈建、目錄同步、審計事件日誌
  - Secrets Manager：支援 CI/CD 機器身份憑證管理
  - 年度第三方安全審計（Cure53 等）公開可用

**優點**：廠商支援、SLA 可用、安全審計公開、企業功能完整。

**缺點**：資源消耗高、SSO/SCIM 需要企業授權金鑰、運維負擔較重、部分功能需付費。

> **適用對象**：20 人以上組織、有法規遵循需求、需要 SSO/SCIM 和審計的企業。

### 3. KeePassXC（搭配 Syncthing/Nextcloud）

KeePassXC[^keepassxc] 是 KeePass 的跨平台分支，採用**無伺服器、檔案式**的密碼管理策略。

- **授權條款**：GPL-2.0 / GPL-3.0
- **技術棧**：C++（Qt）、KDBX4 加密檔案格式
- **部署方式**：無伺服器——安裝桌面應用後透過 Syncthing、Nextcloud、rsync 等同步
- **特色功能**：
  - AES-256 + Argon2 加密，經 20 年以上社群審視
  - 內建 TOTP、支援 YubiKey/Nitrokey 硬體金鑰
  - 第三方行動應用：KeePassDX（Android）、Strongbox（iOS）

**優點**：零伺服器攻擊面、完全離線可用、無任何訂閱與雲端元件。

**缺點**：無內建同步、無團隊分享模型、無網頁 vault、行動應用為第三方開發。

> **適用對象**：極度重視隱私的個人、安全研究人員、記者、需要離線優先儲存的用戶。

### 4. Passbolt

Passbolt[^passbolt] 是以 OpenPGP 為基礎的團隊密碼管理工具，從第一天就以憑證分享為核心設計。

- **授權條款**：AGPL-3.0（社群版）；Pro 與 Enterprise 為專屬授權
- **技術棧**：PHP、MySQL/MariaDB/PostgreSQL、OpenPGP 加密
- **最低需求**：約 1 GB RAM
- **特色功能**：
  - GPG 架構：伺服器僅儲存加密資料，入侵不直接洩露憑證
  - 最佳權限模型：逐項資源 ACL（擁有者、檢視者）
  - 社群版即含完整審計追蹤
  - 無限使用者，無人為功能上限

**優點**：權限模型最佳、審計功能內建、第三方審計定期公開。

**缺點**：GPG 金鑰管理增加入門摩擦、瀏覽器擴充功能為必須、生態系小於 Bitwarden。

> **適用對象**：5-200 人團隊，以共用憑證和審計追蹤為主要需求；受監管產業（法律、醫療、金融）。

### 5. Psono

Psono[^psono] 是開發者導向的自託管密碼管理工具，採用 Apache-2.0 授權，強調 API 完整性。

- **授權條款**：Apache-2.0（社群版伺服器與客戶端）
- **技術棧**：Python（Django）、PostgreSQL、Docker Compose
- **最低需求**：約 512 MB RAM
- **特色功能**：
  - 客戶端 AES-256-GCM 加密，伺服器永不存取明文
  - 完善的 REST API，支援 CI/CD 秘密注入
  - 檔案與秘密管理功能（不限於密碼）

**優點**：Apache-2.0 授權（最友善企業法務）、API 設計優良、無每使用者 GPG 金鑰管理。

**缺點**：社群較小、行動應用不如 Bitwarden 成熟、獨立安全審計較少。

> **適用對象**：開發者團隊、因 AGPL 顧慮需要 Apache-2.0 授權的企業。

### 6. Padloc

Padloc[^padloc] 是以 TypeScript 開發的現代簡潔密碼管理器，具備端對端加密。

- **授權條款**：GPL-3.0
- **技術棧**：TypeScript、Node.js、SQLite/PostgreSQL
- **最低需求**：約 500 MB RAM
- **特色功能**：
  - 簡潔現代的使用者介面
  - 端對端加密、WebAuthn/FIDO2 支援
  - 組織與團隊 vault 分享

**優點**：UI 設計佳、學習曲線低、WebAuthn 支援。

**缺點**：企業控制功能較少、社群和第三方整合生態較小。

> **適用對象**：重視 UI 品質的小型團隊與個人。

### 7. Teampass

Teampass[^teampass] 是以資料夾層級存取控制為基礎的協作式密碼管理器。

- **授權條款**：GPL-3.0
- **技術棧**：PHP、MySQL/MariaDB、Apache/Nginx
- **特色功能**：
  - 2009 年起持續維護的成熟專案
  - 資料夾層級 ACL、LDAP/AD 認證
  - Docker 映像檔可用

**優點**：專案成熟、資料夾式權限管理直覺。

**缺點**：UI 較老舊、共用密碼使用單一對稱金鑰加密（儲存於伺服器）、行動端僅網頁版。

> **適用對象**：需要 LAMP 架構協作密碼管理器的團隊。

### 8. SysPass

SysPass[^syspass] 是專為 IT 團隊設計的密碼管理器，用於管理伺服器、設備與應用程式憑證。

- **授權條款**：GPL-3.0
- **技術棧**：PHP、MySQL、Apache/Nginx
- **最低需求**：約 256 MB RAM
- **特色功能**：
  - 角色式存取控制、群組與設定檔
  - REST API、審計日誌
  - LDAP/AD 整合

**優點**：免費、角色式存取控制、審計日誌。

**缺點**：開發速度減緩（原作者 2026 年已淡出）、UI 老舊、行動支援有限。

> **適用對象**：能自行運維的 IT 團隊，不需要現代 UI 或行動優先體驗。

### 9. AliasVault

AliasVault[^aliasvault] 是隱私優先的密碼管理器，內建郵件別名伺服器。

- **授權條款**：AGPL-3.0 / MIT
- **技術棧**：Docker Compose，內建 Postfix + Dovecot 郵件伺服器
- **最低需求**：約 1 GB RAM、1 vCPU
- **特色功能**：
  - 內建電子郵件別名產生器——每個網站使用獨特信箱
  - 完全端對端加密
  - 身份產生器（建立替代數位身份）

**優點**：獨特的郵件別名功能、端對端加密、開發活躍。

**缺點**：資源消耗高於 Vaultwarden、非團隊/企業設計、郵件伺服器增加運維複雜度。

> **適用對象**：希望每個網站使用獨立信箱的隱私意識個人用戶。

### 10. Passky

Passky[^passky] 是輕量、現代、易於使用的密碼管理器。

- **授權條款**：GPL-3.0
- **技術棧**：PHP/JavaScript、MySQL
- **最低需求**：約 256 MB RAM
- **特色功能**：
  - 軟體與硬體 2FA 免費可用
  - 25+ 語言支援、多種佈景主題

**優點**：非常易用、學習曲線最低、輕量。

**缺點**：社群較小、生態系深度不足、較少被審計。

> **適用對象**：想要簡單輕量、無繁瑣功能的用戶。

### 11. Passit

Passit[^passit] 使用 libsodium 加密的開源密碼管理器，具備群組分享功能。

- **授權條款**：AGPL-3.0
- **技術棧**：Python（Django）、PostgreSQL、libsodium

**優點**：使用現代加密函式庫 libsodium、API 簡潔。

**缺點**：開發速度較慢、功能不如 Bitwarden 完整。

### 12. Password Pusher

Password Pusher[^pwp] 並非密碼 vault，而是**一次性秘密分享工具**，用於暫時安全傳遞憑證。

- **授權條款**：OSL-3.0
- **技術棧**：Ruby on Rails、SQLite/MySQL/PostgreSQL
- **特色功能**：
  - 自毀連結（可設定過期時間與檢視次數）
  - REST API、Slack 整合

> ⚠️ **注意**：這不是密碼管理器——沒有 vault、沒有同步、沒有自動填入功能。應作為完整密碼管理器的輔助工具使用。

### 13. pass（password-store）

pass[^pass] 是標準的 Unix 密碼管理器，使用 GPG 加密與簡潔的檔案階層。

- **授權條款**：GPL-2.0
- **技術棧**：Bash、GPG、Git
- **特色功能**：
  - 最 Unix 哲學的方式——簡單、可組合、可腳本化
  - Git 版本控制內建
  - 擴充套件生態系（pass-otp、pass-audit 等）

**優點**：極輕量、Git 版本控制內建、GPG 加密成熟。

**缺點**：僅 CLI（對非技術使用者學習曲線陡峭）、無網頁 UI、團隊存取控制僅能透過 GPG 金鑰分享。

> **適用對象**：Linux/Unix 進階使用者、開發者、CLI 愛好者。

## 功能比較表

| 工具 | 授權條款 | 技術棧 | RAM 需求 | 部署方式 | 網頁 UI | 行動應用 | 瀏覽器擴充 | 2FA | API | 團隊分享 |
|---|---|---|---|---|---|---|---|---|---|---|
| **Vaultwarden** | AGPL-3.0 | Rust, Docker | ~50 MB | 單一 Docker 容器 | ✓ (BW) | ✓ (BW) | ✓ | TOTP, YubiKey, FIDO2 | ✓ (BW API) | ✓ |
| **Bitwarden** | AGPL-3.0 | C#/.NET | ~2 GB | Docker Compose (6+) | ✓ | ✓ | ✓ | TOTP, FIDO2, YubiKey, Duo | ✓ (+ Secrets) | ✓ (企業) |
| **KeePassXC** | GPL-2.0/3.0 | C++ (Qt) | 無伺服器 | 桌面 + 同步 | ✗ | ✓ (第三方) | ✓ | TOTP, YubiKey | CLI | ✗ (DIY) |
| **Passbolt** | AGPL-3.0 | PHP, MySQL, GPG | ~1 GB | Docker / .deb | ✓ (需擴充) | ✓ | ✓ (必須) | TOTP, YubiKey | ✓ | ✓ (最佳) |
| **Psono** | Apache-2.0 | Python, PostgreSQL | ~512 MB | Docker Compose | ✓ | ✓ | ✓ | TOTP, YubiKey, Duo | ✓ (完善) | ✓ |
| **Padloc** | GPL-3.0 | TypeScript, Node.js | ~500 MB | Docker | ✓ | ✓ | ✓ | TOTP, WebAuthn | 有限 | ✓ |
| **Teampass** | GPL-3.0 | PHP, MySQL | ~512 MB | Docker / LAMP | ✓ | 僅網頁 | ✓ | TOTP, YubiKey | 有限 | ✓ (資料夾) |
| **SysPass** | GPL-3.0 | PHP, MySQL | ~256 MB | LAMP / Docker | ✓ | 僅網頁 | ✓ | TOTP | ✓ | ✓ (角色) |
| **AliasVault** | AGPL-3.0/MIT | Docker 複合 | ~1 GB | Docker Compose | ✓ | PWA | ✓ | ✓ | ✓ | 基本 |
| **Passky** | GPL-3.0 | PHP, JS, MySQL | ~256 MB | Docker | ✓ | ✓ | ✓ | TOTP, 硬體 | 有限 | 基本 |
| **Passit** | AGPL-3.0 | Python/Django | ~512 MB | Docker | ✓ | 僅網頁 | ✓ | ✓ | ✓ | 群組 |
| **Password Pusher** | OSL-3.0 | Ruby on Rails | ~512 MB | Docker | ✓ | ✗ | ✗ | N/A | ✓ | 一次性 URL |
| **pass** | GPL-2.0 | Bash, GPG, Git | 無伺服器 | CLI + Git | ✗ | ✓ (第三方) | ✓ (browserpass) | GPG + pass-otp | CLI + 腳本 | Git/GPG |

## 選擇建議

| 使用情境 | 推薦工具 |
|---|---|
| 個人/家庭自用，硬體有限 | **Vaultwarden**（50MB RAM，單一容器） |
| 5-200 人團隊需憑證分享 | **Passbolt**（最佳分享/審計）或 **Vaultwarden**（更簡單便宜） |
| 企業需 SSO/法規遵循 | **Bitwarden**（官方自託管） |
| 極度重視隱私，不要伺服器 | **KeePassXC** + Syncthing/Nextcloud |
| 需要網站專用信箱別名 | **AliasVault** |
| 法務對 AGPL 有顧慮 | **Psono**（Apache-2.0） |
| CLI 愛好者 / Unix 使用者 | **pass**（password-store） |
| 取代 Slack/Email 傳遞一次性秘密 | **Password Pusher**（輔助工具） |
| 想要簡單現代方案 | **Padloc** |
| IT 團隊管理伺服器憑證 | **SysPass** 或 **Teampass** |

## 結論

FOSS 自託管密碼管理生態系已相當成熟，從極輕量的 Vaultwarden 到企業級的 Bitwarden 官方版皆有對應方案。選擇關鍵取決於：

1. **規模**：個人/家庭建議 Vaultwarden，團隊建議 Passbolt，企業建議 Bitwarden
2. **硬體資源**：資源受限者首選 Vaultwarden（50MB RAM）或免伺服器的 KeePassXC/pass
3. **隱私需求**：極致隱私選 KeePassXC，郵件別名選 AliasVault
4. **法規與授權**：對 AGPL 敏感者選 Psono（Apache-2.0）

多數方案均支援 Docker 部署，可在 10-30 分鐘內完成建置。建議以 Vaultwarden 作為預設起點，因其實踐了最低資源消耗與最高客戶端相容性的最佳平衡。

## 參考資料

[^vaultwarden]: Vaultwarden. (n.d.). *Vaultwarden — Unofficial Bitwarden compatible server*. Retrieved 2026-09-27, from https://github.com/dani-garcia/vaultwarden
[^bitwarden]: Bitwarden Inc. (n.d.). *Bitwarden — Open Source Password Manager*. Retrieved 2026-09-27, from https://bitwarden.com/help/self-hosting/
[^keepassxc]: KeeppassXC Team. (n.d.). *KeePassXC — Cross-platform password manager*. Retrieved 2026-09-27, from https://keepassxc.org/
[^passbolt]: Passbolt SA. (n.d.). *Passbolt — Open source password manager for teams*. Retrieved 2026-09-27, from https://www.passbolt.com/
[^psono]: Psono. (n.d.). *Psono — Self-hosted enterprise password manager*. Retrieved 2026-09-27, from https://psono.com/
[^padloc]: Padloc. (n.d.). *Padloc — Modern password manager*. Retrieved 2026-09-27, from https://padloc.app/
[^teampass]: Nils Teampass. (n.d.). *TeamPass — Collaborative password manager*. Retrieved 2026-09-27, from https://teampass.net/
[^syspass]: SysPass. (n.d.). *SysPass — IT team password manager*. Retrieved 2026-09-27, from https://www.syspass.org/
[^aliasvault]: AliasVault. (n.d.). *AliasVault — Self-hosted password manager with email aliases*. Retrieved 2026-09-27, from https://docs.aliasvault.com/
[^passky]: Passky. (n.d.). *Passky — Simple, modern password manager*. Retrieved 2026-09-27, from https://www.passky.org/
[^passit]: Passit. (n.d.). *Passit — Open source password manager with libsodium*. Retrieved 2026-09-27, from https://passit.io/
[^pwp]: Password Pusher. (n.d.). *Password Pusher — Self-destructing secret sharing*. Retrieved 2026-09-27, from https://github.com/pglombardo/PasswordPusher
[^pass]: Jason A. Donenfeld. (n.d.). *pass — the standard unix password manager*. Retrieved 2026-09-27, from https://www.passwordstore.org/