# 數位雙生中的伺服器硬體描述：標準、本體論與模型

## 概述

在本研究範疇中，「伺服器硬體描述」指在數位雙生（Digital Twin）中對實體伺服器基礎設施進行結構化建模，涵蓋機房位置、機架佈局、線纜連接、網路拓撲、硬體規格等面向。目前業界並不存在單一通用標準，而是分散於 IT 管理、建築元數據、工業 4.0、資料中心管理等多個領域[^dmtf-redfish][^idta-aas]。

## 各領域標準與涵蓋範圍

| 領域 | 標準 / 方案 | 伺服器涵蓋度 | 說明 |
|---|---|---|---|
| IT 系統管理 | DMTF Redfish, DMTF CIM | ★★★★★ | 原生且詳盡的伺服器硬體模型 |
| 建築元數據 | Brick Schema, Project Haystack | ★★ | 主要為建築子系統，可擴展至 IT |
| 工業 4.0 | Asset Administration Shell (AAS), ISO 23247 | ★★★ | 通用框架，無專用伺服器子模型 |
| IoT / Azure | DTDL (Digital Twins Definition Language) | ★★ | 語言而非預定義模型，可自行擴展 |
| 資料中心管理 | openDCIM, OCP Digital Twin Initiative | ★★★★ | 實用導向 DCIM；OCP 正制定 DT 標準 |
| 地理空間 | GeoSPARQL / WGS84 | ★★★★★ | 成熟的地理位置標準，可與其他模型整合 |

## Redfish — 最成熟的伺服器硬體標準

DMTF 制定的 Redfish 是目前對伺服器基礎設施建模最完整的標準，使用 RESTful API 與 JSON Schema[^redfish-chassis]。

### 位置（Location）

`Chassis` 資源提供 `Location` 屬性（v1.2+ 加入），用以描述實體位置。`ChassisType` 列舉值包括 `RackMount`、`Blade`、`Enclosure`、`Rack`、`Drawer`、`StandAlone` 等，涵蓋從單一機箱到整體機櫃的層級。Location 物件可包含 GPS 座標、機架單位（rack unit）、槽位（slot/socket/bay）資訊[^redfish-chassis]。

Chassis 間的階層關係透過 `Links` 物件表達，包含 `ContainedBy`（被誰包含）、`Contains`（包含誰）、`CooledBy`、`PoweredBy`、`ManagedBy` 等關係[^redfish-chassis]。

### 線纜連接與網路拓撲

- `NetworkAdapter` / `NetworkAdapterCollection`：實體網卡模型
- `EthernetInterface`、`Switch`、`Port`、`Fabric`：網路連線與拓撲模型
- Redfish Fabrics 白皮書（DSP2066）詳細說明 Fabric 層級建模，包含 Ethernet 與 CXL Fabric[^redfish-fabrics]

### 硬體規格與供應

`ComputerSystem` 為邏輯伺服器模型，透過集合屬性表示：
- `Processor`、`Memory`、`Storage`、`PCIeDevices`、`Drives`
- `PowerSubsystem`、`ThermalSubsystem`：電源與散熱基礎設施
- `PhysicalSecurity`：機櫃門/面板狀態；`LeakDetectors`：液體冷卻洩漏監控（v1.26+）[^redfish-chassis]

## CIM — 通用資訊模型

DMTF CIM Schema（v2.56）提供完整的 IT 管理類別階層[^dmtf-cim]：

| 類別 | 用途 |
|---|---|
| `CIM_ComputerSystem` | 邏輯伺服器 |
| `CIM_Chassis` / `CIM_Rack` | 實體機殼階層 |
| `CIM_PhysicalLocation` / `CIM_Location` | GPS 座標、地址 |
| `CIM_PhysicalConnector` / `CIM_PhysicalLink` / `CIM_PhysicalComponent` | 線纜與連接 |
| `CIM_NetworkPort` / `CIM_SwitchService` | 網路拓撲 |

CIM 的 OWL/RDF 對映（CIM-OWL）在學術研究中被探索，但尚未廣泛部署。

## Brick Schema 與 Project Haystack

**Brick Schema** 為建築元數據本體論，涵蓋 HVAC、照明、消防、安防、電力系統。Brick 的 alignment 機制可與 BOT（Building Topology Ontology）、REC（RealEstateCore）、VBIS 等對齊。但目前**沒有公開的 IT/伺服器擴展**，若要涵蓋伺服器硬體需自行定義類別與關係[^brick-schema][^brick-extensions]。

**Project Haystack** 針對 IoT 與建築環境，`device` 標籤涵蓋「控制器、網路設備等微處理器硬體」，但未設計用於詳細伺服器規格描述[^haystack]。

## 工業 4.0 標準

### Asset Administration Shell (AAS)

AAS 為工業 4.0 數位雙生標準，由 Industrial Digital Twin Association (IDTA) 維護[^idta-aas]。

**Technical Data 子模型**（IDTA 02003，v2.0）[^aas-techdata]：
- `GeneralInformation`：製造商、型號、產品名稱
- `ProductClassifications`：ECLASS 或 IEC CDD 分類碼
- `TechnicalProperties`：核心技術參數，以 MainSections/SubSections 階層組織
- `FurtherInformation`：有效性聲明

AAS 依賴 **ECLASS** 與 **IEC Common Data Dictionary** 作為跨組織互通的屬性定義。目前 IDTA 尚未發布專門的伺服器/IT 硬體子模型，需要自訂 Technical Data 子模型的具體實例化。

### ISO 23247

數位雙生製造框架，定義四層架構：Observable Manufacturing Elements (OME)、Digital Twin Entity、Digital Twin User、Cross-cutting[^iso23247]。Part 4 定義物理實體到數位表示的對映方式。為通用框架，不包含伺服器專用模型。

### OCP Digital Twin Initiative

Open Compute Project 的資料中心數位雙生策略白皮書（v1.2）定義[^ocp-dt]：
- 資料中心數位雙生成熟度模型
- 設計、驗收、營運、永續等用例
- **以量測效能數據為基礎的伺服器硬體數位雙生**為核心重點

## DTDL — Azure 生態系

Microsoft 的 DTDL（Digital Twins Definition Language）是一種定義數位雙生模型的語言，支援 `Interface`、`Telemetry`、`Property`、`Command`、`Relationship` 原語。Azure Digital Twins 與 IoT Plug and Play 使用此語言。Microsoft 提供智慧建築、電網等產業本體論，但**無 IT/伺服器本體論**[^dtdl]。組織可自行定義伺服器 DTDL 模型，但無標準化共識。

## openDCIM

開源資料中心基礎設施管理（DCIM）工具，提供實用的（非本體論）數據模型[^opendcim]：
- **位置**：多機房、機櫃模板化管理
- **連接**：機櫃內/跨機櫃線纜連接追蹤
- **供應**：資產追蹤、電力/冷卻/空間容量管理
- **容錯**：電力中斷模擬

## 地理空間標準

WGS84（GPS 座標）與 GeoSPARQL（RDF 空間查詢）是成熟的地理位置標準，可與上述模型整合[^geosparql]。CIM 的 `PhysicalLocation` 亦支援 WGS84 座標。

## 學術本體論

- **Data Center Ontology（Memari et al., 2016）**[^memari2016]：以本體論為基礎的資料中心模擬框架，引用 openDCIM 作為數據源，整合觀察與量測本體論，用於資料中心架構方案模擬。
- **Digital Twin Construction Ontology（TUM, 2023）**[^dtc-ontology]：導入 BOT、GeoSPARQL、WGS84 等標準，驗證了位置與空間關係在 Semantic Web 中已有完善處理。

## 結論

描述數位雙生中的伺服器硬體設定，沒有一個標準能完全涵蓋所有面向。最務實的做法是：

1. **位置與機櫃模型**：使用 Redfish Chassis + GeoSPARQL/WGS84
2. **硬體規格**：使用 Redfish ComputerSystem 或 AAS Technical Data + ECLASS
3. **網路拓撲與線纜**：使用 Redfish (NetworkAdapter, Switch, Port, Fabric)
4. **全生命週期管理**：需自行整合上述標準，或參考 OCP Digital Twin 框架建立客製模型

Redfish 在 IT 硬體描述上最完整，AAS 提供跨領域語意互通性，而 OCP 框架正在填補資料中心整體數位雙生的缺口。

---

[^dmtf-redfish]: DMTF. (n.d.). Redfish Specification. Retrieved 2026-09-27, from https://www.dmtf.org/standards/redfish
[^redfish-chassis]: DMTF. (2026). Redfish Chassis Schema v1.26.0. Retrieved 2026-09-27, from https://redfish.dmtf.org/schemas/v1/Chassis.v1_26_0.json
[^dmtf-cim]: DMTF. (n.d.). Common Information Model (CIM). Retrieved 2026-09-27, from https://www.dmtf.org/standards/cim
[^brick-schema]: Brick Consortium. (n.d.). Brick Ontology. Retrieved 2026-09-27, from https://brickschema.org/
[^brick-extensions]: Brick Schema. (n.d.). Extensions and Alignments. Retrieved 2026-09-27, from https://brickschema.readthedocs.io/en/stable/extensions.html
[^haystack]: Project Haystack. (n.d.). Project Haystack Introduction. Retrieved 2026-09-27, from https://project-haystack.org/
[^idta-aas]: Industrial Digital Twin Association. (n.d.). Asset Administration Shell. Retrieved 2026-09-27, from https://industrialdigitaltwin.org/en/
[^aas-techdata]: IDTA. (2024). Technical Data Submodel v2.0. Retrieved 2026-09-27, from https://github.com/admin-shell-io/submodel-templates/tree/main/published/Technical_Data/2/0
[^iso23247]: ISO. (2021). ISO 23247-1:2021 Digital twin framework for manufacturing. Retrieved 2026-09-27, from https://www.iso.org/standard/75066.html
[^ocp-dt]: Open Compute Project. (2024). OCP Digital Twin Whitepaper v1.2. Retrieved 2026-09-27, from https://www.opencompute.org/documents/ocp-digital-twin-whitepaper-v1-2-final-pdf
[^dtdl]: Microsoft. (n.d.). Digital Twins Definition Language. Retrieved 2026-09-27, from https://github.com/Azure/opendigitaltwins-dtdl
[^opendcim]: openDCIM. (n.d.). Open Source Data Center Infrastructure Management. Retrieved 2026-09-27, from http://opendcim.org/
[^memari2016]: Memari, M., et al. (2016). A Data Center Simulation Framework Based on an Ontological Foundation. *Springer*. Retrieved 2026-09-27, from https://link.springer.com/chapter/10.1007/978-3-319-23455-7_3
[^dtc-ontology]: Technical University of Munich. (2023). Digital Twin Construction Ontology. Retrieved 2026-09-27, from https://dtc-ontology.cms.ed.tum.de/ontology/
[^geosparql]: Open Geospatial Consortium. (2012). GeoSPARQL - A Geographic Query Language for RDF Data. Retrieved 2026-09-27, from https://www.ogc.org/standard/geosparql/