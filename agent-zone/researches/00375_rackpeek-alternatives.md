# RackPeek 替代方案調查報告

## 概述

RackPeek 是一套以 YAML 為基礎、以程式碼管理機房基礎設施文件的開源工具，以 CLI + Web UI 形式提供，適合家庭實驗室 (Home Lab) 與小型 IT 環境。本報告調查並比較各類替代方案，從重量級 DCIM/CMDB 到輕量級 YAML 方案、以及現代化 Homelab 視覺化工具。

## 替代方案分類

### 第一類：重量級 DCIM / CMDB / Source of Truth 工具

這類工具功能完整，但部署與維護成本較高，適合大型機房或企業環境。

#### NetBox

- **說明**：業界最知名的開源網路基礎設施 Source of Truth，結合 IPAM 與 DCIM，提供強大的 REST API。可管理機櫃、設備、線纜、IP、VLAN、電路、電源、VPN 等。本身不繪製拓樸圖，需透過插件擴充。[^netbox]
- **主要功能**：IPAM、機櫃層面圖、線纜管理、設備追蹤、自訂欄位、自訂腳本、事件規則、Jinja2 設定渲染、豐富插件生態系、LDAP/OIDC 認證、REST/graphQL API
- **語言**：Python (Django)
- **授權**：Apache 2.0
- **GitHub**：<https://github.com/netbox-community/netbox>（21.6k ★）

#### RackTables

- **說明**：成熟穩定的開源資料中心資產管理系統，自 2006 年以來廣泛部署。以網頁介面管理硬體資產、網路位址、機櫃空間與網路設定。[^racktables]
- **主要功能**：機櫃空間視覺化、類型化機櫃物件、標籤系統、IPAM（IPv4/IPv6）、VLAN 管理、NAT/虛擬路由器、插件架構、可自訂屬性、SNMP 支援
- **語言**：PHP
- **授權**：GPL-2.0
- **GitHub**：<https://github.com/RackTables/racktables>（812 ★）

#### openDCIM

- **說明**：功能豐富的開源 DCIM 工具，專注於資料中心實體庫存管理——空間、電源與冷卻容量。最初由范德堡大學開發。**注意：維護者正在退休，最終版本 26.01 即將發布。**[^opendcim]
- **主要功能**：完整實體資產追蹤、多機房支援、電源/冷卻容量管理、容錯追蹤、拖曳式機櫃視覺化、SNMP 輪詢、LDAP/OIDC 認證、條碼支援、自動探索（ESX、Proxmox、PDU、溫度感測器）
- **語言**：PHP
- **授權**：GPL-3.0
- **GitHub**：<https://github.com/opendcim/openDCIM>（367 ★）

#### Ralph

- **說明**：由 Allegro（波蘭電商公司）開發的全功能資產管理、DCIM 與 CMDB 系統，支援資產生命週期追蹤、資料中心視覺化與後端辦公室支援。[^ralph]
- **主要功能**：資產採購追蹤與生命週期管理、靈活的流程系統、DC 視覺化、自動探索、成本報告、JIRA 整合、RESTful API、多資料中心支援
- **語言**：Python (Django)
- **授權**：Apache 2.0
- **GitHub**：<https://github.com/allegro/ralph>（2.5k ★）

#### RackMonkey

- **說明**：輕量級開源 DCIM 工具，用於追蹤與管理資料中心資產。簡單的網頁介面記錄與視覺化機櫃、伺服器與設備。最後穩定版本為 2009 年，成熟穩定。[^rackmonkey]
- **主要功能**：簡單資產追蹤、機櫃圖表視覺化、設備位置追蹤、連線管理、變更記錄、搜尋功能
- **語言**：Perl
- **授權**：GPL
- **GitHub**：<https://github.com/osamu/rackmonkey>

### 第二類：YAML / Infrastructure as Code 文件化工具（最接近 RackPeek）

#### RacksDB

- **說明**：與 RackPeek 哲學最接近的輕量級 YAML 式 DCIM/CMDB。資料以 YAML 檔案儲存於 Git 中管理，提供 CLI、Python 程式庫、REST API 與 Web UI。可產生 PNG、SVG、PDF 格式的機櫃圖。[^racksdb]
- **主要功能**：YAML 儲存（Git 友善）、標籤式過濾、去中心化架構（無需中央伺服器）、CLI 工具、Python 程式庫、REST API、Web UI、產生 2D 與 3D 機櫃圖、自訂 schema 擴充
- **語言**：Python
- **授權**：MIT
- **GitHub**：<https://github.com/rackslab/RacksDB>（34 ★）

### 第三類：現代 Homelab 視覺化工具（自架 Web GUI）

#### Rackpad

- **說明**：專為家庭實驗室、小型機櫃與網路機房設計的自架基礎設施庫存與操作工作區。在單一 Docker 容器中整合機櫃、設備、埠口、線纜、IPAM、VLAN、WiFi、運算、探索、監控、文件與拓樸視覺化。使用 SQLite。[^rackpad]
- **主要功能**：視覺化拓樸（React Flow）、機櫃層面圖、交換器埠口對映、線纜配接、VLAN/子網路/DHCP/IPAM、儲存拓樸（硬碟、槽位、儲存池）、監控（ICMP、TCP、HTTP、SNMP 含警報）、自動探索（IPAM 子網路掃描）、Proxmox/UniFi/Omada/OPNsense 整合、Markdown 文件、淺色/深色主題、24 種語言、OIDC 認證
- **語言**：TypeScript (React + Fastify)
- **授權**：MIT
- **GitHub**：<https://github.com/Kobii-git/Rackpad>（411 ★）

#### Homelable

- **說明**：自架 Homelab 基礎設施視覺化工具，具備互動式網路圖、即時監控、網路掃描、含埠口配接的機櫃畫布以及內建文件功能。可透過 nmap 自動探索設備，並從 Proxmox/Zigbee/Z-Wave 匯入。非常受歡迎（4k 星）。[^homelable]
- **主要功能**：互動式網路圖畫布、自動探索（nmap -sV）、Proxmox VE 匯入、Zigbee/Z-Wave 透過 MQTT 匯入、健康檢查（ping/TCP/HTTP/SSH）、含埠口對接的機櫃畫布、Markdown 設備文件、程式庫文件、MCP 伺服器（AI 整合）、唯讀 Live View 分享、REST API、Home Assistant 原生支援（HACS）
- **語言**：TypeScript（前端）+ Python（後端）
- **授權**：MIT
- **GitHub**：<https://github.com/Pouzor/homelable>（4.0k ★）

#### Homelab Inventory

- **說明**：自架視覺化工作檯，用於記錄、組裝、驗證與監控 Homelab 硬體。專注於詳細硬體元件追蹤（CPU、RAM、儲存、GPU 等），具備相容性驗證與線纜佈線的無限畫布。[^homelab-inventory]
- **主要功能**：詳細硬體元件庫存（CPU、主機板、RAM、儲存、GPU、PSU 等）、相容性驗證、彩色線纜佈線、無限畫布工作區、硬體目錄（已簽名/驗證）、代理式監控（systemd Linux、FreeBSD）、OIDC 認證、lab.gd 分享、opt-in Ntfy/Webhook 警報、備份/還原、已簽名硬體註冊表
- **語言**：TypeScript (Bun, React, Vite) + Rust (WASM)
- **授權**：MIT
- **GitHub**：<https://github.com/mriverodorta/homelab-inventory>（11 ★）

#### PrivateGlue

- **說明**：簡單的自架網頁應用，用於連結設備、筆記與憑證。專為 Homelab 玩家與 IT 專業人士設計，可追蹤設備、編寫 Markdown 文件與儲存加密密碼。[^privateglue]
- **主要功能**：Markdown 筆記（可連結到設備）、加密密碼管理器（一鍵複製）、設備庫存（含位置與類型標籤）、將筆記與憑證連結到設備、深色模式、Docker 部署
- **語言**：Python (Flask) + SQLite
- **授權**：MIT
- **GitHub**：<https://github.com/marcmylemans/privateglue-public>（15 ★）

### 第四類：網路探索與拓樸文件化工具

#### Scanopy Community Edition

- **說明**：開源網路探索工具，可透過 SNMP、LLDP、CDP、ARP 自動探索並產生互動式拓樸圖。提供四種可切換視圖（實體、邏輯、工作負載、應用程式）。AGPL-3.0 授權，可自架。[^scanopy]
- **主要功能**：自動網路探索（SNMP/LLDP/CDP/ARP）、互動式拓樸圖（4 種切換視圖）、自動更新文件、匯出為 PNG/SVG/PDF/HTML/Mermaid、可嵌入 Wiki、iframe 分享、CSV 匯出
- **授權**：AGPL-3.0（社群版）
- **GitHub**：<https://github.com/scanopy/scanopy>

#### NetDisco

- **說明**：網頁式網路管理工具，適用於小型到超大型網路。透過 SNMP、CLI 或設備 API 收集 IP/MAC 資料至 PostgreSQL。可依 MAC/IP 定位設備、管理交換器埠口、盤點硬體、產生網路拓樸圖。[^netdisco]
- **主要功能**：依 MAC/IP 定位交換器埠口上的設備、管理交換器埠口（停用、變更 VLAN/PoE）、按型號/廠商/OS 盤點硬體、網路拓樸探索、Layer2 對映、PostgreSQL 後端、Docker 支援
- **語言**：Perl + Python
- **授權**：BSD-3-Clause
- **GitHub**：<https://github.com/netdisco/netdisco>（922 ★）

#### Netdot

- **說明**：開源網路文件化工具，透過 SNMP 探索收集、組織與維護網路文件。包含 IPAM 與 Layer2 拓樸探索。[^netdot]
- **主要功能**：SNMP 設備探索、Layer2 拓樸探索（CDP/LLDP/STP）、IPAM（IPv4/IPv6）、DNS/DHCP 設定產生、線纜管理
- **語言**：Perl
- **授權**：GPL
- **GitHub**：<https://github.com/cvicente/Netdot>

## 對照總表

| 工具 | 分類 | 資料格式 | CLI？ | 自架？ | 語言 | 授權 | Stars |
|------|------|----------|-------|--------|------|------|-------|
| **RackPeek** | IaC 文件化 | YAML | ✅ | ✅ | C# (.NET) | AGPL-3.0 | 1.8k |
| **NetBox** | DCIM/CMDB/IPAM | 資料庫 | ✅ (API) | ✅ | Python | Apache 2.0 | 21.6k |
| **RackTables** | DCIM 資產管理 | 資料庫 | ❌ | ✅ | PHP | GPL-2.0 | 812 |
| **openDCIM** | DCIM | 資料庫 | ❌ | ✅ | PHP | GPL-3.0 | 367 |
| **Ralph** | CMDB/DCIM/資產 | 資料庫 | ✅ (API) | ✅ | Python | Apache 2.0 | 2.5k |
| **RackMonkey** | 輕量 DCIM | 資料庫 | ❌ | ✅ | Perl | GPL | — |
| **RacksDB** | YAML DCIM/CMDB | **YAML** | ✅ CLI+API | ✅ | Python | MIT | 34 |
| **Rackpad** | Homelab 庫存 | SQLite | ❌ | ✅ | TypeScript | MIT | 411 |
| **Homelable** | Homelab 視覺化/文件 | 資料庫 | ✅ (API) | ✅ | TS + Python | MIT | 4.0k |
| **Homelab Inventory** | HW 庫存/文件 | SQLite/JSON | ✅ (代理) | ✅ | TS + Rust | MIT | 11 |
| **PrivateGlue** | 簡易 IT 文件 | SQLite | ❌ | ✅ | Python (Flask) | MIT | 15 |
| **Scanopy CE** | 網路拓樸 | 資料庫 | ❌ | ✅ | 不明 | AGPL-3.0 | — |
| **NetDisco** | 網路管理 | PostgreSQL | ✅ | ✅ | Perl + Python | BSD-3 | 922 |

## 建議

若您使用 RackPeek 的核心原因是**以 YAML 作為唯一儲存格式並納入 Git 版本控制**，**RacksDB** 是最直接的替代方案，且用 Python 而非 C#，可能對多數 DevOps 環境更友善。

若您偏好**現代化 GUI 與視覺化操作**，**Rackpad** 與 **Homelable** 均為優秀選擇，後者社群規模更大且具 MCP 伺服器支援。

若需要**企業級 Source of Truth**，**NetBox** 是無可爭議的標準解決方案，但其部署與學習曲線遠比 RackPeek 沉重。

---

[^netbox]: NetBox Community. (n.d.). _NetBox — The Premier Open Source Network Source of Truth_. Retrieved 2026-10-03, from <https://github.com/netbox-community/netbox>

[^racktables]: RackTables Community. (n.d.). _RackTables — Open Source Datacenter Asset Management_. Retrieved 2026-10-03, from <https://github.com/RackTables/racktables>

[^opendcim]: openDCIM Project. (n.d.). _openDCIM — Open Source Data Center Infrastructure Management_. Retrieved 2026-10-03, from <https://github.com/opendcim/openDCIM>

[^ralph]: Allegro. (n.d.). _Ralph — Asset Management, DCIM, and CMDB System_. Retrieved 2026-10-03, from <https://github.com/allegro/ralph>

[^rackmonkey]: Osamu. (n.d.). _RackMonkey — Simple Data Center Asset Management_. Retrieved 2026-10-03, from <https://github.com/osamu/rackmonkey>

[^racksdb]: Rackslab. (n.d.). _RacksDB — YAML-based DCIM/CMDB_. Retrieved 2026-10-03, from <https://github.com/rackslab/RacksDB>

[^rackpad]: Kobii. (n.d.). _Rackpad — Homelab Infrastructure Inventory_. Retrieved 2026-10-03, from <https://github.com/Kobii-git/Rackpad>

[^homelable]: Pouzor. (n.d.). _Homelable — Homelab Infrastructure Visualization_. Retrieved 2026-10-03, from <https://github.com/Pouzor/homelable>

[^homelab-inventory]: mriverodorta. (n.d.). _Homelab Inventory — Hardware Documentation Workbench_. Retrieved 2026-10-03, from <https://github.com/mriverodorta/homelab-inventory>

[^privateglue]: marcmylemans. (n.d.). _PrivateGlue — Simple Self-Hosted IT Documentation_. Retrieved 2026-10-03, from <https://github.com/marcmylemans/privateglue-public>

[^scanopy]: Scanopy. (n.d.). _Scanopy Community Edition — Open Source Network Discovery_. Retrieved 2026-10-03, from <https://github.com/scanopy/scanopy>

[^netdisco]: NetDisco Community. (n.d.). _NetDisco — Web-Based Network Management_. Retrieved 2026-10-03, from <https://github.com/netdisco/netdisco>

[^netdot]: Cvicente. (n.d.). _Netdot — Network Documentation Tool_. Retrieved 2026-10-03, from <https://github.com/cvicente/Netdot>