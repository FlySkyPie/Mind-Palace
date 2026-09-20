# FOSS 郵件伺服器（Email Server）研究報告

## 概述

本報告針對自由開源（FOSS）的郵件伺服器軟體進行調查與比較，涵蓋目前最受關注的 9 個專案。自建郵件伺服器的主要挑戰並非軟體本身，而是寄送信譽（deliverability）——包括 IP 暖身、PTR 記錄、SPF/DKIM/DMARC 等 DNS 設定以及埠 25 是否開放[^sg-ramblings]。

---

## 專案比較

### 1. Stalwart Mail Server

- **語言**：Rust
- **授權**：AGPL v3.0（開源版）/ Stalwart Enterprise License v2（商業版）
- **GitHub Stars**：~14.7k ⭐
- **官網**：https://stalw.art

**功能亮點**：
- 完整 SMTP + IMAP4rev2/rev1 + POP3 + **JMAP**（現代郵件通訊協定）
- 內建 CalDAV/CardDAV/WebDAV（行事曆、聯絡人、檔案儲存）
- 內建反垃圾郵件引擎（LLM 驅動的 spam 過濾、統計分類器、DNSBL、greylisting、釣魚防護）
- 支援 DMARC、DKIM（v1/v2）、SPF、ARC、DANE、MTA-STS、SMTP TLS Reporting
- 內建 Web 管理後台（即時監控、帳戶/網域管理、SMTP 佇列管理、DMARC 報告視覺化）
- 支援 OpenID Connect、OAuth 2.0、LDAP、SCIM v2、2FA-TOTP、app passwords
- 可插拔儲存：RocksDB、FoundationDB、PostgreSQL、MySQL、SQLite、S3、Azure、Redis
- 內建全文檢索（17 種語言）+ 可整合 Meilisearch、ElasticSearch

**部署**：單一二進位檔、Docker、Kubernetes。最低約 512 MB RAM。

**評價**：2025-2026 年最受矚目的現代化郵件伺服器。功能最全面的單一二進位方案，適合技術使用者或低資源 VPS。[^tg]

---

### 2. Mailcow (mailcow: dockerized)

- **語言**：PHP + 多元件（Postfix / Dovecot / Rspamd 等）
- **授權**：GPL v3.0
- **GitHub Stars**：~13.4k ⭐
- **官網**：https://mailcow.email

**功能亮點**：
- Postfix（SMTP）+ Dovecot（IMAP/POP3）+ SOGo Groupware（Webmail、行事曆、通訊錄、ActiveSync）
- Rspamd 反垃圾郵件 + ClamAV 防毒 + Olefy Office 文件巨集掃描
- DKIM 自動產生/簽署、ARC、DMARC、SPF、MTA-STS、DANE
- 全面的 Web 管理後台（網域/使用者管理、DKIM、黑/白名單、隔離區、TFA、imapsync、監控）
- Let's Encrypt 自動 TLS

**部署**：Docker Compose（約 15 個容器），建議至少 4-6 GB RAM。

**評價**：最成熟的「開箱即用」全端方案，適合需要共享行事曆/通訊錄的小型團隊。[^pg-ramblings][^rt]

---

### 3. Mail-in-a-Box

- **語言**：Bash + Python
- **授權**：CC0 1.0（公眾領域）
- **GitHub Stars**：~15.4k ⭐
- **官網**：https://mailinabox.email

**功能亮點**：
- Postfix + Dovecot + Roundcube Webmail
- Nextcloud（CalDAV/CardDAV）+ Z-Push（Exchange ActiveSync）
- SpamAssassin + Postgrey greylisting + fail2ban
- SPF、DKIM、DMARC、DNSSEC、DANE TLSA、MTA-STS、SSHFP 全自動設定
- 內建控制面板 + REST API

**部署**：Ubuntu 22.04 LTS 一鍵安裝腳本，佔用整臺機器。最低 2 GB RAM。

**評價**：最簡單的個人網域郵件方案。但極度固執（opinionated），無法客製化，不支援 Docker。[^tg]

---

### 4. docker-mailserver (DMS)

- **語言**：Shell + 設定檔（Postfix / Dovecot / Rspamd）
- **授權**：MIT
- **GitHub Stars**：~18.9k ⭐（最高）
- **官網**：https://docker-mailserver.github.io

**功能亮點**：
- Postfix（SMTP）+ Dovecot（IMAP/POP3/SASL/Sieve）
- Rspamd、Amavis、SpamAssassin、ClamAV、fail2ban、Postscreen
- OpenDKIM + OpenDMARC、OAuth2（XOAUTH2/OAUTHBEARER SASL）
- 無需 SQL 資料庫，純設定檔方式
- LDAP、SASLauthd、OAuth2 認證

**部署**：單一 Docker 容器，無 Web UI。最低 1 GB RAM。

**評價**：最受歡迎的 Docker 郵件方案。適合基礎設施即程式碼（IaC）風格的使用者，但不含 Webmail 和管理 UI。[^tg]

---

### 5. Maddy Mail Server

- **語言**：Go
- **授權**：GPL v3.0
- **GitHub Stars**：~6.1k ⭐
- **官網**：https://maddy.email

**功能亮點**：
- 單一守護行程取代 Postfix + Dovecot + OpenDKIM + OpenSPF + OpenDMARC
- SMTP（MTA + MX）+ IMAP4rev1（Beta 品質）
- 原生 DANE、MTA-STS、DKIM、SPF、DMARC
- 可整合 rspamd（Milter）+ DNSBL
- Prometheus 內建指標、S3 相容 blob 儲存
- 零停機設定重新載入

**部署**：單一二進位檔 / Docker。支援 SQLite、PostgreSQL。

**評價**：優雅的工程方案，但 IMAP 儲存標示為 Beta，實際生產建議配對 Dovecot。適合偏好單一行程、原生 DANE/MTA-STS 的使用者。由單一開發者維護，bus factor = 1。[^sm]

---

### 6. WildDuck Mail Server

- **語言**：Node.js
- **授權**：EUPL v1.2
- **GitHub Stars**：~2.1k ⭐
- **官網**：https://wildduck.email

**功能亮點**：
- 完整 IMAP4rev1 + POP3 + LMTP
- 無 SPOF（No Single Point of Failure）架構——無狀態實例、MongoDB 分片複寫
- REST API 驅動（所有管理皆可透過 API 完成）
- 內建 2FA（TOTP + WebAuthn/FIDO2）、應用程式密碼、PGP 加密
- 完整 EAI（國際化電子郵件地址）支援
- 附件去重（content-hash based）、壓縮（節省 40-56% 空間）
- Webhooks、HTTP Event Source

**部署**：需搭配 Haraka（入站 SMTP）+ ZoneMTA（出站 SMTP）+ MongoDB + Redis。建議 1000+ 帳號規模。

**評價**：為大規模水平擴展而設計。對小規模部署來說過於複雜，官方亦建議小規模使用 Postfix+Dovecot。[^wd]

---

### 7. iRedMail

- **語言**：Shell 安裝腳本（元件：Postfix/Dovecot/Amavis 等）
- **授權**：GPL v3.0
- **GitHub Stars**：~1.8k ⭐
- **官網**：https://iredmail.org

**功能亮點**：
- Postfix + Dovecot + Roundcube + SOGo（Webmail、行事曆、通訊錄、ActiveSync）
- Amavisd-new + SpamAssassin + ClamAV + DKIM 簽署
- 支援 OpenLDAP 或 MySQL/MariaDB/PostgreSQL 後端
- iRedAdmin 管理介面（免費基礎版 / 付費 Pro 版）

**部署**：安裝腳本支援 CentOS/RHEL/Alma/Rocky、Debian/Ubuntu、FreeBSD、OpenBSD，亦有 Docker 版。

**評價**：極為成熟的專案（2007 年至今），支援最廣泛的作業系統。進階管理功能需付費 iRedAdmin-Pro。[^tg]

---

### 8. Postal

- **語言**：Ruby（Ruby on Rails）+ JavaScript
- **授權**：MIT
- **GitHub Stars**：~16.8k ⭐
- **官網**：https://postalserver.io

**功能亮點**：
- 純 SMTP 伺服器（入站 + 出站），**不含 IMAP/POP3**（非信箱伺服器）
- 內建 Web UI（組織/伺服器/使用者/憑證/統計/訊息佇列/Webhook）
- SpamAssassin + ClamAV 入站過濾
- DKIM 簽署 + DNS 可送達性檢查

**部署**：Docker / 手動安裝（需 MySQL + RabbitMQ）。

**評價**：這是自助託管的 SendGrid/Mailgun 替代品，**非個人信箱伺服器**。適合網站發送交易性郵件。[^tg]

---

### 9. Modoboa

- **語言**：Python 3（Django）+ Vue.js
- **授權**：ISC
- **GitHub Stars**：~3.5k ⭐
- **官網**：https://modoboa.org

**功能亮點**：
- 整合 Postfix + Dovecot（完整 SMTP/IMAP/POP3）
- 內建 Webmail + 行事曆 + 通訊錄
- Amavis 前端 + DNSBL + DMARC 報表擴充
- 模組化擴充系統

**部署**：官方安裝腳本 / Docker。支援 MySQL/MariaDB/PostgreSQL/SQLite。

**評價**：Python/Django 生態使用者的好選擇。模組化設計，DMARC 報表擴充為亮點。[^tg]

---

## GitHub Stars 排名

| 排名 | 專案 | Stars | 語言 |
|------|------|-------|------|
| 1 | docker-mailserver | 18.9k ⭐ | Shell |
| 2 | Postal | 16.8k ⭐ | Ruby |
| 3 | Mail-in-a-Box | 15.4k ⭐ | Bash/Python |
| 4 | Stalwart | 14.7k ⭐ | Rust |
| 5 | Mailcow | 13.4k ⭐ | PHP/多元件 |
| 6 | Maddy | 6.1k ⭐ | Go |
| 7 | Modoboa | 3.5k ⭐ | Python |
| 8 | WildDuck | 2.1k ⭐ | Node.js |
| 9 | iRedMail | 1.8k ⭐ | Shell |

---

## 快速選擇指南

| 使用情境 | 推薦方案 |
|----------|----------|
| 個人自建、想最簡單 | Mail-in-a-Box（一鍵腳本） |
| 現代化單一二進位、低資源 VPS、JMAP | **Stalwart**（512 MB RAM） |
| 小團隊需共享行事曆/通訊錄、精美 UI | **Mailcow**（6+ GB RAM） |
| Docker IaC 風格、不需 UI | docker-mailserver |
| Kubernetes 原生 | Mailu |
| 發送交易性郵件（非信箱） | Postal |
| 大規模水平擴展（1000+ 帳號） | WildDuck |
| 偏好單一守護行程、原生 DANE/MTA-STS | Maddy（IMAP 為 Beta） |
| Python/Django 生態、多網域管理 | Modoboa |

---

## 基礎設施注意事項

所有比較文章皆一致指出：**軟體選擇是容易的部分**，實際挑戰在於[^sg-ramblings][^pg-ramblings]：

1. **埠 25 被封鎖**：DigitalOcean、AWS、GCP、Azure、Linode、Vultr 預設封鎖，建議使用 Hetzner 或 OVH
2. **靜態 IP + PTR 記錄**：不可或缺，家用 ISP 和 CGNAT 無法運作
3. **DNS 記錄至關重要**：SPF、DKIM、DMARC、MX、A、PTR 必須全部正確
4. **IP 暖身**：第 1 天 10 封、第 2 天 50 封，逐步增加至 2-4 週；冷 IP 大量寄送直接進垃圾郵件
5. **寄送信譽（deliverability）** 才是真正決定成敗的因素

---

## 參考資料

[^sg-ramblings]: SumGuy's Ramblings. (n.d.). *Self-Hosted Email 2026: Mailcow vs Mailu vs Stalwart*. Retrieved 2026-09-20, from https://sumguy.com/self-hosted-email-mailcow-mailu-stalwart/

[^tg]: Talos Tools. (n.d.). *7 Best Self-Hosted Email Servers in 2026*. Retrieved 2026-09-20, from https://talos.tools/blog/best-self-hosted-email-server-2026

[^rt]: RottenWiFi. (n.d.). *11 Best Self-Hosted Email Servers in 2026 Compared*. Retrieved 2026-09-20, from https://rottenwifi.com/11-best-self-hosted-email-server-platforms-to-use-in-2026/

[^pg-ramblings]: profor.pro. (n.d.). *Self-hosted email in 2026: mailcow vs Stalwart vs Mailu*. Retrieved 2026-09-20, from https://profor.pro/blog/self-hosted-email-2026-mailcow-stalwart-mailu/

[^sm]: GitHub — foxcpp/maddy. (n.d.). *Maddy Mail Server*. Retrieved 2026-09-20, from https://github.com/foxcpp/maddy

[^wd]: GitHub — zone-eu/wildduck. (n.d.). *WildDuck Mail Server*. Retrieved 2026-09-20, from https://github.com/zone-eu/wildduck

[^ss]: Stalwart Labs. (n.d.). *Stalwart Mail Server Documentation*. Retrieved 2026-09-20, from https://stalw.art/docs/install/get-started

[^md]: Mailcow. (n.d.). *Mailcow: dockerized Documentation*. Retrieved 2026-09-20, from https://docs.mailcow.email/