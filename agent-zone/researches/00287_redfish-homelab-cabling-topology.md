# Redfish 用於描述 Homelab 接線與網路拓撲的可能性探討

## 摘要

本文探討是否有人在 homelab 社群中使用 Redfish（DMTF 組織定義的伺服器管理標準）來描述實體接線、網路拓撲或機櫃佈線。研究發現，Redfish 確實定義了相關的 Schema（Cable、Fabric、Port、NetworkPort 等），足以構成描述實體基礎設施的完整資料模型，但在 homelab 社群中完全沒有任何實際使用案例。社群主流工具為 NetBox、Draw.io、Markdown + Git 等。

## 研究發現

### Redfish 具備相關 Schema

Redfish 標準定義了數個與實體接線直接相關的 Schema[^schema-index]：

1. **Cable Schema** (v1.3.0) — 描述連接兩個端點的纜線，屬性包含：
   - `CableClass`（列舉：Power、Network、Storage、Fan、PCIe、USB、Video、Fabric、Serial、General）
   - `CableType`、`CableStatus`、`LengthMeters`、`UserLabel`、`AssetTag`
   - `UpstreamConnectorTypes` / `DownstreamConnectorTypes`（列舉：RJ45、SFP、SFPPlus、QSFP、USBA、USBC、HDMI 等）
   - `Links` 連接至 `DownstreamPorts`、`UpstreamPorts`、`DownstreamChassis`、`UpstreamChassis`
   - `Manufacturer`、`Model`、`SerialNumber`、`PartNumber`、`SKU`
   - URI: `/redfish/v1/Cables/{CableId}`

2. **Fabric Schema** (v1.4.0) — 表示由一個或多個 Switch、零個或多個 Endpoint 及 Zone 組成的 Fabric[^schema-index]。

3. **Port Schema** (v1.20.0) — 描述 Switch、Controller、Chassis 或其他裝置上可連接的埠，包含 `PortMetrics` (v1.9.1)[^schema-index]。

4. **NetworkPort Schema** (v1.4.3) — 離散的實體網路埠[^schema-index]。

5. **Switch** — 在 Fabric 模型中引用，透過下屬 Port 資源描述實體拓撲[^fabrics-wp]。

Redfish Fabrics 白皮書 (DSP2066) 明確說明：「Redfish 使用 Switch 和 Port 資源描述 Fabric 的實體拓撲。Switch 資源下屬的 Port 資源顯示了 Fabric Switch 如何提供與基礎設施中其他裝置的連線能力。」[^fabrics-wp]

### 社群中完全無人使用

經過多角度搜索，在 homelab 社群中**完全找不到**任何使用 Redfish 記錄接線或拓撲的案例[^search-summary]：

- 無相關部落格文章
- 無 Reddit / Redfish Forum 討論
- 無 GitHub 專案
- 無任何社群提及

### 社群實際使用的工具

Homelab 社群普遍使用以下工具來記錄網路拓撲與接線[^homelab-guide][^homelab-starter]：

- **NetBox** — 專為 IPAM/DCIM 設計，可追蹤裝置、介面、纜線、機櫃、電路
- **Draw.io / diagrams.net** — 視覺化網路拓撲圖
- **Markdown + Git** — 純文字基礎設施即程式碼（最常被推薦的方式）
- **Wiki.js / BookStack / Outline** — 自託管 Wiki
- **Mermaid 圖表** — Markdown 內嵌文字式拓撲圖
- **試算表** — 快速追蹤 IP 分配

### 原因分析

Redfish 本質上是**頻外管理協定**（out-of-band management protocol）[^redfish-about]——它是 BMC（如 iDRAC、iLO、OpenBMC）用來管理硬體的 API。Cable、Fabric、Port 等 Schema 存在的目的是讓執行在實際硬體上的 Redfish 服務（如受管 Switch、PDU、伺服器 BMC）回報其實體互連狀態，而非設計成讓人手動撰寫的文件格式。

在 homelab 環境中使用 Redfish Schema 意味著需要：

1. 自行搭建一個 Redfish 服務實作來存放纜線資料（極度大材小用）
2. 手動編寫符合 Schema 的 JSON 承載（可行但無工具支援）
3. 等待你的硬體（Switch、PDU）實際暴露 Redfish Cable 端點（多數 homelab 等級硬體不支援）

## 結論

Redfish **擁有完整的語彙**來描述 homelab 的接線與網路拓撲——Cable、Fabric、Port/Switch、NetworkPort 與 Chassis Schema 共同構成了描述實體基礎設施的全面資料模型。然而，**homelab 社群中完全沒有任何實際使用案例**。Redfish 仍是一款企業級/工業級的硬體自動化管理協定，而非 grassroots 的 homelab 文件工具。

若有人想開創先例，理論上可以手動構建符合 DMTF Schema 的 Cable、Fabric、Port JSON 資源[^cable-schema]，架設在輕量級 Redfish 服務上，並使用 [Redfish Tacklebox](https://github.com/DMTF/Redfish-Tacklebox) Python 工具來查詢——但這將是首例。

## 參考文獻

[^schema-index]: DMTF. (n.d.). *Redfish Schema Index*. Retrieved 2026-09-27, from https://redfish.dmtf.org/redfish/schema_index

[^fabrics-wp]: DMTF. (n.d.). *Redfish Fabrics White Paper (DSP2066)*. Retrieved 2026-09-27, from https://www.dmtf.org/sites/default/files/standards/documents/DSP2066_1.0.0.pdf

[^cable-schema]: DMTF. (n.d.). *Cable v1.1.5 JSON Schema*. Retrieved 2026-09-27, from https://github.com/DMTF/Redfish-Publications/blob/main/json-schema/Cable.v1_1_5.json

[^redfish-about]: DMTF. (n.d.). *Redfish — DMTF Standard*. Retrieved 2026-09-27, from https://www.dmtf.org/standards/redfish

[^homelab-guide]: ExcaliburSheath. (2025-05-13). *Planning and Documenting Your Homelab Network*. Retrieved 2026-09-27, from https://excalibursheath.com/guide/2025/05/13/planning-and-documenting-your-homelab-network.html

[^homelab-starter]: HomelabStarter. (n.d.). *Homelab Documentation*. Retrieved 2026-09-27, from https://homelabstarter.com/homelab-documentation/

[^search-summary]: Multiple search queries across Reddit, GitHub, blogs, and forums conducted 2026-09-27. No results found matching Redfish + homelab cabling documentation.