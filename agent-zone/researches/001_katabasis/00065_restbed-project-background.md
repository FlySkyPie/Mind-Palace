# Restbed 專案背景調查報告

## 概述

Restbed 是一個由 Corvusoft 公司開發的 C++ 非同步 RESTful 框架，專注於提供企業級 HTTP 通訊能力，支援 WebSocket、Server-Sent Events、SSL/TLS、HTTP Pipelining 等多項功能。專案自 2013 年啟動，至今已超過 13 年。

## 公司資訊

### Corvusoft Limited

- **公司全稱**：Corvusoft Limited
- **公司編號**：SC455983
- **公司類型**：Private Limited Company（私人有限公司）
- **註冊地址**：15 Duncan Road, Helensburgh, Argyll & Bute, Scotland, G84 9DH（曾用址：Suite 2/3, 2nd Floor, 48 West George Street, Glasgow, Scotland, G2 1BP）
- **成立日期**：2013 年 8 月 2 日
- **公司狀態**：Active（活躍中）
- **SIC 代碼**：62012 - Business and domestic software development（商業與家用軟體開發）
- **下次財務報表提交期限**：2027 年 5 月 30 日（最近報表截至 2025 年 8 月 30 日）[^coho]

### 公司定位

Corvusoft 自述為「全球領先的開源軟體支援服務提供商」，核心業務包括：

- 開源軟體技術支援
- 缺陷修復（Defect Resolution）
- 功能開發（Feature Development）
- 倉庫審計（Repository Auditing）

其服務對象為「在關鍵任務環境中使用開源元件的企業」[^corvusoft]。

### 主要人員

- **Benjamin Crowhurst**（1984 年 2 月出生）— 活躍董事，自 2013 年 8 月 2 日起任職，英國籍，居住於蘇格蘭[^coho-officers]。
  - GitHub 帳號：`ben-crowhurst`，為 restbed 專案最主要的貢獻者（822 次提交），亦是 Corvusoft GitHub 組織唯一列名成員[^github-members][^github-contributors]。
- **Laura Bernadette Bruynseels** — 曾任秘書（2024 年 1 月 10 日至 2024 年 8 月 20 日），已離職[^coho-officers]。

## GitHub 儲存庫分析

- **建立時間**：2015 年 6 月 19 日
- **主要語言**：C++
- **星數**：約 2,000
- **分支數**：380
- **觀看數**：101
- **開放議題**：4 個
- **提交數**：1,449 次
- **最後推送**：2026 年 6 月 23 日[^github-api]

### 主要貢獻者

| 貢獻者 | 提交數 | 角色 |
|--------|--------|------|
| ben-crowhurst | 822 | 主要開發者/創辦人 |
| build-o-bot | 139 | 自動化建置機器人 |
| aberaud | 8 | 社群貢獻者 |
| naedanger | 6 | 社群貢獻者 |
| zzl-010 | 6 | 社群貢獻者 |
| 其他 | 各 1-5 | 零星貢獻者（約 20 人）[^github-contributors] |

### 技術架構

- **建置系統**：GNU Autotools（autoconf/automake/libtool）
- **C++ 標準**：C++23（需要 GCC >= 13 或 Clang >= 17）
- **核心依賴**：
  - Asio（header-only，非同步 I/O 核心）
  - OpenSSL（選用，SSL/TLS 支援）
  - Catch2 >= 3.0（選用，測試套件）
- **支援平台**：BSD、Linux、macOS、Windows（MSYS2/Cygwin）[^github-readme]

### 功能特色

WebSocket、Server-Sent Events、Comet Long Polling、SSL/TLS、HTTP Pipelining、路徑/查詢參數、自訂 HTTP 方法、壓縮（GZip/Deflate）、編碼（UTF-32/ASCII）、IPv4/IPv6、C10K 問題處理、認證、錯誤處理、Signal 處理、HTTP 1.0/1.1 支援[^github-readme]。

## 授權與商業模式

Restbed 採用**雙重授權**（Dual License）模式：© 2013-2026 Corvusoft Limited, United Kingdom[^github-readme]。這意味著：

- **開源版本**：提供社群免費使用（依特定開源條款）
- **商業授權**：企業可購買商業授權與技術支援

商業服務透過 `sales@corvusoft.com` 聯繫，提供客製化軟體開發、測試、設計諮詢、培訓、指導與程式碼審查[^github-readme]。

## 資金與創投

**未發現任何創投（Venture Capital）投資紀錄**。Corvusoft 在英國 Companies House 的公開紀錄中並無股權變動顯示外部投資人。該公司很可能為**自籌資金（Bootstrapped）**的微型企業，由 Benjamin Crowhurst 個人擁有與經營。公開資訊中亦未發現任何種子輪、天使輪或其他募資活動。

## 社群生態

- **GitHub 星數**：2,000 — 屬中等規模的 C++ 開源專案
- **分支數**：380
- **活躍度**：近期提交頻率不高（最後推送為 2026 年 6 月），僅 4 個開放議題
- **引用案例**：Solutions Architect, Bellrock Technology 稱之為「akin to embedding NGINX into your companies own product line」[^github-readme]
- **Hacker News 討論**：在 2019 年一篇關於「單一 C++ 伺服器可處理複雜系統」的討論中被提及作為伺服器核心範例[^hn]

## 產業背景價值

Restbed 填補了 C++ 生態系中一個明確的缺口：提供一個現代化、功能完整的 HTTP/RESTful 框架。其競爭對手包括：

- 商業產品：NGINX、Apache HTTP Server
- 開源替代：cpp-httplib、Pistache、Drogon、libmicrohttpd
- 與 NGINX 等伺服器的關鍵差異：Restbed 是作為嵌入應用的函式庫（library），而非獨立伺服器

## 風險與注意事項

- **關鍵人員風險**：主要開發者僅 Benjamin Crowhurst 一人，其餘社群貢獻極為有限
- **維護頻率**：2026 年僅有少數提交，專案維護步調趨緩
- **建置系統**：採用 GNU Autotools，而非 CMake，可能對現代 C++ 開發者構成門檻
- **依賴管理**：需要手動安裝 Asio、OpenSSL、Catch2，缺乏套件管理整合

## 結論

Restbed 是一個由蘇格蘭微型企業 Corvusoft（實質上即 Benjamin Crowhurst 一人公司）維護的 C++ HTTP 框架，自 2013 年持續開發至今。該專案技術扎實、功能完整，但社群規模不大，維護力量集中於單一開發者。商業模式以雙重授權 + 企業支援服務為主，未發現外部創投資金。作為嵌入式的 C++ RESTful 解決方案，它在特定場景（需要自訂 HTTP 伺服器的 C++ 應用）中具有獨特價值。

---

[^coho]: Companies House. (n.d.). CORVUSOFT LIMITED overview (SC455983). Retrieved 2026-09-19, from https://find-and-update.company-information.service.gov.uk/company/SC455983
[^coho-officers]: Companies House. (n.d.). CORVUSOFT LIMITED people (SC455983). Retrieved 2026-09-19, from https://find-and-update.company-information.service.gov.uk/company/SC455983/officers
[^corvusoft]: Corvusoft. (n.d.). Corvusoft — Open-source development services. Retrieved 2026-09-19, from https://www.corvusoft.co.uk
[^github-api]: GitHub API. (n.d.). Repository Corvusoft/restbed. Retrieved 2026-09-19, from https://api.github.com/repos/Corvusoft/restbed
[^github-members]: GitHub API. (n.d.). Corvusoft organization members. Retrieved 2026-09-19, from https://api.github.com/orgs/Corvusoft/members
[^github-contributors]: GitHub API. (n.d.). Corvusoft/restbed contributors. Retrieved 2026-09-19, from https://api.github.com/repos/Corvusoft/restbed/contributors
[^github-readme]: Corvusoft. (n.d.). restbed — README. Retrieved 2026-09-19, from https://github.com/Corvusoft/restbed
[^hn]: Hacker News. (2019-12-29). A lot of complex "scalable" systems can be done with a simple, single C++ server. Retrieved 2026-09-19, from https://news.ycombinator.com/item?id=21907517