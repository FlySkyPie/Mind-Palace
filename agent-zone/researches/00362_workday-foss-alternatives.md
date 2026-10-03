# Workday 的自由開源替代方案

Workday 是一套企業級人力資源、薪資、勞動力管理與財務管理 SaaS 平台。本報告整理可替代 Workday 的自由開源（FOSS）方案，涵蓋 HR、薪資、出勤、與 ERP 等功能範疇。

## 1. Frappe HR

- **說明：** Frappe HR 是專注於 HR 與薪資的開源方案，從 ERPNext 獨立拆分而來。涵蓋員工生命週期管理、請假與出勤、費用申報、績效管理、薪資與稅務，並提供行動應用程式。
- **主要功能：** 員工入職/轉調/離職管理、請假與出勤（含地理定位打卡）、費用申報、績效考核（目標/KRA）、薪資（薪資結構、所得稅級距、非週期付款）、招募、班次/輪值管理、行動 PWA 應用、REST API、多公司支援。
- **授權：** GPL-3.0
- **網站：** <https://frappe.io/hr> | GitHub: <https://github.com/frappe/hrms>
- **知名使用者：** Zerodha（印度最大股票經紀商）、IFTAS、SELCO、Jiva 等。[^frappe-hr]
- **與 Workday 比較：** Frappe HR 專注於 HR 與薪資，不含財務管理模組；若需財務功能，可搭配同體系的 ERPNext。最接近 Workday 的 HR 核心功能，但欠缺原生財務模組。

## 2. ERPNext

- **說明：** 全功能開源 ERP，包含會計、CRM、銷售、採購、製造、專案管理與完整的**人力資源**模組（招募、員工管理、請假、出勤、薪資、費用、績效）。
- **主要功能：** 會計、CRM、HR（員工資料、出勤、請假、薪資）、製造、庫存、採購、專案管理、電子商務、POS、醫療/教育領域模組、多公司/多貨幣、REST API、低代碼自訂。
- **授權：** GPL-3.0-only
- **網站：** <https://frappe.io/erpnext> | GitHub: <https://github.com/frappe/erpnext>
- **知名使用者：** MIT、Libre Technologies 等。[^erpnext-wiki]
- **與 Workday 比較：** ERPNext 是最接近 Workday 全功能的方案，同時涵蓋 HR 與企業財務管理。Workday 也涵蓋財務管理，因此 ERPNext 是最全面的替代選擇。

## 3. OrangeHRM

- **說明：** 全面的 HRM 系統，涵蓋企業 HR 功能。提供自託管開源版（Starter，GPL-3.0）與付費雲端版。全球超過 500 萬使用者。
- **主要功能：** 員工管理、請假管理、時間與出勤、招募、入職、績效管理、訓練、職涯發展、報表分析、行動應用、AI 功能、薪資/生產力工具連接。
- **授權：** GPL-3.0
- **網站：** <https://www.orangehrm.com/> | GitHub: <https://github.com/orangehrm/orangehrm>
- **知名使用者：** Rutgers 大學、Puma、Toyota、Stanley Black & Decker 等。[^orangehrm]
- **與 Workday 比較：** OrangeHRM 專注於 HR，不包含財務管理或完整薪資模組（薪資為付費附加）。適合只需 HR 核心功能、不需 ERP 的組織。

## 4. Odoo（含 HR 模組）

- **說明：** 模組化開源 ERP，包含「Employees」HR 模組。社群版完全開源（LGPL-3.0）。可僅使用 HR 應用（招募、員工、請假、績效、費用），也可擴展至 ERP。
- **主要功能：** 員工目錄、請假管理、出勤監控、績效考核、招募管理、入職計畫、費用管理、數位簽章、組織圖、技能追蹤、與會計/薪資整合。
- **授權：** LGPL-3.0（社群版）
- **網站：** <https://www.odoo.com/app/employees> | GitHub: <https://github.com/odoo/odoo>
- **知名使用者：** Airbus、Toyota、Hyundai 等，全球超過 1200 萬使用者。[^odoo]
- **與 Workday 比較：** Odoo 是最接近 Workday 的商業開源方案之一，具備完整 HR 與 ERP 功能，且模組化可按需選用。但社群版缺少部分進階功能（如完整薪資），需付費企業版。

## 5. Dolibarr ERP CRM

- **說明：** 現代化開源 ERP/CRM，包含人力資源管理模組（員工管理、請假、費用、招募、工時表）。約 100 個模組涵蓋會計、CRM、庫存、發票、POS、製造等。**尚無完整薪資模組。**
- **主要功能：** 員工管理、請假管理、費用申報、招募管理、工時表、會計、CRM、發票、POS、庫存、製造（BOM）、REST/SOAP API、多語言、多貨幣。
- **授權：** GPL-3.0+
- **網站：** <https://www.dolibarr.org/> | GitHub: <https://github.com/Dolibarr/dolibarr>
- **知名使用者：** 全球超過 1,000+ 附加元件在 Dolistore 社群。[^dolibarr]
- **與 Workday 比較：** 輕量級方案，適合中小型組織。若薪規管理是必要功能，則不適用。

## 6. Kimai（工時追蹤）

- **說明：** 開源工時追蹤應用，涵蓋 Workday 勞動力管理中工時表、專案時間追蹤、發票與報表功能。非完整 HRMS。
- **主要功能：** 多人計時（無限使用者）、客戶/專案/活動追蹤、發票（可自訂模板、電子發票）、費用追蹤（外掛）、JSON API、SAML/LDAP/SSO、2FA（TOTP）、角色權限、30+ 語言、預算追蹤。
- **授權：** AGPL-3.0
- **網站：** <https://www.kimai.org/> | GitHub: <https://github.com/kimai/kimai>
- **知名使用者：** CodeWeavers、Pulse of Europe e.V.、Fairtrade Deutschland 等。[^kimai]
- **與 Workday 比較：** 僅涵蓋工時追蹤與發票功能，不涵蓋 HR 管理或薪資。適合專案導向團隊作為勞動力管理的補充工具。

## 7. iDempiere

- **說明：** 社群驅動的成熟開源 ERP/CRM/MFG/SCM/POS，源自 ADempiere/Compiere，經數十年驗證。包含 HR & 薪資模組、出勤模組。Java/PostgreSQL/基於 OSGi 外掛架構。
- **主要功能：** 完整 ERP（會計、CRM、製造、供應鏈、POS）、HR & 薪資、時間與出勤、貸款管理、財務管理、庫存、專案管理、REST API。
- **授權：** GPL-2.0
- **網站：** <https://www.idempiere.org/> | GitHub: <https://github.com/idempiere/idempiere>
- **知名使用者：** 中小企業至大型企業全球社群。[^idempiere]
- **與 Workday 比較：** iDempiere 與 ADempiere 系列是最接近傳統 ERP 的開源方案，HR 與薪資功能成熟，但技術架構較為陳舊（Java/桌面整合）。

## 8. ADempiere

- **說明：** iDempiere 的前身，同為開源 ERP/CRM/MFG/SCM/POS。包含 `org.eevolution.hr_and_payroll` 與 `org.spin.hr_time_and_attendance` 模組。成熟完整。
- **主要功能：** 會計、CRM、製造、供應鏈、POS、HR & 薪資、時間與出勤、資產管理、庫存、JasperReports 整合。
- **授權：** GPL-2.0
- **網站：** <http://www.adempiere.io> | GitHub: <https://github.com/adempiere/adempiere>
- **知名使用者：** 拉丁美洲、歐洲、亞洲廣泛部署。[^adempiere]
- **與 Workday 比較：** 功能全面但技術棧較舊，適合看重成熟度而非現代化 UI 的組織。

## 9. Axelor Open Suite

- **說明：** 模組化開源商業套件，包含人力資源管理模組與 Talent 模組（招募、技能管理）。亦涵蓋 CRM、銷售、財務、庫存、製造。基於 Axelor Open Platform（Java/Python/Angular）。
- **主要功能：** HR 管理（員工生命週期、請假、費用）、Talent 模組（招募、技能）、CRM、會計財務、專案管理、供應鏈、製造、車隊管理、品質管理、多公司/多貨幣/多語言。
- **授權：** AGPL-3.0
- **網站：** <https://axelor.com> | GitHub: <https://github.com/axelor/axelor-open-suite>
- **知名使用者：** 法國及歐洲的製造、服務、分銷產業。[^axelor]
- **與 Workday 比較：** 功能全面但社群相對較小；適合歐洲企業。

## 10. Tryton

- **說明：** 100% 開源商業軟體/ERP，以整潔程式碼與資料完整性著稱。包含會計、銷售、採購、庫存、人力資源模組。模組化架構。
- **主要功能：** 會計（總帳、資產、發票）、CRM、銷售與採購、庫存與物流、人力資源、專案管理、製造、分析、多公司/多貨幣、REST API、工作流程引擎。
- **授權：** GPL-3.0
- **網站：** <https://www.tryton.org/> | GitHub: <https://github.com/tryton/tryton>
- **知名使用者：** 歐洲廣泛使用，尤其是法國合作社與協會。[^tryton]
- **與 Workday 比較：** 薪資功能有限，適合需要輕量 HR 功能搭配完整 ERP 的組織。

## 11. OpenCATS（僅招募/ATS）

- **說明：** 免費開源申請者追蹤系統（ATS）與招募 CRM，僅涵蓋 Workday 的招募功能。
- **主要功能：** 候選人管理、職位發佈、申請追蹤、履歷解析、招募 CRM、電子郵件整合、日曆整合、報表、JSON API。
- **授權：** AGPL-3.0
- **網站：** <https://www.opencats.org> | GitHub: <https://github.com/opencats/OpenCATS>
- **知名使用者：** 招募機構與 HR 部門。[^opencats]
- **與 Workday 比較：** 僅涵蓋招募環節，不涵蓋 HR 管理中其他功能。可與其他 HR 系統搭配使用。

## 比較總結

| 方案 | HR | 薪資 | 出勤 | 財務/ERP | 最佳適用場景 |
|---|---|---|---|---|---|
| **Frappe HR** | ✅ 完整 | ✅ 完整 | ✅ 完整 | ✅（ERPNext） | 專注 HR + 薪資的開源方案 |
| **ERPNext** | ✅ 完整 | ✅ 完整 | ✅ 完整 | ✅ 完整 | 全方位替代 Workday |
| **OrangeHRM** | ✅ 完整 | ❌（付費附加） | ✅ 完整 | ❌ | 獨立 HRMS |
| **Odoo** | ✅ 完整 | ✅（會計整合） | ✅ 完整 | ✅ 完整 | 模組化 ERP + HR |
| **Dolibarr** | ✅ 部分 | ❌ | ✅ 部分 | ✅ 完整 | 輕量 ERP/CRM |
| **Kimai** | ❌（僅工時） | ❌ | ✅ 完整 | ❌（僅發票） | 工時追蹤 |
| **iDempiere** | ✅ 完整 | ✅ 完整 | ✅ 完整 | ✅ 完整 | 成熟企業 ERP |
| **ADempiere** | ✅ 完整 | ✅ 完整 | ✅ 完整 | ✅ 完整 | 傳統企業 ERP |
| **Axelor** | ✅ 完整 | ❌ 部分 | ✅ 部分 | ✅ 完整 | 模組化商業套件 |
| **Tryton** | ✅ 部分 | ✅ 部分 | ❌ 有限 | ✅ 完整 | 整潔現代 ERP |
| **OpenCATS** | ❌（僅ATS） | ❌ | ❌ | ❌ | 招募專用 |

## 結論

- 若需要**最全面的 Workday 替代**（同時涵蓋 HR 與財務管理），**ERPNext** 或 **Odoo** 是最佳選擇。
- 若僅需要**HR 核心功能**（員工管理、請假、薪資），**Frappe HR** 或 **OrangeHRM** 已經足夠。
- 若需要**傳統成熟 ERP**，**iDempiere/ADempiere** 提供歷經考驗的功能。
- 若只需**工時與勞動力管理**，**Kimai** 是專注且成熟的选择。

[^frappe-hr]: Frappe. (n.d.). Frappe HR — The World's #1 Open Source HR and Payroll Software. Retrieved 2026-10-03, from https://frappe.io/hr
[^erpnext-wiki]: Wikipedia. (n.d.). ERPNext. Retrieved 2026-10-03, from https://en.wikipedia.org/wiki/ERPNext
[^orangehrm]: OrangeHRM. (n.d.). OrangeHRM — Open Source Human Resource Management Software. Retrieved 2026-10-03, from https://www.orangehrm.com/
[^odoo]: Odoo. (n.d.). Odoo Employees — Human Resources Software. Retrieved 2026-10-03, from https://www.odoo.com/app/employees
[^dolibarr]: Dolibarr. (n.d.). Dolibarr ERP CRM — Open Source Business Management Software. Retrieved 2026-10-03, from https://www.dolibarr.org/
[^kimai]: Kimai. (n.d.). Kimai — Open Source Time Tracking. Retrieved 2026-10-03, from https://www.kimai.org/
[^idempiere]: iDempiere. (n.d.). iDempiere — Open Source ERP. Retrieved 2026-10-03, from https://www.idempiere.org/
[^adempiere]: ADempiere. (n.d.). ADempiere — Open Source ERP. Retrieved 2026-10-03, from http://www.adempiere.io
[^axelor]: Axelor. (n.d.). Axelor Open Suite. Retrieved 2026-10-03, from https://axelor.com
[^tryton]: Tryton. (n.d.). Tryton — Open Source Business Software. Retrieved 2026-10-03, from https://www.tryton.org/
[^opencats]: OpenCATS. (n.d.). OpenCATS — Open Source Applicant Tracking System. Retrieved 2026-10-03, from https://www.opencats.org