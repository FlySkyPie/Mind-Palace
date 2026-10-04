# DMTF CIM (Common Information Model) 是否有現代繼任者？

## 概述

本報告探討 DMTF（Distributed Management Task Force）所制定的 CIM（Common Information Model，通用資訊模型）是否存在現代的繼任標準。調查結果顯示：**CIM 並未被取代，至今仍是活躍維護的標準**，而 Redfish、PLDM 等新興標準與 CIM 屬於**互補關係**，並非繼任關係。

## 什麼是 DMTF CIM？

DMTF CIM 是由 DMTF 制定的**供應商中立管理資訊模型**，為系統、網路、應用程式與服務提供一套共通的、中立的被管理物件定義[^dmtf-cim]。

核心組成包括：

- **CIM Schema** — 以類別（class）、屬性（property）與關聯（association）組織的模型描述，分為核心模型、通用模型與延伸 schema 等層次。
- **CIM Specification** — 定義 CIM 與其他管理模型的整合方式。
- **CIM Metamodel** — 定義建構新合規模型的語意規則。

CIM 自 3GPP Release 4 起即被引用，廣泛用於電信與企業 IT 的網元素管理系統（EMS）與網路管理系統（NMS）[^3gpp-glossary]。

## 最新狀態：CIM 仍活躍，未被淘汰

CIM **未被廢棄，也無單一繼任者**。DMTF 持續釋出更新版本，最新版本為 **2.56.0（2026 年 1 月 21 日）**[^dmtf-cim-256]。近期版本歷程如下：

| 版本 | 釋出日期 |
|------|----------|
| **2.56.0** | **2026-01-21** |
| 2.55.0 | 2024-02-29 |
| 2.54.1 | 2022-03-08 |
| 2.54.0 | 2020-10-26 |
| 2.53.0 | 2020-03-04 |

DMTF 官網說明此為「CIM 2.0 以來的第 56 次更新」[^dmtf-cim-256]。CIM Forum（CIMF）工作組與 CIM Schema Task Force 持續活躍[^dmtf-wg]。

## Redfish：互補而非繼任

Redfish 是 DMTF 旗下另一個重要的現代標準，但**並非 CIM 的繼任者**，而是作用域不同的互補標準[^dmtf-redfish]：

| 面向 | CIM | Redfish |
|------|-----|---------|
| **目的** | 概念性資訊模型（定義被管理物件「是什麼」） | RESTful API 標準（定義如何安全管理硬體） |
| **領域** | 系統、網路、應用、服務 | 伺服器/基礎設施硬體管理 |
| **形式** | 概念 schema | RESTful API + JSON Schema |
| **最新版本** | 2.56.0 (2026-01-21) | 1.25.0 / 2026.2 (2026-09-14) |

DMTF PMCI 工作組頁面將 CIM 與 Redfish 並列為互補技術[^dmtf-pmci]：

> "The PMCI WG creates intra-platform manageability standards and technologies, which complement DMTF's other standards such as the Redfish API from the Redfish Forum, Security Protocol and Data Models (SPDM) from the SPDM WG, Common Information Model (CIM) profiles..."

簡言之：**CIM 定義「管理什麼」，Redfish 定義「如何管理」**。

## DMTF 現代標準體系的定位

DMTF 當前的核心標準生態如下[^dmtf-standards]：

| 標準 | 用途 |
|------|------|
| **Redfish** | 硬體管理的 RESTful API（DMTF 最廣為人知的現代標準） |
| **MCTP** (Management Component Transport Protocol) | 管理子系統內部元件間的通訊協定 |
| **PLDM** (Platform Level Data Model) | 平台監控、控制、韌體更新、BIOS 設定的資料模型 |
| **SMBIOS** | 系統管理 BIOS 資訊標準 |
| **SPDM** (Security Protocols and Data Models) | 裝置身分驗證、認證與金鑰交換 |

這些標準中無一被定位為 CIM 的繼任者。CIM 仍扮演抽象資訊模型的角色，而 Redfish/PLDM 等則提供具體的管理互動協定。

## 結論

**DMTF CIM 沒有現代繼任者。** 它仍是活躍維護的標準（最新版 2026 年 1 月），並未被任何單一標準取代。

但需注意兩個趨勢：

1. **在硬體/伺服器管理領域**，Redfish 已成為 DMTF 最主流的現代標準——但它與 CIM 是互補關係，並非繼任。
2. **在平台內管理（intra-platform manageability）領域**，MCTP、PLDM、SPDM 等 PMCI 標準是 DMTF 近年投入較多的方向。

因此正確的理解是：CIM 與 Redfish 等新標準**共存於 DMTF 生態系統中**，各自服務不同的管理層面。

## 參考文獻

[^dmtf-cim]: DMTF. (n.d.). CIM — Common Information Model. Retrieved 2026-10-01, from https://www.dmtf.org/standards/cim
[^dmtf-cim-256]: DMTF. (2026). DMTF Releases CIM 2.56.0. Retrieved 2026-10-01, from https://www.dmtf.org/content/dmtf-releases-cim-256-0
[^dmtf-redfish]: DMTF. (n.d.). Redfish. Retrieved 2026-10-01, from https://www.dmtf.org/standards/redfish
[^dmtf-standards]: DMTF. (n.d.). Standards. Retrieved 2026-10-01, from https://www.dmtf.org/standards
[^dmtf-pmci]: DMTF. (n.d.). PMCI — Platform Management Communications Infrastructure. Retrieved 2026-10-01, from https://www.dmtf.org/standards/pmci
[^dmtf-wg]: DMTF. (n.d.). Working Groups. Retrieved 2026-10-01, from https://www.dmtf.org/about/working-groups
[^3gpp-glossary]: 3GPP Explorer. (n.d.). DMTF Glossary. Retrieved 2026-10-01, from https://3gpp-explorer.com/glossary/dmtf/