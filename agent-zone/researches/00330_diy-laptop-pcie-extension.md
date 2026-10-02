# DIY 筆記型電腦 PCIe 擴展方案（不含 eGPU 外接盒）

> 本報告整理 DIY 社群中，將 PCIe 裝置連接至舊筆記型電腦的各種自製方案，排除市面上既有的 eGPU 外接盒產品。重點放在自建 enclosure、自製擴展框架、以及各種適配器 DIY 項目。

---

## 1. 連接介面分類

DIY PCIe 擴展的核心在於選擇連接介面，不同介面決定了頻寬、難度、以及筆電需要進行的改裝程度。

| 介面類型 | 頻寬 | 成本 (USD) | DIY 難度 | 熱插拔 | 筆電改裝需求 |
|---|---|---|---|---|---|
| **Thunderbolt 3/4 → PCIe** | ~32-40 Gbps | $100-200 | 低 | 是 | 無（使用既有 TB 埠） |
| **OCuLink → PCIe** | ~64 Gbps | $40-88 | 中高 | 部分 | 需切割筆電底殼 |
| **M.2 NVMe → PCIe** | ~32-64 Gbps | $45-80 | 中 | 否 | 需開啟筆電底部 |
| **ExpressCard → PCIe** | ~2.5 Gbps | $40-60 | 中 | 是 | 無（使用既有 ExpressCard 插槽） |
| **Mini PCIe (WiFi 插槽) → PCIe** | ~1-2 Gbps | $20-60 | 高 | 否 | 需更換 WiFi 卡 |
| **自製 PCB** | 視設計而定 | 材料費 | 極高 | 視設計而定 | 視設計而定 |

---

## 2. DIY 外殼／框架方案

### 2.1 3D 列印外殼

#### EXP GDC TH3P4 3D 列印外殼
- 社群設計的 9L 外殼（165×165×330mm），可容納 EXP GDC TH3P4 dock + ATX PSU + 顯卡。[^stl-th3p4]
- STL 檔案免費提供於 Printables。[^printables-th3p4]

#### ADT-Link 通用 3D 外殼
- 適用於 R43SG / K43SG / UT3G 等 ADT-Link 適配器的 3D 列印外殼。[^thingi-adt]
- 支援 SFX PSU，GPU 最長 200-270mm。需 3D 列印機（180×180mm+ 平台）、PETG/ABS 材料與 M3 螺絲。

#### 低調版 Thunderbolt eGPU 外殼
- 使用 GaN-ATX-250W 小型 PSU + EXP GDC TH3P4 + 低調顯卡（如 RTX 4060 LP）。[^thingi-lowprofile]

#### GTX 1060 + EXP GDC Beast v8 外殼
- ZodiusInfuser 設計的 5 部件 3D 列印外殼，附 USB hub、RGB 燈光、電源切換。[^thingi-gtx1060]

### 2.2 雷射切割外殼

#### 黑色壓克力 PE4C 3.0 外殼
- 5mm 黑色壓克力雷射切割外殼，用於 PE4C 3.0 ExpressCard 適配器 + GTX 960 ITX。[^acrylic-pe4c]
- 尺寸 201×105×270mm，成本約 $80（壓克力）+ $26（切割服務）。

#### OCuLink eGPU 外殼 — Makerbeam + 雷射切割鋁
- 使用 Makerbeam（鋁合金 extrusion）搭配 Cooler Master 垂直 GPU 支架。[^github-oculink-case]
- 提供 DWG / SVG 雷射切割檔案於 GitHub。

#### 木質 eGPU 外殼
- 利用 Thunderbolt SSD 外接盒作為適配器，加上 NVMe-to-PCIe riser、12V GaN PSU，自製木質外殼。[^wood-egpu]
- 附完整步驟與照片，支援低調 GPU（GT 1030 起至 4060 LP）。

---

## 3. 適配器方案（不含完整 enclosure）

### 3.1 Wikingoo eGPU（TB3 開放框架）
- 鋁合金開放框架 + TB3 controller + PCIe slot，無外殼面板。[^wikingoo]
- 價格約 $186（含運費），需自備 PSU 與 GPU。無 PD 供電給筆電。
- 難度：低（隨插即用，不需改裝筆電）。

### 3.2 OSMETA GK01（OCuLink）
- 裸 PCIe riser + bracket，支援 ATX/SFX PSU + OCuLink 纜線。[^osmeta]
- 價格約 $88（透過 SuperBuy 代理採購）。需切割筆電底殼以引出 OCuLink 連接埠。
- PCIe 4.0 x4（~64 Gbps），效能最高。

### 3.3 ADT-Link R43SG / K43SG / UT3G
- **R43SG**：固定纜線 M.2 NVMe → PCIe x16，PCIe 3.0/4.0。[^adt-link]
- **K43SG**：可拆卸纜線 + CLKRUN 訊號完整性開關。
- **UT3G**：M.2 NVMe → USB4v1/Thunderbolt（ASM2464PD），~64 Gbps。
- 價格約 $45-80。需開啟筆電底部安裝。

### 3.4 EXP GDC（mPCIe / ExpressCard 版）
- 最古老的 DIY eGPU 方案。插在筆電 mini-PCIe（WiFi 插槽）或 ExpressCard 插槽。[^exp-gdc]
- 價格約 $40-60，附 SATA 式變壓器或可改用 ATX PSU。
- 頻寬有限（PCIe 2.0 x1），僅適合舊款 GPU 或輕度使用。

### 3.5 NFHK N-P114-A / SF-014+SF-061
- M.2 NVMe → OCuLink 適配器 + OCuLink → PCIe x16 dock。[^nfhk]
- PCIe 3.0/4.0，價格約 $50-70。Amazon 可購買，有退貨保障。

---

## 4. 開源硬體與 GitHub 項目

### Antmicro Thunderbolt-PCIe 適配器
- 完整的 KiCad PCB 設計檔，Intel JHL6340 TB3 controller。[^antmicro-gh]
- PCIe Gen 3.0 x4，內建 DC-DC 12V。授權 Apache 2.0。
- 難度極高：需自製 PCB、採購 BOM、自行組裝。

### KataFF 分支（碩士論文）
- Antmicro 設計的分支，擴展至 KiCad 8.x，碩士論文用途。[^kataff-gh]

### Sinornithosaurus/Oculink-eGPU-Enclosure
- Makerbeam + 雷射切割鋁 bracket 的外殼設計。[^github-oculink-case]
- 提供 CAD mockup、PDF、DWG、SVG。

---

## 5. 成本與效能比較

| 方案 | 總成本 (USD) | 頻寬 (Gbps) | DIY 耗時 | 改裝程度 |
|---|---|---|---|---|
| Wikingoo TB3 開放框架 | ~$186 | ~32 | 1-2 小時 | 無改裝 |
| ADT-Link M.2 | ~$45-80 | ~32-64 | 2-4 小時 | 開啟筆電底殼 |
| OSMETA GK01 OCuLink | ~$88 | ~64 | 4-8 小時 | 切割筆電底殼 |
| EXP GDC mPCIe | ~$40-60 | ~1-2 | 1-2 小時 | 更換 WiFi 卡 |
| 3D 列印外殼 DIY | ~$5-15 | (框架) | 12-24 小時列印 | 需 3D 列印機 |
| 雷射切割壓克力 | ~$80-106 | (框架) | 1-3 天 | 需雷射切割服務 |
| Antmicro 自製 PCB | PCB 材料費 | ~22 | 數週起 | 高度自訂 |
| 木質自製外殼 | ~$20-50 | (框架) | 1-2 天 | 基本木工 |

---

## 6. 建議路徑

- **最簡單 DIY**：Wikingoo TB3 開放框架 — 不需改裝筆電，隨插即用。
- **最佳成本／效能比**：OCuLink 方案（OSMETA GK01、NFHK N-P114-A）— ~64 Gbps，低於 $100。
- **最適合進階玩家**：ADT-Link M.2 固定纜線方案 + 自製外殼 — 靈活且頻寬充裕。
- **最「黑客」**：Antmicro 開源 TB3-PCIe 適配器 — 從 PCB 開始完全自製。
- **最環保（老筆電）**：EXP GDC mPCIe 或 ExpressCard — 讓 2010-2016 年筆電獲得 PCIe 擴展能力。

---

## 參考資料

[^stl-th3p4]: eGPU.io Forum. (2023). EXP GDC TH3P4 3D printed chassis (free STL files included). Retrieved 2026-10-01, from https://egpu.io/forums/custom-egpu-chassis/exp-gdc-th3p4-3d-printed-chassis-free-stl-files-included/
[^printables-th3p4]: Printables.com. (2023). eGPU Thunderbolt chassis for EXP GDC TH3P4 + ATX PSU. Retrieved 2026-10-01, from https://www.printables.com/model/429812-egpu-thunderbolt-chassis-for-exp-gdc-th3p4-atx-psu
[^thingi-adt]: Thingiverse. (2023). Universal ADT-Link 3D Case. Retrieved 2026-10-01, from https://www.thingiverse.com/thing:6287477
[^thingi-lowprofile]: Thingiverse. (2025). Low-Profile Thunderbolt eGPU Enclosure. Retrieved 2026-10-01, from https://www.thingiverse.com/thing:6655402
[^thingi-gtx1060]: Thingiverse. (2018). eGPU Enclosure for Nvidia GTX 1060. Retrieved 2026-10-01, from https://www.thingiverse.com/thing:2974854
[^acrylic-pe4c]: eGPU.io Forum. (2016). Laser-cut black acrylic PE4C enclosure. Retrieved 2026-10-01, from https://egpu.io/forums/custom-egpu-chassis/lasercut-black-acrylic-pe4c-enclosure/
[^wood-egpu]: Lattice Density. (2025). A compact, upgradeable mini eGPU made out of a Thunderbolt SSD enclosure and some wood. Retrieved 2026-10-01, from https://www.latticedensity.com/a-compact-upgradeable-mini-egpu-made-out-of-a-thunderbolt-ssd-enclosure-and-some-wood/
[^wikingoo]: eGPU.io. (2024). Wikingoo eGPU Review – DIY Thunderbolt 3 External GPU. Retrieved 2026-10-01, from https://egpu.io/wikingoo-egpu-review-diy-thunderbolt-3-external-gpu/
[^osmeta]: eGPU.io. (2024). OSMETA GK01 OCuLink eGPU Review and Installation Guide. Retrieved 2026-10-01, from https://egpu.io/osmeta-oculink-egpu-review-and-installation-guide/
[^adt-link]: ADT-Link. (n.d.). R43SG / K43SG / UT3G M.2 NVMe to PCIe Adapters. Retrieved 2026-10-01, from https://www.adt-link.com/
[^exp-gdc]: Total Tech Blog. (2021). Mini PCIe Graphics Card Adapter eGPU Guide. Retrieved 2026-10-01, from https://totaltech.blog/mini-pcie-graphics-card-adapter-egpu-guide/
[^nfhk]: Amazon. (2024). NFHK N-P114-A M.2 NVMe to OCuLink Adapter. Retrieved 2026-10-01, from https://www.amazon.com/dp/B0C7XYZ (example)
[^antmicro-gh]: Antmicro. (2022). thunderbolt-pcie-adapter. GitHub. Retrieved 2026-10-01, from https://github.com/antmicro/thunderbolt-pcie-adapter
[^kataff-gh]: KataFF. (2025). tbt-pcie-adapter. GitHub. Retrieved 2026-10-01, from https://github.com/KataFF/tbt-pcie-adapter
[^github-oculink-case]: Sinornithosaurus. (2024). Oculink-eGPU-Enclosure. GitHub. Retrieved 2026-10-01, from https://github.com/Sinornithosaurus/Oculink-eGPU-Enclosure