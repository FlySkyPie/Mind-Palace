# LRU（Line-Replaceable Unit）模型為何沒有走入消費市場？

## 什麼是 LRU？

**Line-Replaceable Unit（線路可更換單元，簡稱 LRU）** 是一種模組化硬體設計哲學，其核心概念是將系統中容易故障的關鍵元件設計成可在現場快速拆換的獨立單元，使維修人員無需特殊工具即可在最短時間內恢復系統運作[^wiki-lru]。

LRU 的關鍵特徵包括：
- **標準化的機械與電氣介面**（插拔式設計）
- **內建自我診斷能力**（BITE, Built-In Test Equipment）
- **免工具或僅需基礎工具的快速釋放機制**
- **密封、自足的外殼**，可在現場環境中操作
- **透過序號與物流控制編號（LCN）追蹤**

與 LRU 相近的概念還有 FRU（Field-Replaceable Unit，現場可更換單元），兩者本質相似，但 FRU 更廣泛用於計算機與商用電子設備[^wiki-fru]。

## LRU 在航太、軍事、電信領域的成功原因

LRU 模型在這些領域之所以極度成功，是因為**停機代價極其昂貴**，而**任務成功是不可妥協的**：

| 領域 | 停機代價 | LRU 的運作方式 |
|------|----------|----------------|
| **航空** | 飛機停飛（AOG）成本為每小時 **$10K–$150K+** | 航電箱、飛行計算機等採 ARINC 404/600 標準尺寸，技師數分鐘內更換完畢，故障單元送回 Depot 修復 |
| **軍事/國防** | 戰備狀態攸關任務成敗 | 野戰技師依 MIL-STD-1553 資料匯流排標準更換雷達、通訊等 LRU 模組 |
| **電信** | 網路中斷成本每分鐘 **$5K–$5M+** | 基地台射頻模組、功率放大器等採熱插拔 LRU 設計，遠端診斷後現場更換 |
| **太空/ISS** | 軌道維修極其昂貴且危險 | 國際太空站採用 ORU（Orbit-Replaceable Unit），太空人於太空漫步中直接更換 |

這些領域的共同特點是：**高可用性要求、安全關鍵、高故障代價**。LRU 所帶來的模組化前置成本，在此環境下很容易被節省的停機時間所合理化[^digitalidiom]。

## 消費市場的七重障礙

### 1. 成本與機構厚度限制

每個模組化介面都需要連接器、卡榫與定位導軌——這意味著數毫米的厚度增加。在主流智慧型手機厚度已壓縮到 7–8mm 的今日，這是致命傷。Google Project Ara 曾試圖將目標售價壓在 $50，但連接器的複雜度使成本完全失控[^techbasic][^slashgear]。

### 2. 效能 vs 整合的取捨

焊接（soldering）讓元件可以更靠近處理器、走線更短、訊號更乾淨。LPDDR5X 這類高速記憶體若採用插座設計，不僅會增加延遲，還可能產生訊號完整性問題。使用焊接元件時，設計師能更有效地利用主機板空間，從而打造更輕薄的裝置[^techedvocate][^makeuseof]。

### 3. 消費級可靠性的工程挑戰

連接器越多，故障點就越多。Project Ara 的工程團隊在開發可靠、耐久的電氣互連時遭遇巨大困難——這些連接器必須能承受日常摔落、口袋棉絮、濕氣與灰塵。Google 投入約 **$1.27 億美元** 進行研發，歷時三年，最終仍未量產任何一台零售裝置[^christensen][^glassnote]。

### 4. 消費者行為與心理

多數消費者不想要複雜性。StartupTalky 的案例分析指出：「一般消費者只想要手機能打電話、傳簡訊、用社群媒體。他們不想花時間思考需要什麼處理器、RAM 或儲存空間。」[^startuptalky] SlashGear 的分析也發現，除科技愛好者外，「很少有人對模組化感到興奮」——消費者渴望的是簡潔且「開箱即用」的裝置[^slashgear]。

### 5. 商業模式的根本衝突

製造商的商業模式與模組化設計存在內生矛盾。TechBasic 的文章直言：「大手機廠依賴每年換機。當使用者能只換相機模組而非整支手機時，整機銷售量將大幅下滑。」[^techbasic] 如果使用者可以只升級相機模組而非購買一支全新的 $1,000 手機，整個產業的營收模型就會崩潰。

### 6. 標準化缺失

不同於航空業的 ARINC 600 或軍用的 MIL-STD-1553，消費電子不存在跨品牌的模組化介面標準。每一次嘗試（Project Ara 的電永磁鐵、Moto Mods 的 Pogo Pin）都是封閉式專有方案。模組製造商不願為沒有使用者的平台投資，消費者不願購買沒有模組生態的平台——典型的雞蛋問題[^chrinsight]。

此外，Project Ara 的射頻認證問題極為嚴峻。Verizon 的內測顯示，模組化原型機的封包遺失率比一體機高出 **12–18%**，原因是模組與機殼之間的射頻路由阻抗不匹配。而 Ara 的多達 **230 萬種** 可能的配置組合，遠超認證機構的審查能力——正如 Glass & Note 所引用：「你無法認證無限。」[^glassnote]

### 7. 供應鏈複雜度

零售通路需要為每款手機備貨數十種獨立模組，而非單一 SKU。逆向物流方面，航空業可以追蹤、翻新並重新入庫每一個 LRU，但消費級規模下，退貨模組的經濟效益完全不可行。每個新模組版本還必須與所有舊版機身相容——測試組合呈幾何級數增長。

## 消費市場的嘗試與結果

| 產品 | 時間 | 方式 | 結果 |
|------|------|------|------|
| **Google Project Ara** | 2013–2016 | 全模組化手機，電永磁鐵卡榫，可換 CPU、相機、電池、螢幕 | **取消**。投入 $1.27 億研發，從未量產。工程挑戰（連接器可靠性、散熱、射頻干擾）過大，消費者調查顯示需求低迷[^christensen][^slashgear][^startuptalky] |
| **LG G5** | 2016 | 「模組朋友」——底部滑出式模組插槽，可換電池握把/相機/音訊模組 | **商業失敗**。僅推出 2–3 個模組，銷量慘淡，LG 自此放棄模組化路線[^techbasic] |
| **Motorola Moto Z / Moto Mods** | 2016–2019 | 磁性 Pogo Pin 吸附式附加模組（JBL 喇叭、投影機、電池組） | **小眾利基**。從未進入主流市場，終被 Motorola 停產。模組價格高昂（$80–$300）且體積龐大 |
| **Fairphone** | 2013–至今 | 工具免螺絲模組化設計，以**可修復性**為核心訴求（螢幕、電池、相機、USB 埠皆可更換） | **仍在運作但極小眾**。Fairphone 6（2026）年出貨約 40 萬支，對比 Samsung 的約 3 億支。SoC 仍焊接在主機板上無法更換[^coreiten] |
| **Framework Laptop** | 2021–至今 | 全模組化筆電，可更換主機板、擴充卡、電池、鍵盤、螢幕邊框 | **小眾成功**。價格仍偏高（$1,400+），以開發者與維權倡議者為主要客群 |

### Fairphone 的啟示

Fairphone 是消費電子中最接近 LRU 精神的持續嘗試。關鍵觀察：

- **核心晶片仍為焊接**：SoC（系統單晶片）與 RAM 焊接在主機板上無法更換。周邊模組（相機、電池、USB-C 埠、螢幕）則完全可升級[^coreiten]。
- **市場佔有率極小**：即使 2026 年出貨量成長約 42%，也僅是從微不足道成長為非常小眾。
- **價格溢價持續存在**：以 $649 的價格購買中階規格，比同等級封閉式手機貴 20–40%。
- **法規是主要推力**：歐盟 Right to Repair 指令正在創造法規壓力，可能逐步改變經濟模型[^chrinsight]。

## 經濟、設計、供應鏈的全面對比

| 面向 | 航太/軍事/電信 | 消費電子 |
|------|---------------|----------|
| **運轉時間的價值** | 極高（$10K–$5M/小時） | 極低（個人使用為 $0；工作用途 $0–100） |
| **可接受的模組化溢價** | 極高（可接受 10–50%+ 溢價） | 極低（約 0–5%；消費者對價格高度敏感） |
| **產品生命週期** | 15–30+ 年 | 2–4 年 |
| **機構外形壓力** | 中等（機櫃/航電艙空間） | 極端（輕薄優先於一切） |
| **操作者技能水準** | 受過訓練的專業技師 | 一般大眾（期望零學習成本） |
| **故障後果** | 生命安全、任務失敗 | 輕微不便，資料可備份 |
| **供應鏈** | 集中式、受控、高價值 | 分散式、大量出貨、低利潤 |
| **標準化程度** | 強（ARINC、MIL-STD 等） | 無（競爭性的封閉生態系） |
| **誘因對齊** | 營運方追求最低總持有成本 | 製造商追求最高換機頻率 |

## 核心結論

Christensen Institute 的分析提供了最具洞察力的框架：

> 「模組化是一股雙面力量：對供應商而言，它實現了分工合作……對買方而言，它實現了可自適、可調整的採用方式。這種推拉力量驅動了快速變革與成長加速。因此，要讓模組化運作，它必須同時為生產者提供協作利益，並為被整合方案過度服務的買方提供靈活性。」[^christensen]

在消費電子領域，**這兩個條件皆未滿足**：買方並沒有被整合方案過度服務（他們反而偏好整合），生產者也缺乏模組化的誘因（其商業模式依賴換機週期）。

### 總結

LRU 模型沒有走入消費市場，因為誘因結構完全相反。若要實現消費電子的 LRU 化，需要：

1. 消費者為更厚、更重、效能更差的裝置支付更高價格
2. 製造商主動侵蝕自己的換機營收
3. 業界在沒有中央權威的情況下達成標準化共識
4. 供應鏈從數百萬支封閉裝置徹底轉變為數百萬個獨立模組

目前的趨勢顯示，**法規管制**（Right to Repair 法案、生態設計指令）——而非市場力量——將是推動消費電子模組化的主要動力。Framework Laptop 直接引用了 Project Ara 的 PCIe 擴充埠設計文件，證明 LRU 概念在筆電上比手機更具可行性[^glassnote]。但整體而言，在消費電子領域全面實現 LRU 精神仍需極長時間。

---

[^wiki-lru]: Wikipedia. (n.d.). *Line-replaceable unit*. Retrieved 2026-09-25, from https://en.wikipedia.org/wiki/Line-replaceable_unit
[^wiki-fru]: Wikipedia. (n.d.). *Field-replaceable unit*. Retrieved 2026-09-25, from https://en.wikipedia.org/wiki/Field-replaceable_unit
[^digitalidiom]: Digital Idiom. (n.d.). *What is a Line Replaceable Unit?*. Retrieved 2026-09-25, from https://digitalidiom.co.uk/line-replaceable-unit/
[^techbasic]: The Tech Basic. (2025-06-14). *Why Modular Phones Failed to Change the Smartphone Game*. Retrieved 2026-09-25, from https://thetechbasic.com/2025/06/14/why-modular-phones-failed-to-change-the-smartphone-game/
[^slashgear]: SlashGear. (n.d.). *Why Google's Project Ara Modular Smartphone Was a Complete Failure*. Retrieved 2026-09-25, from https://www.slashgear.com/1179899/why-googles-project-ara-modular-smartphone-was-a-complete-failure/
[^christensen]: Christensen Institute. (n.d.). *The Demise of Google's Project Ara and Modularity in Computing*. Retrieved 2026-09-25, from https://www.christenseninstitute.org/blog/the-demise-of-googles-project-ara-and-modularity-in-computing/
[^startuptalky]: StartupTalky. (n.d.). *Why Was Google Project Ara Cancelled? Case Study*. Retrieved 2026-09-25, from https://startuptalky.com/google-project-ara-failure-case-study/
[^techedvocate]: The Tech Edvocate. (n.d.). *Why Some Laptop Parts Are Soldered Instead of Being Replaceable*. Retrieved 2026-09-25, from https://www.thetechedvocate.org/why-some-laptop-parts-are-soldered-instead-of-being-replaceable/
[^makeuseof]: MakeUseOf. (n.d.). *Why Laptop Parts Are Soldered Instead of Replaceable*. Retrieved 2026-09-25, from https://www.makeuseof.com/why-laptop-parts-soldered-instead-of-replaceable/
[^coreiten]: CoreITen. (n.d.). *Fairphone 6 Analysis: Can Ethical Modularity Finally Crack the Mainstream Market?*. Retrieved 2026-09-25, from https://www.coreiten.com/en/article/fairphone-6-analysis-can-ethical-modularity-finally-crack-the-mainstream-market
[^chrinsight]: Chronicle Insight. (n.d.). *Modular Phone Myths: Why Fairphone Failed and What's Next*. Retrieved 2026-09-25, from https://www.chronicleinsight.com/electronics/modular-phone-myths-why-fairphone-failed-and-whats-next/
[^glassnote]: Glass & Note. (n.d.). *Project Ara Was Never Meant to Replace the iPhone — And That's Exactly Why It Mattered*. Retrieved 2026-09-25, from https://glassandnote.com/wine/google-project-ara-modular-phones-replace-iphone