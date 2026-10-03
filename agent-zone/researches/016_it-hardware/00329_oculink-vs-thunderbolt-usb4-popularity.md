# OCuLink 為何沒有 Thunderbolt/USB4 普及？

## 摘要

OCuLink 是 PCI-SIG 制定的外部 PCI Express 連接標準，技術上能以極低延遲直接傳輸原始 PCIe 訊號，頻寬和效能均優於 Thunderbolt 4，甚至在某些實測中勝過 Thunderbolt 5。然而它在消費市場的普及率遠低於 Thunderbolt 與 USB4。本報告從技術特性、生態系統、使用體驗、歷史發展等面向分析箇中原因。

## 1. 背景：什麼是 OCuLink？

### 1.1 技術原理

OCuLink（Optical Copper Link）是 PCI-SIG 於 2016 年標準化的外部 PCI Express 連接技術[^pcisig]。其核心設計理念是**不做任何協定轉換**——直接將主機端的 PCIe Root Complex 透過纜線延伸至外部裝置，訊號在電氣層面與內部 PCIe 通道完全相同[^csdn]。

這與 Thunderbolt/USB4 形成鮮明對比：後兩者將 PCIe 流量封裝在通道協定堆疊中（PCIe → Thunderbolt/USB4 通道 → USB-C 實體層 → 解碼），引入額外延遲與損耗。

### 1.2 硬體規格

| 組態 | 原始傳輸率 | 編碼 | 有效頻寬（單向） |
|---|---|---|---|
| PCIe 3.0 ×4（OCuLink 4i） | 8 GT/s × 4 = 32 GT/s | 128b/130b | ~31.5 Gbps（~3.94 GB/s） |
| PCIe 4.0 ×4（OCuLink 4i） | 16 GT/s × 4 = 64 GT/s | 128b/130b | **~63 Gbps（~7.88 GB/s）** |
| PCIe 4.0 ×8（OCuLink 8i） | 16 GT/s × 8 = 128 GT/s | 128b/130b | **~126 Gbps（~15.75 GB/s）** |

OCuLink 使用兩種實體連接器：**SFF-8611**（垂直插拔，多用於伺服器背板）與 **SFF-8612**（水平插拔附鎖定扣，用於消費級外接裝置）[^evezone]。

## 2. 技術比較：OCuLink vs Thunderbolt vs USB4

### 2.1 頻寬與效能

| 項目 | OCuLink（4i） | Thunderbolt 4 | Thunderbolt 5 | USB4 v1 | USB4 v2 |
|---|---|---|---|---|---|
| 原始雙向頻寬 | 64 Gbps（PCIe 4.0 ×4） | 40 Gbps | 80 Gbps（120 Gbps boost） | 40 Gbps | 80 Gbps |
| 有效 PCIe 吞吐量 | **~7.88 GB/s** | ~3.94 GB/s | ~7.88 GB/s | ~2.86–3.94 GB/s | ~7.88 GB/s |
| 與內接 ×16 插槽效能差距 | **5–10% 損失** | 15–30% 損失 | 5–14% 損失 | 10–20% 損失 | 5–14% 損失 |

獨立實測數據（ONEXPLAYER X1、AtomMan X7 Ti 手持裝置搭配同一外接 GPU）[^xda]：

| 測試項目 | OCuLink（64 Gbps） | USB4（40 Gbps） |
|---|---|---|
| 3DMark PCIe 頻寬（AtomMan X7 Ti） | **6.70 GB/s** | 2.42 GB/s |
| Forza Horizon 5（99th %ile FPS） | **~149 FPS** | ~60 FPS |
| Elden Ring（99th %ile FPS） | **~69 FPS** | ~18 FPS |

### 2.2 延遲

OCuLink 的直接 PCIe 路徑僅增加極少延遲（5–10% 效能損失），因為沒有封包化、協定轉換或重新成幀的過程。Thunderbolt 4 因協定堆疊開銷，損失達 15–30%[^evezone]。

### 2.3 功能比較

| 功能 | OCuLink | Thunderbolt 4/5 | USB4 |
|---|---|---|---|
| 影像傳輸 | ❌ 不支援 | ✅ DisplayPort 通道 | ✅ DisplayPort Alt Mode |
| 電力傳輸（PD） | ❌ 不支援 | ✅ 最高 100–240W | ✅ 最高 240W |
| 熱插拔 | ❌ 有限支援 | ✅ 完整支援 | ✅ 完整支援 |
| 單線解決方案 | ❌ 需三條線 | ✅ 一條線搞定 | ✅ 一條線搞定 |
| Daisy-chain | ❌ 不支援 | ✅ 最多 6 裝置 | ❌ 非原生 |
| 授權費 | **無（開放標準）** | 有（Intel） | 有（USB-IF） |

## 3. OCuLink 未普及的主要原因

### 3.1 缺乏影像與電力傳輸（核心致命缺陷）

OCuLink 是「純 PCIe 裸介面」，不包含影像傳輸或電力傳輸功能[^csdn]。這意味著使用者需要：

1. OCuLink 資料線
2. 外接 GPU 的獨立電源供應
3. 從 GPU 到螢幕的獨立影像線（HDMI/DisplayPort）

與 Thunderbolt/USB4 的「一條 USB-C 線同時傳資料、影像、電力」相比，OCuLink 的使用體驗明顯繁瑣，對主流消費者缺乏吸引力。

### 3.2 缺乏行銷推動力與認證體系

Thunderbolt 由 **Intel 積極推動**，包含完善的認證計劃與品牌行銷，儘管需要授權費，但這也資助了生態系統的建立。OCuLink 是 PCI-SIG 的開放標準，沒有行銷預算、沒有消費者可見的認證標章、沒有單一公司主導推廣[^pcisig]。

### 3.3 雞蛋問題（生態系統彼此等待）

消費者不會購買沒有連接埠的裝置，製造商也不會在沒有消費者需求的情況下增加連接埠。Thunderbolt 透過 Intel/Apple 在 premium 筆記型電腦上強力推動打破了這個循環。OCuLink 多年來停留在伺服器領域，直到 2023–2024 年才由中國迷你 PC 與手持裝置廠商帶入消費市場[^evezone]。

### 3.4 無法與 USB-C 共用連接埠

OCuLink 使用專有的 SFF-8611/8612 連接埠，無法與 USB-C 相容。要增加 OCuLink 支援，裝置需要**第二個專用連接埠**——對於追求輕薄的筆記型電腦來說難以接受。相比之下，Thunderbolt 3/4/5 與 USB4 共用 USB-C 連接埠，一個連接埠即可支援充電、螢幕、集線器、儲存、**與 eGPU**[^howtogeek]。

### 3.5 設定複雜度

OCuLink 的使用者體驗對一般消費者不友善：

- 需進入 BIOS 開啟 "Above 4G Decoding"
- 可能需設定 PCIe Bifurcation
- 需管理獨立電源供應
- 需冷插拔（無法熱插拔）
- 短纜線長度限制桌面配置

這些技術門檻對主流消費者太高[^lenvanta]。

### 3.6 廠商專有分支導致碎片化

Lenovo 使用 **TGX**（基於 SFF-8611 但專有），ASUS ROG 使用 **XG Mobile**（同樣專有）[^chargerlab]。這些不相容於標準 SFF-8612 的 OCuLink 擴充塢，進一步分裂了本就不大的生態系統。

### 3.7 纜線長度限制

| 纜線類型 | 可靠最大長度 |
|---|---|
| 被動銅纜 | ~0.5m 至 1m（建議小於 1m） |
| 主動增強纜線 | 最長 ~6m（PCIe 4.0 ×4） |
| 光纖 OCuLink | 最長 100m（但極少採用） |

超過 1m 後訊號完整性顯著下降，可能導致降速或斷線[^reddit-cable]。相比之下，Thunderbolt/USB4 的被動銅纜可靠長度可達 2–3 公尺。

## 4. 現狀與近期發展（2025–2026）

### 4.1 消費級裝置支援

OCuLink 在 2024–2026 年間迎來了**顯著增長**，主要集中在手持遊戲 PC、迷你 PC 與小眾筆記型產品線[^oculinknet]：

- **GPD**：Win Max 2、Win 4、Win Mini、GPD G1（eGPU 擴充塢）
- **ONEXPLAYER**：ONEXPLAYER X1、ONEXGPU（eGPU 擴充塢）
- **AYANEO**：FLIP DS、FLIP KB
- **Minisforum**：AtomMan X7 Ti、MS-A1、UM780 XTX、DEG1/DEG2 eGPU 擴充塢
- **Lenovo**：ThinkBook 14/16 i Gen 6+（TGX 介面，基於 OCuLink）
- **ASUS ROG**：Flow Z13/X16/X13 2023（XG Mobile 介面）
- **Framework**：Framework Laptop 16（宣布 OCuLink 支援）

### 4.2 2026 年被稱為「OCuLink 主流化之年」

多位分析師指出 2026 年是 OCuLink 在消費級市場真正起飛的一年[^evezone-mainstream]。原因包括：

- PCIe 5.0 時代 OCuLink 的頻寬優勢進一步擴大
- 手持 PC 市場蓬勃，這些裝置對 eGPU 效能需求強烈
- OCuLink 擴充塢價格低廉（~$99 USD）降低了進入門檻
- Thunderbolt 5 授權成本仍高，限制了其在中低價位裝置的滲透

## 5. 結論

| 面向 | OCuLink | Thunderbolt / USB4 |
|---|---|---|
| **哲學** | 原始效能優先，開放標準 | 便利性優先，多功能合一 |
| **最佳場景** | 固定式 eGPU、伺服器、進階玩家 | 筆記型電腦、行動使用者、一般周邊 |
| **效能** | 外接 GPU 最高效能（5–10% 損失） | 對多數使用者足夠好（15–30% 損失） |
| **成本** | 低（無晶片/授權費） | 較高（控制器晶片 + 授權費） |
| **生態系統** | 小眾但快速成長 | 廣泛普及 |

**OCuLink 並非 Thunderbolt/USB4 的直接競爭者**——兩者服務不同的市場定位。OCuLink 追求極致效能，代價是犧牲便利性與多功能性；Thunderbolt/USB4 追求普遍適用性，代價是效能損耗與成本增加。

對於追求**最高 eGPU 幀率**且願意接受較複雜設定的使用者，OCuLink 是更優選擇。對於追求**一條線搞定一切**的一般使用者，Thunderbolt/USB4 仍是正確答案。

## 參考文獻

[^pcisig]: PCI-SIG. (n.d.). OCuLink Specification Overview. Retrieved 2026-10-01, from https://pcisig.com/specification-overview/oculink

[^csdn]: CSDN. (2025). OCuLink 接口详解 — 与 Thunderbolt 的区别. Retrieved 2026-10-01, from https://blog.csdn.net/weixin_36012152/article/details/164391314

[^evezone]: EveZone. (2026). OCuLink vs Thunderbolt 5 vs USB4: Choosing the Right eGPU Connection. Retrieved 2026-10-01, from https://evezone.evetech.co.za/deep-dives/oculink-vs-thunderbolt-5-vs-usb4-choosing-the-right-egpu-connection

[^xda]: XDA Developers. (2025). Why Thunderbolt 5 Didn't Replace OCuLink for eGPU. Retrieved 2026-10-01, from https://www.xda-developers.com/why-thunderbolt-5-didnt-replace-oculink-for-egpu/

[^howtogeek]: How-To Geek. (2024). What Is OCuLink? Retrieved 2026-10-01, from https://www.howtogeek.com/what-is-oculink/

[^lenvanta]: Lenvanta. (2025). OCuLink eGPU Explained: What You Need to Know. Retrieved 2026-10-01, from https://www.lenvanta.com/guides/oculink-egpu-explained

[^chargerlab]: ChargerLab. (2024). Lenovo and ROG Laptops Are the First to Support OCuLink. Retrieved 2026-10-01, from https://www.chargerlab.com/lenovo-and-rog-laptops-are-the-first-to-support-oculink-which-is-faster-than-thunderbolt-4/

[^oculinknet]: OCuLink.net. (2026). OCuLink Device Database. Retrieved 2026-10-01, from https://www.oculink.net/

[^evezone-mainstream]: EveZone. (2026). OCuLink Goes Mainstream in 2026: Why the PCIe-Direct eGPU Standard Is Finally Catching On. Retrieved 2026-10-01, from https://evezone.evetech.co.za/quick-bytes/oculink-goes-mainstream-in-2026-why-the-pcie-direct-egpu-standard-is-finally-catching-on/

[^reddit-cable]: Reddit r/eGPU. (2025). Question: Length of OCuLink Cable. Retrieved 2026-10-01, from https://www.reddit.com/r/eGPU/comments/1cy1ez8/question_length_of_oculink_cable/

[^toms]: Tom's Hardware. (2025). OCuLink Outpaces Thunderbolt 5 in RTX 5070 Ti Tests. Retrieved 2026-10-01, from https://www.tomshardware.com/pc-components/gpus/oculink-outpaces-thunderbolt-5-in-nvidia-rtx-5070-ti-tests-latter-up-to-14-percent-slower-on-average-in-gaming-benchmarks

[^toms4090]: Tom's Hardware. (2025). High-End External GPUs Still Suffer a Performance Hit: OCuLink Tests Show Up to a 23% Drop with RTX 4090. Retrieved 2026-10-01, from https://www.tomshardware.com/pc-components/gpus/high-end-external-gpus-still-suffer-a-performance-hit-oculink-tests-show-up-to-a-23-drop-with-an-rtx-4090

[^pcgamer]: PC Gamer. (2025). State of Play: eGPUs. Retrieved 2026-10-01, from https://www.pcgamer.com/hardware/graphics-cards/state-of-play-egpus/

[^cloudzat]: Cloudzat. (2026). OCuLink vs Thunderbolt 5 eGPU. Retrieved 2026-10-01, from https://cloudzat.com/oculink-vs-thunderbolt-5-egpu/