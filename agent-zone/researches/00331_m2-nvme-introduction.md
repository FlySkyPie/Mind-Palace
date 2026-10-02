# M.2 NVMe SSD 介紹

> 本報告提供 M.2 NVMe SSD 的完整介紹，包含技術規格、效能指標、選購建議與注意事項。

## 什麼是 M.2 NVMe

**M.2** 是一種**外型規格**（物理形狀與連接介面），而非通訊協定。它是一種小型、類似口香棒狀的模組，直接插入主機板的 M.2 插槽。**NVMe**（Non-Volatile Memory Express）則是專為快閃記憶體設計的**通訊協定**。

相比之下，**SATA SSD** 使用較舊的 SATA III 介面（上限約 600 MB/s）與 AHCI 協定——後者原本是為傳統硬碟設計的。SATA SSD 可以採用 2.5 吋或 M.2 外型規格（即 M.2 SATA）。

**核心差異**：NVMe 硬碟透過 PCI Express 通道（與顯示卡相同的高頻寬路徑）直接與 CPU 通訊，完全繞過 SATA 控制器，從而消除 SATA 的瓶頸。[^pcworld-sandisk]

| 特徵 | SATA SSD | NVMe SSD |
|---|---|---|
| 介面 | SATA III | PCIe |
| 協定 | AHCI | NVMe |
| 最大循序速度 | ~550–600 MB/s | 最高 14,000+ MB/s (Gen5) |
| 常見外型 | 2.5 吋、M.2 SATA | M.2 |
| 延遲 | 較高 | 更低 (< 0.1ms) |
| 每 GB 價格 | 較低 | 較高 |

**重要區別**：並非所有 M.2 SSD 都是 NVMe SSD。M.2 SATA 硬碟外觀與 NVMe 完全相同，但速度被限制在 SATA 等級。購買時請務必確認介面類型。[^diskgenius]

## NVMe 協定 vs AHCI

### AHCI（Advanced Host Controller Interface）

AHCI 於 2000 年代初期為機械硬碟設計，主要限制包括：

- **1 個命令佇列**，每佇列僅 **32 個命令**
- 高驅動程式負擔與高延遲
- 作業系統序列化處理命令，形成快閃記憶體的瓶頸

### NVMe（Non-Volatile Memory Express）

NVMe 專為快閃儲存設計：

- 支援最高 **65,535 個命令佇列**，每佇列最高 **65,536 個命令**
- 極低延遲——命令送達與回應回傳速度大幅加快
- 消除 AHCI 的協定開銷
- 充分利用 PCIe 通道直接與 CPU 通訊
- 原生支援平行 I/O 操作

**比喻**：AHCI 像是透過單一收費站處理資料（一條佇列、一次 32 輛車）；NVMe 則是 65,535 個收費站同時處理。[^minitool]

## M.2 外型規格（2242、2260、2280、22110）

### 尺寸命名規則

M.2 SSD 尺寸以 **WWLL** 格式標示：前兩位 = 寬度（mm），後幾位 = 長度（mm）。所有標準 M.2 SSD 均為 **22mm 寬**。[^wikipedia-m2]

| 尺寸代碼 | 尺寸（mm） | 常見用途 |
|---|---|---|
| **2230** | 22 × 30 | 掌機遊戲 PC（Steam Deck、ROG Ally）、輕薄筆記型電腦 |
| **2242** | 22 × 42 | 小型筆記型電腦、迷你 PC |
| **2260** | 22 × 60 | 部分筆記型電腦、舊款 ultrabook |
| **2280** | 22 × 80 | **最常見**——桌上型電腦與主流筆記型電腦標準 |
| **22110** | 22 × 110 | 企業級、高容量硬碟、部分高階主機板 |

### 金鑰（防呆插槽）

M.2 硬碟使用連接端的物理凹槽來防止插入不相容的插槽：[^atpinc]

| 金鑰類型 | 凹槽位置 | 支援介面 | PCIe 通道數 | 典型用途 |
|---|---|---|---|---|
| **B-key** | 第 11 針後 | PCIe x2、SATA | 最多 2 | 特殊/舊款裝置 |
| **M-key** | 第 59 針後 | PCIe x4 | 最多 4 | **高效能 NVMe SSD**——最常見 |
| **B+M-key** | 兩個凹槽 | PCIe x2、SATA | 最多 2 | 最大機械相容性 |

**重要**：金鑰相符並不保證 SSD 能使用——插槽的電路設計也須支援對應介面。請務必查閱主機板說明書。

## PCIe 世代（Gen3、Gen4、Gen5）

每代 PCIe 約**加倍**前一代的頻寬：[^kingston-evezone]

| 世代 | 每條通道原始訊號 | x4 頻寬（NVMe） | 典型實際循序讀取 |
|---|---|---|---|
| **PCIe 3.0** | 8 GT/s | ~4 GB/s | ~3,500 MB/s |
| **PCIe 4.0** | 16 GT/s | ~8 GB/s | ~7,000 MB/s |
| **PCIe 5.0** | 32 GT/s | ~16 GB/s | ~14,000+ MB/s |

消費者 NVMe 硬碟使用 **4 條 PCIe 通道（x4）**，上述數字反映 x4 配置。

**向後相容**：Gen4 硬碟插入 Gen3 插槽時以 Gen3 速度運作；Gen5 硬碟插入 Gen4 插槽時以 Gen4 速度運作。

**2026 年建議**：PCIe 4.0 提供最佳價格/效能平衡；Gen5 適用於長時間創意工作負載（4K/8K 影片、AI、3D 渲染）；Gen3 仍適合預算組裝與日常使用。

## NAND 快閃技術（TLC、QLC、3D NAND）

### 單元類型比較

| NAND 類型 | 每單元位元 | 耐用度（P/E 循環） | 1TB 硬碟典型 TBW | 成本 | 典型用途 |
|---|---|---|---|---|---|
| **SLC** | 1 | 50,000–100,000 | 極高 | 最高 | 企業、工業 |
| **MLC** | 2 | 3,000–10,000 | ~600–1,200 TBW | 中等 | 目前消費者市場少見 |
| **TLC** | 3 | 1,500–3,000 | ~300–1,200 TBW | 實惠 | **2026 年消費者主流** |
| **QLC** | 4 | 1,000–1,500 | ~100–600 TBW | 最低 | 高容量/預算儲存 |

### 3D NAND

3D NAND 將儲存單元堆疊在**垂直層**中，相較於舊款平面 NAND 提供更高密度。2026 年 SK Hynix 達到 **321 層**——一項重要里程碑。優點包括：每晶圓更高密度、更低每位元成本、更高效能、更小體積。[^newegg]

**TLC** 仍是 2026 年消費者 NVMe 的主流選擇（Samsung 990 PRO、WD Black SN850X 等），而 **QLC** 在 2026 年有重大突破，效能較前代提升 56%，使 4TB+ 消費者硬碟更加可行。

## NVMe 的使用場景

### 建議使用 NVMe：
- **遊戲玩家**：更快的遊戲載入、大型開放世界遊戲流暢素材串流、支援 DirectStorage
- **內容創作者**：4K/8K 影片剪輯、大量 RAW 照片、3D 渲染、AI/ML 工作負載——此為 NVMe 速度最有感的使用場景
- **高效能使用者**：執行 VM、編譯程式碼、管理大型資料庫
- **PS5 使用者**：需 PCIe Gen4 NVMe 才能擴充儲存
- **新組裝電腦**：主機板原生支援 NVMe，價格已與 SATA 接近

### 建議使用 SATA：
- **老電腦升級**：僅支援 SATA 的 5 年以上舊筆記型電腦
- **次要儲存**：以更低價格獲得大容量儲存（媒體庫、遊戲檔案）
- **辦公室/基本工作**：瀏覽網頁、電子郵件、試算表——與 NVMe 差異不明顯

### 混合策略（許多人推薦）：
使用 500GB–1TB NVMe 安裝作業系統與常用應用程式（熱資料），搭配較大、較便宜的 SATA SSD 作為大量儲存（冷資料）。[^pcworld]

## 重要規格解讀

### 循序讀寫（Sequential Read/Write）
測量處理**大型連續資料區塊**的速度。

| 硬碟類型 | 循序讀取 | 循序寫入 |
|---|---|---|
| SATA SSD | ~550 MB/s | ~500 MB/s |
| NVMe Gen3 | ~3,500 MB/s | ~3,000 MB/s |
| NVMe Gen4 | ~7,000 MB/s | ~5,000–6,500 MB/s |
| NVMe Gen5 | ~14,000 MB/s | ~10,000+ MB/s |

### IOPS（每秒輸入/輸出操作次數）
測量處理**大量小型隨機檔案**的速度——這決定日常使用反應速度的重要性質。

| 硬碟類型 | 隨機 4K 讀取 IOPS | 隨機 4K 寫入 IOPS |
|---|---|---|
| SATA SSD | ~80,000–100,000 | ~70,000–90,000 |
| NVMe Gen3 | ~200,000–400,000 | ~150,000–300,000 |
| NVMe Gen4 | ~500,000–1,000,000 | ~400,000–800,000 |
| NVMe Gen5 | ~1,000,000–1,500,000+ | ~800,000+ |

### 延遲（Latency）

| 硬碟類型 | 典型延遲 |
|---|---|
| HDD | ~5–10 ms |
| SATA SSD | ~0.5–1 ms |
| NVMe SSD | **< 0.1 ms（通常 0.02–0.05 ms）** |

### 其他重要規格
- **TBW**（Terabytes Written）：寫入耐用量。1TB TLC 硬碟典型值 300–1,200 TBW
- **隨機 4K 效能**：比循序速度更能反映日常使用體驗[^evezone]

## NVMe 的缺點

### 1. 發熱問題
NVMe 硬碟（尤其 Gen4 與 Gen5）產生大量熱能。持續寫入可將控制器溫度推至 80°C 以上，觸發**效能降速保護**。Gen5 硬碟**必須**搭配散熱片。[^laptopjudge]

### 2. 價格溢價
1TB 容量差距已縮小至 $20–40 美元，但 4TB+ 高容量仍有顯著 NVMe 溢價。

### 3. 相容性限制
- 需要支援 PCIe/NVMe 的 M.2 插槽
- 舊主機板（2015 年前）可能缺乏 NVMe 支援
- 金鑰（B vs M）與實體尺寸需相符
- 部分主機板使用 M.2 插槽時會停用 SATA 連接埠

### 4. 日常使用邊際效益遞減
對於網頁瀏覽、電子郵件、辦公室工作，Gen3 與 Gen5 NVMe 的使用體驗幾乎無差異。

### 5. M.2 不支援熱插拔
與 SATA 或 U.2 不同，M.2 模組無法在系統運作時插入或移除。

## 無 DRAM vs 有 DRAM 的 NVMe 硬碟

### DRAM 型 SSD
內建專屬 DRAM 晶片，儲存 **FTL（Flash Translation Layer）** 映射表。
- **優點**：最快、最一致的效能；最佳隨機 I/O；更低寫入放大；更高耐用度
- **範例**：Samsung 990 PRO、WD Black SN850X
- **隨機 4K IOPS**：400,000+（即使 90% 容量仍保持一致）

### HMB（Host Memory Buffer）SSD
透過 NVMe HMB 功能**借用 32–64 MB 系統記憶體**來快取 FTL 映射表。
- **優點**：比 DRAM 型便宜；實際使用效能優異
- **範例**：WD Blue SN580、Kingston NV2、Crucial P3
- **隨機 4K IOPS**：150,000–300,000

### 無 DRAM SSD（無快取）
FTL 映射表完全儲存在低速 NAND 上。
- **缺點**：**顯著較慢**；隨機 4K IOPS 降至 40,000–80,000；超過 70% 容量時效能大幅下降
- **建議**：**避免作為系統碟**——最初的低價不值得長期效能損失[^computercompatibility]

## 散熱與降速機制

### 溫度範圍

| 狀態 | 溫度 | 意義 |
|---|---|---|
| 閒置 | 30–50°C | 正常 |
| 持續負載 | 50–70°C | 合理目標範圍 |
| 警告 | 70–80°C | 可能影響一致性，建議改善通風 |
| 降速可能 | 80–85°C | **多數硬碟在此範圍啟動降速保護** |
| 危險 | 85°C 以上 | 降速幅度顯著，應停止並降溫 |

### 降速運作方式
NVMe 硬碟在控制器上設置溫度感測器。當溫度接近降速門檻（通常 80–85°C）：

1. 硬碟動態降低效能（降低 IOPS/throughput）讓散熱進行
2. 此現象表現為**持續寫入曲線先高後急降**——大量寫入 10–60 秒後效能驟降
3. 控制器通常比 NAND 高 10–15°C

### 散熱解決方案

| 方案 | 適用場景 |
|---|---|
| **被動散熱片** | Gen4 與 Gen5 桌上型；鋁合金設計效果最佳 |
| **銅箔** | 空間受限環境（筆記型電腦） |
| **主動散熱（風扇）** | Gen5 持續工作負載、密封環境 |
| **主機板 M.2 散熱屏蔽** | 許多主機板內建——使用前記得移除保護塑膠膜 |

### 實務建議
- Gen3：多數不需散熱片
- Gen4：**建議使用**（尤其持續寫入時）
- Gen5：**必須使用**——無適當散熱時數秒內即降速
- 在筆記型電腦中，間隙僅 2–3 mm，薄銅箔或低高度散熱墊為唯一選項

## 結論

M.2 NVMe SSD 代表了消費者儲存技術的重大進展。選擇時應考量預算、使用場景與主機板相容性。對於 2026 年的多數使用者，PCIe 4.0 NVMe SSD 搭配 TLC NAND 提供最佳價格與效能的平衡點。

---

[^pcworld-sandwich]: PCWorld. (2024). NVMe vs. M.2 vs. SATA SSD: What's the difference? Retrieved 2026-10-01, from https://www.pcworld.com/article/558324/nvme-vs-m-2-vs-sata-ssd-whats-the-difference.html
[^diskgenius]: DiskGenius. (2024). NVMe SSD vs SATA SSD: What are the differences? Retrieved 2026-10-01, from https://www.diskgenius.com/resource/nvme-ssd-vs-sata-ssd.html
[^minitool]: MiniTool. (2024). AHCI vs NVMe: What's the difference. Retrieved 2026-10-01, from https://www.minitool.com/backup-tips/ahci-vs-nvme.html
[^wikipedia-m2]: Wikipedia. (2024). M.2. Retrieved 2026-10-01, from https://en.wikipedia.org/wiki/M.2
[^atpinc]: ATP Inc. (2024). What is M.2 M-B-BM Key Socket 3. Retrieved 2026-10-01, from https://www.atpinc.com/blog/what-is-m.2-M-B-BM-key-socket-3
[^kingston-evezone]: Evezone. (2024). Every NVMe SSD specification explained – Read speed, write speed, latency, and PCIe. Retrieved 2026-10-01, from https://evezone.evetech.co.za/deep-dives/every-nvme-ssd-specification-explained-read-speed-write-speed-latency-and-pcie/
[^newegg]: Newegg Insider. (2026). SSD lifespan decoded: Understanding NAND types and write endurance. Retrieved 2026-10-01, from https://www.newegg.com/insider/ssd-lifespan-decoded-understanding-nand-types-and-write-endurance-in-2026/
[^pcworld]: PCWorld. (2024). What type of SSD should you buy? Retrieved 2026-10-01, from https://www.pcworld.com/article/394015/what-type-of-ssd-should-you-buy.html
[^laptopjudge]: LaptopJudge. (2024). NVMe SSD safe temperature range. Retrieved 2026-10-01, from https://laptopjudge.com/nvme-ssd-safe-temperature-range/
[^computercompatibility]: ComputerCompatibility. (2024). SSD DRAM cache HMB explained. Retrieved 2026-10-01, from https://computercompatibility.com/ssd-dram-cache-hmb-explained/