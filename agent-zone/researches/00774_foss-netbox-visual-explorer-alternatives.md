# NetBox Visual Explorer 的自由開源替代方案調查

## 摘要

NetBox Labs 於 2026 年 3 月發表 **NetBox Visual Explorer (NVE)**，這是一組疊加在既有 NetBox 資料之上的互動式視覺化功能，包含機房平面圖（2D/3D）、機櫃立面圖（rack elevation）、線纜拓撲圖（cable topology）、IP 位址空間視覺化（sunburst / heatmap）、電力配送鏈（power chain）、WAN 地理地圖、L2/L3 相依性以及 BGP 拓撲[^nve]。

NVE 並非自由開源軟體，而是 NetBox Cloud / Enterprise 商業方案的付費功能：機櫃、線纜、IP 空間、WAN 與相依性的「核心視圖」屬於 Professional 方案，而平面圖、電力鏈與 BGP 拓撲等「資料中心視圖」則屬於 Premium 方案[^nbpricing]。因此，想要在自架環境取得同類能力的使用者，必須自行組合自由開源軟體（FOSS）。

本報告以 NVE 的功能清單為對照基準，盤點可用的 FOSS 替代方案，並逐一標註其 GitHub 星數、授權條款與視覺化能力。核心結論是：**沒有任何單一 FOSS 專案完整涵蓋 NVE 全部功能，但透過 NetBox 或 Nautobot 外掛生態系組合，可覆蓋約八成功能；其中平面圖、拓撲圖與 IPAM 視覺化都有成熟替代品，而電力鏈與 3D 平面圖則較為薄弱。**

---

## 一、對照基準：NetBox Visual Explorer 的功能範圍

依據官方發表文章，NVE 包含下列視覺化模組[^nve]：

| 模組 | 說明 |
|---|---|
| 平面圖（floorplan） | 匯入 PDF / DXF / 圖片平面圖並校正比例，放置機櫃、劃分冷熱通道區域、疊加電力或密度指標、迷你地圖導覽、機櫃間距離量測；支援 2D 與 3D |
| 機櫃立面圖（rack elevation） | 單一機櫃細節或並排比較，顯示前後面板、依角色著色，可拖曳調整並寫回 NetBox |
| 電力鏈（power chain） | 從配電盤（panel）經饋線（feed）到裝置的電力配送追蹤，用於辨識備援缺口與單點故障 |
| 線纜拓撲（cable topology） | 埠對埠線纜連線的互動圖形，可高亮來源到目的地的路徑；大型網路自動聚合避免「毛球圖」 |
| IPAM 視覺化 | 以 sunburst 或 heatmap 呈現位址空間，可下鑽前綴、找出壅塞子網、依位址族或 VRF 過濾 |
| WAN 地理地圖 | 將站點標於世界地圖並以弧線呈現電路連線，可依供應商、電路型別或租戶過濾 |
| 相依性與 BGP 視圖 | 呈現裝置關係與對等連線工作階段 |

官方宣稱已針對數千座機櫃、數萬台裝置的環境測試[^nve]。

---

## 二、功能對照總表

下表將 NVE 的七個模組對應到 FOSS 方案。星數與授權以 GitHub API 於 2026-11-21 查詢之資料為準。

| NVE 模組 | 最接近的 FOSS 方案 | 星數 | 授權 |
|---|---|---|---|
| 平面圖（2D） | NetBox Floorplan Plugin[^fpp]；Nautobot Floor Plan App[^nbfp]；openDCIM 影像對位圖[^odcimfeat] | 129 / 15 / 371 | LGPL-3.0 / Apache-2.0 / GPL-3.0 |
| 平面圖（3D） | net3d[^net3d] | 29 | MIT |
| 機櫃立面圖 | openDCIM Graphical Cabinet Viewer[^odcimfeat]；phpIPAM RACK management[^phpipamfeat]；NetBox 內建 rack elevation[^nbdcim] | 371 / 2,828 / 21,914 | GPL-3.0 / GPL-3.0 / Apache-2.0 |
| 電力鏈 | netbox-topology-views 的 Show Power Feeds[^ntvreadme]；openDCIM 電源鏈與故障模擬[^odcimfeat] | 1,113 / 371 | Apache-2.0 / GPL-3.0 |
| 線纜拓撲 | netbox-topology-views[^ntv]；nextbox-ui-plugin[^nextbox]；Nautobot Topology Views[^nbntv] | 1,113 / 643 / 7 | Apache-2.0 / MIT / Apache-2.0 |
| IPAM 視覺化 | phpIPAM visual subnet display[^phpipamfeat]；ipv4-heatmap[^ipv4hm]；netbox-free-ip[^freeip] | 2,828 / 156 / 2 | GPL-3.0 / GPL-2.0 / 未標示 |
| WAN 地理地圖 | netbox-geo[^nbgeo]；netbox2mapgl（搭配 Grafana MapGL）[^mapgl] | 10 / 4 | Apache-2.0 / GPL-3.0 |
| 拓撲自動探索 | Netdisco[^netdisco]；SuzieQ[^suzieq]；Topolograph[^topolograph]；LibreNMS[^librenms] | 944 / 909 / 245 / 5,018 | BSD-3-Clause / Apache-2.0 / MIT / GPL-3.0 |

---

## 三、核心平台：DCIM / IPAM 骨幹

### 3.1 NetBox（自架社群版）

NetBox 本身以 Apache-2.0 授權，是 NVE 的資料來源，自架版即為 FOSS[^netbox]。它內建機櫃立面圖（前後視圖、裝置擺放）與線纜追蹤（cable trace）等視覺化能力[^nbdcim]。也就是說，即使不購買 NVE，自架 NetBox 已提供最基礎的機櫃與線纜視覺呈現；NVE 的價值在於更進階的平面圖、聚合拓撲與 IPAM 圖形。

### 3.2 Nautobot

Nautobot 由 Network to Code 維護，Apache-2.0 授權，定位為「網路 Source of Truth 與自動化平台」，是 NetBox 的分支後繼者，同樣具備 DCIM/IPAM 資料模型與外掛（App）生態系[^nautobot]。對於想以 Nautobot 為主體的使用者，社群已提供拓撲與平面圖 App（見第四節）。

### 3.3 openDCIM

openDCIM 是 GPL-3.0 授權的機房資產管理（DCIM）應用，以 PHP 撰寫[^opendcim]。其視覺化能力與 NVE 重疊度相當高[^odcimfeat]：

- **影像對位圖**：可上傳自訂圖片並為每座機櫃建立可點擊區域。
- **疊加圖層**：在平面圖上疊加電力、空間、溫度與重量容量。
- **電力鏈映射**：裝置 → 電源排（power strip）→ 配電盤 → 來源饋線的完整路徑映射。
- **圖形化機櫃檢視器**：使用者自備圖片後可呈現機櫃立面。
- **故障容忍模擬**：模擬某個配電盤或來源饋線中斷後受影響的裝置，功能上對應 NVE 的電力鏈備援缺口辨識。
- **交換器連線映射**：可記錄被歸類為交換器之裝置的網路連線。

其最新版本為 3.2[^odcimfeat]，GitHub 倉庫在 2026-11-21 仍有提交[^opendcim]。

### 3.4 RackTables

RackTables 是 GPL-2.0 授權的機房與 IP 管理工具，具備 IPv4 位址空間樹（IP tree）與機櫃／機架（row）物件化模型[^racktables]。其 IP tree 可顯示已配置與未配置的網段，並可選擇是否掃描 used/spare IP 以兼顧效能[^racktables]。這是 NVE IPAM 視圖的簡化替代，但屬靜態樹狀呈現而非 sunburst/heatmap。

### 3.5 phpIPAM

phpIPAM 是 GPL-3.0 授權的 PHP 網頁式 IPAM，官方功能清單明列「Visual subnet display」（視覺化子網顯示）與「RACK management」（機櫃管理）[^phpipamfeat]。其視覺化子網可呈現子網內每位址的使用狀態（近似 heatmap），機櫃管理則提供機櫃圖面。社群標籤包含 `network-visualization`[^phpipam]。最新版本為 1.9（2026-11 發布）[^phpipamfeat]。

### 3.6 TeemIP

TeemIP 是建構於 iTop CMDB 之上的 IPAM 工具套件，核心模組以 AGPL-3.0 授權[^teemip]。它管理 IPv4/IPv6 空間（block、subnet、range、address）、VLAN、DNS 網域、WAN 連結與 AS 號碼，並提供 iTop 整合的分層 IP 區塊呈現；另有多個擴充模組，包含線纜管理（teemip-cable-mgmt，GPL-3.0）與 IP 探索[^teemip]。相對於 NVE，TeemIP 的強項是階層式 IP 管理與工作流程，而非圖形化拓撲。

---

## 四、NetBox 外掛生態系（最直接的替代路徑）

由於 NVE 是 NetBox 的加值層，最直接的 FOSS 替代就是安裝社群外掛。

### 4.1 netbox-topology-views（拓撲 + 電力鏈）

- **星數**：1,113（2026-11-21 查詢，最後推送 2026-11-20）[^ntv]
- **授權**：Apache-2.0[^ntv]
- **能力**：依 NetBox 中的線纜建立拓撲視圖，可依名稱、站點、標籤、裝置角色過濾，支援匯出 XML（供 draw.io / diagrams.net 使用）或 PNG[^ntvreadme]。
- **與 NVE 對應**：其篩選面板提供「Show Cables」（實線，線纜顏色）、「Show Logical Connections」（黃色虛線，跨 patch panel 的端對端連線）、「Show Circuit Terminations」（藍色虛線）、**「Show Power Feeds」（電力排與電力埠連線，實線）**、「Show Wireless Links」（藍色點線）[^ntvreadme]。最後一項即為 NVE 電力鏈的輕量替代。
- **座標與分組**：支援 Coordinate Groups，可讓同一拓撲有多組節點排列；節點預設由物理引擎自動布局，拖曳後則固定儲存[^ntvreadme]。這部分對應 NVE 的互動式拓撲操作。

### 4.2 nextbox-ui-plugin（次世代拓撲視圖）

- **星數**：643（最後推送 2025-05-24）[^nextbox]
- **授權**：外掛程式碼為 MIT；內嵌的 topoSphere SDK 為專有授權，但對 NextBox UI Plugin 與 NetBox Community 免費[^nextboxreadme]
- **能力**：自適應且可擴展的網路拓撲圖，具備進階自動布局演算法、延伸過濾器與多種匯出格式；支援深色模式；可依裝置角色自動分層（border / edge / core / distribution / leaf / spine / access）並依裝置型號或標籤選擇圖示[^nextboxreadme]。它亦宣稱可透過 Dockerfile 安裝於 netbox-docker 環境[^nextboxreadme]。
- **注意**：此專案的 `topoSphere SDK` 為專有元件，嚴格而言並非 100% FOSS；但其外掛本體與社群使用免費，因此列為準 FOSS 選項。

### 4.3 netbox-floorplan-plugin（平面圖）

- **星數**：129（最後推送 2026-10-23）[^fpp]
- **授權**：LGPL-3.0[^fpp]
- **能力**：為站點或 location 提供平面圖繪製，機櫃與未上架裝置以相對於實體房間的位置繪於畫布上，並可點擊跳轉至對應物件；支援標籤、區域、牆面與著色等中繼資料、鍵盤操作、匯出 SVG，以及每張平面圖上傳或連結一張背景圖[^fppreadme]。
- **限制**：機櫃要顯示真實比例，需為該機櫃指定 rack type 並設定外寬與深度，否則會以預設尺寸繪製[^fppreadme]。
- **與 NVE 差異**：不支援 PDF / DXF 自動校正比例（NVE 有此功能），也不支援 3D[^nve]。此專案原始 fork 自已封存的 `tbotnz/netbox_floorplan`（80 星、LGPL-3.0、已封存）[^tbotnz]。

### 4.4 netbox_drawio（CESNET，draw.io 整合）

- **星數**：12（建立於 2026-08-20，最後推送 2026-10-02）[^drawio]
- **授權**：Apache-2.0[^drawio]
- **能力**：將 NetBox 與 draw.io（diagrams.net）整合，適合以「圖表即程式碼」方式產出網路圖。這是 NVE 互動拓撲的靜態檔案式替代，而非即時互動圖。

### 4.5 netbox-geo（WAN 地理地圖）

- **星數**：10（最後推送 2023-02-15）[^nbgeo]
- **授權**：Apache-2.0（README 標示）[^nbgeoreadme]
- **能力**：以 GeoDjango 為基礎，可將站點、線纜、電路等在地圖上視覺化[^nbgeoreadme]。這是 NVE WAN 地理地圖最直接對應的 FOSS 外掛，但專案自 2023 年起未再更新，且相容性僅列 NetBox 3.4[^nbgeoreadme]，屬低維護度選項。
- **替代**：`netbox2mapgl`（4 星、GPL-3.0）會將 NetBox 拓撲同步到 Grafana 的 MapGL 面板，等於以 Grafana 為地理視覺化層[^mapgl]。

### 4.6 net3d（3D 基礎設施視覺化）

- **星數**：29（建立於 2026-06-14，最後推送 2026-09-29）[^net3d]
- **授權**：MIT[^net3d]
- **能力**：以 TypeScript 撰寫，從 NetBox 或 Infrahub 讀取資料，提供「地球 → 站點 → 機櫃」三層的 3D 基礎設施視覺化[^net3d]。這是 NVE 3D 平面圖與 WAN 地圖的少數開源對應，但專案相當年輕、社群規模小。

### 4.7 其他相關外掛

- **netbox-free-ip**（2 星，授權未標示）：受 phpIPAM 啟發，以彩色格狀呈現前綴內的可用 IP 區段，支援快速新增／移除與 CSV 匯出，屬 NVE IPAM heatmap 的微型替代[^freeip]。
- **nrx**（125 星、Apache-2.0）：NetBox 拓撲匯出工具，可輸出為 Containerlab、Cisco CML、NVIDIA Air 格式，常搭配 Graphite 視覺化[^nrx]。
- **netreplica/graphite**（91 星、Apache-2.0）：網路拓撲視覺化前端，支援 NetBox（需搭配 nrx 匯出）、Containerlab、Netlab；不需在 NetBox 上安裝外掛，並可用 Jinja2 樣板自訂拓撲資料定義[^graphite]。
- **ntmap**（165 星、MIT）：以 NetBox 為資料源的網路拓撲地圖，支援依站點與標籤呈現實體鏈路；但最後推送為 2023-01-10，維護度低[^ntmap]。
- **shumoku**（185 星、AGPL-3.0）：以 YAML 描述拓撲並產生 SVG/HTML 圖（diagram as code），具 NetBox 整合[^shumoku]。
- **stratagram**（2 星、Apache-2.0）：Rust 撰寫的網路圖工具，含桌面版、瀏覽器編輯器與 NetBox 外掛；專案極新（2026-11 建立）[^stratagram]。
- **grafana-rackelevation-panel**（2 星、Apache-2.0）：Grafana 面板外掛，可依 NetBox / CSV / SQL / Prometheus 資料逐 U 繪製機櫃立面，並依裝置狀態或指標著色[^rackpanel]。

---

## 五、Nautobot App 生態系

### 5.1 nautobot-app-floor-plan（平面圖）

- **星數**：15（最後推送 2026-11-10）[^nbfp]
- **授權**：Apache-2.0[^nbfplicense]
- **能力**：為 Nautobot Location 建立網格化平面圖表示，並標示各機櫃在其中的相對位置[^nbfpreadme]。與 NetBox 版本相比，它採用網格（grid）而非自由畫布，功能較簡化，但有官方組織維護、與 Nautobot 版本同步較佳。

### 5.2 nautobot-topology-views（拓撲）

- **星數**：7（最後推送 2026-10-29）[^nbntv]
- **授權**：Apache-2.0[^nbntv]
- **能力**：為 NetBox 的 `netbox-topology-views` 之 Nautobot 移植版，功能對應原專案的拓撲繪製[^nbntv]。社群規模明顯較小。

---

## 六、以「實際網路資料」自動產生拓撲的工具

NVE 的拓撲來自 NetBox 中人工維護的線纜資料；若希望拓撲直接由網路設備探索而來，可考慮下列 FOSS 工具。

### 6.1 Netdisco

- **星數**：944（最後推送 2026-11-20）[^netdisco]
- **授權**：BSD-3-Clause[^netdisco]
- **能力**：以 Perl 撰寫的網頁式網路管理工具，透過 SNMP 探索並呈現網路拓撲；GitHub 標籤明確包含 `network-visualization`[^netdisco]。

### 6.2 SuzieQ

- **星數**：909（最後推送 2026-08-29）[^suzieq]
- **授權**：Apache-2.0[^suzieq]
- **能力**：多廠牌網路可觀測性框架，支援 Cumulus、EOS、IOS、IOS-XE、IOS-XR、Junos（QFX/EX/MX/SRX）、Panos、NX-OS、SONIC 與 Linux[^suzieqreadme]。可透過 CLI、GUI、REST API 分析，具備**路徑（path）視覺化**（含 EVPN overlay）與 **asserts** 機制，並能呈現 OSPF、BGP、EVPN 等控制平面狀態[^suzieqreadme]。它對應 NVE 的「BGP 與相依性視圖」，且是「由真實資料驅動」而非人工登錄。另有商業版 SuzieQ Enterprise[^suzieqreadme]。

### 6.3 Topolograph

- **星數**：245（最後推送 2026-11-20）[^topolograph]
- **授權**：MIT[^topolograph]
- **能力**：可依單一網路設備的 LSDB 輸出，視覺化 OSPF / IS-IS 拓撲，並檢查來源到目的的最短路徑，以及沿路徑發生鏈路或節點失效後的結果[^topolograph]。支援 Cisco、Juniper、Huawei、Nokia、MikroTik、FRR、Quagga、Extreme、Ericsson、Ruckus、Palo Alto 等廠牌[^topolograph]。

### 6.4 LibreNMS 與 OpenNMS

- **LibreNMS**：5,018 星、GPL-3.0（含第三方套件例外），社群版網路監控系統，支援 SNMP 與 RRD，具備網路地圖能力[^librenms]。
- **OpenNMS**：1,183 星，授權標示為非 SPDX 標準（Other），企業級開源網路管理平台[^opennms]。

---

## 七、IP 位址空間視覺化的專門工具

NVE 的 IPAM 視圖提供 sunburst 與 heatmap 兩種呈現[^nve]。FOSS 面向：

| 專案 | 星數 | 授權 | 視覺化型態 |
|---|---|---|---|
| ipv4-heatmap[^ipv4hm] | 156 | GPL-2.0 | Hilbert 曲線 IPv4 熱圖，輸出 4096×4096 PNG，每像素代表一個 /24 |
| rust-ipv4heatmap[^rusthm] | 13 | MIT | ipv4-heatmap 的 Rust 重寫版，支援 JSON 註解與 SVG 圖例 |
| hrbrmstr/ipv4-heatmap[^hrbrm] | 34 | GPL（衍生自 measurement-factory） | 加入 viridis 與 ColorBrewer 色盤的 C 版分支 |
| phpIPAM visual subnet[^phpipamfeat] | 2,828 | GPL-3.0 | 子網內 IP 使用狀態的視覺化格狀呈現 |
| netbox-free-ip[^freeip] | 2 | 未標示 | NetBox 前綴內可用 IP 區段的彩色格狀圖 |

需要特別指出：**在調查中並未找到專門提供 IPAM sunburst 視覺化的獨立 FOSS 專案**。sunburst 型態通常是一般視覺化函式庫（如 D3.js）的應用，需自行開發；這點是 NVE IPAM 視圖中最難直接替代的部分。

---

## 八、觀察與建議

### 8.1 組合建議

若以自架 NetBox 為主體，最貼近 NVE 功能的 FOSS 組合為：

1. **netbox-topology-views** 取得線纜拓撲、邏輯連線、電路終端、電力饋線與無線鏈路視圖。
2. **netbox-floorplan-plugin** 取得 2D 平面圖與機櫃位置。
3. **net3d** 取得 3D 與地球層級的空間視覺化（可視為 WAN 地圖的替代）。
4. **nextbox-ui-plugin** 取得自動布局的次世代拓撲圖與角色分層。
5. **phpIPAM** 或 **netbox-free-ip** 補足子網層級的 IP 使用狀態視覺化。
6. 若需要由真實網路資料自動產生拓撲，另部署 **Netdisco** 或 **SuzieQ**。

### 8.2 能力落差

| NVE 能力 | FOSS 覆蓋程度 | 說明 |
|---|---|---|
| 機櫃立面圖 | 完整 | NetBox 內建、openDCIM、phpIPAM 皆有 |
| 線纜／邏輯拓撲 | 接近完整 | netbox-topology-views 具備路徑高亮與自動聚合以外的多數功能 |
| 平面圖 2D | 部分 | 有外掛，但不支援 PDF/DXF 比例校正 |
| 電力鏈 | 部分 | 僅有連線層級的呈現（netbox-topology-views 的 Show Power Feeds）與 openDCIM 的故障模擬，缺乏 NVE 的互動鏈狀追蹤 |
| IPAM heatmap | 部分 | phpIPAM 與 netbox-free-ip 提供子網格狀；全域 heatmap 可用 ipv4-heatmap 離線產生 |
| IPAM sunburst | 缺乏 | 未找到專門的 FOSS 實作 |
| WAN 地理地圖 | 部分 | netbox-geo 與 netbox2mapgl 可用，但維護度低 |
| 3D 平面圖 | 缺乏 | 僅 net3d 提供接近的地球→站點→機櫃 3D 視圖，但非機房內部 3D |

### 8.3 規模化考量

NVE 的核心賣點之一是針對數千座機櫃、數萬台裝置的效能調校，並具備大型網路的拓撲自動聚合[^nve]。在 FOSS 側，netbox-topology-views 的節點布局同時提供物理引擎自動布局與手動座標儲存[^ntvreadme]，但在極大規模下的效能表現並無公開基準可供比較；這是選擇 FOSS 方案時必須自行驗證的風險點。

### 8.4 授權注意

- `nextbox-ui-plugin` 內嵌的 **topoSphere SDK 為專有授權**，僅對該外掛與 NetBox Community 免費[^nextboxreadme]。若需再散布或商用於其他情境，須先確認授權範圍。
- `shumoku` 採用 **AGPL-3.0**[^shumoku]，若以網路服務形式提供修改版，需依 AGPL 開源。
- `netbox-floorplan-plugin` 為 **LGPL-3.0**[^fpp]，屬弱 copyleft，對 NetBox 本體（Apache-2.0）的整合通常無影響。
- openDCIM、phpIPAM、RackTables、LibreNMS 為 GPL 系（GPL-3.0 或 GPL-2.0），若嵌入商業產品需注意 copyleft 義務。

---

## 九、結論

NetBox Visual Explorer 是將 NetBox 資料轉為空間與圖形理解的商業加值層。在自由開源領域：

- **最成熟的替代路徑是 NetBox 外掛生態系**，其中 `netbox-topology-views`（1,113 星）與 `nextbox-ui-plugin`（643 星）在拓撲視覺化上已有相當規模的社群；`netbox-floorplan-plugin`（129 星）補上平面圖。
- **openDCIM 是唯一在單一 FOSS 專案內同時覆蓋平面圖、機櫃立面、電力鏈與故障模擬的選項**，但其技術棧較舊（PHP 8 / MySQL）且網路拓撲呈現較弱。
- **IPAM 的 sunburst 視覺化在 FOSS 中缺乏現成實作**，是 NVE 最難直接替代的功能。
- **拓撲若希望由真實資料自動產生**，Netdisco（944 星）、SuzieQ（909 星）與 Topolograph（245 星）是可互補的選擇，且 SuzieQ 直接對應 NVE 的 BGP 與相依性視圖。

---

## 參考文獻

[^nve]: Beevers, K. (2026, March 23). See your infrastructure: Introducing NetBox Visual Explorer. NetBox Labs. Retrieved 2026-11-21, from https://netboxlabs.com/blog/see-your-infrastructure-introducing-netbox-visual-explorer/

[^nbpricing]: NetBox Labs. (n.d.). NetBox Labs platform pricing: Cloud and Enterprise. Retrieved 2026-11-21, from https://netboxlabs.com/pricing/

[^netbox]: NetBox Community. (n.d.). netbox: The premier source of truth powering network automation [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/netbox-community/netbox

[^nbdcim]: NetBox Labs. (n.d.). NetBox documentation: DCIM. Retrieved 2026-11-21, from https://netboxlabs.com/docs/netbox/

[^nautobot]: Nautobot. (n.d.). nautobot: Network source of truth and network automation platform [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/nautobot/nautobot

[^opendcim]: openDCIM. (n.d.). openDCIM: An open source (GPL v3) data center inventory management (DCIM) application [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/opendcim/openDCIM

[^odcimfeat]: openDCIM.org. (n.d.). openDCIM features. Retrieved 2026-11-21, from https://opendcim.org/features.html

[^racktables]: RackTables. (n.d.). RackTables current development repository [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/RackTables/racktables

[^phpipamfeat]: phpIPAM. (n.d.). phpIPAM feature list. Retrieved 2026-11-21, from https://phpipam.net/documents/features/

[^phpipam]: phpIPAM. (n.d.). phpipam development repository [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/phpipam/phpipam

[^teemip]: TeemIP. (n.d.). teemip-core-ip-mgmt: Core modules of TeemIP, a web based IP address management tool [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/teemIP/teemip-core-ip-mgmt

[^ntv]: NetBox Community. (n.d.). netbox-topology-views: A NetBox plugin that draws topology views [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/netbox-community/netbox-topology-views

[^ntvreadme]: NetBox Community. (n.d.). NetBox Topology Views plugin README. GitHub. Retrieved 2026-11-21, from https://raw.githubusercontent.com/netbox-community/netbox-topology-views/develop/README.md

[^nextbox]: iDebugAll. (n.d.). nextbox-ui-plugin: Next-gen topology visualization plugin for NetBox powered by topoSphere SDK [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/iDebugAll/nextbox-ui-plugin

[^nextboxreadme]: iDebugAll. (n.d.). NextBox UI plugin README. GitHub. Retrieved 2026-11-21, from https://raw.githubusercontent.com/iDebugAll/nextbox-ui-plugin/master/README.md

[^fpp]: NetBox Community. (n.d.). netbox-floorplan-plugin: NetBox floorplan plugin [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/netbox-community/netbox-floorplan-plugin

[^fppreadme]: NetBox Community. (n.d.). NetBox Floorplan plugin README. GitHub. Retrieved 2026-11-21, from https://raw.githubusercontent.com/netbox-community/netbox-floorplan-plugin/main/README.md

[^tbotnz]: tbotnz. (n.d.). netbox_floorplan [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/tbotnz/netbox_floorplan

[^drawio]: CESNET. (n.d.). netbox_drawio: Drawio integration for NetBox [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/CESNET/netbox_drawio

[^nbgeo]: wholesailnetworks. (n.d.). netbox-geo: NetBox GeoDjango plugin for visualizing sites, cables, circuits, etc. [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/wholesailnetworks/netbox-geo

[^nbgeoreadme]: wholesailnetworks. (n.d.). NetBox geospatial plugin README. GitHub. Retrieved 2026-11-21, from https://raw.githubusercontent.com/wholesailnetworks/netbox-geo/main/README.md

[^mapgl]: hamptik. (n.d.). netbox2mapgl: Synchronise NetBox topology to MapGL in Grafana [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/hamptik/netbox2mapgl

[^net3d]: Hildenbrand, H. (n.d.). net3d: 3D infrastructure visualization from NetBox or Infrahub (globe → site → rack) [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/hervehildenbrand/net3d

[^freeip]: Estylowy. (n.d.). netbox-free-ip: NetBox 4.x plugin inspired by PHP IPAM [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/Estylowy/netbox-free-ip

[^nrx]: Netreplica. (n.d.). nrx topology exporter: NetBox to Containerlab, CML, NVIDIA Air [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/netreplica/nrx

[^graphite]: Netreplica. (n.d.). Graphite: Visualization for network topologies [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/netreplica/graphite

[^ntmap]: den-it. (n.d.). ntmap: Network topology map using NetBox as a data source [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/den-it/ntmap

[^shumoku]: konoe-akitoshi. (n.d.). shumoku: Generate network topology diagrams from YAML [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/konoe-akitoshi/shumoku

[^stratagram]: shinsterneck. (n.d.). stratagram: Network diagrams in minutes [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/shinsterneck/stratagram

[^rackpanel]: Kentrow. (n.d.). grafana-rackelevation-panel: Grafana panel plugin that draws your racks unit by unit [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/Kentrow/grafana-rackelevation-panel

[^nbfp]: Nautobot. (n.d.). nautobot-app-floor-plan: Easily view floor plans in Nautobot [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/nautobot/nautobot-app-floor-plan

[^nbfpreadme]: Nautobot. (n.d.). Nautobot Floor Plan README. GitHub. Retrieved 2026-11-21, from https://raw.githubusercontent.com/nautobot/nautobot-app-floor-plan/develop/README.md

[^nbfplicense]: Nautobot. (n.d.). Nautobot Floor Plan license. GitHub. Retrieved 2026-11-21, from https://raw.githubusercontent.com/nautobot/nautobot-app-floor-plan/develop/LICENSE

[^nbntv]: gioccher. (n.d.). nautobot-topology-views: A Nautobot app that draws topology views [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/gioccher/nautobot-topology-views

[^netdisco]: Netdisco. (n.d.). netdisco: A web-based network management tool [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/netdisco/netdisco

[^suzieq]: Netenglabs. (n.d.). suzieq: Using network observability to operate and design healthier networks [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/netenglabs/suzieq

[^suzieqreadme]: Netenglabs. (n.d.). SuzieQ README. GitHub. Retrieved 2026-11-21, from https://raw.githubusercontent.com/netenglabs/suzieq/develop/README.md

[^topolograph]: Vadims06. (n.d.). Topolograph: Online project which can visualize OSPF/ISIS topology [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/Vadims06/topolograph

[^librenms]: LibreNMS. (n.d.). librenms: Community-based GPL-licensed network monitoring system [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/librenms/librenms (授權確認見 https://raw.githubusercontent.com/librenms/librenms/master/LICENSE.txt)

[^opennms]: OpenNMS. (n.d.). opennms: Enterprise-grade open-source network management platform [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/OpenNMS/opennms

[^ipv4hm]: Measurement Factory. (n.d.). ipv4-heatmap: Generate Hilbert curve heatmaps of the IPv4 address space [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/measurement-factory/ipv4-heatmap

[^rusthm]: Rudis, B. (n.d.). rust-ipv4heatmap: Generate IPv4 12th order Hilbert heatmaps [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/hrbrmstr/rust-ipv4heatmap

[^hrbrm]: Rudis, B. (n.d.). ipv4-heatmap [Computer software]. GitHub. Retrieved 2026-11-21, from https://github.com/hrbrmstr/ipv4-heatmap
