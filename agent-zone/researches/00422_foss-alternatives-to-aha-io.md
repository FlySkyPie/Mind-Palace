# Aha! (aha.io) 的 FOSS 替代方案研究

## 概述

Aha! 是一套產品藍圖 (Product Roadmap) 軟體，主要用於產品策略規劃、功能優先級排序、版本／發佈規劃，以及向利害關係人溝通產品方向。本報告調查其自由開源 (FOSS) 替代方案。

---

## 替代方案一覽

### 1. OpenProject

- **網站：** <https://www.openproject.org>
- **說明：** 最成熟的開源專案管理軟體之一，內建專屬的**產品藍圖 (Product Roadmap)** 模組，支援版本／發佈規劃與藍圖視覺化，是功能上最接近 Aha! 的開源選擇。
- **主要功能：** 產品藍圖、版本／發佈規劃、工作套件 (Work Packages)、甘特圖、敏捷板 (Scrum/Kanban/SAFe)、時間追蹤、預算管理、會議管理、GitHub/GitLab 整合、Wiki、討論區。
- **授權：** GPL-3.0
- **部署方式：** 自架 (Docker、套件管理) + 雲端 SaaS

### 2. Plane

- **網站：** <https://plane.so>
- **說明：** 現代化 UI 的開源專案管理平台，定位為 Jira/Linear/ClickUp 的替代品。提供 Initiatives（倡議）與 Epics（史詩）層級，適合高層次產品藍圖規劃。
- **主要功能：** Initiatives & Epics（藍圖層級規劃）、循環／衝刺 (Cycles/Sprints)、甘特圖、多種視圖（看板、試算表、列表、甘特）、AI 功能、Wiki、儀表板、可從 Jira/Linear/Asana/ClickUp 遷移。
- **授權：** AGPL-3.0
- **部署方式：** 自架 (Docker、Kubernetes) + 雲端 SaaS

### 3. WeKan

- **網站：** <https://wekan.fi>（原始碼：<https://github.com/wekan/wekan>）
- **說明：** MIT 授權的協作式看板應用，支援 Scrum 流程並內建**藍圖 (Roadmap)** 視圖。已被組織用於多達 30,000 名使用者的規模。
- **主要功能：** Scrum Roadmap 視圖、產品待辦清單、衝刺規劃與報告、速度追蹤、甘特圖、泳道、多板日曆、IFTTT 規則引擎、可從 Trello/Jira/Asana 匯入、支援 234 種語言。
- **授權：** MIT
- **部署方式：** 自架 (Snap、Docker、Kubernetes、AppImage) + 雲端 SaaS

### 4. Leantime

- **網站：** <https://leantime.io>
- **說明：** 以目標為導向的專案管理系統，專為「非專案經理」設計，考量 ADHD、自閉症與讀寫障礙使用者的體驗。將策略性目標與具體任務連結。
- **主要功能：** 目標與指標追蹤、里程碑（含時間線／甘特視圖）、看板／表格／列表視圖、點子板、回顧、時間表、白板、目標與任務連結、個人工作儀表板、AI 功能。
- **授權：** AGPL-3.0
- **部署方式：** 自架 (Docker、PHP + MySQL/MariaDB) + 代管／SaaS

### 5. Taiga

- **網站：** <https://www.taiga.io>
- **說明：** 專為跨職能敏捷團隊打造的開源專案管理工具。提供待辦清單管理、衝刺規劃與看板，適合敏捷脈絡下的產品藍圖規劃。
- **主要功能：** 待辦清單管理、衝刺、看板、Epics／User Stories、Wiki、多種專案視圖、角色權限管理、敏捷報表。
- **授權：** MPL-2.0
- **部署方式：** 自架 (Docker) + 雲端 SaaS

### 6. Vikunja

- **網站：** <https://vikunja.io>
- **說明：** 自架任務管理工具，強調「你真正擁有的任務管理器」。提供甘特圖視圖，適合時間線式的產品藍圖規劃。
- **主要功能：** 甘特圖、看板、表格視圖、列表視圖、任務關係（子任務、阻斷任務）、標籤、儲存篩選、CalDAV 整合、分享連結、快速新增、附件。
- **授權：** AGPL-3.0
- **部署方式：** 自架 (Docker、二進位套件) + 雲端 SaaS

### 7. Focalboard

- **網站：** <https://www.focalboard.com>（原始碼：<https://github.com/mattermost-community/focalboard>）
- **說明：** Mattermost 社群維護的開源看板工具，簡單易用，支援桌面用戶端與伺服器版。（注意：獨立版本目前處於維護模式。）
- **主要功能：** 看板、目標追蹤、卡片式任務管理、多板管理、與 Mattermost 整合。
- **授權：** AGPL-3.0（Mattermost 家族專案）
- **部署方式：** 自架 (Ubuntu 個人伺服器、Docker) + 桌面應用 (macOS、Windows、Linux)

### 8. Kanboard

- **網站：** <https://kanboard.org>（原始碼：<https://github.com/kanboard/kanboard>）
- **說明：** 極簡、輕量的自架看板軟體。效能佳，適合不需要複雜功能的使用者。（目前處於維護模式。）
- **主要功能：** 看板、任務管理、時間追蹤、分析、多專案、使用者管理、Markdown 支援、Webhook、API。
- **授權：** MIT
- **部署方式：** 自架 (Docker、PHP + MySQL/PostgreSQL/SQLite)

### 9. AppFlowy

- **網站：** <https://www.appflowy.com>
- **說明：** 開源的 Notion 替代品，提供整合式協作空間。雖然更偏向通用知識管理，但其資料庫與看板功能亦可作為藍圖追蹤用途。
- **主要功能：** 資料庫／試算表、看板、Wiki／文件、AI 功能、跨平台 (macOS、Windows、Linux、iOS、Android)、Flutter + Rust 技術棧。
- **授權：** AGPL-3.0
- **部署方式：** 自架 + 雲端 SaaS + 桌面應用

### 10. Restya

- **網站：** <https://restya.com>（原始碼：<https://github.com/restyaboard/Restyaboard>）
- **說明：** 開源的 Trello 類似看板工具。可用於視覺化產品待辦清單與發佈規劃。
- **主要功能：** 多板、卡片管理、核對清單、附件、標籤、自訂欄位、投票、日曆視圖、甘特視圖、整合。
- **授權：** AGPL-3.0（社群版）／ Proprietary（企業版）
- **部署方式：** 自架 (Docker) + 雲端 SaaS

---

## 綜合比較

| 工具 | 授權 | 自架 | SaaS | 產品藍圖能力 |
|---|---|---|---|---|
| **OpenProject** | GPL-3.0 | ✅ | ✅ | 專屬產品藍圖模組、版本／發佈規劃 |
| **Plane** | AGPL-3.0 | ✅ | ✅ | Initiatives、Epics、Cycles |
| **WeKan** | MIT | ✅ | ✅ | Scrum Roadmap 視圖、產品待辦清單 |
| **Leantime** | AGPL-3.0 | ✅ | ✅ | 目標、里程碑、甘特時間線 |
| **Taiga** | MPL-2.0 | ✅ | ✅ | 待辦清單與衝刺管理 |
| **Vikunja** | AGPL-3.0 | ✅ | ✅ | 甘特圖、任務關聯 |
| **Focalboard** | AGPL-3.0 | ✅ | ❌ | 看板式專案組織 |
| **Kanboard** | MIT | ✅ | ❌ | 極簡看板 |
| **AppFlowy** | AGPL-3.0 | ✅ | ✅ | 資料庫、看板、Wiki |
| **Restya** | AGPL-3.0 | ✅ | ✅ | 看板式規劃 |

---

## 建議

- **若需最完整的產品藍圖功能**（版本／發佈規劃、利害關係人溝通）：**OpenProject** 是最接近 Aha! 的開源選擇，因其內建專門的產品藍圖模組。
- **若偏好現代化 UI 與 Initiative 層級規劃**：**Plane** 是出色的選擇，UI 品質高且功能完整。
- **若需輕量但功能完整的選項**：**WeKan** 以 MIT 授權提供 Scrum Roadmap 視圖，部署靈活。
- **若注重目標導向規劃與包容性設計**：**Leantime** 是獨特選擇，將策略目標與日常任務連結。

---

## 參考資料

[^plane-gh]: Plane. (n.d.). Plane — Open-source project management. Retrieved 2026-10-01, from https://github.com/makeplane/plane
[^plane-web]: Plane. (n.d.). Plane — Project management tool. Retrieved 2026-10-01, from https://plane.so
[^openproject-gh]: OpenProject. (n.d.). OpenProject. Retrieved 2026-10-01, from https://github.com/opf/openproject
[^openproject-web]: OpenProject. (n.d.). OpenProject — open-source project management software. Retrieved 2026-10-01, from https://www.openproject.org
[^openproject-roadmap]: OpenProject. (n.d.). Product Development. Retrieved 2026-10-01, from https://www.openproject.org/collaboration-software-features/product-development/
[^leantime-gh]: Leantime. (n.d.). Leantime. Retrieved 2026-10-01, from https://github.com/Leantime/leantime
[^leantime-web]: Leantime. (n.d.). Leantime. Retrieved 2026-10-01, from https://leantime.io
[^wekan-gh]: WeKan. (n.d.). WeKan. Retrieved 2026-10-01, from https://github.com/wekan/wekan
[^taiga-gh]: Taiga. (n.d.). Taiga. Retrieved 2026-10-01, from https://github.com/kaleidos-ventures/taiga
[^vikunja-gh]: Vikunja. (n.d.). Vikunja. Retrieved 2026-10-01, from https://github.com/go-vikunja/vikunja
[^vikunja-web]: Vikunja. (n.d.). Features. Retrieved 2026-10-01, from https://vikunja.io/features/
[^focalboard-gh]: Focalboard. (n.d.). Focalboard. Retrieved 2026-10-01, from https://github.com/mattermost-community/focalboard
[^kanboard-gh]: Kanboard. (n.d.). Kanboard. Retrieved 2026-10-01, from https://github.com/kanboard/kanboard
[^appflowy-gh]: AppFlowy. (n.d.). AppFlowy. Retrieved 2026-10-01, from https://github.com/AppFlowy-IO/AppFlowy