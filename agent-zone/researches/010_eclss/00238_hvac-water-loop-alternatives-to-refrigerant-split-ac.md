# 空調系統替代方案：從冷媒分離式到水循環模組化熱交換

## 摘要

現代居家與商業建築普遍使用冷媒分離式空調（split AC），但此系統依賴銅管傳送高壓冷媒，安裝需專業銅管燒焊，線路修改困難，且冷媒洩漏對環境有害。本報告探討更接近國際太空站（ISS）水循環熱交換架構、以及更具模組化與易維護性的替代方案，涵蓋冰水主機（chiller）+ 風機盤管（fan coil unit）、水力模組冰水主機、汲水式熱泵、輻射空調、以及可變冷媒流量（VRF）系統，並比較其與 ISS 主動熱控系統（ATCS）的設計哲學異同。

---

## 1. 問題背景：冷媒分離式空調的限制

冷媒分離式空調利用壓縮機驅動冷媒（如 R-410A、R-32）在室內機與室外機之間循環，透過蒸發與冷凝進行熱交換[^split-overview]。雖然成本低、安裝簡單，但有以下根本限制：

- **冷媒洩漏環境風險**：冷媒多為強效溫室氣體，部分舊型冷媒甚至破壞臭氧層[^refrigerant-impact]。
- **管線修改困難**：銅管連接需燒焊，線路一旦固定就難以調整或延伸。
- **專業技術門檻**：冷媒系統需具備特定證照的技師才能維修。
- **管長限制**：冷媒管有最長距離限制，超出會影響效率[^line-length]。
- **室外機數量問題**：大型建築若每間房裝一台分離式，外牆會被室外機佔滿。

---

## 2. ISS 熱控系統（ATCS）的設計哲學

ISS 的主動熱控系統採用兩層式設計，其核心理念值得參考[^iss-tcs]：

### 內部水循環（ITCS：Internal Thermal Control System）

- **工作流體**：純水（非冷媒）
- **雙迴路設計**：
  - 低溫迴路（LTL）：約 3–7°C，處理人員舒適冷卻與除濕
  - 中溫迴路（MTL）：約 16–18°C，冷卻電子設備
- **泵浦模組（PPA）**：設計為軌道更換單元（ORU），太空人可在艙內徒手更換
- **熱收集**：透過冷板（cold plate）和熱交換器連接各實驗架

### 外部氨循環（ETCS：External Thermal Control System）

- **工作流體**：無水氨（ammonia），因其優異的熱傳性能與低凝固點
- **輻射散熱**：氨經由泵浦送至 P1/S1 桁架上的散熱板，以紅外線輻射至太空
- **泵浦總成（PFCS）**：同樣設計為 ORU，太空人可透過艙外活動（EVA）更換

### ISS 模組化設計要點

| 特性 | ISS 設計 |
|------|----------|
| 熱傳介質 | 水（內部）→ 氨（外部） |
| 模組更換 | 所有關鍵組件均為 ORU，可單獨更換 |
| 快速接頭 | 使用專用快拆接頭，支援 EVA 操作 |
| 冗餘設計 | 泵浦與閥門總成三重冗餘 |
| 系統層級 | 內部收集 → 界面交換 → 外部排放，分層明確 |

---

## 3. 替代方案比較

### 3.1 冰水主機 + 風機盤管系統（Chiller + FCU）

**運作原理**：中央冰水主機產生冰水（約 3–7°C），經由水管分送至各樓層的風機盤管（FCU），FCU 內的風扇吹過冰水盤管產生冷風[^chilled-water]。

**優點**：
- ✅ **水為熱傳介質**——水管內流動的是水而非冷媒，洩漏無害
- ✅ **管線修改容易**——水管可使用標準螺紋或壓接接頭，不需燒焊
- ✅ **單點冷媒管理**——冷媒僅封閉於冰水主機機房內，末端 FCU 完全無冷媒
- ✅ **獨立區域控制**——每個 FCU 可獨立啟停與溫控
- ✅ **容易擴充**——只要水管主幹有預留，即可新增 FCU

**缺點**：
- ❌ 需機械機房放置冰水主機
- ❌ 需水質處理（防鏽、防垢、防菌）
- ❌ 初期建置成本高於分離式冷氣
- ❌ 需較複雜的配管設計與水力平衡

**ISS 相似度**：★★★★★ 最高。直接對應 ISS ITCS 的水循環設計。

---

### 3.2 模組化冰水主機（如 Trane Thermafit™ 系列）

**運作原理**：將冰水主機設計為可堆疊的小型模組（每模組 15–80 噸），每組最多並聯 12 個模組[^modular-chiller]。

**優點**：
- ✅ **真正的模組化**——容量不足時直接疊加同型模組
- ✅ **熱備援**——個別模組故障時，其餘模組持續運作
- ✅ **可通過標準門框**——體積小，適合既有建築改造
- ✅ **水冷或氣冷皆可**——水冷型可完全將冷媒封閉於機房
- ✅ **低 GWP 冷媒選項**——使用 R-454B、R-513A 等環保冷媒

**ISS 相似度**：★★★★☆ 極高。接近 ISS 的 ORU 更換哲學，模組可單獨抽換。

---

### 3.3 汲水式（水對水）熱泵系統（WSHP：Water-Source Heat Pump）

**運作原理**：建築內各區域裝設獨立的水源熱泵機組，所有機組連接至同一條共用水迴路（common water loop）[^wshp]。機組可選擇從迴路取熱（暖氣模式）或排熱至迴路（冷氣模式）。

**優點**：
- ✅ **同時冷暖**——一個區域冷氣排出的熱可被另一區域暖氣使用
- ✅ **無室外機**——所有機組安裝於室內天花板或機房
- ✅ **各機獨立**——任何一台故障不影響其他區域
- ✅ **更換容易**——個別機組可獨立更換或升級

**ISS 相似度**：★★★★☆ 高。類似 ISS 將熱從一個區域轉移到另一個區域的設計。

---

### 3.4 輻射空調系統（Radiant Cooling & Chilled Beam）

**運作原理**：冷水（約 16–19°C，需高於露點）流經天花板或地板內的管路，透過輻射與自然對流進行冷卻[^chilled-beam]。

**優點**：
- ✅ **極靜音**——無風扇噪音
- ✅ **節能風扇動力**——水攜熱能力約為空氣的 540 倍，可減少 60–80% 送風量[^water-efficiency]
- ✅ **低維護**——無活動部件（被動式），僅需每 5 年清理鰭片
- ✅ **整合設計**——可結合照明、消防灑水器於同一天花板模組

**缺點**：
- ❌ 在潮濕環境有結露風險
- ❌ 不適合高濕度空間（健身房、廚房、劇院）
- ❌ 暖氣效能有限，常需輔助暖氣
- ❌ 天花板高度可能受限

**ISS 相似度**：★★★☆☆ 中等。使用水循環但缺乏模組化熱交換概念。

---

### 3.5 可變冷媒流量系統（VRF / VRV）

**運作原理**：一台室外機連接多台室內機，透過變頻壓縮機精確調節冷媒流量。部分機型可同時提供冷暖（熱回收型）[^vrf]。

**優點**：
- ✅ 節能（比傳統分離式省 55%）[^vrf-efficiency]
- ✅ 同時冷暖
- ✅ 無大型風管
- ✅ 區域獨立控制

**缺點**：
- ❌ 仍然使用冷媒作為熱傳介質
- ❌ 仍需專業冷媒技師維護
- ❌ 管線仍有距離限制
- ❌ 仍非真正的水循環系統

**ISS 相似度**：★★☆☆☆ 低。仍屬冷媒系統，與水循環概念有本質差異。

---

## 4. 綜合比較表

| 系統 | 熱傳介質 | 模組化 | 修改彈性 | 維護難度 | ISS 相似度 | 適合場景 |
|------|----------|--------|----------|----------|-----------|---------|
| 冷媒分離式 | 冷媒 | 低 | 低（銅管需燒焊） | 需冷媒技師 | ★☆☆☆☆ | 住宅、小空間 |
| Chiller + FCU | 水 | 中高 | 高（水管接頭） | 低（FCU 無冷媒） | ★★★★★ | 中大型建築 |
| 模組化 Chiller | 水 | 極高 | 高（堆疊擴充） | 低（模組抽換） | ★★★★★ | 商業、醫療 |
| WSHP | 水+冷媒 | 中 | 高（獨立機組） | 中（各機有冷媒） | ★★★★☆ | 飯店、公寓 |
| 輻射空調 | 水 | 中 | 中 | 低（無活動件） | ★★★☆☆ | 辦公、學校 |
| VRF | 冷媒 | 中 | 中 | 需冷媒技師 | ★★☆☆☆ | 辦公、商業 |

---

## 5. 結論與建議

若要在現代建築中實現接近 ISS 水循環熱交換、且便於修改線路與維護的系統，**最推薦的方案是冰水主機（Chiller）+ 風機盤管（FCU）系統**，理由如下：

1. **水為熱傳介質**：完全對應 ISS ITCS 的水迴路設計，洩漏無害、容易處理
2. **模組化擴充**：只要預留水管主幹，後續加裝 FCU 如同新增電燈插座般簡單
3. **簡易維護**：末端 FCU 不含冷媒，一般水電師傅即可維修
4. **管線修改容易**：水管使用標準接頭（壓接、螺紋、快拆），不需燒焊

若預算允許且希望更接近 ISS 的 ORU 設計哲學，可選用 **模組化冰水主機**（如 Trane Thermafit 系列），能在不停機的情況下抽換故障模組，實現真正熱備援。

而對於已裝潢完成的既有建築改造，**水源熱泵（WSHP）** 是較輕量的替代方案，室內機可獨立更換且無需室外機，但仍需專業冷媒處理。

---

## 參考資料

[^split-overview]: Wikipedia. (n.d.). Air conditioning. Retrieved 2026-09-26, from https://en.wikipedia.org/wiki/Air_conditioning

[^refrigerant-impact]: United Nations Environment Programme. (n.d.). Ozone-depleting substances. Retrieved 2026-09-26, from https://www.unep.org/ozonaction/who-we-are/about-montreal-protocol

[^line-length]: Daikin. (n.d.). VRV System Design Manual. Retrieved 2026-09-26, from https://www.daikin.eu/

[^iss-tcs]: NASA. (n.d.). International Space Station Active Thermal Control System (ATCS). Retrieved 2026-09-26, from https://www.nasa.gov/international-space-station/

[^chilled-water]: Wikipedia. (n.d.). Chilled water. Retrieved 2026-09-26, from https://en.wikipedia.org/wiki/Chilled_water

[^modular-chiller]: Trane. (n.d.). Thermafit™ Modular Chillers. Retrieved 2026-09-26, from https://www.trane.com/commercial/north-america/us/en/products-systems/chillers/modular-chillers.html

[^wshp]: Wikipedia. (n.d.). Heat pump — Water-source heat pump. Retrieved 2026-09-26, from https://en.wikipedia.org/wiki/Heat_pump

[^chilled-beam]: Wikipedia. (n.d.). Chilled beam. Retrieved 2026-09-26, from https://en.wikipedia.org/wiki/Chilled_beam

[^water-efficiency]: Environmental Protection Agency. (n.d.). Chilled Beam Systems. Retrieved 2026-09-26, from https://www.epa.gov/

[^vrf]: Wikipedia. (n.d.). Variable refrigerant flow. Retrieved 2026-09-26, from https://en.wikipedia.org/wiki/Variable_refrigerant_flow

[^vrf-efficiency]: U.S. Department of Energy. (n.d.). Variable Refrigerant Flow Systems. Retrieved 2026-09-26, from https://www.energy.gov/