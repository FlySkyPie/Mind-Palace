# PCI Express (PCIe) 介紹

## 概述

**PCI Express**（Peripheral Component Interconnect Express，正式縮寫 **PCIe** 或 **PCI-E**）是一種高效能電腦內部擴充匯流排標準，由 **PCI-SIG**（PCI Special Interest Group）制定與維護，用於連接顯示卡、固態硬碟（SSD）、網路卡、音效卡等周邊裝置。[^wiki-pcie]

PCIe 採用**序列點對點**（serial, point-to-point）架構，取代舊式的 PCI（Peripheral Component Interconnect）共享並列匯流排，以及專用於顯示卡的 AGP（Accelerated Graphics Port）。[^wiki-pcie]

## 歷史發展

### 開發起源（2001）

PCI Express 的研發始於 Intel 內部，最初代號為 **HSI**（High Speed Interconnect），後改名 **3GIO**（3rd Generation I/O）。由 Intel 工程師組成的 **Arapaho Work Group**（AWG）起草規範。[^wiki-pcie]

2001 年 8 月，Compaq、Dell、IBM、Intel、Microsoft 聯合宣布與 PCI-SIG 合作，定義新一代序列 I/O 互連架構，代號 **Arapahoe**。[^intel-press-2001]

### 正式命名與發布（2002–2003）

2002 年 4 月，Arapaho Work Group 將 3GIO 1.0 規範移交 PCI-SIG，並正式更名為 **PCI Express**。[^eetimes-2002] 2003 年，PCI-SIG 發布 PCI Express 1.0a 規格。[^wiki-pcie]

### 版本演進

| 版本 | 發布年份 | 傳輸率 (per lane) | 吞吐量 (per lane) | 編碼方式 | 主要變更 |
|:---|:---|:---:|:---|:---|:---|
| 1.0a | 2003 | 2.5 GT/s | 250 MB/s | 8b/10b | 初始版本 |
| 1.1 | 2005 | 2.5 GT/s | 250 MB/s | 8b/10b | 釐清規範，完全相容 |
| 2.0 | 2007 | 5.0 GT/s | 500 MB/s | 8b/10b | 頻寬翻倍 |
| 2.1 | 2009 | 5.0 GT/s | 500 MB/s | 8b/10b | 導入 3.0 的管理功能 |
| 3.0 | 2010 | 8.0 GT/s | ~985 MB/s | 128b/130b | 編碼開銷降至 ~1.54% |
| 4.0 | 2017 | 16.0 GT/s | ~1.969 GB/s | 128b/130b | 頻寬翻倍 |
| 5.0 | 2019 | 32.0 GT/s | ~3.938 GB/s | 128b/130b | 頻寬翻倍 |
| 6.0 | 2022（硬體 2025） | 64.0 GT/s | ~7.563 GB/s | PAM-4 + FEC | 引入 PAM-4 調變 |
| 7.0 | 2025 | 128.0 GT/s | ~15.125 GB/s | PAM-4 | 鎖定 AI/雲端/800GbE |
| 8.0 | 預計 2028 | 256.0 GT/s | ~30.25 GB/s | PAM-4（規劃） | x16 雙向達 1 TB/s |

> **GT/s** = Gigatransfers per second，為原始序列位元傳輸率。**吞吐量**為扣除編碼開銷後的有效資料承載量。

詳細版本里程碑：[^wiki-pcie][^phoronix-pcie7][^tomshardware-pcie7][^servethehome-pcie8]

- **PCIe 3.0**（2010）：從 8b/10b 改為 128b/130b 編碼，效率從 80% 提升至 ~98.46%。
- **PCIe 4.0**（2017）：AMD Zen 2（Ryzen 3000）率先支援消費級平台。
- **PCIe 5.0**（2019）：Intel 第 12 代 Core（Alder Lake）為首款消費級 x86 CPU 支援。
- **PCIe 6.0**（2022 規格發布，2025 硬體問世）：引入 PAM-4 調變及前向錯誤更正（FEC）。
- **PCIe 7.0**（2025 年 6 月 11 日最終規格發布）：128 GT/s，x16 雙向達 512 GB/s。
- **PCIe 8.0**（2025 年 8 月 5 日宣布開發）：目標 256 GT/s，x16 雙向達 1 TB/s，目前 Draft 0.5（2026 年 5 月）已發布。

## 架構與工作原理

### 點對點拓撲

與舊 PCI 的共享並列匯流排不同，PCIe 採用**點對點序列連結**：每個裝置直接連接至主機端（Root Complex），擁有專屬頻寬，無需與其他裝置競爭或進行匯流排仲裁。[^wiki-pcie]

### Lane（通道）系統

**Lane** 是 PCIe 的基本建構單元。每條 Lane 由兩組差動信號對（共 4 條信號線）組成：一組用於發送（TX），一組用於接收（RX），形成**全雙工**（full-duplex）位元組串流。[^wiki-pcie]

多條 Lane 可合併使用，常見配置：

| 配置 | Lane 數 | 接腳數 | 插槽長度 | 常見用途 |
|:---|:---:|:---:|:---:|:---|
| **x1** | 1 | 22 pins | 25 mm | Wi-Fi 卡、音效卡、GbE 網路卡 |
| **x4** | 4 | 64 pins | 39 mm | NVMe SSD、RAID 控制器 |
| **x8** | 8 | 98 pins | 56 mm | 高階網路卡、儲存 HBA |
| **x16** | 16 | 164 pins | 89 mm | 顯示卡、AI 加速器 |

### 資料條帶化

在多 Lane 連結中，資料以位元組為單位交錯分配到各 Lane（striping），接收端需進行去歪斜（deskew）重新排列。最大 Lane 間歪斜：2.5/5/8 GT/s 分別為 20/8/6 ns。[^wiki-pcie]

### 三層協定架構

PCI Express 為分層協定：[^wiki-pcie]

1. **交易層（Transaction Layer）**：處理封裝/解封裝、請求和回應分離（split transactions）、信用基流量控制（credit-based flow control）。
2. **資料鏈結層（Data Link Layer）**：封包排序、ACK/NAK 重播可靠傳遞、附加 LCRC 錯誤檢測碼。
3. **實體層（Physical Layer）**：分為邏輯子層與電氣子層，負責序列化/反序列化（SerDes）、時脈回復、編碼/解碼。

## 實體規格

### 供電能力

| 卡類型 | +12V 電流限制 | 最大功率 |
|:---|:---:|:---:|
| x1 卡 | 0.5 A | 10 W |
| x4/x8/x16 非顯示卡 | 2.1 A | 25 W |
| x16 顯示卡（初始化後） | 5.5 A | 75 W |

輔助供電連接器：
- **6-pin**：增加 75 W
- **8-pin**：增加 150 W
- **12VHPWR / 12V-2×6**（PCIe 5.0 引入）：最高 600 W

### 向後相容

- 實體向下相容：x1 卡可插入 x4/x8/x16 插槽（反之不行）。
- 電氣向下相容：舊版本裝置可在新版本插槽運作，雙方自動協商最高支援版本和 Lane 數。
- 軟體向下相容：PCIe 保留 PCI 的程式模型，作業系統無需修改即可支援基本功能。

## 與其他介面比較

| 特性 | 舊 PCI | PCI Express |
|:---|:---|:---|
| 拓撲 | 共享並列匯流排 | 點對點序列 |
| 傳輸模式 | 半雙工 | 全雙工 |
| 頻寬分配 | 所有裝置共享 | 每裝置專用 |
| 時脈 | 以最慢周邊為準 | 獨立運作 |
| 封裝 | 非封包式 | 封包化 |

| 特性 | AGP | PCI Express |
|:---|:---|:---|
| 推出年份 | 1997 | 2003 |
| 最高頻寬 | 2.133 GB/s (AGP 8×) | 242 GB/s (PCIe 6.0 x16) |
| 適用範圍 | 僅顯示卡 | 通用 |
| 傳輸模式 | 半雙工 | 全雙工 |
| 狀態 | 已淘汰 | 主流標準 |

## 應用場景

### 消費級

- **顯示卡**：最常見的 PCIe x16 應用。PCIe 在 2010 年左右完全取代 AGP 成為顯示卡標準介面。
- **NVMe SSD**：透過 M.2（最多 4 條 PCIe Lane）或直接 PCIe 插槽連接。
- **網路卡**：GbE 通常使用 x1；10GbE+ 使用 x4/x8。
- **無線網路卡**、音效卡、USB 控制器。

### 企業級 / 資料中心

- **AI/ML 加速器**：高頻寬 x16 連接 GPU/TPU。
- **高速網路**：100/200/400/800 GbE 網路卡。
- **叢集互連**：光纖 PCIe 用於高效能運算。
- **CXL（Compute Express Link）**：基於 PCIe 5.0 的新型互連標準，用於處理器、記憶體與加速器間的資料共享。

### 筆記型電腦／行動裝置

- **Thunderbolt 3/4/5**：透過 USB-C 承載 PCIe 協定，支援外接 GPU（eGPU）。
- **M.2**：NVMe SSD 和無線網卡的主要尺寸規格。
- **ExpressCard / OCuLink**：筆記型電腦外部擴充。
- **M-PCIe**：將 PCIe 引入智慧型手機/平板（如 iPhone NVMe 儲存）。

## PCI-SIG 組織

PCI-SIG 成立於 1992 年，最初為 Intel PCI 規範的相容性計畫，2000 年正式成為非營利組織，總部位於美國奧勒岡州比佛頓。截至 2024 年擁有 900+ 會員公司，董事會包含 AMD、ARM、Dell、IBM、Intel、NVIDIA、Qualcomm 等。[^wiki-pcisig]

主席為 **Al Yanes**（IBM 傑出工程師）。會員年費 $5,000。

## 未來發展

- **PCIe 7.0**（2025 年已發布）：128 GT/s，目標 AI、雲端運算、800Gb 乙太網路。預計 2027 年初步相容性測試，2028–2029 年實際裝置問世。[^phoronix-pcie7]
- **PCIe 8.0**（開發中，目標 2028）：256 GT/s，x16 雙向 1 TB/s。已發布 Draft 0.5（2026 年 5 月）。[^servethehome-pcie8]
- **PCIe Optical Interconnect**：光纖 PCIe 規範，可跨伺服器機櫃延伸 PCIe 連接距離。
- **CXL**：基於 PCIe 的新一代互連標準。

---

[^wiki-pcie]: Wikipedia. (2026). PCI Express. Retrieved 2026-10-01, from https://en.wikipedia.org/wiki/PCI_Express
[^wiki-pcisig]: Wikipedia. (2026). PCI-SIG. Retrieved 2026-10-01, from https://en.wikipedia.org/wiki/PCI-SIG
[^intel-press-2001]: Intel. (2001-08-03). Compaq, Dell, IBM, Intel, Microsoft Work with PCI-SIG on Arapahoe. Retrieved 2026-10-01, from https://www.intel.com/pressroom/archive/releases/2001/20010803corp.htm
[^eetimes-2002]: EE Times. (2002-04-18). Renamed 3GIO Interface Moves Toward Standardization. Retrieved 2026-10-01, from https://www.eetimes.com/renamed-3gio-interface-moves-toward-standardization/
[^phoronix-pcie7]: Phoronix. (2025-06-11). PCI Express 7.0 Specification Officially Released. Retrieved 2026-10-01, from https://www.phoronix.com/news/PCI-Express-7.0-PCIe-7.0
[^tomshardware-pcie7]: Tom's Hardware. (2025-06-11). PCIe 7.0 Spec Finalized with up to 512GB/s Speeds. Retrieved 2026-10-01, from https://www.tomshardware.com/tech-industry/pcie-7-0-spec-finalized-with-up-to-512gb-s-speeds-pci-sig-targets-1tb-s-for-8-0-as-exploration-phase-begins
[^servethehome-pcie8]: ServeTheHome. (2026-05). PCI-SIG PCIe 8.0 Specification Draft 0.5 Released. Retrieved 2026-10-01, from https://www.servethehome.com/pci-sig-pcie-8-0-specification-draft-0-5-released/