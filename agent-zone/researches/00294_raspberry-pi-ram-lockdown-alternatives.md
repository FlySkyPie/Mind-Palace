# Raspberry Pi 韌體鎖定 RAM 升級事件與替代方案調查

## 事件概述

2026 年 9 月，Raspberry Pi 官方證實其韌體自 2024 年底起，已將主機板鎖定至出廠時的 RAM 配置[^geerling]。這意味著即使使用者解焊原始 RAM 晶片並更換為更高容量（或相同容量）的晶片，主機板也無法開機或無法正確識別新記憶體。韌體在每次開機時，會檢查 RAM 是否符合儲存在 SoC 一次性可編程（OTP, One-Time Programmable）記憶體中的配置資料[^tomshardware]。

此消息由 Raspberry Pi 工程師兼論壇版主 PhilE 在官方論壇回應一位使用者時公開，該使用者無法在 Compute Module 5（原為 2GB 版本）上成功換裝 4GB 晶片[^forums]。

## 時間線

- **2024 年 9 月 23 日**：韌體版本 `2024-09-23-2712` 首次實裝 RAM 鎖定機制，更新日誌僅記載「微調以符合製造測試」——**未說明 RAM 鎖定**[^geerling]
- **2024 年 9 月 10 日（及更早版本）**：最後一個不強制 RAM 鎖定的韌體版本
- **2026 年 9 月 21 日**：Jeff Geerling 發布影片及部落格文章，事件開始廣為人知[^geerling]
- **2026 年 9 月 22 日後**：Tom's Hardware、Hackaday、TweakTown、TechPowerUp 等媒體陸續報導[^tomshardware][^hackaday][^tweaktown][^techpowerup]

## 受影響型號

| 型號 | 狀態 | 說明 |
|------|------|------|
| **Raspberry Pi 5** | ✅ 已確認 | 所有 RAM 配置（1/2/4/8/16GB） |
| **Compute Module 5 (CM5)** | ✅ 已確認 | 觸發此次公開揭露的型號 |
| **Compute Module 4 (CM4)** | ⚠️ 可能受影響 | 韌體鎖定適用於搭載焊接式記憶體的板卡 |
| **Raspberry Pi 4** | ⚠️ 可能受影響 | 仍接收韌體更新，鎖定機制可能已包含 |

所有採用焊接式 LPDDR4x 記憶體且接收 2024 年底後韌體更新的新機型均受影響[^tomshardware]。

## 韌體鎖定造成的限制

1. **RAM 容量升級**：無法將低規格型號（如 2GB）更換為高容量晶片（如 4GB 或 8GB）[^phile]
2. **同容量晶片更換**：即使是從另一塊 Raspberry Pi 上取下相同容量的 RAM 晶片，也可能無法使用——韌體如今會將每塊主機板個別的記憶體密度、組織方式與時序參數寫入 OTP[^phile]
3. **維修更換**：若 RAM 晶片故障，無法可靠地更換為新晶片[^geerling]

## Raspberry Pi 的官方理由

PhilE 在論壇中說明：此措施是為了**遏止轉售商詐騙**[^phile]。部分轉售商購入低規格 Pi 板（1GB、2GB），解焊原始 RAM，換上廉價/未驗證的高容量晶片，再冒充原廠 8GB 裝置轉售。這些裝置因記憶體時序有問題或焊接不良而出現穩定度問題時，使用者會轉向 **Raspberry Pi 求助**，而非向轉售商索賠[^tomshardware]。

Raspberry Pi 表示此鎖定機制實施於 AI 需求引發的 2025–2026 RAM 價格飆漲之前，因此是主動防詐措施，而非對市場條件的反應[^geerling]。

## 已知解決方法

可**刷寫舊版韌體**（版本 `2024-09-10-2712` 或更早）以繞過鎖定。但這同時代表放棄所有後續的韌體修復、功能改進及安全性更新[^geerling]。

目前**沒有**官方的「認可此修改」機制。Jeff Geerling 等人建議 Raspberry Pi 應引入類似舊款 Pi 超頻用的「保固位元」（warranty bit）——設為單次可寫的韌體旗標，在修改硬體後自願放棄官方支援，而非直接封鎖硬體變更[^geerling]。Raspberry Pi 至今尚未提供此類方案。

---

# Raspberry Pi 替代方案全面比較

2026 年全球 LPDDR4 記憶體價格因 AI 需求較 2025 年底飆漲約 7 倍，導致 Pi 5 價格上漲至約 €137（4GB）及約 €190（8GB），與競爭對手的價差因此縮小[^chipradar]。以下為主要替代方案比較。

## 快速總覽

| 板卡 | SoC | 最大 RAM | NVMe | 乙太網路 | WiFi | 預估價格 (2026) | 最適合 |
|------|-----|----------|------|----------|------|-----------------|--------|
| **Raspberry Pi 5** | BCM2712 (4×A76) | 16GB | PCIe 2.0 x1 (HAT) | 1GbE | WiFi 5 | ~$110–175 | 生態系、初學者 |
| **Orange Pi 5 Plus** | RK3588 (4×A76+4×A55) | 32GB | PCIe 3.0 x4 (原生) | 2×2.5GbE | 無（M.2 插槽） | ~$85–100（進口） | 最佳 RK3588 性價比 |
| **Orange Pi 5 Pro** | RK3588S | 16GB | PCIe 3.0 NVMe | 1GbE | WiFi 6 | ~$110 (8GB) | 搭載 WiFi 的緊湊型 RK3588 |
| **Radxa Rock 5B/5B+** | RK3588 | 32GB | PCIe 4.0 x4（原生） | 2.5GbE | 無（M.2 插槽） | ~$130–265 | 高效能使用者、NVMe 速度 |
| **Banana Pi M7** | RK3588 | 32GB | PCIe 3.0 x4 | 2×2.5GbE | WiFi 6 | ~$160+ | 全功能 RK3588 |
| **Odroid M2** | RK3588S2 | 16GB | PCIe 3.0 NVMe | 2.5GbE | 無 | ~$184 | ARM Docker/Proxmox |
| **Radxa X4** | Intel N100 (x86) | 16GB | PCIe 3.0 x4 | 2.5GbE + PoE | WiFi 6 | ~$60–80 | x86 相容性 |
| **Orange Pi Zero 3** | H618 (4×A53) | 4GB | 無 | 1GbE | WiFi 5 | ~$25（進口） | 預算 IoT |
| **NVIDIA Jetson Orin Nano Super** | ARM + GPU | 8GB | NVMe | 1GbE | WiFi | ~$249 | 邊緣 AI / ML |
| **Khadas VIM4** | A311D2 (4×A73+4×A53) | 8GB | M.2 插槽 | 1GbE | WiFi 6 | ~$120–150 | 媒體/緊湊型 |
| **BeagleBone Black** | AM3358 (1×Cortex-A8) | 512MB | 無 | 100MbE | 無 | ~$55–75 | 工業 GPIO/PRU |
| **Pine64 RockPro64** | RK3399 (2×A72+4×A53) | 4GB | PCIe x4 插槽 | 1GbE | 無 | ~$60–80 | 較舊但仍可用 |
| **ASUS Tinker Board 3N** | RK3568 (4×A55) | 4GB | M.2 B-key | 2×GbE | WiFi 5 | 工業定價 | 工業 IoT |
| **Libre Computer Le Potato** | S905X (4×A53) | 2GB | 無 | 1GbE | 無 | ~$35 | 低價 Linux 學習 |

## 重點替代方案分析

### 1. Rockchip RK3588 三強（Orange Pi 5 / Radxa Rock 5B / Banana Pi M7）

三者均採用 **RK3588**——8 核心晶片（4×Cortex-A76 @ 2.4GHz + 4×Cortex-A55 @ 1.8GHz），搭載 Mali-G610 GPU（約 610 GFLOPS，為 Pi 5 VideoCore VII 的 10 倍）及 6 TOPS NPU[^chipradar]。

**Geekbench 6 效能：** 單核約 850 分，多核約 4800 分。Pi 5 在單核勝出（約 1604 分），但 RK3588 在多核表現上明顯領先[^sbccompare]。

| 功能 | Orange Pi 5 Plus | Radxa Rock 5B/5B+ | Banana Pi M7 |
|------|-----------------|-------------------|--------------|
| **價格** | ~$85–100（限進口） | ~$130–265 | ~$160+ |
| **WiFi** | ❌ 無（需 M.2 模組） | ❌ 無（需 M.2 模組） | ✅ WiFi 6 + BT 5.2 |
| **RAM 選項** | 最高 32GB LPDDR4X | 最高 32GB LPDDR4X | 最高 32GB LPDDR4X |
| **NVMe** | PCIe 3.0 x4 原生 | PCIe 4.0 x4 原生 | PCIe 3.0 x4 原生 |
| **網路** | 2× 2.5GbE | 1× 2.5GbE | 2× 2.5GbE |
| **eMMC** | 插槽 | 插槽 | 64/128GB 焊接 |
| **HDMI IN** | ✅ | ❌ | ❌ |
| **2026 購買管道** | 限進口（AliExpress） | 歐美較易取得 | 管道有限 |

**Orange Pi 5 Pro（RK3588S）** 為更緊湊的變體，內建 **WiFi 6**。單一 Gigabit 乙太網路（而非 Plus 的雙 2.5GbE），採用 LPDDR5 RAM，8GB 版本約 $110[^armbian-orangepi5pro]。

### 2. Radxa X4 — x86 架構替代方案

採用 **Intel N100**（Alder Lake-N，4 核 4 執行緒，3.4GHz 渦輪加速）的信用卡大小 x86 SBC[^radxa-x4]。

| 規格 | Radxa X4 |
|------|----------|
| **CPU** | Intel N100 (x86) |
| **RAM** | 最高 16GB LPDDR5 |
| **儲存** | M.2 PCIe 3.0 x4 NVMe |
| **網路** | 2.5GbE + PoE 支援 |
| **無線** | WiFi 6 + BT 5.2 |
| **GPIO** | 40-pin Pi 相容（+ RP2040 協處理器！） |
| **價格** | ~$60 (4GB) – $80 (8GB) |
| **效能** | Geekbench 6：單核 1,243 / 多核 2,981 |

**重要性：** 完整 x86 相容性——所有 Docker 映像檔、Linux 套件甚至 Windows 應用程式皆無需 ARM 轉譯層。內建 RP2040 晶片負責 GPIO，不犧牲即時 I/O 能力[^sbccompare-radxax4]。

### 3. NVIDIA Jetson Orin Nano Super

- **AI 效能：** 67 TOPS（INT8）
- **RAM：** 8GB（CPU/GPU 共享）
- **價格：** ~$249
- **最適合：** **AI 推論**——電腦視覺、LLM、機器人學、TensorFlow/PyTorch 部署
- **軟體生態系：** NVIDIA JetPack SDK、CUDA、TensorRT、DeepStream——業界最佳 AI 生態系[^nvidia-jetson]
- **2026 注意：** NVIDIA 於 2026 年中將 Jetson 模組價格調漲高達 101%，但 Nano Super DevKit 仍維持 $249

### 4. Odroid M2（Hardkernel）— RK3588S2

- **RAM：** 8GB 或 16GB LPDDR5
- **儲存：** M.2 PCIe 3.0 NVMe + eMMC 插槽
- **網路：** 2.5GbE（無 WiFi）
- **價格：** 約 €184（8GB）
- **注意：** ✅ 40-pin GPIO，但 **與 Pi 不相容**——HAT 無法使用[^odroid-m2]
- **最適合：** Docker 堆疊、Proxmox、不依賴 GPIO 相容性的自託管 ARM 工作負載

### 5. 預算級：Orange Pi Zero 3

- **SoC：** H618（4×A53 @ 1.5GHz）
- **RAM：** 最高 4GB
- **價格：** 約 $25（直接進口）
- **WiFi：** WiFi 5
- **注意：** Amazon 市場賣家售價高達 €119，失去價格優勢[^raspberrytips]

## 效能比較（Geekbench 6）

| 板卡 | 單核 | 多核 |
|------|------|------|
| **Raspberry Pi 5** | ~1,604 | ~4,200 |
| **Radxa X4 (Intel N100)** | ~1,243 | ~2,981 |
| **Orange Pi 5 Pro (RK3588S)** | ~803 | ~3,018 |
| **Raspberry Pi 4** | ~900 | ~2,900 |
| **Pine64 RockPro64** | ~400 | ~1,800 |
| **Orange Pi Zero 3** | ~350 | ~1,000 |

Pi 5 在單核效能方面領先所有 ARM SBC——適合 Python 腳本、Home Assistant、Node-RED 等偏重單緒的應用。RK3588 板卡在多緒工作負載上勝出，Radxa X4（x86）則在軟體相容性上佔有優勢[^sbccompare][^sbccompare-radxax4]。

## 社群與軟體支援評比

| 板卡 | 社群規模 | 文件品質 | 作業系統選項 | GPIO 相容 |
|------|---------|---------|-------------|-----------|
| **Raspberry Pi 5** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Raspberry Pi OS、Ubuntu 等 | ✅（原生） |
| **Radxa Rock 5B** | ⭐⭐⭐ | ⭐⭐⭐ | Armbian、Ubuntu、Radxa OS | ✅ 40-pin |
| **Orange Pi 5 系列** | ⭐⭐⭐ | ⭐⭐ | Armbian、Orange Pi OS | ✅ 40-pin |
| **Odroid M2** | ⭐⭐ | ⭐⭐⭐ | Hardkernel OS、Ubuntu | ❌ 與 Pi 不相容 |
| **Banana Pi M7** | ⭐⭐ | ⭐⭐ | Armbian、Debian | ✅ 40-pin |
| **NVIDIA Jetson Orin** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | JetPack（Ubuntu 基礎） | ❌ 無標準 GPIO |
| **Radxa X4 (x86)** | ⭐⭐⭐ | ⭐⭐⭐ | 所有 x86 Linux、Windows | ✅ 40-pin + RP2040 |
| **BeagleBone Black** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | Debian、Ubuntu、Android | ⚠️ 2×46-pin (Capes) |
| **Khadas VIM4** | ⭐⭐ | ⭐⭐⭐ | Ubuntu、Armbian、OOWOW | ⚠️ 40-pin（非標準） |

## 2026 年重大注意事項

全球 RAM 危機已影響所有廠商。記憶體價格約**翻倍**——「亞洲替代方案較便宜」的常識在 2026 年已不再成立[^chipradar]：

- Raspberry Pi 5（4GB）：$60 → $110
- Orange Pi 5B（16GB）：$160 → $312（漲幅 95%）
- Radxa Rock 5B+（8GB）：約 $251

Raspberry Pi 生態系——文件、HAT、外殼、社群教學、本地零售商、長期支援——往往使 Pi 5 儘管價格較高仍是較佳選擇，尤其對初學者而言。

## 按使用情境推薦

| 情境 | 推薦 | 理由 |
|------|------|------|
| **最佳整體替代方案** | **Radxa Rock 5B+** | RK3588，原生 PCIe 4.0 NVMe，最高 32GB RAM，社群支援良好，約 $130–265 |
| **最佳性價比** | **Orange Pi 5 Plus（進口）** | RK3588 約 $85–100，但僅限 AliExpress，無保固 |
| **最佳 x86 替代方案** | **Radxa X4** | Intel N100 約 $60–80，完整軟體相容性 |
| **最佳 AI/ML** | **NVIDIA Jetson Orin Nano Super** | 67 TOPS，CUDA 生態系，$249 |
| **最佳 Home Assistant** | **Raspberry Pi 5** 或 **Intel N100 mini-PC** | 官方 HAOS 支援或 x86 Docker |
| **最佳 Docker/自託管 (ARM)** | **Odroid M2** | RK3588S2，2.5GbE，NVMe，注意 GPIO 不相容 |
| **最便宜（進口）** | **Orange Pi Zero 3** | 4GB RAM 約 $25 |
| **工業/即時控制** | **BeagleBone Black** | PRU 協處理器，即時 I/O |

---

[^geerling]: Geerling, J. (2026, September 21). Raspberry Pi locks down Pi 5 RAM upgrades in firmware. Retrieved 2026-09-27, from https://www.jeffgeerling.com/blog/2026/raspberry-pi-ram-lockdown/

[^tomshardware]: Tom's Hardware. (2026, September 22). Raspberry Pi locks boards to factory RAM capacities in firmware. Retrieved 2026-09-27, from https://www.tomshardware.com/raspberry-pi/raspberry-pi-locks-boards-to-factory-ram-capacities-in-firmware-engineer-tells-diy-modders-dont-waste-your-time-trying-repairs-or-upgrades-company-cites-shady-reseller-scams

[^hackaday]: Hackaday. (2026, September 22). Raspberry Pi Locks Down RAM Upgrades. Retrieved 2026-09-27, from https://hackaday.com/2026/09/22/raspberry-pi-locks-down-ram-upgrades/

[^tweaktown]: TweakTown. (2026, September 23). Raspberry Pi confirms firmware has blocked RAM upgrades since late 2024. Retrieved 2026-09-27, from https://www.tweaktown.com/news/113707/raspberry-pi-confirms-firmware-has-blocked-ram-upgrades-since-late-2024/index.html

[^techpowerup]: TechPowerUp. (2026, September 23). Raspberry Pi Confirms Boards Are Locked to Factory RAM Size via Firmware. Retrieved 2026-09-27, from https://www.techpowerup.com/352973/raspberry-pi-confirms-boards-are-locked-to-factory-ram-size-via-firmware

[^phile]: Raspberry Pi Forums. (2026). PhilE response to CM5 RAM upgrade thread. Retrieved 2026-09-27, from https://forums.raspberrypi.com/viewtopic.php?t=399212

[^forums]: GitHub - rpi-eeprom issue #761. Retrieved 2026-09-27, from https://github.com/raspberrypi/rpi-eeprom/issues/761

[^chipradar]: ChipRadar. (2026). Raspberry Pi 5 Alternatives 2026: Best RK3588 SBCs Reviewed. Retrieved 2026-09-27, from https://chipradar.io/blog/raspberry-pi-5-alternatives-2026

[^raspberrytips]: Raspberry Tips. (n.d.). Raspberry Pi Alternatives — Single Board Computer Comparison. Retrieved 2026-09-27, from https://raspberry.tips/en/raspberrypi-tutorials/raspberry-pi-alternatives-sbc

[^sbccompare]: SBC Compare. (n.d.). Orange Pi 5 Pro vs Radxa Rock 5B benchmark comparison. Retrieved 2026-09-27, from https://sbc.compare/orange-pi-5-pro/radxa-rock-5b

[^sbccompare-radxax4]: SBC Compare. (n.d.). Radxa X4 benchmarks. Retrieved 2026-09-27, from https://sbc.compare/radxa-x4

[^radxa-x4]: Radxa. (n.d.). Radxa X4 — Intel N100 SBC. Retrieved 2026-09-27, from https://radxa.com/products/x/x4/

[^nvidia-jetson]: NVIDIA. (n.d.). Jetson Orin Nano Super Developer Kit. Retrieved 2026-09-27, from https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/jetson-orin/nano-super-developer-kit/

[^odroid-m2]: Hardkernel. (n.d.). Odroid M2. Retrieved 2026-09-27, from https://www.hardkernel.com/shop/odroid-m2/

[^armbian-orangepi5pro]: Armbian. (n.d.). Orange Pi 5 Pro board support. Retrieved 2026-09-27, from https://armbian.com/boards/orangepi5pro

[^pine64]: Pine64. (n.d.). RockPro64 4GB Single Board Computer. Retrieved 2026-09-27, from https://pine64.com/product/rockpro64-4gb-single-board-computer/

[^beagleboard]: BeagleBoard.org. (n.d.). BeagleBone Black. Retrieved 2026-09-27, from https://www.beagleboard.org/boards/beaglebone-black

[^asus-tinker]: ASUS. (n.d.). Tinker Board 3N. Retrieved 2026-09-27, from https://www.asus.com/us/networking-iot-servers/aiot-industrial-solutions/tinker-board-series/tinker-board-3n/

[^khadas]: Khadas. (n.d.). VIM4. Retrieved 2026-09-27, from https://www.khadas.com/vim4