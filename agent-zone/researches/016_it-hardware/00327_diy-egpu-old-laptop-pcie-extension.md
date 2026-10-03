# DIY 舊筆記型電腦 PCIe 外接擴充（eGPU）完全指南

## 1. 何謂 eGPU？與舊筆電 PCIe 的關係

**eGPU**（External Graphics Processing Unit，外部圖形處理單元）是將桌上型顯示卡安置在筆電外部，透過某種外部介面連接的技術。核心概念是：筆電內部的 PCIe 匯流排可以透過其中一個擴充埠（ExpressCard、mini-PCIe、M.2/NGFF、Thunderbolt 或 OCuLink）**向外延伸**，用來驅動一個全尺寸的桌上型 GPU[^egpuio]。

對舊筆電而言，eGPU 是一種讓擁有良好 CPU 但缺乏強效 dGPU 的機器重新具備遊戲/渲染能力的方式——不需購買新筆電，而是從內部插槽拉出一條連接線到外部轉接板，接上 GPU 和電源，你的老 ThinkPad/Latitude 就能驅動現代遊戲[^makeuseof]。

> **關鍵概念**：PCIe 通道早已存在於筆電內部——它們原本用於 Wi-Fi 卡（mini-PCIe/M.2）、SSD（M.2 NVMe）或 ExpressCard 插槽。你只是在「劫持」那條現有的 PCIe 路徑，並將其延伸至機殼外部。

## 2. DIY 方法 — 從舊筆電延伸 PCIe

以下按照速度從最慢/最老到最快/最新排列：

### A. ExpressCard (EC) → PCIe x16
- **常見於**：~2005–2013 年的商務筆電（ThinkPad T/X 系列、Dell Latitude、HP EliteBook）
- **頻寬**：~2 Gbps（PCIe 1.1 x1）或 ~4 Gbps（PCIe 2.0 x1）— 非常慢
- **轉接器**：EXP GDC Beast（ExpressCard 版本）、BPlus PE4C-EC
- **實用性**：適合極舊的筆電；預期 GPU 效能約 50–70%。最適合低階 GPU（GTX 750 Ti、GTX 1050）[^egpuio_builds]

### B. mini-PCIe (mPCIe) → PCIe x16
- **常見於**：~2010–2016 年的筆電（Wi-Fi 卡插槽）
- **頻寬**：~4 Gbps（PCIe 2.0 x1）或 ~8 Gbps（PCIe 3.0 x1）
- **轉接器**：EXP GDC Beast（mPCIe 版本）、PE4L、PE4C
- **備註**：通常需移除 Wi-Fi 卡（或使用 mPCIe 分路器）。部分筆電有兩個 mPCIe 插槽（一個用於 Wi-Fi，一個用於 WWAN）
- **實用性**：比 ExpressCard 好，但仍受限於 x1 通道[^techinferno]

### C. M.2 NGFF (A/E key) → PCIe x16
- **常見於**：~2013–2018 年的筆電（WWAN 卡插槽，通常為 M.2 2242）
- **頻寬**：~8 Gbps（PCIe 3.0 x1）或某些筆電有 x2 通道（~16 Gbps）
- **轉接器**：EXP GDC Beast（NGFF 版本）、ADT-Link
- **備註**：「NGFF」是 M.2 的舊稱。A/E key 用於無線卡，通常限制為 x1 PCIe[^egpuio]

### D. M.2 NVMe (M-key) → PCIe x16
- **常見於**：~2015 年以後的筆電（SSD 插槽）
- **頻寬**：最高 **32 Gbps**（PCIe 3.0 x4）或 **64 Gbps**（PCIe 4.0 x4）— 這是甜蜜點
- **轉接器**：ADT-Link R43SG、ADT-Link K43SG、EXP GDC OCuP4v2、NFHK N-P114-A
- **DIY eGPU 之王**：使用主要 NVMe SSD 插槽。可從 SATA SSD 開機，或使用短 M.2 延長線搭配轉接器
- **實用性**：效能極佳——PCIe 3.0 x4 可達桌上型效能的 8–15% 以內，4.0 x4 差距更小[^xda]

### E. Thunderbolt 3/4/USB4 → PCIe x16（經由轉接板）
- **常見於**：~2017 年以後的高階筆電
- **頻寬**：~22 Gbps（TB3/TB4，考慮編碼開銷後）
- **轉接器**：EXP GDC TH3P4G2/G3、Wikingoo D3-650（DIY TB3 擴充塢）
- **備註**：商業產品（Razer Core X、Akitio Node）成本 $200–400 美元。DIY TB3 板約 $120–150 美元
- **缺點**：編碼/解碼後的 PCIe（非直接連接）引入延遲，頻寬開銷約 10–20%[^egpuio]

### F. OCuLink（光銅鏈接）→ PCIe x16
- **常見於**：~2022 年以後的迷你 PC、遊戲掌機（GPD Win Max 2、OneXPlayer）、Lenovo ThinkBook Gen 6+
- **頻寬**：最高 **64 Gbps**（PCIe 4.0 x4）。PCIe 5.0 OCuLink 轉接器已達 **128 Gbps**[^oculink]
- **轉接器**：OSMETA GK01、EXP GDC OCuP4v2、NFHK N-P114-A、ADT-Link K993G
- **新王者**：透過纜線直接延伸 PCIe，無需編碼/解碼。比 Thunderbolt 穩定得多，不會隨機斷線
- **缺點**：極少筆電有原生 OCuLink 連接埠。通常需使用 **M.2 NVMe → OCuLink** 內部轉接卡（小型 riser 卡），再透過纜線連接外部[^oculink]

## 3. 每種方法所需的硬體

### 對於 ExpressCard / mini-PCIe / NGFF M.2 A/E：

| 元件 | 項目 |
|------|------|
| **轉接器** | **EXP GDC Beast**（v8.5 或 v9.0）— 經典 DIY eGPU 擴充塢。附帶符合你插槽類型的纜線。約 $50–80 美元 |
| **替代方案** | **BPlus PE4C**（結構更穩固，較舊但可靠） |
| **GPU** | 任何桌上型 GPU。建議中階（GTX 1060/RTX 2060/3050/4060）以避免 PCIe 瓶頸 |
| **PSU** | 標準 ATX 電源供應器（如 500W EVGA/Corsair/Seasonic）或緊湊型 PicoPSU（150W，適合低功耗 GPU） |
| **外殼** | 開放式框架、紙箱、3D 列印外殼，或自製木/壓克力殼 |
| **纜線** | SATA 轉 6+2 pin PCIe（給 GPU），24-pin ATX 轉接板 |

### 對於 M.2 NVMe（M-key）— 直接或 OCuLink：

| 元件 | 項目 |
|------|------|
| **轉接器** | **ADT-Link R43SG**（固定纜線，PCIe 3.0 x4，約 $30–40 美元）或 **ADT-Link K43SG**（更長纜線） |
| **OCuLink 變體** | **ADT-Link K993G**（PCIe 5.0 x4 OCuLink，128 Gbps）、**NFHK N-P114-A**（Amazon，約 $40–60 美元）、**OSMETA GK01**（約 $60–100+ 美元，含外殼） |
| **DIY TB3 變體** | **EXP GDC TH3P4G3**（約 $120–150 美元，含 TB3 控制器板 + PSU 支架） |
| **GPU** | 任何桌上型 GPU。PCIe 4.0 GPU（RTX 3000+、AMD 5000+）在 OCuLink/M.2 NVMe 上表現最佳 |
| **PSU** | 標準 ATX 電源。M.2/OCuLink 板只需 24-pin ATX 供電 |
| **M.2 Riser** | 對 OCuLink 而言：一個小型 NVMe → OCuLink riser 卡（約 $15 美元）放在筆電 M.2 插槽內，OCuLink 纜線拉出。通常需要 3D 列印的底部蓋板修改 |
| **外殼** | 許多 OCuLink 板以裸板形式出貨——需自行建造外殼，或購買 OSMETA GK01 外殼 |

### 對於 Thunderbolt 3/4 DIY：

| 元件 | 項目 |
|------|------|
| **轉接器** | **EXP GDC TH3P4G3**（約 $140 美元，AliExpress）— 含 TB3 控制器、PCIe x16 插槽、85W PD 充電 |
| **替代方案** | **Wikingoo D3-650**（一體式 TB3 eGPU 擴充塢，約 $180 美元） |
| **GPU** | 查閱 eGPU.io 相容性列表——部分較新的 40 系列/70 系列 GPU 有 TB3 相容問題 |
| **PSU** | 隨板附帶的支架或外部變壓器 |
| **Thunderbolt 纜線** | 隨轉接器附帶 |

## 4. 限制與相容性

### 頻寬摘要（來自 eGPU.io 實測數據）[^egpuio_perf]：

| 介面 | 理論頻寬 | 實測 H2D Write（MiB/s） | 相對 GPU 效能* |
|------|---------|----------------------|--------------|
| PCIe 3.0 x16（桌上型內部） | 128 Gbps | ~12,000 | 100% |
| PCIe 4.0 x4（M.2/OCuLink） | 64 Gbps | ~6,371 | ~92% |
| PCIe 3.0 x4（M.2 NVMe/TB3） | 32 Gbps | ~2,939 | ~81% |
| TB3/TB4（編碼後） | 22 Gbps | ~2,250 | ~75%（含開銷） |
| TB2 / M.2 x2 | 16 Gbps | ~1,413 | ~65% |
| mPCIe2 / EC2 | 4 Gbps | ~369 | ~40–50% |
| EC1 / mPCIe1 | 2 Gbps | ~189 | ~30% |

*以 RTX 4090 在 TechPowerUp 的 PCIe 縮放測試為基準：x16 4.0 = 100%，x4 4.0 = 92%，x4 3.0 = 81%*

### BIOS/UEFI 限制（常見地雷）：
- **Secure Boot**：部分 UEFI 實作會阻擋外部 PCIe 裝置的列舉。可能需要關閉 Secure Boot 或切換至 Legacy/CSM 模式
- **Thunderbolt Security**：在 TB3 筆電上，需將 BIOS Thunderbolt 安全層級設為「No Security」或「Legacy」模式
- **iGPU vs dGPU**：許多舊筆電無法停用內建 GPU——必須在連接至 eGPU 的外部螢幕上使用，才能獲得最佳效能
- **Whitelist**：部分 Lenovo/HP 筆電的 BIOS 設有白名單，會封鎖內部插槽上的未識別 PCIe 裝置
- **Error 43（Nvidia）**：常見的 Windows 問題，當 Nvidia 驅動程式偵測到 GPU 透過外部/封裝的 PCIe 鏈接執行時會停用 GPU。可透過 `nvidia-error43-fixer.efi` 或 DDU 重新安裝（僅以 eGPU 作為唯一顯示）來修復
- **PCIe 鏈接速度/超時**：具備省電功能的筆電可能會將 PCIe 鏈接從 x4 動態降至 x2 或 x1。需停用 Link State Power Management、Intel SpeedStep 或 C-States[^reddit_egpu]

### 其他陷阱：
- **M.2 插槽已被 SSD 佔用**：如果唯一的 M.2 插槽是你的開機碟，你需要（a）改用 SATA SSD 釋放 M.2 給 eGPU，（b）使用分叉 riser（筆記型電腦上罕見），或（c）從 USB 開機碟開機
- **無法使用內建螢幕**：使用 ExpressCard/mPCIe/M.2 直接轉接器時，**無法**使用筆電的內建螢幕玩遊戲——PCIe 鏈接是單向的，且 iGPU 無法從外部 GPU 回讀。你**必須**使用連接至 eGPU 的外部螢幕
- **熱插拔**：OCuLink 不支援熱插拔。Thunderbolt 支援（但操作不當可能引發 BSOD）[^oculink]

## 5. 從零開始打造 DIY 外殼/擴充塢

**完全可以。** DIY eGPU 的本質就在於自製外殼。常見方法：

### A. 3D 列印外殼
- **TH3P4G3 eGPU case** — Printables.com 型號 #1348707[^printable_th3]
- **ATX eGPU Enclosure for OCuLink** — MakerWorld 型號 #1381001[^makerworld_atx] — 可容納 ATX PSU + 雙風扇 GPU
- **EGPU Enclosure（通用）** — MakerWorld 型號 #604908[^makerworld_egpu]
- **OCuLink eGPU Enclosure（MakerBeam）** — GitHub: Sinornithosaurus/Oculink-eGPU-Enclosure[^github_oculink] — 使用 MakerBeam 鋁合金框架 + 垂直 GPU 支架
- 更多模型可查閱 Printables.com 的 eGPU 標籤[^printable_egpu_tag]

### B. 從零自建（木/壓克力/金屬）
常見步驟：
1. 將轉接板（EXP GDC 或 ADT-Link）固定在平坦底座上（合板、壓克力板、鋁 L 型支架）
2. 使用垂直 GPU 支架安裝 GPU，或直接水平放置在支柱上
3. 安裝 ATX PSU（通常放在另一端以平衡重量）
4. 加入通風孔：120mm 風扇孔、網狀側板
5. 可選：加入把手、VESA 壁掛支架、RGB 燈光

### C. 開放式「測試平臺」框架
- 許多人直接用束帶將 GPU 固定在金屬框架上，或甚至直接敞開放在桌上。可使用舊主機板托盤或購買 $15 美元的開放式礦機框架

### D. 利用舊 PC 機殼
- 使用小型 ITX 機殼（如 Cooler Master NR200）——只需保持側板開啟，或為纜線開鑿一個孔洞

## 6. 知名專案與指南

| 資源 | 網址 | 重點 |
|------|------|------|
| **eGPU.io** | https://egpu.io | 第一大社群。數千份使用者提交的建置指南、採購指南、基準測試 |
| **eGPU.io Builds Database** | https://egpu.io/best-external-graphics-card-builds/ | 1000+ 份建置的可搜尋表格。可按機型、eGPU 連接埠類型、GPU、OS 過濾 |
| **Tech-Inferno Forums** | https://www.techinferno.com | eGPU 社群的歷史發源地。仍有 PE4C vs EXP GDC、mPCIe/ExpressCard 設定的優質討論 |
| **Reddit r/eGPU** | https://www.reddit.com/r/eGPU/ | 活躍社群。適合疑難排解及「我的筆電能否使用？」問題 |
| **MUO DIY eGPU 指南** | https://www.makeuseof.com/how-build-diy-egpu/ | TH3P4 擴充塢組裝的逐步實作指南 |
| **XDA Developers** | https://www.xda-developers.com/ways-to-turn-an-old-gpu-into-an-external-one/ | 三種方法的概覽：預製、DIY TB3、DIY OCuLink |
| **Geeky Gadgets OCuLink 擴充塢** | https://www.geeky-gadgets.com/building-and-affordable-oculink-gpu-dock/ | 低成本的 OCuLink 擴充塢建置指南（搭配 RTX 4060） |
| **GitHub: OCuLink eGPU Enclosure** | https://github.com/Sinornithosaurus/Oculink-eGPU-Enclosure | 基於 MakerBeam 的 OCuLink 外殼建置文件 |
| **GitHub: eGPU USB4 疑難排解** | https://github.com/dreamforse/eGPU-USB4-windows-fix-thunderbolt | Windows 11 的 USB4/TB eGPU 偵測/穩定性修復指南 |

## 7. OCuLink 深入探討

OCuLink 是近年 DIY eGPU 領域**最令人振奮的發展**。

### 什麼是 OCuLink？
OCuLink（SFF-8611/SFF-8612 連接器）是一種**直接的 PCIe 延長纜線**，最初來自伺服器儲存領域。它透過一條薄型、靈活的纜線承載未經編碼的原始 PCIe 通道[^oculink]。

### 為何重要：
- **無編碼開銷**（不同於 Thunderbolt 將 PCIe 包入類似 DisplayPort 的協定中，損失約 15–30% 頻寬）
- **可靠地達到 64 Gbps**（PCIe 4.0 x4），ADT-Link K993G 現已達到 **128 Gbps**（PCIe 5.0 x4）
- **接近桌上型效能**——OCuLink 使用者報告 64 Gbps 實測 H2D Write（~6,371 MiB/s），相較之下 TB3 僅約 ~2,250 MiB/s
- **無隨機卡頓/斷線**——困擾 Thunderbolt eGPU 設定的問題不再出現
- **原生支援正逐步出現**：Lenovo ThinkBook 14 G6+ 配備原生 TGX（OCuLink）連接埠。GPD、OneXPlayer、MinisForum 正在為裝置加入 OCuLink 連接埠[^egpuio]

### 如何在沒有原生 OCuLink 連接埠的筆電上使用：
1. 打開筆電，找到 M.2 NVMe SSD 插槽
2. 將 SSD 更換為**短 M.2 2230**，或將開機移至 SATA
3. 在 M.2 插槽中安裝 **M.2 NVMe → OCuLink riser 卡**（如 NFHK SF-014、ADT-Link OCuLink riser）
4. 透過機殼縫隙將 OCuLink 纜線拉出（通常需移除底部蓋板或 3D 列印修改後的底部蓋板）
5. 另一端連接至 OCuLink → PCIe x16 轉接板（如 OSMETA GK01、EXP GDC OCuP4v2）
6. 如往常般加入 GPU 和 PSU

### 耐久性備註（來自 eGPU.io nando4）：
OCuLink SFF 連接器的插拔次數評級為**最低 50 次**（並非某些人所稱的 10,000 次）。請將纜線視為「半永久性」——不要每天斷開[^oculink]。

## 8. 成功率與實用效能預期

### 按方法區分：

| 方法 | 成功率 | 難度 | 成本（轉接器+外殼） | 相對 GPU 效能 |
|------|-------|------|------------------|-------------|
| **M.2 NVMe（PCIe 3.0 x4）** | ★★★★☆ | 中 | ~$30–50 美元 | ~81% |
| **M.2 NVMe → OCuLink（PCIe 4.0 x4）** | ★★★★☆ | 中高 | ~$50–100 美元（轉接器+riser） | ~92% |
| **Thunderbolt 3/4 DIY（TH3P4G3）** | ★★★☆☆ | 低 | ~$120–150 美元 | ~75% |
| **ExpressCard / mPCIe / NGFF** | ★★★☆☆ | 中 | ~$50–80 美元 | ~30–50% |
| **Thunderbolt 3 商業外殼** | ★★★★★ | 極低 | ~$200–400 美元 | ~75% |

### 成功實用技巧：

1. **使用外部螢幕**——務必如此。內部螢幕迴圈（Optimus/Primus）會額外損失 10–20% 的效能
2. **中階 GPU 甜蜜點**——RTX 3060/4060 或 RX 6600/7600。高階卡（RTX 4090）在 PCIe x4 下受嚴重瓶頸，尤其在 1080p 解析度
3. **PCIe 4.0 至關重要**——如果你的筆電有 PCIe 4.0 M.2 插槽（Intel 11 代+ 或 AMD Ryzen 4000+），務必使用 PCIe 4.0 轉接器和 GPU
4. **Nvidia vs AMD**——Nvidia 卡在處理縮減的 PCIe 通道時略優於 AMD。支援 PCIe 4.0 的 AMD 卡（RX 5000+）在 OCuLink 上表現良好
5. **先查閱 eGPU.io 建置資料庫**——在購買任何東西之前，先搜尋你的確切筆電型號。數百筆建置已有紀錄
6. **BIOS 準備檢查清單**：
   - 關閉 Secure Boot
   - 關閉 Fast Boot
   - 將 Thunderbolt 安全層級設為「No Security」（如使用 TB）
   - 若可行則關閉 CSM/Legacy（僅使用 UEFI）
   - 若 BIOS 允許則關閉 iGPU
   - 將 PCIe 電源管理設為「Performance」

### 常見失敗點：
- 筆電只有一個 M.2 插槽（開機碟衝突）
- BIOS 白名單（Lenovo、較舊的 HP）
- Nvidia Error 43（可使用 error-43-fixer EFI 指令稿修復）
- 由於省電功能導致 PCIe 鏈接降至 x1/x2（停用 C-States/Link State Power Management）
- PSU 瓦數不足（任何具 6-pin 的 GPU 都需要 500W+）

## 總結建議

**對於舊筆電（2012–2018）：** 如果它有可拆卸的 Wi-Fi 卡插槽（mPCIe）或 WWAN 插槽（M.2 A/E），使用 **EXP GDC Beast** 搭配 GTX 1050 Ti / RX 570 / GTX 1650。你將獲得約 50–60% 的桌上型效能。總成本：約 $80–120 美元。

**對於較新的筆電（2018–2024）：** 如果它有空的 M.2 NVMe 插槽，使用 **ADT-Link R43SG** 或 **OCuLink 轉接器**搭配 RTX 3060/4060。你將獲得約 81–92% 的桌上型效能。總成本：約 $30–150 美元（轉接器），另加 GPU 和 PSU。

**對於配備 Thunderbolt 的筆電：** 使用 **EXP GDC TH3P4G3** 或 **TH3P4G2** DIY 擴充塢。效能約為桌上型的 75%。總成本：約 $120–150 美元（板）。

---

[^egpuio]: eGPU.io. (n.d.). eGPU — External Graphics Processing Unit. Retrieved 2026-10-01, from https://egpu.io
[^egpuio_builds]: eGPU.io. (n.d.). Best External Graphics Card Builds. Retrieved 2026-10-01, from https://egpu.io/best-external-graphics-card-builds/
[^egpuio_perf]: eGPU.io. (n.d.). Builds — Bandwidth/Acronym Reference Table. Retrieved 2026-10-01, from https://egpu.io/builds#perf
[^makeuseof]: MakeUseOf. (n.d.). How to Build a DIY eGPU. Retrieved 2026-10-01, from https://www.makeuseof.com/how-build-diy-egpu/
[^techinferno]: Tech-Inferno Forums. (n.d.). BPlus PE4C or EXP GDC Beast NGFF/m.2. Retrieved 2026-10-01, from https://www.techinferno.com/index.php?/topic/10448-bplus-pe4c-or-exp-gdc-beast-ngffm2/
[^xda]: XDA Developers. (n.d.). Ways to Turn an Old GPU Into an External One. Retrieved 2026-10-01, from https://www.xda-developers.com/ways-to-turn-an-old-gpu-into-an-external-one/
[^oculink]: eGPU.io Forums. (n.d.). OCuLink — What It Is, Why You Need It, and Where to Get It. Retrieved 2026-10-01, from https://egpu.io/forums/custom-egpu-chassis/oculink-what-it-is-why-you-need-it-and-where-to-get-it/
[^reddit_egpu]: Reddit r/eGPU. (n.d.). Retrieved 2026-10-01, from https://www.reddit.com/r/eGPU/
[^geeky_gadgets]: Geeky Gadgets. (n.d.). Building an Affordable OCuLink GPU Dock. Retrieved 2026-10-01, from https://www.geeky-gadgets.com/building-and-affordable-oculink-gpu-dock/
[^github_oculink]: Sinornithosaurus. (n.d.). OCuLink eGPU Enclosure. GitHub. Retrieved 2026-10-01, from https://github.com/Sinornithosaurus/Oculink-eGPU-Enclosure
[^printable_th3]: Printables.com. (n.d.). TH3P4G3 eGPU Case Enclosure (Model #1348707). Retrieved 2026-10-01, from https://www.printables.com/model/1348707-th3p4g3-egpu-case-enclosure
[^makerworld_atx]: MakerWorld. (n.d.). ATX eGPU Enclosure for DIY OCuLink eGPU (Model #1381001). Retrieved 2026-10-01, from https://makerworld.com/en/models/1381001-atx-egpu-enclosure-for-diy-oculink-egpu-s
[^makerworld_egpu]: MakerWorld. (n.d.). EGPU Enclosure (Model #604908). Retrieved 2026-10-01, from https://makerworld.com/en/models/604908-egpu-enclosure
[^printable_egpu_tag]: Printables.com. (n.d.). eGPU Tag. Retrieved 2026-10-01, from https://www.printables.com/tag/egpu