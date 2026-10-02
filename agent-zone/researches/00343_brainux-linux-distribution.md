# Brainux — SHARP Brain 専用 Linux 發行版

## 概述

Brainux 是一套基於 **Debian GNU/Linux** 的 Linux 發行版，專為 **SHARP Brain** 系列電子辭典所設計[^1]。這些裝置原生搭載 **Windows CE** 作業系統，Brainux 可與之共存或完全取代，釋放硬體的完整潛能[^2]。

Brainux 專案由社群 **Brain Hackers** 維護，創始人為 **puhitaku**[^3]。

## 開發動機

根據 Brainux 官方團隊的說法，之所以要在 Brain 上執行 Linux，主要基於以下幾個理由[^2]：

1. **釋放真正的駭客潛能** — 核心、驅動程式乃至所有軟體皆可自訂，實現 Windows CE 無法達成的改造。
2. **滿足硬體探索的好奇心** — 專案源於想探索封閉系統之外的各種可能性。
3. **保留那份驚喜與感動** — 重現學生時代 Brain 初次帶給使用者的震撼。
4. **為現今的學生開路** — 讓新一代 Brain 使用者也能接觸程式與系統改裝。

## 主要特色

| 特色 | 說明 |
|------|------|
| **Debian GNU/Linux 基底** | 完整 APT 套件管理器，可使用 Debian 套件庫 |
| **自訂硬體驅動** | 鍵盤、LCD 顯示、觸控面板、音訊、電源管理等驅動 |
| **輕量足跡** | 針對有限的 RAM 與 CPU 最佳化 |
| **U-Boot 引導** | 自訂引導程式，支援 SD 卡開機 |
| **雙重開機** | 可從 Windows CE 的「App Menu」啟動，或直接 SD 卡開機 |
| **SD 卡映像檔** | 類似 Raspberry Pi OS 的「寫入即可用」模式 |
| **brain-config 工具** | 類似 `raspi-config` 的設定工具[^4] |
| **USB Ethernet Gadget** | 透過 USB 將 Brain 變成有線網路裝置（支援 RNDIS/NCM） |
| **鍵盤重新映射** | 特殊按鍵組合以輸入符號 |
| **活躍社群** | Discord 即時討論、GitHub 組織擁有 37+ 倉庫 |

## 支援的硬體

| 支援程度 | 型號範圍 |
|----------|----------|
| **完整開機** | PW-Sx1 至 PW-Sx7 世代（4 位數型號） |
| **部分支援** | G4200、G5200、A7200–A7400、A9100–A9300 等 |
| **尚未支援** | 3 位數型號（如 PW-GC610）及單位數型號（如 PW-H1） |
| **SoC** | NXP i.MX28 及 NXP i.MX7（依世代不同） |
| **SD 卡** | 建議 4GB 以上 |

## 技術細節

- **登入資訊**：使用者名稱 `user`，密碼 `brain`[^5]
- **關機指令**：`sudo shutdown -h now` 或按電源按鈕（避免 SD 卡損毀）
- **音訊**：Yamaha 與 Rohm Smart Amp — 播放與錄音仍在研究中（有限支援）
- **Wi-Fi**：仍在調查中（SDIO Wi-Fi 晶片或 USB 網卡途徑）

## 最新版本（2026-03-25 釋出）[^6]

- **Linux 核心**：升級至 6.1
- **Debian 基底**：升級至 Debian 13.4 "Trixie"
- **USB Ethernet Gadget**：從 RNDIS 轉換為 NCM（macOS、Windows、Linux 皆可用）
- **USB Video Class** 驅動啟用
- 套件更新：移除 `midori`、`neofetch` 替換為 `fastfetch`
- 版本資訊檔（`brainux_version`）置於開機分割區

## 發展歷程

| 時間 | 里程碑 |
|------|--------|
| **2019 年初** | puhitaku 發現 U-Boot 與 Linux 可能在 Brain 上執行 |
| **2019 年 8 月** | U-Boot 成功引導 |
| **2020 年 3 月** | Linux 核心成功啟動 |
| **2020 年** | Brain Hackers 社群成立 |
| **2022 年** | 首批 Brainux 正式版本；BrainLILO 用於 App Menu 開機 |
| **2023 年** | 鍵盤驅動改善；brain-config 工具 |
| **2024 年** | PW-A7400 支援；電源關閉修復；SPIDEV 啟用 |
| **2026 年** | Linux 6.1、Debian 13 "Trixie"、USB Ethernet NCM |

## 相關連結

- 官方網站：https://brainux.org
- 官方 Wiki：https://wiki.brainux.org/
- GitHub 組織：https://github.com/brain-hackers
- Discord：Brain Hackers 社群（透過官方網站取得邀請）

[^1]: Brain Hackers. (n.d.). Brainux — SHARP Brain 専用 Linux ディストリビューション. Retrieved 2026-10-01, from https://brainux.org
[^2]: Brain Hackers. (n.d.). About Brainux. Retrieved 2026-10-01, from https://brainux.org/about/
[^3]: brain-hackers. (n.d.). brain-hackers/README. Retrieved 2026-10-01, from https://github.com/brain-hackers/README
[^4]: Brain Hackers. (n.d.). brain-config — Brainux Wiki. Retrieved 2026-10-01, from https://wiki.brainux.org/linux/brain-config/
[^5]: Brain Hackers. (n.d.). Get Started — Brainux Wiki. Retrieved 2026-10-01, from https://wiki.brainux.org/beginners/get-started/
[^6]: brain-hackers. (2026-03-25). buildbrain Releases. Retrieved 2026-10-01, from https://github.com/brain-hackers/buildbrain/releases