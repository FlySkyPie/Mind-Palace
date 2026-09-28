# openDCIM 是否支援非標準／自訂機櫃尺寸？

openDCIM 是一套免費、開源的資料中心基礎設施管理（DCIM）網頁應用程式，採 GPL v3 授權[^opendcim-site]。本報告探討 openDCIM 對非標準機櫃尺寸（如自訂高度、寬度、深度）的支援程度。

## 自訂機櫃高度（U 數）— ✅ 支援

openDCIM **完全支援自訂機櫃高度**，不限制為標準 42U。資料庫 `fac_Cabinet` 表格中的 `CabinetHeight` 欄位為純整數（`int(11)`）[^schema]，在機櫃管理介面（`cabinets.php`）中，該欄位為自由輸入的文字框，僅驗證為可選的數字與空格，無預設選項或下拉選單[^cabinets-php]：

```html
<div><input type="text" class="validate[optional,custom[onlyNumberSp]]" name="cabinetheight" ...></div>
```

`Cabinet.class.php` 建構子中，`CabinetHeight` 僅以 `intval()` 儲存，無固定清單或限制[^cabinet-class]。

官方 Wiki 說明文件指出：
> **CabinetHeight** — 機櫃的高度，以 RU 為單位。目前最常見的是 42U，且能通過大多數門[^wiki]。

因此任何整數 U 值（1U、2U、10U、27U、48U、54U 等）皆可輸入。

## 自訂機櫃寬度（如非 19 吋）— ❌ 不支援

openDCIM 的資料模型中**完全沒有機櫃寬度的概念**。`fac_Cabinet` 表格中不存在 `CabinetWidth` 欄位[^schema]，`Cabinet.class.php` 中沒有寬度或深度屬性[^cabinet-class]，機櫃管理表單也沒有寬度/深度輸入框。

系統中唯一與寬度相關的概念是**2D 地圖座標**（`MapX1`、`MapX2`、`MapY1`、`MapY2`），這些座標用於在資料中心平面圖上定義可點擊的矩形區域，目的是導航而非表示實體機櫃尺寸[^cabinet-class]。

### GitHub Issue #1273 的證據

2021 年 5 月 24 日，一位使用者提交了功能請求（Issue #1273）[^issue-1273]：
> 「請問是否有支援其他機櫃尺寸，例如 10 吋機櫃？對我來說一些裝置特別有用。」

此請求**未有任何開發人員回覆或採取任何行動即被關閉**，未附加任何評論或程式碼變更。這進一步確認 openDCIM 目前不支援非標準機櫃寬度。

## 其他非標準機櫃幾何 — ❌ 不支援

- **機櫃深度**：資料模型中無相關概念。
- **自訂機櫃寬度**：完全不支援。
- **半深裝置**：`fac_Device` 表格有 `HalfDepth` 布林欄位（`tinyint(1)`）[^schema]，但這是裝置層級的屬性（用於標示深度僅為機櫃一半的裝置），非機櫃層級的尺寸。

## 現有可行替代方案

唯一能表示非標準機櫃尺寸的方式是透過**自由文字欄位**，例如在 Model（型號）或 Notes（備註）欄位中標註（如「10 吋寬機櫃」），但這些欄位不會影響任何計算、視覺化或報表。機櫃立面圖僅考量 `CabinetHeight`（垂直 U 槽數量）— 寬度始終被視為單一欄位（標準 19 吋）。

## 總結

| 功能 | 是否支援 | 說明 |
|------|----------|------|
| 自訂機櫃高度（任意 U 值） | ✅ 支援 | 自由輸入整數欄位，無限制 |
| 自訂機櫃寬度（非 19 吋） | ❌ 不支援 | 資料模型中無寬度欄位 |
| 自訂機櫃深度 | ❌ 不支援 | 資料模型中無深度欄位 |
| 10 吋機櫃支援 | ❌ 不支援 | GitHub Issue #1273 關閉未處理 |
| 半深裝置 | ✅ 部分支援 | 裝置層級有 `HalfDepth` 旗標，但非機櫃層級深度 |

## 參考資料

[^opendcim-site]: openDCIM. (n.d.). openDCIM - Open Source Data Center Infrastructure Management. Retrieved 2026-09-27, from https://opendcim.org/
[^schema]: openDCIM. (n.d.). create.sql - fac_Cabinet table schema. Retrieved 2026-09-27, from https://raw.githubusercontent.com/opendcim/openDCIM/master/create.sql
[^cabinets-php]: openDCIM. (n.d.). cabinets.php - Cabinet management UI. Retrieved 2026-09-27, from https://github.com/opendcim/openDCIM/blob/master/cabinets.php
[^cabinet-class]: openDCIM. (n.d.). Cabinet.class.php - Cabinet class. Retrieved 2026-09-27, from https://raw.githubusercontent.com/opendcim/openDCIM/master/classes/Cabinet.class.php
[^wiki]: openDCIM. (n.d.). Infrastructure Management. Retrieved 2026-09-27, from https://github-wiki-see.page/m/opendcim/openDCIM/wiki/Infrastructure-Management
[^issue-1273]: GitHub user. (2021, May 24). Issue #1273 - Additional Cabinet Size like 10in Rack. Retrieved 2026-09-27, from https://github.com/opendcim/openDCIM/issues/1273