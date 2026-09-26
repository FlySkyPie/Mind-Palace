# Part-DB 開源替代方案調查

## 背景

Part-DB 是一套電子零件庫存管理系統，用於追蹤電阻、電容、IC 等電子元件的庫存數量、存放位置、供應商資訊，並提供 BOM (Bill of Materials) 管理、條碼標籤生成、供應商價格追蹤等功能。其官方儲存庫位於 GitHub: https://github.com/Part-DB/Part-DB-server，採用 AGPL-3.0 授權，以 PHP/Symfony 開發[^pdb-server]。

本報告調查並比較 Part-DB 的 FOSS（自由及開源軟體）替代方案，協助使用者根據自身需求選擇最合適的工具。

## 調查結果

### 1. InvenTree 🏆 首選推薦

| 屬性 | 內容 |
|------|------|
| **GitHub** | https://github.com/inventree/InvenTree |
| **Stars** | 7,600+ |
| **授權** | MIT |
| **技術棧** | Python / Django + React |
| **狀態** | ✅ 積極維護中 |

InvenTree 是目前最受歡迎且最活躍的開源電子零件庫存管理系統，擁有最大的社群與最完整的生態系。它涵蓋零件追蹤、庫存控制、BOM 管理、供應商管理以及生產製造支援[^inventree]。

**核心功能：**
- 零件分類與參數化搜尋
- 多倉位庫存管理與盤點
- 多層級 BOM (Bill of Materials) 管理，自動計算可用庫存
- 供應商零件連結（Digi-Key、Mouser 等）與價格/交期追蹤
- 採購單與生產單管理
- REST API 與 Python 模組供外部整合
- 外掛系統可擴充功能
- KiCad 整合（透過 Ki-nTree 及官方外掛）
- **原生行動 App**（Android & iOS）
- Docker 一鍵部署
- 支援 PostgreSQL、MySQL、MariaDB、SQLite
- 39 種語言翻譯

**獨特優勢：** 比 Part-DB 更大的社群規模、現代化 React 前端、行動 App 支援、MIT 更寬鬆授權、活躍外掛生態。InvenTree 文件明確提到 PartKeepr 是「有價值的先驅與靈感來源」[^inventree-readme]。

---

### 2. PartKeepr ⚠️ 已封存

| 屬性 | 內容 |
|------|------|
| **GitHub** | https://github.com/partkeepr/PartKeepr |
| **Stars** | 1,500+ |
| **授權** | GPL-3.0 |
| **技術棧** | PHP / Symfony2 |
| **狀態** | ❌ **2025 年 7 月封存，唯讀狀態** |

PartKeepr 曾是電子零件庫存管理的首選方案，但官方儲存庫已於 2025 年 7 月 2 日封存，不再維護。它需要 PHP 7.0-7.1（已極度過時），不建議用於新部署[^partkeepr]。

**歷史功能：**
- 零件分類與進階搜尋
- 封裝 (Footprint) 管理與自動圖片顯示
- 條碼生成
- 多使用者權限系統
- REST API（JSON-LD）
- 庫存位置管理

**遷移建議：** PartKeepr 使用者建議遷移至 InvenTree 或 Part-DB v1。

---

### 3. Part-DB v1（對照參考）

| 屬性 | 內容 |
|------|------|
| **GitHub** | https://github.com/Part-DB/Part-DB-server |
| **Stars** | 1,800+ |
| **授權** | AGPL-3.0 |
| **技術棧** | PHP / Symfony 7 |
| **狀態** | ✅ 積極維護中 |

此為 Part-DB 現代化重寫版本（取代已棄用的 Part-DB-legacy），採用 Symfony 6+ 開發。具備下列特色功能[^pdb-server]：

- 條碼/標籤生成 + 網路攝影機掃描
- 專案管理與 BOM 追蹤
- KiCad 整合
- 雲端供應商整合（Octopart、Digi-Key、Farnell、LCSC、TME）
- AI 網頁資料萃取（從任意購物網站）
- 瀏覽器外掛快速提交零件
- SAML SSO、2FA（TOTP + Webauthn/U2F）
- MCP Server 供 AI Agent 存取
- 事件日誌與版本還原

---

### 4. Ki-nTree（InvenTree + KiCad 橋接工具）

| 屬性 | 內容 |
|------|------|
| **GitHub** | https://github.com/sparkmicro/Ki-nTree |
| **Stars** | 250+ |
| **授權** | GPL-3.0 |
| **技術棧** | Python |
| **狀態** | ✅ 積極維護中 |

Ki-nTree 是一個自動化零件建立工具，作為 KiCad 與 InvenTree 之間的橋樑。它能從供應商 API 自動擷取零件資料，同時建立 KiCad 符號、封裝、3D 模型以及 InvenTree 零件記錄[^kintree]。

**核心功能：**
- 從 Digi-Key、Mouser、Element14、LCSC、TME API 自動擷取零件資料
- 自動產生 KiCad 符號、封裝與 3D 模型
- 在 InvenTree 中建立含完整參數與價格的零件
- 同步 KiCad 與 InvenTree 之間的零件資料
- 圖形化操作介面

**注意：** 這不是完整的庫存管理系統，而是 InvenTree 使用者的輔助工具。

---

### 5. InteractiveHtmlBom（BOM 視覺化工具）

| 屬性 | 內容 |
|------|------|
| **GitHub** | https://github.com/openscopeproject/InteractiveHtmlBom |
| **Stars** | 4,600+ |
| **授權** | MIT |
| **技術棧** | Python |
| **狀態** | ✅ 積極維護中 |

InteractiveHtmlBom 是一個 BOM 生成外掛，支援 KiCad、EasyEDA、Eagle、Fusion360、Allegro，能產生互動式 HTML BOM 搭配 PCB 視覺渲染。它不是完整庫存管理系統，而是優秀的 BOM 視覺化工具[^ibom]。

---

### 6. Odoo Inventory（通用 ERP 庫存模組）

| 屬性 | 內容 |
|------|------|
| **GitHub** | https://github.com/odoo/odoo |
| **Stars** | 42,000+（整個 Odoo 專案） |
| **授權** | LGPL-3.0（社群版） |
| **技術棧** | Python、PostgreSQL |
| **狀態** | ✅ 極度活躍 |

Odoo 是成熟的通用 ERP 系統，其庫存與 MRP 模組能涵蓋部分零件庫存管理需求，但它並非專為電子零件設計，缺乏參數化搜尋（如電阻值、容值查詢）、封裝管理等電子業特定功能，對業餘愛好者而言過於龐大[^odoo]。

---

## 比較總表

| 專案 | 範疇 | 活躍狀態 | Stars | 授權 | 技術棧 | 適用場景 |
|------|------|----------|-------|------|--------|----------|
| **InvenTree** | 完整庫存 + BOM + 製造 | ✅ 活躍 | 7.6k | MIT | Python/Django | **最佳全方位方案** |
| **Part-DB v1** | 完整零件庫存 | ✅ 活躍 | 1.8k | AGPL | PHP/Symfony | Part-DB 升級使用者 |
| **PartKeepr** | 零件庫存 | ❌ 已封存 | 1.5k | GPL | PHP/Symfony2 | 僅供歷史參考 |
| **Ki-nTree** | 零件建立橋接 | ✅ 活躍 | 251 | GPL | Python | InvenTree + KiCad 使用者 |
| **InteractiveHtmlBom** | BOM 視覺化 | ✅ 活躍 | 4.6k | MIT | Python | BOM 檢視需求 |
| **Odoo** | 通用 ERP | ✅ 極活躍 | 42k+ | LGPL | Python | 已有 Odoo 部署者 |

---

## 總結與建議

**InvenTree 是 Part-DB 最推薦的 FOSS 替代方案**，理由如下：

1. **最大社群與最活躍開發** — 7,600+ Stars、77 Watchers、18,000+ Commits
2. **現代化技術棧** — Python/Django 後端 + React 前端，相較 Part-DB 的 PHP/Symfony 更現代
3. **行動 App 支援** — 原生 Android 與 iOS App，可在現場掃描條碼管理庫存
4. **完整 KiCad 整合** — 透過 Ki-nTree 無縫建立零件
5. **外掛系統** — 無需修改核心即可擴充功能
6. **MIT 寬鬆授權** — 比 Part-DB 的 AGPL-3.0 更具彈性
7. **功能對等** — 涵蓋 Part-DB 所有核心功能（BOM 管理、供應商連結、庫存追蹤、條碼、參數化搜尋、REST API）

若偏好 PHP 生態或需要 Part-DB 特有的 AI 網頁資料萃取功能，則 **Part-DB v1 本身** 仍是活躍開發中的優秀選擇。

## 參考來源

[^pdb-server]: Part-DB. (n.d.). Part-DB Server. Retrieved 2026-09-25, from https://github.com/Part-DB/Part-DB-server
[^inventree]: InvenTree. (n.d.). InvenTree. Retrieved 2026-09-25, from https://github.com/inventree/InvenTree
[^inventree-readme]: InvenTree. (n.d.). InvenTree README. Retrieved 2026-09-25, from https://github.com/inventree/InvenTree#readme
[^partkeepr]: PartKeepr. (n.d.). PartKeepr. Retrieved 2026-09-25, from https://github.com/partkeepr/PartKeepr
[^kintree]: Spark Micro. (n.d.). Ki-nTree. Retrieved 2026-09-25, from https://github.com/sparkmicro/Ki-nTree
[^ibom]: OpenScope Project. (n.d.). InteractiveHtmlBom. Retrieved 2026-09-25, from https://github.com/openscopeproject/InteractiveHtmlBom
[^odoo]: Odoo. (n.d.). Odoo. Retrieved 2026-09-25, from https://github.com/odoo/odoo