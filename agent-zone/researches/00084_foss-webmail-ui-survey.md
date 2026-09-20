# FOSS 郵件伺服器 Web UI 調查報告

## 背景

本報告旨在調查可用於郵件伺服器的「自由及開源軟體」（FOSS）網頁郵件客戶端（Webmail Client），提供專案概覽、功能比較與維護狀態，協助選擇適合自架郵件的 Web UI 解決方案。

## 重點推薦

以下為目前**最成熟、最活躍**的三個 FOSS 網頁郵件客戶端：

| 專案 | 授權 | 語言 | GitHub Stars | 最新版 | 維運狀態 |
|------|------|------|-------------|--------|---------|
| **Roundcube** | GPL-3.0 | PHP | ~7,200 | 1.7.4 (2026-09) | ✅ 非常活躍 |
| **SnappyMail** | AGPL-3.0 | PHP | ~1,700 | 2.38.2 (2024-10) | ✅ 活躍 |
| **Cypht** | LGPL-2.1 | PHP | ~1,700 | 2.12.2 (2026-08) | ✅ 非常活躍 |

---

## 1. Roundcube

- **官網**：https://roundcube.net
- **原始碼**：https://github.com/roundcube/roundcubemail
- **授權**：GNU GPL-3.0（佈景與外掛有例外條款）
- **語言**：PHP 8.1+
- **需資料庫**：是（MariaDB、MySQL、PostgreSQL、SQLite）

Roundcube 是最知名、部署最廣泛的自架網頁郵件客戶端，具有類似桌面應用的使用者介面，支援多國語系。其功能涵蓋 MIME 郵件解析、通訊錄、資料夾管理、郵件搜尋、拼字檢查、PGP 加密，並可透過外掛（plugins）與佈景（skins）擴充。[^roundcube-gh]

**2026 年維運狀況**：非常活躍。1.7.x 為當前穩定分支（1.7.0 於 2026-05 發表，歷經約四年開發），每月皆有安全性更新。1.6.x 為 LTS 分支（最低維護模式），僅接收重要安全性修補。1.5.x 已終止支援。[^roundcube-releases]

**1.7 重大變更**：最低要求 PHP 8.1+，捨棄 MS SQL Server 與 Oracle 支援、終止 Internet Explorer 支援；新增 Markdown 郵件撰寫/渲染、改進 OAuth2/OIDC 支援、強制 `public_html/` 入口點強化安全性、進階郵件搜尋語法。[^roundcube-releases]

---

## 2. SnappyMail

- **原始碼**：https://github.com/the-djmaze/snappymail
- **授權**：GNU AGPL-3.0
- **語言**：PHP 7.4+
- **需資料庫**：否（使用 flat-file/JSON 儲存）

SnappyMail 是 RainLoop Webmail Community Edition 的**強化分叉**（fork），以輕量、現代、快速為訴求。不須資料庫，壓縮後約 315 KB（較 RainLoop 小約 66%），支援 ES2020 JavaScript 無 polyfill，無 jQuery。[^snappymail-gh]

**功能亮點**：支援 Sieve 指令碼、OpenPGP.js v5 / GnuPG / Mailvelope、S/MIME、Kolab groupware、暗色模式、Docker 部署。內建 Squire HTML 編輯器（取代 CKEditor，小 18 倍）。[^snappymail-gh]

**維運狀況**：活躍。儘管曾有專案已死的謠傳，但截至 2026-03 仍有提交紀錄。v2.38.0 修復了 CVE-2024-45800（mXSS 漏洞）。

**RainLoop 狀態**：RainLoop 原始專案（MIT 授權）已於 2024-11 **封存（archived）**，官方建議使用者遷移至 SnappyMail。[^rainloop-gh]

---

## 3. Cypht

- **官網**：https://cypht.org
- **原始碼**：https://github.com/cypht-org/cypht
- **授權**：GNU LGPL-2.1
- **語言**：PHP、JavaScript
- **需資料庫**：否

Cypht 被描述為「郵件帳號的 RSS 閱讀器」，是一個輕量級的**郵件聚合器**。它不是取代既有郵件帳號，而是將多個帳號合併到單一介面。同時也支援 RSS/Atom 新聞訂閱。[^cypht-gh]

**獨特功能**：支援 IMAP、SMTP、JMAP、EWS（Exchange Web Services）、POP3 多種協定；無限帳號數、跨帳號搜尋、用戶端加密（libsodium/AES-256-CBC）、2FA（TOTP）、CSS 追蹤阻擋、Sieve 伺服端過濾、排程寄信。[^cypht-features]

**維運狀況**：非常活躍。2026 年已發行 7+ 個版本（v2.10.0 ~ v2.12.2），每月安全性修補。Composer 下載量 16.2 萬次，Docker 拉取 15.8 萬次。社群每月舉行線上會議。[^cypht-gh]

---

## 4. Mailpile

- **官網**：https://www.mailpile.is
- **原始碼（v1）**：https://github.com/mailpile/Mailpile
- **重寫版（Moggie）**：https://github.com/mailpile/moggie
- **授權**：AGPL-3.0
- **語言**：Python（v1 為 Python 2，Moggie 為 Python 3）

Mailpile 是以隱私為核心的網頁郵件客戶端，具有自訂搜尋引擎、標籤式郵件管理、友善的 OpenPGP 加密、CLI 與 API。[^mailpile-gh]

**⚠️ 重要狀態**：**Mailpile v1 已停止開發**。程式碼基於已 EOL 的 Python 2，不再接受 PR 與 issue。開發團隊正在進行 Python 3 重寫，專案名稱為 **Moggie**。[^mailpile-blog]

**Moggie 現狀**：微服務架構（搜尋引擎、中繼資料儲存、PGP 操作、IMAP 連線等），資料以 AES 加密儲存。目前已有可運作的**終端機 UI（TUI）**唯讀郵件客戶端，但**尚無 Web UI**，尚未有穩定版本。不適合生產環境使用。[^moggie-gh]

---

## 5. 其他值得注意的專案

| 專案 | 語言 | 授權 | 說明 | 維運狀態 |
|------|------|------|------|---------|
| **SquirrelMail** | PHP | GPL-2.0 | 經典輕量 IMAP 客戶端，不需 JS 即可運作 | ⏸ 不再活躍維護 |
| **Afterlogic WebMail Lite** | PHP | AGPL-3.0 | 支援 OpenPGP、社群登入、30+ 語言 | ✅ 有維護（亦有商業版） |
| **Nextcloud Mail** | PHP/JS | AGPL-3.0 | Nextcloud 生態系的郵件應用，需 Nextcloud 平臺 | ✅ 活躍 |
| **SOGo** | Obj-C/JS | GPL-2.0 | 完整協作套件（含郵件、行事曆、通訊錄），支援 ActiveSync | ✅ 活躍 |
| **Isotope Mail** | Java/React | Apache-2.0 | 微服務架構網頁郵件，Docker 部署 | 🚧 早期階段 |
| **Horde (IMP)** | PHP | GPL-2.0 | 協作框架含 Webmail 元件 | ⏸ 單體庫已封存 |

---

## 6. 綜合建議

| 使用情境 | 推薦方案 | 理由 |
|---------|---------|------|
| 最穩定、最多外掛 | **Roundcube** | 最大社群、最成熟、豐富外掛生態系、安全性更新頻繁 |
| 輕量快速、不需資料庫 | **SnappyMail** | 零資料庫需求、體積極小、現代化架構、支援 PGP |
| 多帳號聚合、RSS 整合 | **Cypht** | 獨特的聚合器模式、支援多種協定（含 EWS/JMAP）、活躍開發 |
| Nextcloud 使用者 | **Nextcloud Mail** | 與 Nextcloud 生態系深度整合 |
| 完整協作套件（郵件+行事曆） | **SOGo** | 支援 ActiveSync + CalDAV/CardDAV |

若需與既有 IMAP/SMTP 郵件伺服器搭配，**Roundcube** 是最穩妥的選擇；若追求極簡部署與高效能，**SnappyMail** 是最佳選項；若管理多個郵件帳號且有 RSS 需求，**Cypht** 提供獨特的聚合體驗。

---

[^roundcube-gh]: Roundcube. (n.d.). roundcube/roundcubemail. GitHub. Retrieved 2026-09-20, from https://github.com/roundcube/roundcubemail

[^roundcube-releases]: Roundcube. (2026-09-06). Security updates 1.6.19 and 1.7.4. roundcube.net. Retrieved 2026-09-20, from https://roundcube.net/news/2026/09/06/security-updates-1.6.19-and-1.7.4

[^snappymail-gh]: The Djmaze. (n.d.). the-djmaze/snappymail. GitHub. Retrieved 2026-09-20, from https://github.com/the-djmaze/snappymail

[^rainloop-gh]: RainLoop. (n.d.). RainLoop/rainloop-webmail. GitHub. Retrieved 2026-09-20, from https://github.com/RainLoop/rainloop-webmail

[^cypht-gh]: Cypht. (n.d.). cypht-org/cypht. GitHub. Retrieved 2026-09-20, from https://github.com/cypht-org/cypht

[^cypht-features]: Cypht. (n.d.). Features. cypht.org. Retrieved 2026-09-20, from https://cypht.org/features

[^mailpile-gh]: Mailpile. (n.d.). mailpile/Mailpile. GitHub. Retrieved 2026-09-20, from https://github.com/mailpile/Mailpile

[^mailpile-blog]: Einarsson, B. R. (2022-11-30). Rebooting Mailpile. mailpile.is. Retrieved 2026-09-20, from https://www.mailpile.is/blog/2022-11-30_Rebooting_Mailpile.html

[^moggie-gh]: Mailpile. (n.d.). mailpile/moggie. GitHub. Retrieved 2026-09-20, from https://github.com/mailpile/moggie