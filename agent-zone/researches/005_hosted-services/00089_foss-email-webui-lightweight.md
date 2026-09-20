# 輕量級 FOSS 網頁郵件用戶端（Webmail）調查報告

## 摘要

本報告針對可在既有郵件伺服器（IMAP/SMTP，如 Dovecot）前端運行的自由開源（FOSS）網頁郵件用戶端進行調查，重點關注伺服器端資源消耗低、輕量化的方案。經評估，**SnappyMail** 為綜合最優選擇（無需資料庫、PHP 後端、活躍維護），**Cypht** 為多帳戶聚合情境的最佳替代方案，**lilmail** 則為極致輕量的單一二進位方案。

---

## 篩選標準

1.  **FOSS 授權** — 必須為開放原始碼且可自由部署
2.  **網頁介面** — 透過瀏覽器操作，非桌面或終端機應用
3.  **輕量低資源** — 伺服端 RAM/CPU/儲存需求低，無大量依賴
4.  **前端角色** — 作為既有郵件伺服器（IMAP/SMTP）的前端 UI，非完整郵件伺服器
5.  **活躍維護** — 近年仍有提交或發行版本

---

## 方案比較

### 1. SnappyMail（首選推薦）

| 面向 | 內容 |
|---|---|
| **授權** | AGPL-3.0 |
| **技術棧** | PHP 7.4+，前端 ES2020 JavaScript |
| **資料庫** | **不需要** — 使用 `data/` 目錄的純檔案儲存 |
| **維護狀態** | ✅ 非常活躍：1.7k stars、7,184 commits、221 forks，持續發行版本[^snappy] |
| **資源使用** | 極低 — 無資料庫、前端 JS 壓縮後約 358 KB（比 RainLoop 小 66%）、官方 Docker 映像約 50 MB |
| **特色功能** | 暗色模式、PGP 加密（OpenPGP.js v5 + GnuPG + Mailvelope）、Kolab 群組軟體支援、進階 Sieve 編輯器、OAuth2、DX/Exchange 外掛、多帳號、聯絡人（可選 SQLite/MySQL/PG）、fail2ban 整合、行動裝置適配 |
| **部署要求** | PHP-FPM + nginx/Apache，需 PHP 延伸模組 mbstring、Zlib、json、libxml、dom |

SnappyMail 衍生自 RainLoop，繼承其簡潔架構但大幅瘦身前端，且完全不需要資料庫，使其成為輕量級 webmail 的首選[^snappy-wiki]。

---

### 2. Cypht

| 面向 | 內容 |
|---|---|
| **授權** | LGPL-2.1 |
| **技術棧** | PHP 後端 + 無框架原生 JavaScript |
| **資料庫** | **不需要**（核心功能無需資料庫，部分模組可選 SQLite） |
| **維護狀態** | ✅ 非常活躍：1.7k stars、7,609 commits、243 forks，近期仍有提交，每月社群會議[^cypht] |
| **資源使用** | 極低 — 伺服器不儲存郵件，目錄結構與訊息清單快取於用戶端連線階段（session）中 |
| **特色功能** | 統一收件匣（跨多個 IMAP 帳號 + RSS + JMAP + Exchange）、跨帳號搜尋、Sieve 篩選、SMTP 發送、2FA（TOTP）、OAuth2、用戶端加密（libsodium/AES-256-CBC）、RSS 閱讀器、鍵盤快捷鍵、全模組化架構 |
| **獨特定位** | 不取代既有郵件帳號，而是將多個帳號整合至單一介面[^cypht-features] |

Cypht 的架構設計理念與傳統 webmail 截然不同，它以「聚合器」而非「郵件客戶端」為核心，伺服器端的儲存與運算需求極低。

---

### 3. lilmail

| 面向 | 內容 |
|---|---|
| **授權** | MIT OR Apache-2.0 |
| **技術棧** | Go (Fiber) + 伺服器渲染 HTML/HTMX/Alpine.js — **單一自包含二進位檔 (~24 MB)** |
| **資料庫** | **不需要** — 使用嵌入式 bbolt |
| **維護狀態** | ⚠️ 活躍但年輕：51 stars、253 commits，截至 2026 年 8 月仍有發行版本[^lilmail] |
| **資源使用** | **極致低** — 官方宣稱可「**舒適運行於 64 MB RAM**」、無建置步驟、無需 CDN、可離線／斷網運作 |
| **特色功能** | IMAP/SMTP、OAuth2/OIDC (PKCE, XOAUTH2)、JWZ 對話串接、CalDAV 日曆、CardDAV 聯絡人、REST `/v1` JSON API、Web Push、IMAP IDLE、排程寄信、暗色模式、多帳號 |

lilmail 是唯一以單一二進位檔形式運作的方案，部署極簡，非常適合資源受限的環境（如樹莓派、低配 VPS）。但專案尚年輕，社群規模小，生產環境需審慎評估。

---

### 其他評估方案

#### Roundcube — 業界標準但較重

- **授權：** GPL-3.0（含佈景主題／外掛例外條款）
- **技術棧：** PHP + jQuery
- **資料庫：** **需要** MariaDB/MySQL/PostgreSQL/SQLite[^roundcube]
- **維護狀態：** ✅ 非常活躍（7.2k stars、13,770 commits）
- **評估：** Roundcube 功能完整、生態系龐大，但資料庫依賴與較大的前端 JS 堆疊使其不符合本次「輕量低資源」的核心標準。若資源充足，仍是可靠的選擇。

#### Mailur — 設計理念吻合但開發停滯

- **授權：** GPL-3.0
- **技術棧：** Python 3 (aiohttp) + Vue.js
- **資料庫：** **不需要** — 直接以 Dovecot IMAP 作為主要儲存
- **維護狀態：** ❌ **暫停開發**（README 標註 "On pause..."，最後更新 2022-12-23）[^mailur]
- **評估：** Mailur 的設計（Dovecot 原生、無資料庫）完全符合本次調查目標，但開發已停滯，不建議用於新生產部署。

#### Mailpile — 知名專案但已停擺

- **授權：** AGPL-3.0
- **技術棧：** Python（自有搜尋引擎、標籤式管理）
- **維護狀態：** ❌ README 明確指出開發已暫停，等待 Python 3 重寫[^mailpile]
- **評估：** 曾獲大量關注（8.8k stars），但目前不適合新部署。

---

## 決策矩陣

| 專案 | 技術棧 | 需資料庫 | 估計伺服器 RAM | 維護狀態 | 最佳適用情境 |
|---|---|---|---|---|---|
| **SnappyMail** | PHP | 否 | 約數十 MB（PHP-FPM） | ✅ 非常活躍 | 通用首選 |
| **Cypht** | PHP | 否（可選 SQLite） | 極低（用戶端快取） | ✅ 非常活躍 | 多帳號聚合 |
| **lilmail** | Go 單一二進位 | 否（嵌入式 bbolt） | ~64 MB | ⚠️ 活躍但年輕 | 極致最小化部署 |
| Mailur | Python + Vue | 否（Dovecot 儲存） | 低 | ❌ 2022 停滯 | 不建議生產使用 |
| Roundcube | PHP | 是（SQL） | 中低 | ✅ 非常活躍 | 功能完整標準方案 |

---

## 結論

針對「低伺服器資源的 FOSS 網頁郵件前端」需求，推薦優先順序如下：

1.  **SnappyMail** — 無資料庫、PHP 生態成熟、前端輕量化、專案活躍，為絕大多數情境的最佳選擇。
2.  **Cypht** — 若需同時管理多個郵件帳號（含 RSS、Exchange），其獨特的用戶端快取架構使伺服器負載極低。
3.  **lilmail** — 若環境資源極度受限（如 64 MB RAM VPS、容器化邊緣部署），單一二進位檔的 Go 方案最為理想。

---

[^snappy]: SnappyMail. (n.d.). SnappyMail — Simple, modern, lightweight & secure webmail. Retrieved 2026-09-20, from https://github.com/the-djmaze/snappymail
[^snappy-wiki]: SnappyMail. (n.d.). Installation instructions. Retrieved 2026-09-20, from https://github.com/the-djmaze/snappymail/wiki/Installation-instructions
[^cypht]: Cypht. (n.d.). Cypht — Lightweight open source webmail. Retrieved 2026-09-20, from https://github.com/cypht-org/cypht
[^cypht-features]: Cypht. (n.d.). Features — Cypht. Retrieved 2026-09-20, from https://cypht.org/features/
[^lilmail]: vul-os. (n.d.). lilmail — A single binary mail client. Retrieved 2026-09-20, from https://github.com/vul-os/lilmail
[^roundcube]: Roundcube. (n.d.). Roundcube webmail — A browser-based multilingual IMAP client. Retrieved 2026-09-20, from https://github.com/roundcube/roundcubemail
[^mailur]: naspeh. (n.d.). Mailur — Lightweight webmail that uses Dovecot as storage. Retrieved 2026-09-20, from https://github.com/naspeh/mailur
[^mailpile]: Mailpile. (n.d.). Mailpile — A modern, fast web-mail client. Retrieved 2026-09-20, from https://github.com/mailpile/Mailpile