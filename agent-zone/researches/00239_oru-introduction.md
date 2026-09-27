# ORU (Orbital Replacement Unit) — 軌道更換單元

## 什麼是 ORU？

**Orbital Replacement Unit（軌道更換單元，簡稱 ORU）** 是一種設計成模組化的太空載具組件，當其壽命到期或發生故障時，可由太空人在艙外活動（EVA／太空漫步）中或由機器人系統在軌道上直接更換。此概念相當於航空領域的 **線可更換單元（Line-Replaceable Unit, LRU）**，可說是 LRU 的太空版本。[^wiki-oru]

ORU 的核心思想是：**不 repair，replace**——不需在太空中修復故障的設備，只需將整個模組拆下、換上備用模組即可。

## ORU 的設計特徵

為了讓太空人或機械手臂能在失重環境下順利操作，ORU 的設計包含以下關鍵特徵[^wiki-oru]：

- **標準化介面** — ORU 配有夾具、連接器和緊固件，設計上相容於太空人手持工具（如 Pistol Grip Tool，使用標準 7/16 吋雙倍高度六角頭螺栓）或機器人手臂。
- **機器人抓取特徵** — 由 **Dextre（加拿大機械手臂 SPDM）** 處理的 ORU，使用專用的 **OTCM（ORU/Tool 更換機構）** 來抓取，常見的接口包括 H-fixtures、Micro-fixtures（Micro-squares）、Micro-Conical Fittings、MTC Targets。
- **抓握夾具（Grapple Fixture）** — 任何配有抓握夾具的 ORU 都可被 **Canadarm2（太空站遙控機械手臂系統）** 移動。
- **存放平台** — 備用 ORU 存放在 ISS 桁架結構上的 **ESPs（外部存放平台）** 或 **ELCs（ExPRESS 物流載具）**。

## 國際太空站（ISS）上的 ORU 範例

ISS 上幾乎所有關鍵的外部系統都設計成 ORU 形式，涵蓋熱控、電力、姿態控制、生命支援、通訊及機器人等領域[^wiki-oru]。

### 熱控系統 ORU

| ORU 名稱 | 重量 | 功能 |
|---|---|---|
| Pump Module（泵模組） | 780 lbs | 循環氨冷卻劑，驅動外部主動熱控系統（EATCS） |
| Ammonia Tank Assembly（氨罐組件） | 1,702 lbs | 儲存氨冷卻劑 |
| Nitrogen Tank Assembly（氮罐組件） | 550 lbs | 提供高壓氮氣來控制氨的流動 |
| Flex Hose Rotary Coupler（軟管旋轉耦合器） | ~900 lbs | 在熱散熱器旋轉接頭兩側傳輸液態氨 |
| Heat Rejection System Radiator（散熱器） | 2,475 lbs | 八面板可展開散熱器，透過輻射排除廢熱 |

### 電力系統 ORU

| ORU 名稱 | 重量 | 功能 |
|---|---|---|
| Battery Charge/Discharge Unit（電池充放電單元） | 235 lbs | 太陽照期間充電、陰影期供電 |
| Main Bus Switching Unit（主匯流排切換單元） | 220 lbs | 電力配電中心 |

### 姿態控制 ORU

- **Control Moment Gyroscope（控制力矩陀螺儀，CMG）** — 600 lbs，內含 220 lbs 不鏽鋼飛輪，轉速 6,600 rpm，角動量 3,600 ft-lb-sec。Z1 桁架上共有 4 顆運作中。[^wiki-oru]

### 其他 ORU

- **Plasma Contactor Unit（電漿接觸器）** — 350 lbs，用於消除太空站表面靜電累積。
- **Latching End Effector（鎖定末端效應器）** — 415 lbs，Canadarm2 和 Dextre 的「手」。
- **Mobile Transporter Trailing Umbilical System-Reel Assembly** — 354 lbs，類似自動收線的園藝水管。

## 哈伯太空望遠鏡（HST）的 ORU

哈伯太空望遠鏡從設計之初就內建了約 **70 個軌道更換單元**，包括科學儀器和有限壽命件（如電池）[^wiki-hst-oru]。

- 重要的 ORU 包括 **Wide Field Camera 3**、**Cosmic Origins Spectrograph**、**Advanced Camera for Surveys**，以及用來修正哈伯主鏡像差的 **COSTAR**。
- 從 1993 年到 2009 年，共進行了 **5 次維修任務**、23 次太空漫步，更換或升級了數十個 ORU，使哈伯從原本設計的約 15 年壽命延長至超過 30 年。[^nasa-hst-missions]
- 最後一次維修任務 **STS-125（SM4，2009 年）** 安裝了兩個新儀器、修復了兩個受損儀器、更換了全部 6 顆陀螺儀和電池。[^nasa-sts125]

## ORU 的補給方式

ORU 透過多種運輸方式送達 ISS[^wiki-oru]：

- **太空梭** — 使用 Integrated Cargo Carrier（ICC）、Lightweight MPESS Carrier（LMC）等載具
- **HTV（白鸛號）** — 日本 H-II Transfer Vehicle，透過其暴露平台運送 ORU
- **商業補給任務** — 如 SpaceX Dragon、Cygnus 等

## ORU 對太空任務的重要性

1. **延長任務壽命** — 模組化可更換的設計使太空站的使用壽命能大幅超越原始設計年限。[^wiki-oru]
2. **降低任務風險** — 故障組件可在軌道上更換，無需發射全新太空載具。
3. **實現技術升級** — 如哈伯望遠鏡多次升級科學儀器，每次升級都大幅提升觀測能力。
4. **成本效益** — 更換單一元件的成本遠低於建造和發射全新的系統。
5. **推動模組化設計理念** — ORU 概念深刻影響了現代太空載具設計，包括商業太空站和未來深空探測船。

## 總結

ORU 是現代太空任務中不可或缺的設計哲學。它將複雜的太空載具拆解為一個一個可獨立更換的模組，讓維修和升級從「不可能」變為「可執行」。從 ISS 的泵浦模組到哈伯的科學儀器，ORU 直接決定了這些重要太空資產的使用壽命和價值，也為未來月球和火星任務中的設備維護提供了設計範本。

[^wiki-oru]: Wikipedia. (n.d.). *Orbital replacement unit*. Retrieved 2026-09-25, from https://en.wikipedia.org/wiki/Orbital_replacement_unit
[^wiki-hst-oru]: Wikipedia. (n.d.). *Orbital replacement unit (HST)*. Retrieved 2026-09-25, from https://en.wikipedia.org/wiki/Orbital_replacement_unit_(HST)
[^nasa-hst-missions]: NASA. (n.d.). *Missions to Hubble*. Retrieved 2026-09-25, from https://science.nasa.gov/mission/hubble/observatory/missions-to-hubble/
[^nasa-sts125]: NASA. (n.d.). *STS-125*. Retrieved 2026-09-25, from https://www.nasa.gov/mission/sts-125/