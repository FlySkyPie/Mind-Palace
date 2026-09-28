# 10 吋伺服器機櫃是否符合標準？

## 結論：不符合正式國際標準，但屬於業界默契 (de facto standard)

10 吋（10-inch）伺服器機櫃**並非任何正式國際標準所規範的規格**。主導實體機櫃標準的三大機構——EIA-310-D（美國電子工業聯盟）、IEC 60297（國際電工委員會）與 DIN 41494（德國標準化研究所）——均僅定義了 19 吋（482.6 mm）規格系列。[^eia] 然而，由於越來越多廠商採用相近的尺寸，10 吋機櫃已成為一種**業界默契（de facto standard）**，尤其在家庭實驗室（homelab）與小型辦公室市場中逐漸普及。

## 10 吋 vs 19 吋：關鍵尺寸對照

10 吋機櫃的核心設計哲學是「繼承 19 吋標準的一半寬度」，且沿用相同的 1U 高度與垂直孔距：

| 尺寸 | 10 吋機櫃 | 19 吋機櫃（參考） |
|---|---|---|
| 面板總寬度 | **254 mm**（10 吋） | 482.6 mm（19 吋） |
| 螺孔中心距 | **236.525 mm**（9.312 吋） | 464.2–465.8 mm |
| 導軌內徑（淨寬） | ~222 mm（8.75 吋） | 450.85 mm |
| 建議最大裝置寬度 | ~210 mm（8.45 吋） | ~438 mm |
| **1U（機櫃單位）高度** | **44.45 mm（1.75 吋）——與 19 吋相同** | 44.45 mm |
| **垂直孔距規律** | **12.70 / 15.88 / 15.88 mm 交替——與 19 吋相同** | 相同 |
| 深度 | **無標準**——常見 150–300 mm（常用 200 mm 或 260 mm） | 依用途而定，典型 600–1200 mm |

[^eia]: EIA-310-D 是定義 19 吋機櫃的正式標準，最後更新為 1996 年的 Rev E。該標準全文不涵蓋 10 吋規格。Wikipedia. (n.d.). *19-inch rack*. Retrieved 2026-09-27, from https://en.wikipedia.org/wiki/19-inch_rack

### 寬度數據來源

上述三個關鍵寬度並非隨意決定，而是從 19 吋標準的比例縮放而來：面板邊緣至螺孔中心的偏移量（約 17.5 mm）以及螺孔中心至導軌內緣的間距（約 14.25 mm）均直接繼承，因此只要驗算即可確認尺寸一致性。[^dimensions]

[^dimensions]: PickLog. (2024). *10-inch rack dimensions: The de facto standard explained*. Retrieved 2026-09-27, from https://www.picklog.cc/blog/10-inch-rack-dimensions

## 標準化狀態對照

| 面向 | 10 吋機櫃 | 19 吋機櫃 |
|---|---|---|
| 標準化程度 | 業界默契（de facto），無正式標準 | 正式標準：EIA-310、IEC 60297、DIN 41494 |
| 面板寬度 | 254 mm（約 19 吋的一半） | 482.6 mm |
| 1U 高度 | 相同（44.45 mm/U） | 相同 |
| 裝置供應鏈 | 有限；歐洲/英國較充足，北美稀缺 | 全球通用，隨處可得 |
| 常見應用場景 | 家庭實驗室、小型辦公室、可攜式設備、空間受限場域 | 資料中心、企業 IT、電信、專業影音 |
| UPS 可用性 | **極度有限**——市面上不存在原生 10 吋機櫃安裝式 UPS[^ups] | 規格齊全，選擇眾多 |
| 深度規格 | 多為淺機櫃（200–260 mm 為主） | 深機櫃（600–1200 mm 為主流） |
| 市場定位 | 利基／玩家／成長中 | 成熟／工業等級 |

[^ups]: Project MINI RACK 的 GitHub Issue #1 標題即為「Find the perfect 1U 10" Mini Rack UPS」，說明「since one doesn't exist」。截至 2026 年已有超過 150 則留言，仍無完美解決方案。Geerling, J. (2025). *Project MINI RACK, Issue #1: Find the perfect 1U 10" Mini Rack UPS*. Retrieved 2026-09-27, from https://github.com/geerlingguy/mini-rack/issues/1

## 普及程度

**尚未全面普及，但在家庭實驗室與消費級市場中成長顯著。**

- **歐洲／英國**：接受度較高。主要品牌包括 Delock、Digitus、DSIT、Rack Magic、Intellinet、Lanberg 等，多款壁掛式迷你機櫃可供選擇。
- **北美**：供應有限。主要品牌有 DeskPi（RackMate T0/T1/T2 系列）、NavePoint、Middle Atlantic、KENUCO。知名科技 YouTuber Jeff Geerling 表示當地採購不易。[^jeff]
- **亞洲**：市場情況不一，以 OEM 代工製造為主。

[^jeff]: Geerling, J. (2025). *Project MINI RACK*. Retrieved 2026-09-27, from https://mini-rack.jeffgeerling.com/

該規格在 **r/minilab**、**r/homelab** 等 Reddit 社群以及 Jeff Geerling 發起的 **Project MINI RACK**（截至 2026 年已有 300+ 組建置案例）推動下，人氣持續上升。[^minilab]

[^minilab]: Geerling, J. (2025). *Project MINI RACK: The 10" Mini Rack Standard*. Retrieved 2026-09-27, from https://deepwiki.com/geerlingguy/mini-rack/2-10%22-mini-rack-standard

## 已知限制

1. **深度沒有統一標準**——不同廠商的深度不同（200 mm vs 260 mm 為常見差異），且同一品牌內不同配件可能無法跨深度相容。
2. **無原生 10 吋機櫃式 UPS**——如上所述，這是社群最關切的痛點之一。[^ups]
3. **變壓器管理**——迷你機櫃內部空間狹小，外部電源變壓器的收納與整線是一大難題。
4. **螺絲規格混亂**——部分使用 10-32 攻牙孔，部分使用 12-24、方孔搭配籠式螺帽，或 M6 公制螺絲，互不通用。
5. **區域供應不均**——北美不易購買，歐洲較齊全。

## 總結

10 吋伺服器機櫃 **不是** EIA、IEC 或 DIN 定義的正式標準，而是一種**業界默契（de facto standard）**。它繼承了 19 吋機櫃的 1U 高度（44.45 mm）與垂直孔距規律，但寬度縮減約一半（螺孔中心距 236.5 mm，面板寬 254 mm）。在家庭實驗室、小型辦公室與可攜式設備場景中需求漸增，歐洲接受度高於北美。雖然在企業級資料中心尚未普及，但在玩家／消費級市場中持續成長。