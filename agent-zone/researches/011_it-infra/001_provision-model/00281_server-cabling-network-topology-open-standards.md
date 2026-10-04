# 開放格式與標準：伺服器接線與網路拓撲描述

## 概述

本報告整理目前業界與學術領域中，用於描述**伺服器接線（cabling）**、**網路拓撲（network topology）**、以及**資料中心基礎設施（data center infrastructure）** 的開放格式與標準。涵蓋圖形描述語言、IETF YANG 資料模型、雲端服務拓撲規範、國際佈線標準、開源落地工具等類別。

---

## 一、圖形拓撲描述格式（Graph-Based Formats）

### 1.1 GraphML
- **類型**：XML 為基礎的開放圖形格式
- **用途**：描述節點（設備）、邊（線纜連接）及其屬性；支援自訂擴充標籤（如設備型號、埠號、線纜 ID）
- **常見搭配**：yED、draw.io 等圖形編輯器
- 非正式標準，但為圖形社群廣為採用的事實開放格式
- **網址**：http://graphml.graphdrawing.org/

### 1.2 DOT 語言（Graphviz）
- **類型**：純文字圖形描述語言（`.gv` / `.dot`）
- **用途**：以程式碼描述節點與邊的關係；廣泛用於自動生成網路拓撲圖（Diagram as Code）
- **網址**：https://graphviz.org/doc/info/lang.html

### 1.3 GML（Graph Modeling Language）
- GraphML 的前身，較簡潔但擴展性較低
- **網址**：https://en.wikipedia.org/wiki/Graph_Modeling_Language

### 1.4 NetJSON
- **類型**：JSON 為基礎的開放資料交換格式，專為電腦網路設計
- **定義類型**：`DeviceConfiguration`、`DeviceMonitoring`、`NetworkGraph`、`NetworkRoutes`、`NetworkCollection`
- **優勢**：輕量、適合 Layer 2 / Layer 3 拓撲資料、裝置配置與連結狀態
- 具備正式 JSON Schema 規範
- **網址**：https://netjson.org/

---

## 二、IETF YANG 資料模型（網路拓撲相關 RFC）

YANG 為 IETF 標準的網路管理資料建模語言。以下 RFC 直接定義拓撲／佈線相關模型：

### 2.1 RFC 8345 — A YANG Data Model for Network Topologies（基礎抽象模型）
- 定義 `ietf-network` 與 `ietf-network-topology` 模組
- 作為其他技術特定模型的基底（可被 augments）
- **網址**：https://datatracker.ietf.org/doc/rfc8345/

### 2.2 RFC 8542 — A YANG Data Model for Fabric Topology in Data-Center Networks
- 專為資料中心交換機 Fabric 拓撲設計（Spine-Leaf、POD 結構）
- 適合描述 ToR 交換機與伺服器間的佈線拓撲
- **網址**：https://datatracker.ietf.org/doc/rfc8542/

### 2.3 RFC 8944 — A YANG Data Model for Layer 2 Network Topologies
- 對 RFC 8345 增加 Layer 2 屬性（VLAN、Bridge Domain、MAC Table）
- 適合描述從跳接面板到交換機埠的 L2 佈線關係
- **網址**：https://datatracker.ietf.org/doc/rfc8944/

### 2.4 RFC 8795 — A YANG Data Model for TE Topologies
- 流量工程拓撲模型，涵蓋光纖／DWDM 連線，適用於資料中心互連（DCI）佈線
- **網址**：https://datatracker.ietf.org/doc/rfc8795/

### 2.5 其他相關草案
- **Layer 3 / IP 拓撲**（擴充 RFC 8345）
- **網路庫存拓撲**（draft-wzwb-ivy-network-inventory-topology）
- **Ethernet TE 拓撲**（draft-ietf-ccamp-eth-client-te-topo-yang）

---

## 三、OpenConfig YANG 模型（營運商主導、廠商中立）

### 3.1 網路實例／L2／L3 模型
- 定義介面、VLAN、路由等裝置配置，為拓撲文件化的基礎
- **網址**：https://www.openconfig.net/projects/models/

### 3.2 平台收發器模型（openconfig-platform-transceiver）
- 定義光纖收發器（SFP/QSFP）與埠之間的關係，包括線纜類型、連接器型態
- **網址**：https://github.com/openconfig/public/blob/master/release/models/platform/openconfig-platform-transceiver.yang

### 3.3 光傳輸模型
- ROADM、光放大器、波長路由器等模型，用於 DCI 與骨幹光纜拓撲描述

---

## 四、TOSCA（OASIS 標準）

### OASIS TOSCA v2.0
- **全稱**：Topology and Orchestration Specification for Cloud Applications
- **格式**：YAML 或 XML
- **用途**：描述服務拓撲的元件與關係；雖為雲端應用設計，但也適用於網路服務與基礎設施拓撲建模
- 使用 Topology Template 定義節點類型與關係類型
- **網址**：https://docs.oasis-open.org/tosca/TOSCA/v2.0/TOSCA-v2.0.html

---

## 五、DMTF 通用資訊模型（CIM）

### CIM（Common Information Model）
- **類型**：DMTF 業界標準
- **內涵**：物件導向的管理資訊模型，定義實體硬體、連接器、線纜與網路拓撲的類別
- **相關類別**：`CIM_PhysicalConnector`、`CIM_PhysicalLink`、`CIM_NetworkPort`、`CIM_ConnectivityCollection`
- 可用 CIM-XML 或 WBEM 序列化
- **網址**：https://www.dmtf.org/standards/cim

---

## 六、結構化佈線標準（實體基礎設施文件化）

### 6.1 TIA/EIA 標準（美國／國際）

| 標準 | 內容 |
|------|------|
| **ANSI/TIA-568** | 商業建築佈線標準：星狀拓撲、距離限制、連接器類別、線纜等級（Cat5e/6/6a/8） |
| **ANSI/TIA-606** | 佈線管理標準：標籤、識別碼、色彩編碼、記錄保存（Class 1–4） |
| **ANSI/TIA-607** | 電信接地與綁定標準 |
| **ANSI/TIA-942** | 資料中心電信基礎設施標準：佈線拓撲（MDA/HDA/ZDA/EDA）、冗餘、路徑規劃 |
| **ANSI/TIA-569** | 電信通道與空間標準（線槽、導管、機房） |

### 6.2 ISO/IEC 標準（國際）

| 標準 | 內容 |
|------|------|
| **ISO/IEC 11801 系列** | 客戶場所通用佈線標準（辦公室、資料中心、住家、工業、建築設備） |
| **ISO/IEC 14763 系列** | 佈線管理（文件化、標籤、測試） |
| **ISO/IEC 30129** | 電信綁定網路 |

### 6.3 歐洲 CENELEC 標準

| 標準 | 內容 |
|------|------|
| **EN 50173 系列** | 通用佈線系統（對應 ISO/IEC 11801） |
| **EN 50174 系列** | 佈線安裝規劃與文件化 |
| **EN 50600 系列** | 資料中心設施與基礎設施（Part 2-5 涵蓋佈線） |

### 6.4 BICSI
- 出版 **TDMM（Telecommunications Distribution Methods Manual）**
- 提供 RCDD 認證，為佈線設計與安裝的業界權威指引
- **網址**：https://www.bicsi.org/standards

---

## 七、開源落地工具（DCIM / 網路真實來源）

### 7.1 NetBox
- **定位**：Network Source of Truth & Infrastructure Resource Modeling（IRM）
- **資料模型涵蓋**：據點、機櫃、設備、設備型號、介面、**線纜**（實體終端對終端連接）、跳接面板（前／後埠）、電路、電源連接、console 連接
- 線纜（Cable）為核心物件，支援端到端線路追蹤（cable trace）
- 提供 REST API、GraphQL、PostgreSQL 結構化 schema
- **網址**：https://github.com/netbox-community/netbox
- **線纜模型文件**：https://netbox.readthedocs.io/en/stable/features/devices-cabling/

### 7.2 Nautobot
- NetBox 分支（Network to Code），資料模型相似但更可擴展
- 支援自訂應用（Apps）、關聯（Relationships）、任務（Jobs）、GraphQL
- **網址**：https://github.com/nautobot/nautobot

### 7.3 RackTables
- 開源 DCIM：資產、機櫃空間、IP 位址、埠對埠線纜連接、跳接面板埠映射
- 比 NetBox 簡單但專注於實體基礎設施
- **網址**：http://www.racktables.org/

### 7.4 openDCIM
- 開源資料中心基礎設施管理：資產、機櫃、空間、電力、冷卻、線纜、電源路徑
- **網址**：https://www.opendcim.org/

### 7.5 RacksDB / RacksLab
- 結構化資料庫 schema 建模資料中心基礎設施：設備、機櫃位置、佈線、電力
- **網址**：https://rackslab.io/en/solutions/racksdb/

---

## 八、Containerlab 拓撲定義格式

### Containerlab YAML 拓撲定義檔（`.clab.yml`）
- 宣告式 YAML 格式，定義節點（設備種類、映像檔）與連結（端點對端點佈線）
- 支援 groups、labels、binding 定義
- 雖然主要用於網路模擬實驗室，但格式本身可作為基礎設施即程式碼的拓撲描述
- **網址**：https://containerlab.dev/manual/topo-def-file/

---

## 九、其他相關格式

### 9.1 Draw.io / Diagrams.net MXFile 格式
- 開放 XML 格式儲存網路拓撲圖
- 可版本控制，適合「動態圖表即文件」工作流程
- **網址**：https://www.drawio.com/

### 9.2 SNMP MIBs（Management Information Bases）
- LLDP-MIB（IEEE 802.1AB）可自動發現 Layer 2 實體連線（埠對埠）
- ENTITY-MIB 可發現設備機箱與元件
- 非拓撲描述格式本身，但可作為自動化拓撲探索的基礎
- **網址**：https://www.ietf.org/rfc/rfc2922.txt

### 9.3 NetBox / Nautobot YAML Bootstrap Schema
- 用於匯入／匯出基礎設施資料的 YAML schema（設備、線纜、機櫃、介面）
- 可作為可攜式佈線文件化格式
- **網址**：https://docs.nvidia.com/switch-infrastructure/config-manager/config-manager/nautobot/bootstrap-schema

### 9.4 IEEE 802.3（Ethernet 標準）
- 定義銅纜（雙絞線）與光纖的實體層規格、傳輸距離、連接器類型
- 非文件格式，但定義了所有結構化佈線需支援的技術規格
- **網址**：https://standards.ieee.org/standard/802_3-2022.html

---

## 綜合比較表

| 類別 | 標準／格式 | 狀態 | 最佳用途 |
|------|-----------|------|---------|
| 圖形格式 | GraphML、DOT、GML | 開放／事實標準 | 拓撲圖交換 |
| JSON 網路格式 | NetJSON | 開放規範 | 輕量網路拓撲＋配置資料 |
| IETF YANG 模型 | RFC 8345、8542、8944、8795 | IETF 標準 | 程式化拓撲描述、自動化 |
| 營運商 YANG | OpenConfig | 業界事實標準 | 廠商中立裝置＋佈線＋光學模型 |
| 雲端／服務拓撲 | TOSCA（OASIS） | OASIS 標準 | 雲端／NFV 網路服務拓撲 |
| 管理 Schema | DMTF CIM | 業界標準 | 抽象管理實體佈線＋裝置 |
| 實體佈線 | TIA-568、TIA-942、TIA-606 | ANSI/TIA 標準 | 結構化佈線設計、標籤、文件化 |
| 國際佈線 | ISO/IEC 11801、14763 | ISO 標準 | 全球佈線設計、安裝、管理 |
| 歐洲佈線 | EN 50173、50174、50600 | CENELEC 標準 | 歐洲佈線與資料中心基礎設施 |
| 真實來源工具 | NetBox、Nautobot、RackTables、openDCIM | 開源（多種授權） | 記錄實際佈線、跳接面板、線路 |
| 實驗室拓撲 | Containerlab YAML | 開源 | 基礎設施即程式碼拓撲定義 |
| 圖表即程式碼 | Draw.io（XML）、Graphviz DOT | 開放格式 | 版本控制中的可視化佈線／拓撲圖 |

---

## 結論

目前並**沒有一個單一的「開放標準」能夠涵蓋所有伺服器接線與網路拓撲描述的需求**，但有多個互補的層次：

1. **實體佈線標準**（TIA-568、ISO/IEC 11801、EN 50173）定義了該如何佈線，但不定義數位格式。
2. **標籤與管理標準**（TIA-606、ISO/IEC 14763）定義了該如何記錄與標識，但仍不強制特定數位格式。
3. **資料模型與序列化格式**（YANG、NetJSON、TOSCA、CIM、GraphML）提供了機器可讀的數位格式，但各有其生態系與適用場景。
4. **開源工具**（NetBox、Nautobot）提供了最完整的實作——結合結構化資料庫、API、以及標準化的線纜管理模型，是當前業界最推薦的「記錄真實佈線」方案。

對於需要**程式化描述與自動化**的場景，IETF YANG（特別是 RFC 8345 + RFC 8542）與 NetBox/Nautobot 的資料模型是最完整的選擇。對於**人可讀的文件與圖表**，Graphviz DOT 與 Draw.io MXFile 最實用。

---

[^graphml]: Graph Drawing Community. (n.d.). GraphML Format Specification. Retrieved 2026-09-27, from http://graphml.graphdrawing.org/
[^dot]: Graphviz Project. (n.d.). DOT Language. Retrieved 2026-09-27, from https://graphviz.org/doc/info/lang.html
[^netjson]: NetJSON Community. (n.d.). NetJSON Data Interchange Format. Retrieved 2026-09-27, from https://netjson.org/
[^rfc8345]: IETF. (2018). RFC 8345: A YANG Data Model for Network Topologies. Retrieved 2026-09-27, from https://datatracker.ietf.org/doc/rfc8345/
[^rfc8542]: IETF. (2019). RFC 8542: A YANG Data Model for Fabric Topology in Data-Center Networks. Retrieved 2026-09-27, from https://datatracker.ietf.org/doc/rfc8542/
[^rfc8944]: IETF. (2021). RFC 8944: A YANG Data Model for Layer 2 Network Topologies. Retrieved 2026-09-27, from https://datatracker.ietf.org/doc/rfc8944/
[^rfc8795]: IETF. (2020). RFC 8795: A YANG Data Model for TE Topologies. Retrieved 2026-09-27, from https://datatracker.ietf.org/doc/rfc8795/
[^openconfig]: OpenConfig. (n.d.). OpenConfig YANG Models. Retrieved 2026-09-27, from https://www.openconfig.net/projects/models/
[^openconfig-optics]: OpenConfig. (n.d.). openconfig-platform-transceiver YANG. Retrieved 2026-09-27, from https://github.com/openconfig/public/blob/master/release/models/platform/openconfig-platform-transceiver.yang
[^tosca]: OASIS. (2023). TOSCA v2.0 Specification. Retrieved 2026-09-27, from https://docs.oasis-open.org/tosca/TOSCA/v2.0/TOSCA-v2.0.html
[^cim]: DMTF. (n.d.). Common Information Model (CIM). Retrieved 2026-09-27, from https://www.dmtf.org/standards/cim
[^tia568]: Wikipedia. (n.d.). ANSI/TIA-568. Retrieved 2026-09-27, from https://en.wikipedia.org/wiki/ANSI/TIA-568
[^tia942]: TIA. (n.d.). TIA-942 Standard. Retrieved 2026-09-27, from https://tiaonline.org/standard/tia-942/
[^tia606]: ANdCable. (n.d.). Data Center Cable Labeling Standard. Retrieved 2026-09-27, from https://andcable.com/cable-management/data-center-cable-labeling-standard/
[^iso11801]: ISO. (n.d.). ISO/IEC 11801-1. Retrieved 2026-09-27, from https://www.iso.org/standard/66182.html
[^iso14763]: ISO. (n.d.). ISO/IEC 14763-1. Retrieved 2026-09-27, from https://www.iso.org/standard/73337.html
[^en50600]: En-Standard. (n.d.). EN 50600-1. Retrieved 2026-09-27, from https://www.en-standard.eu/bs-en-50600-1-2019/
[^netbox]: NetBox Community. (n.d.). NetBox. Retrieved 2026-09-27, from https://github.com/netbox-community/netbox
[^netbox-cables]: NetBox Documentation. (n.d.). Devices and Cabling. Retrieved 2026-09-27, from https://netbox.readthedocs.io/en/stable/features/devices-cabling/
[^nautobot]: Nautobot Community. (n.d.). Nautobot. Retrieved 2026-09-27, from https://github.com/nautobot/nautobot
[^racktables]: RackTables Project. (n.d.). RackTables. Retrieved 2026-09-27, from http://www.racktables.org/
[^opendcim]: openDCIM Project. (n.d.). openDCIM. Retrieved 2026-09-27, from https://www.opendcim.org/
[^racksdb]: RacksLab. (n.d.). RacksDB. Retrieved 2026-09-27, from https://rackslab.io/en/solutions/racksdb/
[^containerlab]: Containerlab Project. (n.d.). Topo Definition File. Retrieved 2026-09-27, from https://containerlab.dev/manual/topo-def-file/
[^bicsi]: BICSI. (n.d.). Standards. Retrieved 2026-09-27, from https://www.bicsi.org/standards
[^en50174]: GlobalSpec. (n.d.). EN 50174-1. Retrieved 2026-09-27, from https://standards.globalspec.com/std/14306102/en-50174-1
[^gml]: Wikipedia. (n.d.). Graph Modeling Language. Retrieved 2026-09-27, from https://en.wikipedia.org/wiki/Graph_Modeling_Language
[^ieee8023]: IEEE. (2022). IEEE 802.3-2022. Retrieved 2026-09-27, from https://standards.ieee.org/standard/802_3-2022.html
[^drawio]: draw.io. (n.d.). draw.io. Retrieved 2026-09-27, from https://www.drawio.com/
[^lldp]: IETF. (2004). RFC 2922: LLDP MIB. Retrieved 2026-09-27, from https://www.ietf.org/rfc/rfc2922.txt