# DMTF CIM（Common Information Model）概述

## 簡介

**Common Information Model（CIM）** 是由 **Distributed Management Task Force（DMTF）** 制定與維護的開放標準，提供一個通用、物件導向的管理資訊模型，用以描述 IT 環境中各類管理元素（電腦系統、網路、應用程式、服務、儲存等）的結構、狀態與關係[^dmtf-cim]。

CIM 的核心目標是定義一套**廠商中立、協定無關**的共同詞彙，使不同廠商的管理工具能基於同一套語意模型進行互通操作。

## 目的與目標

1. **提供共同詞彙** — 定義管理物件的類別、屬性與關係，使不同廠商與管理系統間能相互理解
2. **實現語意互通** — 讓語意豐富的管理資訊可在異質系統間交換
3. **模型與實作分離** — CIM 只描述「管理什麼」，不依賴特定通訊協定、平台或程式語言
4. **支援廠商延伸** — 標準模型允許廠商增補產品專屬的功能定義
5. **作為基礎標準** — CIM 是 DMTF 其他標準的核心資料模型，包括 WBEM、SMASH、DASH 等[^dmtf-wbem]

## 歷史沿革

| 時間 | 里程碑 |
|------|--------|
| **1992** | DMTF 成立（原名 Desktop Management Task Force），推出 Desktop Management Interface（DMI） |
| **1999** | 組織更名為 Distributed Management Task Force，同年首次發佈 CIM |
| **2000s** | CIM 成為 WBEM、SMI-S（儲存管理）、SMASH（伺服器管理）、DASH（桌面管理）的基礎 |
| **至今** | CIM 持續由 DMTF CIM Forum 維護，最新 Schema 版本 **2.56.0**（2026-01-21 發佈）[^dmtf-schemas] |

DMTF 近年也發展 Redfish（現代硬體管理）、SPDM（安全協定與資料模型）等新標準。

## 核心概念

### 1. 物件導向基礎

CIM 基於 UML（統一建模語言）設計，使用以下物件導向結構[^wikipedia-cim]：

- **類別（Class）** — 代表被管理元素，如 `CIM_ComputerSystem`、`CIM_OperatingSystem`
- **屬性（Property）** — 描述類別實例的狀態
- **方法（Method）** — 類別可執行的操作
- **關聯（Association）** — 類別之間的關係
- **繼承（Inheritance）** — 通用基礎類別可衍生出更具體的類別
- **多型（Polymorphism）** — 類別階層中常見

### 2. 三層類別階層

CIM 將類別定義分為三個抽象層級[^dmtf-tutorial]：

| 層級 | 說明 | 範例 |
|------|------|------|
| **Core（核心）** | 適用於**所有**管理領域的基礎物件 | `__Parameters`、`__SystemSecurity` |
| **Common（共同）** | 特定管理領域通用，不依賴特定平台 | `CIM_ComputerSystem`、`CIM_OperatingSystem` |
| **Extended（延伸）** | 特定平台或技術的擴充 | `Win32_ComputerSystem`（繼承自 `CIM_UnitaryComputerSystem`） |

此設計讓開發者可針對 Common 層級撰寫管理邏輯，即可跨平台運作；廠商則可在 Extended 層級添加平台特定細節。

### 3. Schema 結構

CIM Schema 由三層組成[^wikipedia-cim-schema]：

- **Core Model** — 所有管理領域通用的資訊模型
- **Common Model** — 特定管理領域的定義（系統、網路、應用、儲存等）
- **Extension Schemas** — 廠商或技術特定的擴充

### 4. 元模型（Metamodel）

CIM Metamodel（DSP0004）定義了如何建構符合標準的新模型與 Schema，包括合法的語句類型、語法規則，以及類別、屬性、方法、關聯等物件導向結構的規範。

### 5. Managed Object Format（MOF）

CIM 使用 MOF 作為描述類別與實例的語言，定義於 CIM Specification（DSP0221）。

## CIM 與 DMTF、WBEM 的關係

```
DMTF（標準組織）
├── CIM（Common Information Model） — 資料模型，定義「管理什麼」
└── WBEM（Web-Based Enterprise Management） — 通訊協定，定義「怎麼管理」
```

| 角色 | 說明 |
|------|------|
| **DMTF** | 制定與維護 CIM 及 WBEM 等標準的產業聯盟，成員包括 Broadcom、Cisco、Dell、HPE、Intel、Lenovo、Verizon 等 |
| **CIM** | 資料模型 — 定義管理物件的類別、屬性、關係，與通訊協定無關 |
| **WBEM** | 通訊協定集合 — 定義如何透過網路發現、存取與操作 CIM 模型化的資源 |

WBEM 的四項核心標準[^dmtf-wbem]：
1. **CIM** — 資料模型本身
2. **CIM-XML Encoding** — CIM 物件的 XML 表示方式
3. **CIM Operations over HTTP** — 傳輸機制（又稱 CIM-XML）
4. **DMTF Schema** — 實際的 Schema 定義

### WBEM 通訊協定家族

| 協定 | 傳輸 | 承載格式 | 標準編號 |
|------|------|----------|----------|
| **CIM-XML** | HTTP(S) | XML | DSP0200 + DSP0201 |
| **WS-Management** | HTTP(S), SOAP | XML/SOAP | DSP0226 + DSP0227 |
| **CIM-RS** | HTTP(S), RESTful | JSON | DSP0210 + DSP0211 |

所有協定支援兩種訊息類型：
- **Operational messages** — 請求和回應（類似 RPC）
- **Export messages** — 非同步事件通知

## 實際應用與實作

### 1. 儲存管理 — SNIA SMI-S

Storage Networking Industry Association（SNIA）採用 CIM 作為 **Storage Management Initiative – Specification（SMI-S）** 的基礎，定義儲存區域網路（SAN）的標準物件模型。 Cisco、IBM 等儲存廠商使用 SMI-S 實現跨廠商的交換器、光纖通道、分區與儲存陣列管理[^snia-smis]。

### 2. 伺服器管理 — DMTF SMASH

**Systems Management Architecture for Server Hardware（SMASH）** 使用 CIM 定義伺服器管理輪廓（Profile），VMware 及伺服器廠商透過 CIM 基礎的 SMASH API 管理伺服器硬體。

### 3. 桌面管理 — DMTF DASH

**Desktop and mobile Architecture for System Hardware（DASH）** 定義以 CIM 為基礎的桌面電腦管理標準。

### 4. Microsoft Windows — WMI

Microsoft 的 **Windows Management Instrumentation（WMI）**（自 Windows 2000 起提供）是 CIM 的實作。 Microsoft 也釋出了 **Open Management Interface（OMI）** 作為 CIM/WBEM 的開放原始碼實作[^microsoft-omi]。

### 5. Linux — SBLIM / OpenLMI

- **Standards Based Linux Instrumentation for Manageability（SBLIM）** 提供 Linux 上的 CIM/WBEM 實作
- **Open Linux Management Infrastructure（OpenLMI）** 提供一組完整的工具用於設定、管理與監控遠端 Linux 伺服器

### 6. OpenPegasus

**OpenPegasus** 是一套完整的開放原始碼 CIM/WBEM 實作（MIT 授權，主要使用 C++），包含 WBEM 伺服器、物件管理器、使用者端基礎架構、Provider 介面（C++ 與 CMPI）及 CIM 儲存庫，支援 Linux 與 Windows[^openpegasus]。

### 7. PyWBEM

**PyWBEM** 是以純 Python 撰寫的 WBEM 使用者端函式庫，可透過 CIM-XML 協定向 WBEM 伺服器發出 CIM 操作。

### 8. IBM 系統

IBM z/OS 與 i/OS 平台包含以 CIM 為基礎的管理功能。

### 9. Micro Focus / Novell

Micro Focus Open Enterprise Server 及 Novell 系統使用 CIM/WBEM 進行系統管理，SLES 15 上以 SFCB（System Flow Control Block）作為 CIM Object Manager[^microfocus]。

## 總結

DMTF CIM 是一套成熟且廣泛應用的管理資訊模型標準，其核心價值在於提供一個**廠商中立、協定無關**的共同語意框架。 雖然 CIM 本身較少直接出現在一般開發者的視野中，但作為 WBEM、WMI、SMI-S、SMASH、DASH 等標準的基礎資料模型，它在企業級系統管理領域有著深遠的影響力。

## 參考資料

[^dmtf-cim]: DMTF. (n.d.). Common Information Model (CIM). Retrieved 2026-10-01, from https://www.dmtf.org/standards/cim
[^dmtf-wbem]: DMTF. (n.d.). Web-Based Enterprise Management (WBEM). Retrieved 2026-10-01, from https://www.dmtf.org/standards/wbem
[^dmtf-schemas]: DMTF. (n.d.). DMTF Schema Repository. Retrieved 2026-10-01, from https://schemas.dmtf.org/
[^wikipedia-cim]: Wikipedia. (n.d.). Common Information Model (computing). Retrieved 2026-10-01, from https://en.wikipedia.org/wiki/Common_Information_Model_(computing)
[^wikipedia-cim-schema]: Wikipedia. (n.d.). CIM Schema. Retrieved 2026-10-01, from https://en.wikipedia.org/wiki/CIM_Schema
[^dmtf-tutorial]: DMTF Tutorial. (n.d.). DMTF CIM/WBEM Overview. Retrieved 2026-10-01, from https://www.scss.tcd.ie/dave.lewis/3ba33/dmtftutorial.pdf
[^snia-smis]: SNIA. (n.d.). Storage Management Initiative Specification (SMI-S). Retrieved 2026-10-01, from https://www.snia.org/tech_activities/smi
[^microsoft-omi]: DMTF. (n.d.). Open Management Interface (OMI) – Open Source Implementation of DMTF CIM/WBEM Standards. Retrieved 2026-10-01, from https://www.dmtf.org/content/open-management-interface-omi-%E2%80%93-open-source-implementation-dmtf-cimwbem-standards
[^openpegasus]: OpenPegasus. GitHub Repository. Retrieved 2026-10-01, from https://github.com/OpenPegasus/OpenPegasus
[^microfocus]: Micro Focus. (n.d.). CIM in Open Enterprise Server. Retrieved 2026-10-01, from https://www.microfocus.com/documentation/open-enterprise-server/24.4/oes_implement_lx/bx5fl0y.html