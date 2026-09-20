# GNU Mailman 介紹

## 什麼是 GNU Mailman？

**GNU Mailman** 是由 GNU 專案維護的自由開源**郵件列表（mailing list）管理軟體**，以 Python 撰寫，採用 GPL 授權。它是目前網際網路上最廣泛部署的開源郵件列表管理系統之一，從開源專案社群（如 Python Software Foundation）、學術機構到企業組織皆有大量使用者。[^gnu-site]

[^gnu-site]: GNU Project. (n.d.). GNU Mailman. Retrieved 2026-09-20, from https://www.list.org/

## 核心功能

Mailman 提供了一套完整的郵件列表管理功能[^features]：

- **網頁化管理**：透過瀏覽器建立、刪除、管理郵件列表；使用者可自行管理訂閱偏好
- **多語言支援**：網頁與郵件通知支援近 20 種語言，可依站點、列表、使用者分別設定
- **自動退信處理**：自動偵測退信（bounce），對無效地址進行停權或移除
- **內容過濾**：基於 MIME 的內容過濾、正規表達式主題過濾
- **摘要傳送**：支援每日摘要與 MIME 摘要
- **垃圾郵件過濾**：內建防垃圾郵件機制
- **隱私權限控制**：封閉式訂閱、私人封存、隱藏成員名單
- **VERP 退信檢測**：精準辨識每個收件者的退信
- **DMARC 處理**：針對現代郵件驗證政策進行 From: rewriting
- **郵件命令**：支援 Majordomo 風格的電子郵件指令操作

[^features]: GNU Project. (n.d.). Features of GNU Mailman. Retrieved 2026-09-20, from https://www.list.org/features.html

## 版本演進

### Mailman 1（1999 年）

由 John Viega 最初開發，因硬碟毀損遺失原始碼後由 Ken Manheimer 於 CNRI 重建，Barry Warsaw 後續接手。1999 年 7 月 30 日釋出 1.0 版。目前已完全終止。[^wikipedia]

[^wikipedia]: Wikimedia Foundation. (2025). GNU Mailman. Retrieved 2026-09-20, from https://en.wikipedia.org/wiki/GNU_Mailman

### Mailman 2（2000 年代 — 最新 2.1.39）

重大重新設計，建立兩條至高原則：絕不遺失訊息、絕不重複投遞。引入 runners（佇列處理程序）架構，採多程序取代多執行緒。2.1 系列成為最廣泛部署的版本，最新版為 2021 年的 2.1.39。目前僅維護模式，不再新增功能。[^mm2-status]

[^mm2-status]: Mailman Host. (n.d.). Mailman 2 vs Mailman 3. Retrieved 2026-09-20, from https://www.mailmanhost.com/mailman-2-vs-mailman-3/

### Mailman 3（2015 年 — 最新 3.3.10）

從底層到上層全面重寫，採用元件化架構。核心特色[^mm3-arch]：

- **Mailman Core**：核心引擎，LMTP 收信 + REST API
- **Postorius**：Django 基礎的現代化 Web 管理介面
- **HyperKitty**：支援全文搜尋、討論串檢視的現代化封存系統
- 儲存從 pickle 檔案改為 SQL 資料庫（PostgreSQL / SQLite）
- REST API 提供完整程式化管理能力
- 完整虛擬網域支援

最新穩定版為 3.3.10（Tom Sawyer），2024 年 10 月 1 日釋出，活躍開發中。

[^mm3-arch]: The Architecture of Open Source Applications. (n.d.). GNU Mailman. Retrieved 2026-09-20, from https://kant2002.github.io/aosabook/en/v2/mailman.html

## 架構

### Mailman 2 架構（單體式）

全部功能整合在單一程式碼庫中：MailList 物件採用 Python mixin classes，狀態儲存在 pickle 檔案（config.pck）；訊息依序通過 pipeline 處理；runners 分別處理 incoming、outgoing、archive、bounce 等佇列；Master Watcher 管理子程序生命週期。

### Mailman 3 架構（元件式）

```
┌──────────────────────────────────────────────────────┐
│                  Mailman 3 Suite                       │
├─────────────────┬─────────────────┬──────────────────┤
│   Mailman Core  │   Postorius     │   HyperKitty     │
│   (核心引擎)     │  (Web 管理介面)  │   (封存系統)      │
├─────────────────┼─────────────────┼──────────────────┤
│  - LMTP 接收     │  - 列表管理      │  - 全文搜尋       │
│  - 審核/過濾     │  - 訂閱管理      │  - 討論串檢視    │
│  - 退信處理      │  - 使用者設定    │  - Atom feed     │
│  - DMARC 處理    │  - OAuth2 登入  │  - 直接回覆      │
│  - REST API     │                 │                  │
├─────────────────┴─────────────────┴──────────────────┤
│               資料庫 (PostgreSQL)                       │
└──────────────────────────────────────────────────────┘
```

核心引擎以 Zope Component Architecture (ZCA) 實現依賴注入，審核邏輯由 rules（規則）/ chains（鏈）組合，取代 Mailman 2 的 pipeline 設計。佇列處理透過 SHA1 分片支援平行處理。[^mm3-arch-detail]

[^mm3-arch-detail]: Mailman 3 Documentation. (n.d.). Architecture Overview. Retrieved 2026-09-20, from https://docs.mailman3.org/projects/mailman/en/latest/src/mailman/docs/architecture.html

## 安裝方式

### 1. Docker 安裝（官方推薦）

官方維護的 Docker Compose 設定，一條指令即可啟動 Core + Postorius + HyperKitty + PostgreSQL。[^docker-mm]

[^docker-mm]: maxking. (n.d.). docker-mailman. Retrieved 2026-09-20, from https://github.com/maxking/docker-mailman

### 2. 原始碼安裝

```bash
pip install mailman
mailman init
mailman start
```

### 3. 套件管理

Debian/Ubuntu 可用 `apt install mailman`（主要為 Mailman 2）。安裝後需設定 MTA（建議 Postfix）將郵件路由至 Mailman Core 的 LMTP 埠 8024，並以反向代理（Nginx / Caddy）提供 HTTPS。[^install-guide]

[^install-guide]: Mailman 3 Documentation. (n.d.). Pre-Installation Guide. Retrieved 2026-09-20, from https://docs.list.org/en/latest/pre-installation-guide.html

## 與其他方案比較

### Mailman 3 vs Sympa vs Mailtrain

| 面向 | Mailman 3 | Sympa | Mailtrain |
|------|-----------|-------|-----------|
| 用途 | 討論列表 | 企業/學術列表 | 電子報（單向） |
| 語言 | Python | Perl | Node.js |
| 雙向討論 | ✅ | ✅ | ❌ |
| REST API | ✅ 完整 | 有限 | ✅ |
| LDAP 整合 | 外掛 | ✅ 原生 | ❌ |
| SSO 支援 | OAuth2 | Shibboleth/CAS/SAML | ❌ |[^comparison]

[^comparison]: Pistack. (2026). Mailman 3 vs Sympa vs Mailtrain. Retrieved 2026-09-20, from https://www.pistack.xyz/posts/mailman-3-vs-sympa-vs-mailtrain-self-hosted-mailing-list-guide-2026/

### Mailman vs Groups.io（SaaS）

Mailman 需要自行維護伺服器與郵件送達率，但資料完全自主掌控；Groups.io 為 SaaS 方案，零維護但有供應商鎖定風險。[^groupsio]

[^groupsio]: Groups.io. (n.d.). Mailman vs Groups.io. Retrieved 2026-09-20, from https://groups.io/static/mailman-vs-groups-io

### Mailman vs Discourse（郵件列表模式）

Mailman 以郵件為第一介面，Discourse 以網頁論壇為主、郵件為輔。Mailman 適合以 email 為核心的工作流程，Discourse 適合需要論壇功能（徽章、信任系統、外掛生態系）的社群。[^discourse]

[^discourse]: Discourse. (n.d.). Features. Retrieved 2026-09-20, from https://www.discourse.org/features

## 優缺點摘要

**優點**：自由開源、成熟穩定（1999 年起）、功能完整（退信/過濾/DMARC）、社群龐大、REST API 可程式化整合、HyperKitty 提供現代化封存體驗。

**缺點**：安裝較複雜（多元件）、郵件送達率需自行維護、非設計給單向電子報使用、Mailman 3 部分進階設定尚無 Web UI。

## 結論

GNU Mailman 是開源郵件列表管理的事實標準。Mailman 3 代表其現代化方向——採用 REST API、Django 前端、SQL 資料庫與全文搜尋封存，適合從社群專案到大型組織的各種場景。對於以 email 為核心的社群溝通，它仍是最成熟的自託管選擇。