# 3DS 形式之 FOSS 硬體手持裝置專案調查

## 摘要

本報告調查網際網路上可取得之自由／開源硬體（FOSS Hardware）手持遊戲機專案，特別聚焦於近似 Nintendo 3DS 形式因素——即具備雙螢幕、摺疊貝殼機（clamshell）或相近尺寸之設計。調查結果顯示 DSpi／DSpi3 為目前最接近 3DS 的開源硬體專案；RISCBoy 則為最完整的從零開始開源遊戲機設計。

## 1. 背景

Nintendo 3DS 作為一代雙螢幕摺疊手持遊戲機，其獨特的形式因素在開源硬體社群中一直缺乏直接對應的開放設計。本報告旨在盤點目前存在的 FOSS 手持裝置專案，並評估其與 3DS 形式的接近程度。

## 2. 調查結果

### 2.1 DSpi 系列 — 最接近 3DS 的開源硬體專案

DSpi 是由創作者 **borpendy** 發起的雙螢幕 Linux 手持遊戲機專案，其設計目標明確為 DS 與 3DS 模擬用途[^dspi]。關鍵規格：

- **DSpi v1**：採用 Raspberry Pi Compute Module 5，雙 800×480 電容觸控螢幕，5000mAh 電池，Xbox 風格控制器（含 3DS 滑墊），RP2040 控制器晶片搭載 GP2040CE 韌體，立體聲喇叭，全 3D 列印外殼（使用 GBA SP 轉軸），儲藏室包含 KiCad PCB 檔案、3D 列印檔案、BOM、系統映像與韌體，授權為 CC BY-NC-SA 4.0[^dspi_repo]。
- **DSpi3（v3）**：第三代設計，升級為雙 720p 顯示器（較前代 480p 提升），雙電容觸控，重新設計散熱系統，更窄的機身。採用 **Modu-PI** 模組化主機板系統[^dspi3]。專案說明指出：「DSpi3 是 DS-pi 專案的第三代，目標是建立開源的雙螢幕遊戲手持裝置，專注於 DS 與 3DS 模擬。」[^dspi3_desc]

### 2.2 Modu-PI — 模組化開源 CM5 手持架構

同為 borpendy 開發的模組化基礎架構，專為 CM5 手持裝置快速原型設計。Modu-PI 提供 BMS（電池管理系統）、Coulomb 計數器、PD 觸發器、雙 DSI 埠、SD 卡槽、擴充槽（含 STM32 電源管理 MCU）等核心功能。已發表之子板設計包含：DSpi 後繼機、雙 7 吋手持裝置、控制器大小 PC，以及筆記型電腦大小之 Cyberdeck[^modupi]。

### 2.3 RISCBoy — 從 CPU 到 PCB 完全開源的手持遊戲機

由 **Wren6991** 開發的 GBA 形式開源手持裝置，雖非雙螢幕設計，但在開源程度上最為徹底：

- 自製 **RISC-V RV32IMC CPU**（可合成的 Verilog 實作）
- 自製光柵圖形管線（raster graphics pipeline）與顯示控制器
- 匯流排架構、記憶體控制器、UART、SPI、PWM、GPIO
- 完整 **KiCad PCB 佈局**（4 層板、5×5cm、每 10 片約 $65）
- 目標為 **iCE40-HX8k FPGA**，使用全開源工具鏈（Yosys、nextpnr、Icestorm）
- 已通過 RISC-V 相容性測試套件與 riscv-formal 驗證[^riscboy]
- 專案自述：「這是一個來自 RISC-V 於 2001 年就已存在的平行宇宙的 Gameboy Advance。」[^riscboy_desc]

### 2.4 其他相關開源手持專案

| 專案名稱 | 形式因素 | 開源硬體檔案 | 備註 |
|---|---|---|---|
| NucDeck | Steam Deck 形式 | KiCad PCB、3D 列印、BOM | Intel NUC 為基礎，CERN-OHL-S-2.0 授權[^nucdeck] |
| Picopad | RP2040 單螢幕 | 硬體電路圖公開 | GPLv3，搭售 DIY 套件[^picopad] |
| FrameDeck | Steam Deck 形式 | KiCad PCB、3D 列印 | 使用 Framework 主機板，GPL-3.0[^framedeck] |
| ESPboy | ESP8266 迷你手持 | 完全開源 | 約 $12 可自組，亦為 IoT 開發平台[^espboy] |
| GameShell | 模組化積木式 | 軟體開源，硬體部分開源 | 四核 Cortex-A7，商業化販售[^gameshell] |

### 2.5 綜合比較

在 55+ 個獨立／DIY 手持裝置專案的完整清單[^awesome_indie]中，**DSpi／DSpi3 是目前唯一同時滿足以下條件的開源專案**：

1. ✅ 雙螢幕設計
2. ✅ 貝殼摺疊機身
3. ✅ KiCad PCB 原始檔開放
4. ✅ 3D 列印外殼檔案開放
5. ✅ BOM 與組裝說明完整
6. ✅ 明確以 DS／3DS 模擬為設計目標

## 3. 結論

若目標是尋找近似 3DS 的開源硬體手持裝置，**DSpi 系列（尤其是 DSpi3）為最直接對應的專案**。其具備完整的 KiCad PCB 設計檔、3D 列印外殼、完整 BOM，且從設計之初即以雙螢幕 DS／3DS 模擬為核心目標。此外，Modu-PI 提供了模組化的基礎架構，適合想自行設計手持裝置形式因素的開發者。對於追求從零開始完全開源的設計，RISCBoy 則是架構上最令人印象深刻的全自製開源遊戲機。

---

[^dspi]: borpendy. (n.d.). DSpi — Dual screen Linux handheld, powered by the Raspberry Pi CM5. GitHub 儲藏室. Retrieved 2026-09-27, from https://github.com/borpendy/DSpi

[^dspi_repo]: borpendy. (n.d.). DSpi README. GitHub. Retrieved 2026-09-27, from https://github.com/borpendy/DSpi

[^dspi3]: pcannon67. (n.d.). DSpi3 — 3rd generation of the DS-pi project. GitHub 儲藏室. Retrieved 2026-09-27, from https://github.com/pcannon67/dspi3

[^dspi3_desc]: pcannon67. (n.d.). DSpi3 repository description. GitHub. Retrieved 2026-09-27, from https://github.com/pcannon67/dspi3

[^modupi]: borpendy. (n.d.). Modu-PI — Modular base for CM5 handhelds. GitHub 儲藏室. Retrieved 2026-09-27, from https://github.com/borpendy/Modu-PI

[^riscboy]: Wren6991. (n.d.). RISCBoy — Open source game console designed from scratch. GitHub 儲藏室. Retrieved 2026-09-27, from https://github.com/Wren6991/RISCBoy

[^riscboy_desc]: Wren6991. (n.d.). RISCBoy repository description. GitHub. Retrieved 2026-09-27, from https://github.com/Wren6991/RISCBoy

[^nucdeck]: dmcke5. (n.d.). NucDeck — Open source DIY handheld gaming PC. GitHub 儲藏室. Retrieved 2026-09-27, from https://github.com/dmcke5/NucDeck

[^picopad]: Pajenicko. (n.d.). Picopad — RP2040 open source game console. GitHub 儲藏室. Retrieved 2026-09-27, from https://github.com/Pajenicko/Picopad

[^framedeck]: redglitch2. (n.d.). FrameDeck — First Open Source Framework Powered Handheld. GitHub 儲藏室. Retrieved 2026-09-27, from https://github.com/redglitch2/FrameDeck

[^espboy]: ESPboy Team. (n.d.). ESPboy — Open source ESP8266/ESP32 handheld multi-gadget. 官方網站. Retrieved 2026-09-27, from https://www.espboy.com/

[^gameshell]: ClockworkPi. (n.d.). GameShell — Modular open source handheld. 官方網站. Retrieved 2026-09-27, from https://www.clockworkpi.com/gameshell

[^awesome_indie]: oshaboy. (n.d.). Awesome Indie Handhelds — Curated list of indie/DIY/hobbyist handhelds. GitHub 儲藏室. Retrieved 2026-09-27, from https://github.com/oshaboy/awesome-indie-handhelds