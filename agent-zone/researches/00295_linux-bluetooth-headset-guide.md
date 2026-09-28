# Linux PC 使用藍芽耳機須知

## 前言

在 Linux PC 上使用藍芽設備（尤其是耳機）時，有許多與 Windows/macOS 不同的技術細節需要注意。本文涵蓋 2026 年主流藍芽音訊技術現狀、常見問題與解決方案。

---

## 1. Linux 藍芽音訊架構

Linux 的藍芽音訊由三層組成：

```
藍芽耳機 ↔ BlueZ（藍芽協定棧）↔ PipeWire / PulseAudio（音訊伺服器）↔ ALSA（核心音訊驅動）
```

| 層級 | 角色 | 說明 |
|------|------|------|
| **BlueZ** | 藍芽協定棧 | 核心層的 `bluetoothd` 負責配對、掃描、連線管理 |
| **PipeWire（建議）** | 音訊伺服器 | 2024 年起主流發行版預設採用，原生支援藍芽音訊 |
| **PulseAudio（舊版）** | 音訊伺服器 | 舊發行版使用，需額外套件才能支援藍芽 |
| **ALSA** | 核心音訊驅動 | 最底層，一般使用者無需直接操作 |

**2026 年建議：使用 PipeWire。** Fedora 34+、Ubuntu 22.10+、Debian 12+、Arch Linux 等均預設採用 PipeWire。[^archwiki]

檢查當前使用的音訊伺服器：

```bash
pactl info | grep "Server Name"
# 看到 "PulseAudio (on PipeWire)" → 已在用 PipeWire
# 看到 "PulseAudio" → 仍在使用舊版 PulseAudio
```

---

## 2. 藍芽音訊 Profile 的關鍵限制

這是最重要也最容易被誤解的一點。

| Profile | 用途 | 音質 | 方向 |
|---------|------|------|------|
| **A2DP** | 高品質立體聲播放 | 優秀（最高 990 kbps） | 單向：裝置 → 耳機 |
| **HSP** | 基本通話 | 低（8 kHz 單聲道） | 雙向 |
| **HFP** | 現代通話 | 中等（16 kHz 寬頻） | 雙向 |
| **LE Audio / BAP** | 新一代低功耗音訊 | 可調（LC3 編碼） | 彈性 |

**核心限制**：藍芽通訊協定無法同時提供高品質立體聲輸出與麥克風輸入。聽音樂時走 A2DP（高音質純輸出），一旦應用程式開啟麥克風（如 Discord、Zoom），系統會自動切換至 HFP/HSP（低音質單聲道）。**這不是 Linux 的 Bug，而是藍芽協定本身的頻寬限制**，在 Windows/macOS 上也一樣。[^bigiron]

---

## 3. 音訊編碼（Codec）比較

| 編碼 | 最高位元率 | 音質 | 2026 年 Linux 支援 |
|------|-----------|------|-------------------|
| **SBC** | ~328 kbps | 普通（標準強制） | PipeWire/PulseAudio 均支援 |
| **SBC-XQ** | ~452 kbps | 良好 | PipeWire 內建（預設之一） |
| **AAC** | ~250 kbps | 良好（Apple 生態系） | PipeWire 內建 |
| **aptX** | ~352 kbps | 優秀（Qualcomm） | PipeWire 內建 |
| **aptX HD** | ~576 kbps | 優秀 | PipeWire 內建 |
| **LDAC** | 990 kbps | 優秀（Sony） | PipeWire 內建 |
| **LC3**（LE Audio）| 可調 | 優秀 + 低功耗 | 實驗性，預設關閉 |

PipeWire 已內建所有主流編碼支援，無需額外安裝。[^fosslinux] 若仍使用 PulseAudio 則需安裝 `libldac`、`libfreeaptx0` 等套件。

---

## 4. 配對流程（建議步驟）

使用 `bluetoothctl` 命令列工具：

```bash
bluetoothctl
power on
agent on
default-agent
scan on
# 等待裝置出現，記下 MAC 位址
pair AA:BB:CC:DD:EE:FF
trust AA:BB:CC:DD:EE:FF    # ← 很多人跳過這步
connect AA:BB:CC:DD:EE:FF
scan off
exit
```

**`trust` 是關鍵** — 沒有這步，耳機不會在下次自動連線。[^archwiki]

---

## 5. 常見問題與排除

### 5.1 連線後無聲音

最常見原因：系統卡在 HFP/HSP profile。手動切回 A2DP：

```bash
pactl list cards short                  # 找到藍芽卡的索引號
pactl set-card-profile <索引> a2dp_sink  # 切換到高音質模式
```

### 5.2 音訊斷斷續續、爆音

| 原因 | 解決方案 |
|------|---------|
| 2.4 GHz Wi-Fi 干擾（藍芽使用同頻段） | 將 Wi-Fi 切換至 5 GHz |
| USB 3.0 雜訊干擾 | 將藍芽接收器移至 USB 2.0 埠或使用延長線 |
| 高 bitrate 編碼在干擾環境下不穩 | 換用更穩定的編碼（如 SBC-XQ） |
| 藍芽 USB 自動暫停（最常見原因） | 停用 USB autosuspend |

停用藍芽 USB autosuspend：

```bash
echo 'options btusb enable_autosuspend=0' | sudo tee /etc/modprobe.d/btusb.conf
# 重新啟動或重新載入 btusb 模組
```

這能解決約 60% 的隨機斷線問題。[^fosslinux_troubleshoot]

### 5.3 麥克風相關

- 用 Discord/Zoom 等通話軟體時，系統會自動從 A2DP 切到 HFP，音質大幅下降
- **最佳解決方案**：使用筆電內建麥克風或獨立 USB 麥克風，耳機保持在 A2DP 模式純輸出
- 若仍想用耳機麥克風，可停用自動切換：

```bash
wpctl settings --save bluetooth.autoswitch-to-headset-profile false
```

或建立設定檔：

```ini
# ~/.config/wireplumber/wireplumber.conf.d/80-disable-autoswitch.conf
wireplumber.settings = {
  bluetooth.autoswitch-to-headset-profile = false
}
```

### 5.4 配對成功但沒出現音訊裝置

檢查順序：
1. PipeWire 是否安裝且正常運行
2. 是否同時有 PulseAudio 在運行（兩者衝突）
3. 裝置是否以非音訊 profile 連線

### 5.5 從睡眠/休眠喚醒後沒聲音

```bash
systemctl --user restart pipewire wireplumber
```

### 5.6 音量最小聲還是太大

停用硬體音量控制：

```ini
# /etc/wireplumber/wireplumber.conf.d/80-bluez-properties.conf
monitor.bluez.properties = {
  bluez5.enable-hw-volume = false
}
```

### 5.7 雙系統（Dual-boot）配對失效

Windows 和 Linux 使用同一個藍芽 Dongle 的 MAC 位址時，配對金鑰會被覆蓋。解決方案：每次切換系統時重新配對。[^bigiron]

### 5.8 連上了但完全沒聲音（DAC/喇叭）

某些裝置需要 AVRCP 回報「Playing」狀態才會取消靜音：

```ini
# ~/.config/wireplumber/wireplumber.conf.d/60-bluez-dummy-avrcp.conf
monitor.bluez.properties = {
  bluez5.dummy-avrcp-player = true
}
```

---

## 6. 電池電量檢視

啟用 BlueZ 實驗功能（否則無法取得電量）：

```ini
# /etc/bluetooth/main.conf
Experimental = true
```

重啟藍芽服務後可用以下方式檢視：

```bash
bluetoothctl info AA:BB:CC:DD:EE:FF    # 尋找 "Battery Percentage"
upower -e                               # 列出電源裝置
upower -i /org/freedesktop/UPower/devices/headset_dev_XX_XX_XX_XX_XX_XX
```

GNOME 桌面環境（Ubuntu 24.04+）可在「設定 → 電源」中直接看到藍芽裝置電量。[^itsfoss]

---

## 7. LE Audio 與 LC3 — 2026 年現狀

LE Audio（藍芽 5.2+）是新一代音訊標準，潛在優勢：
- 延遲 < 20ms
- 比 LDAC/aptX HD 節省約 40% CPU 資源
- Auracast™ 廣播功能
- 更好的續航力

**2026 年 Linux 支援狀態**：
- BlueZ + PipeWire 已實作 LE Audio
- LC3 預設關閉（仍屬實驗性）
- 需 Linux kernel 6.4+、新版 BlueZ、新版 PipeWire
- 啟用 `Experimental = true` 後尚無法同時使用 Classic BT 與 LE Audio，需斷線重連來切換
- 還不建議日常使用[^collabora]

---

## 8. 最佳實踐總結

1. **使用 PipeWire** — 現代發行版預設，對藍芽支援遠優於 PulseAudio
2. **配對流程**：`pair` → `trust` → `connect`，缺一不可
3. **聽音樂時確認在 A2DP mode**，若聲音品質很差，先檢查是否在 HFP
4. **停用自動切換到 HFP**（除非你真的需要用耳機麥克風）
5. **用 5 GHz Wi-Fi** 避免與藍芽的 2.4 GHz 干擾
6. **停用 USB autosuspend** 解決斷線問題
7. **喚醒後重啟服務**：`systemctl --user restart pipewire wireplumber`
8. **不必執著編碼** — SBC-XQ 對多數人已足夠好，且更穩定
9. **通話時用獨立麥克風**是最簡單的解法
10. **LE Audio 值得關注**但還不成熟

---

## 參考資料

[^archwiki]: Arch Linux Wiki. (n.d.). Bluetooth headset. Retrieved 2026-09-26, from https://wiki.archlinux.org/title/Bluetooth_headset

[^bigiron]: Big Iron. (2026). Bluetooth on Linux — BlueZ Pairing Pain and the Headset Codec Truth. Retrieved 2026-09-26, from https://www.bigiron.cc/guides/bluetooth-on-linux-bluez-pairing-pain-and-headset-codec-truth

[^fosslinux]: FOSS Linux. (2026). PipeWire 2.0 Pro-Audio Guide including LE Audio. Retrieved 2026-09-26, from https://www.fosslinux.com/156774/pipewire-2-0-pro-audio-guide-low-latency-music-production-and-bluetooth-le-audio-on-linux.htm

[^fosslinux_troubleshoot]: FOSS Linux. (2026). Linux Bluetooth Setup and Troubleshooting: Complete Guide (BlueZ 5.85, PipeWire 1.6.2). Retrieved 2026-09-26, from https://www.fosslinux.com/158534/linux-bluetooth-setup-troubleshooting.htm

[^itsfoss]: It's FOSS. (2024). How to Check Bluetooth Device Battery in Ubuntu. Retrieved 2026-09-26, from https://itsfoss.com/ubuntu-bluetooth-battery-status/

[^collabora]: Collabora. (2025). Implementing Bluetooth LE Audio & Auracast on Linux Systems. Retrieved 2026-09-26, from https://www.collabora.com/news-and-blog/blog/2025/11/24/implementing-bluetooth-le-audio-and-auracast-on-linux-systems/