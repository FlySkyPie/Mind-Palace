# Part-DB 廣義庫存管理 FOSS 替代方案調查

## 背景

Part-DB 是一套開源電子零件庫存管理系統，採用 AGPL-3.0 授權，以 PHP/Symfony 開發，支援存放位置（Storage Location）的樹狀階層管理、容量控制旗標（滿載/單一零件/限既有零件）、擁有者權限控管、條碼標籤生成等功能[^pdb-server]。

本報告跳出電子零件框架，尋找 **廣義庫存管理系統**，以 **存放位置（儲位）管理** 為核心篩選條件，比較各 FOSS 替代方案。

## 主要替代方案

### 1. InvenTree — 最佳綜合替代

| 屬性 | 內容 |
|------|------|
| 授權 | MIT |
| 技術棧 | Python/Django + React |
| GitHub Stars | ~7,600 ⭐ |
| 官方網站 | https://inventree.org |

**存放位置管理：**

InvenTree 支援**樹狀層級（cascading）庫存位置**，可無限深度嵌套。位置類型（Stock Location Type）可自訂圖示與分類（抽屜、層架、箱子等）。支援**結構性位置（Structural Location）**，用於僅作為組織節點而不實際存放物品的位置（如整面層架），而實際物品放在其下層子位置。也支援**外部位置（External Location）**，用於標示非即時可取用的存放點[^inventree-stock]。

**通用庫存能力：**

模型不受限於電子零件。零件可標記為組裝件（Assembly）、元件（Component）、可採購（Purchaseable）、可銷售（Salable）、耗材（Consumable）、虛擬（Virtual）等類型。支援實體單位系統（公釐、公斤、公升、個、打等），可自訂單位。類別（Category）為樹狀結構，可任意分類物品[^inventree-part]。

**限制：**

- 尚無儲位尺寸/容積追蹤（已列入規劃）
- 無進階倉儲管理系統（WMS）功能（揀貨包裝、波次揀貨等）
- 無內建 RFID 整合
- 定位於 SME/玩家等級，可能無法應付企業級交易量

---

### 2. ERPNext — 最完整的 ERP 方案

| 屬性 | 內容 |
|------|------|
| 授權 | GPL-3.0 |
| 技術棧 | Python/Frappe Framework |
| GitHub Stars | ~39,600 ⭐ |
| 官方網站 | https://erpnext.com |

**存放位置管理：**

ERPNext 沒有獨立的「儲位」或「料架」實體，而是透過**倉庫（Warehouse）樹狀結構**來表示所有儲存位置。官方文件明確指出：**「ERPNext 中的『倉庫』一詞較廣義，可以視為『儲存位置』。你可以建立子倉庫來代表實際地點內的層架。」**典型結構為：**倉庫 > 房間 > 走道 > 層架 > 儲位**[^erpnext-warehouse]。

每個葉節點倉庫（對應到實際的儲位/料箱）獨立記錄庫存數量，同一物品可存在於不同儲位。支援**倉庫類型（Warehouse Type）** 標籤，用於分類（如冷藏區、危險品區、待檢區）[^erpnext-warehouse]。

**通用庫存能力：**

作為完整 ERP，庫存管理僅是其一模組。另有採購、銷售、製造、會計、專案、人資等模組。支援序號追蹤、批次/批號追蹤、多公司多倉庫。

**限制：**

- 進階倉儲功能（如動態儲位指派、揀貨路徑最佳化）部分位於企業版
- 學習曲線較陡，需要了解 Frappe 框架
- 若只需庫存管理，完整 ERP 可能過重

---

### 3. OpenBoxes — 倉儲物流專用方案

| 屬性 | 內容 |
|------|------|
| 授權 | EPL-1.0 |
| 技術棧 | Groovy/Grails |
| GitHub Stars | ~900 ⭐ |
| 官方網站 | https://www.openboxes.com |

**存放位置管理：**

由 Partners In Health 為醫療供應鏈（2010 海地地震後）開發，支援三層嵌套結構：**倉庫（Depot）> 區域（Zone Location）> 儲位（Bin Location）**。儲位類型分為：一般儲位（General）、收貨區（Receiving，系統自動建立）、保留儲位（Hold Bin，放入後庫存即不可出貨）、及計劃中的越庫（Cross-docking）[^openboxes-bin]。

每一筆庫存同時追蹤：**儲位位置 × 產品 × 批號/序號 × 有效日期**。完整的揀貨（FEFO 策略）、入庫上架、內部移轉、補貨、循環盤點等倉儲流程支援[^openboxes-wf]。

**通用庫存能力：**

不限特定產業，原始設計雖為醫療，但可管理任何物品。支援多組織多倉庫（總公司可為子公司統一採購）。

**限制：**

- 社群較小（~900 stars），生態系較弱
- Groovy/Grails 技術棧相對小眾，第三方擴充較少
- UI 質感較樸素

---

### 4. Odoo Community Edition — 全方位商業套件

| 屬性 | 內容 |
|------|------|
| 授權 | LGPL-3.0（社群版） |
| 技術棧 | Python |
| GitHub Stars | ~54,700 ⭐ |
| 官方網站 | https://odoo.com |

**存放位置管理：**

庫存/倉庫模組支援多倉庫、多儲位、儲位類別，可定義儲位間的補貨規則與入庫策略（put-away strategies）。支援條碼掃描。儲位同樣可用樹狀層級組織[^odoo-inventory]。

**通用庫存能力：**

完整 ERP 套件：CRM、會計、製造、電商、HR、專案管理等。社群版完全開源。

**限制：**

- 部分進階庫存功能位於企業版
- 模組眾多，自訂與調整需熟悉 Odoo 架構

---

### 5. Grocy — 輕量家庭庫存

| 屬性 | 內容 |
|------|------|
| 授權 | MIT |
| 技術棧 | PHP |
| GitHub Stars | ~9,500 ⭐ |
| 官方網站 | https://grocy.info |

**存放位置管理：**

支援簡單的位置指派（如「冰箱」、「儲藏室」），每個庫存項目可指定一個位置名稱，但**無樹狀層級**，無料架/儲位/料箱的階層結構。

**通用庫存能力：**

專為家庭/食品管理設計。支援有效日期追蹤、購物清單自動生成、食譜管理、家事提醒等。不適合商業或工業用途。

**限制：**

- 地理位置管理僅有基本標籤層級，無階層結構
- 以家庭使用為設計目標，無法應付商業庫存管理

---

### 6. PartKeepr — 已歸檔

| 屬性 | 內容 |
|------|------|
| 授權 | GPL-3.0 |
| 技術棧 | PHP/Symfony |
| GitHub Stars | ~1,500 ⭐（已歸檔） |
| 官方網站 | https://www.partkeepr.org |

原為 Part-DB 的主要競爭對手，同樣專注電子零件管理。2025-07-02 歸檔為唯讀狀態。官方 README 建議轉移至 InvenTree。**不建議新部署使用**[^partkeepr]。

---

### 7. Snipe-IT — IT 資產管理

| 屬性 | 內容 |
|------|------|
| 授權 | AGPL-3.0 |
| 技術棧 | PHP/Laravel |
| GitHub Stars | ~15,000 ⭐ |
| 官方網站 | https://snipeitapp.com |

**非推薦選項**。Snipe-IT 主要設計為 **IT 資產管理**（筆電、伺服器、軟體授權），支援的是「位置」（建築物/樓層/辦公室），而非庫存儲位的料架/儲位/料箱層級。不適合一般物品庫存管理[^snipeit]。

---

## 比較總表

| 系統 | 儲位階層 | 多倉庫 | 批號/序號追蹤 | 通用庫存 | 積極維護 |
|------|----------|--------|--------------|---------|---------|
| **InvenTree** | ✅ 無限樹狀階層 + 位置類型 | ✅ (頂層位置) | ✅ | ✅ | ✅ |
| **ERPNext** | ✅ 倉庫樹狀結構 (走道/層架/儲位) | ✅ | ✅ | ✅ | ✅ |
| **OpenBoxes** | ✅ 三層 (Depot/Zone/Bin) | ✅ (多設施) | ✅ (批號追蹤) | ✅ | ✅ |
| **Odoo CE** | ✅ 倉庫/儲位類別 | ✅ | ✅ | ✅ | ✅ |
| **Grocy** | ⚠️ 單層位置標籤 | ❌ | ⚠️ 有效日期 | ✅ (家庭) | ✅ |
| **PartKeepr** | ✅ 有樹狀結構 | ❌ | ❌ | ❌ (電子) | ❌ 已歸檔 |
| **Snipe-IT** | ❌ 僅建築物/樓層 | ❌ | ❌ | ❌ (IT 資產) | ✅ |

## 推薦建議

### 第一推薦：InvenTree

最接近 Part-DB 的使用體驗，但支援通用庫存。存放位置管理為一級功能（first-class entity），有樹狀階層、位置類型、結構性位置等豐富功能。MIT 授權靈活，Python/Django 技術棧擴充容易。非常適合從 Part-DB 轉移的使用者。

### 第二推薦：ERPNext

若已有或預計導入完整 ERP，ERPNext 的倉庫樹狀結構可以達到同樣細緻的儲位管理，且擁有更強大的採購、銷售、製造、會計整合。適合中型以上組織。

### 第三推薦：OpenBoxes

若倉儲物流為核心需求（多設施、FEFO 揀貨、循環盤點），OpenBoxes 是三者中最專業的選擇。但社群較小，需注意長期維護風險。

---

## 引用

[^pdb-server]: Part-DB. (n.d.). Part-DB-server. Retrieved 2026-09-25, from https://github.com/Part-DB/Part-DB-server

[^inventree-stock]: InvenTree. (n.d.). Stock & Stock Location documentation. Retrieved 2026-09-25, from https://docs.inventree.org/en/stable/stock/

[^inventree-part]: InvenTree. (n.d.). Part Model documentation. Retrieved 2026-09-25, from https://docs.inventree.org/en/stable/part/

[^erpnext-warehouse]: ERPNext. (n.d.). Warehouse documentation. Retrieved 2026-09-25, from https://docs.erpnext.com/docs/v14/user/manual/en/stock/warehouse

[^openboxes-bin]: OpenBoxes. (n.d.). Managing Bin and Zone Locations. Retrieved 2026-09-25, from https://help.openboxes.com/article/289-managing-bin-and-zone-locations

[^openboxes-wf]: OpenBoxes. (n.d.). Location Type and Supported Activities. Retrieved 2026-09-25, from https://help.openboxes.com/article/33-location-type-and-supported-activities

[^odoo-inventory]: Odoo. (n.d.). Inventory documentation. Retrieved 2026-09-25, from https://www.odoo.com/app/inventory

[^partkeepr]: PartKeepr. (n.d.). GitHub repository. Retrieved 2026-09-25, from https://github.com/partkeepr/PartKeepr

[^snipeit]: Snipe-IT. (n.d.). GitHub repository. Retrieved 2026-09-25, from https://github.com/grokability/snipe-it