# Linux 筆記型電腦硬體資訊完整傾印方法

本文件說明在 Linux 系統上完整擷取所有硬體資訊的命令與工具，適合用於診斷、回報問題、或是記錄裝置規格。

## 總覽工具

以下為涵蓋多種硬體範疇的全方位工具：

- **`inxi`** — 系統、CPU、GPU、音訊、網路、儲存、感應器、電池等，是論壇診斷的黃金標準。命令：`inxi -Fxxxz`
- **`hwinfo`** — 幾乎所有硬體的超集合。命令：`hwinfo --short` 或 `hwinfo`
- **`lshw`** — CPU、記憶體、磁碟、網路、圖形。支援 JSON/XML/HTML 輸出。命令：`sudo lshw` 或 `sudo lshw -json`
- **`HardInfo`** / **`KInfoCenter`** — GUI 硬體瀏覽工具

[^inxi]: inxi — Linux 硬體資訊工具. Retrieved 2026-10-03, from https://github.com/smxi/inxi
[^lshw]: lshw — HardWare Lister. Retrieved 2026-10-03, from https://ezix.org/project/hardware/lshw

## 分類工具詳細說明

### CPU / 處理器

| 工具 | 顯示內容 | 命令 |
|------|---------|------|
| `lscpu` | 架構、廠商、型號、核心數、執行緒、快取、旗標 | `lscpu` |
| `lshw -C cpu` | 詳細 CPU 樹狀資訊 | `sudo lshw -C cpu` |
| `cat /proc/cpuinfo` | 核心回報的原始 CPU 資料（每核心） | `cat /proc/cpuinfo` |
| `cpuid` | 低階 CPUID 指令識別 | `cpuid` |
| `dmidecode -t processor` | SMBIOS/DMI 處理器資訊（插槽類型、升級） | `sudo dmidecode -t processor` |

[^lscpu]: GNU Core Utilities — lscpu. Retrieved 2026-10-03, from https://man7.org/linux/man-pages/man1/lscpu.1.html

### 記憶體 (RAM)

- `sudo dmidecode -t memory` — 每條記憶體的容量、類型（DDR4/DDR5）、速度、製造商、料號、序號
- `sudo lshw -C memory` — 記憶體樹狀資訊
- `free -h` — 已用/可用/總計記憶體
- `lsmem` — 記憶體區間列表

### 儲存裝置

| 工具 | 顯示內容 | 命令 |
|------|---------|------|
| `lsblk` | 所有區塊裝置、分割區、掛載點、檔案系統 | `lsblk -a` 或 `lsblk -f` |
| `fdisk -l` | 詳細分割表 | `sudo fdisk -l` |
| `lshw -C disk` | 磁碟型號、供應商、序號 | `sudo lshw -C disk` |
| `blkid` | UUID、檔案系統類型、LABEL | `blkid` |
| `smartctl -a /dev/sda` | SMART 屬性、健康狀態、溫度、使用時數 | `sudo smartctl -a /dev/sda`[^smartctl] |
| `nvme list` | NVMe 磁碟列表 | `sudo nvme list`[^nvme] |

[^smartctl]: smartctl — SMART Control. Retrieved 2026-10-03, from https://www.smartmontools.org
[^nvme]: NVMe CLI. Retrieved 2026-10-03, from https://github.com/linux-nvme/nvme-cli

### PCI 與 USB

- `lspci -vvv` — 所有 PCI/PCIe 裝置的完整細節
- `lspci -t` — PCI 裝置樹狀拓撲
- `lsusb -v` — 所有 USB 裝置的詳細資訊

### 網路

- `sudo lshw -C network` — 網路卡型號、驅動、韌體
- `ip addr` — 介面與 IP 位址
- `ip link show` — 所有網路介面與 MAC 位址
- `iw dev` — 無線介面與能力

### BIOS / UEFI / 韌體

- `sudo dmidecode -t bios` — BIOS 版本、日期、UEFI 支援
- `sudo dmidecode -t system` — 製造商、產品名稱、序號、UUID
- `sudo dmidecode -t baseboard` — 主機板型號、晶片組
- `sudo dmidecode -t chassis` — 機殼類型（筆記型/桌上型/伺服器）

[^dmidecode]: dmidecode — DMI table decoder. Retrieved 2026-10-03, from https://www.nongnu.org/dmidecode/

### 圖形 (GPU)

- `sudo lshw -C display` — GPU 樹：驅動、解析度、供應商
- `lspci | grep -i vga` — GPU 在 PCI 匯流排上的型號
- `nvidia-smi`（NVIDIA 專用）— GPU 名稱、記憶體、驅動版本、溫度[^nvidia]

[^nvidia]: NVIDIA System Management Interface. Retrieved 2026-10-03, from https://developer.nvidia.com/cuda-downloads

### 音訊

- `sudo lshw -C audio` — 音訊裝置樹
- `aplay -l` — ALSA 音效裝置
- `pactl list` — PulseAudio 接收器/來源/音效卡

### 感應器與電池

- `sensors` — CPU/GPU 溫度、風扇速度、電壓[^sensors]
- `acpi -V` — 電池狀態、電量、變壓器、散熱
- `upower -i $(upower -e | grep BAT)` — 完整電池詳細資料[^upower]

[^sensors]: lm-sensors — Hardware monitoring. Retrieved 2026-10-03, from https://github.com/lm-sensors/lm-sensors
[^upower]: UPower — System power monitoring. Retrieved 2026-10-03, from https://upower.freedesktop.org/

### OS / 核心

- `uname -a` — 核心版本、架構、主機名稱
- `hostnamectl` — 作業系統名稱、核心、虛擬化
- `lsb_release -a` — 發行版資訊

## 完整傾印腳本

以下指令可一次傾印所有硬體資訊至目錄 `~/hardware-dump/`：

```bash
mkdir -p ~/hardware-dump && cd ~/hardware-dump

inxi -Fxxxz                                  2>&1 | tee inxi.txt
sudo hwinfo --short                          2>&1 | tee hwinfo-short.txt
sudo lshw -short                             2>&1 | tee lshw-short.txt
sudo lshw -json                              2>&1 | tee lshw.json   # 機器可解析

lscpu                                        2>&1 | tee lscpu.txt
sudo dmidecode -t processor                  2>&1 | tee dmidecode-processor.txt
sudo dmidecode -t memory                     2>&1 | tee dmidecode-memory.txt
sudo dmidecode -t bios -t system -t chassis  2>&1 | tee dmidecode-system.txt
sudo dmidecode                               2>&1 | tee dmidecode-full.txt

lsblk -a                                     2>&1 | tee lsblk.txt
sudo fdisk -l                                2>&1 | tee fdisk.txt
blkid                                        2>&1 | tee blkid.txt
df -h                                        2>&1 | tee df-h.txt
sudo smartctl -a /dev/nvme0n1                2>&1 | tee smart-nvme0.txt

lspci -vvv                                   2>&1 | tee lspci-vvv.txt
lsusb -v                                     2>&1 | tee lsusb.txt

ip addr                                      2>&1 | tee ip-addr.txt
sudo lshw -C network                         2>&1 | tee lshw-network.txt

sensors                                      2>&1 | tee sensors.txt
acpi -V                                      2>&1 | tee acpi.txt
upower -i $(upower -e | grep BAT)            2>&1 | tee battery.txt

uname -a                                     2>&1 | tee uname.txt
hostnamectl                                  2>&1 | tee hostnamectl.txt
```

也有一行指令版本：

```bash
sudo sh -c 'echo "=== INXI ==="; inxi -Fxxxz; echo; echo "=== HWINFO ==="; hwinfo --short; echo; echo "=== LSHW ==="; lshw -short; echo; echo "=== DMIDECODE ==="; dmidecode -t bios -t system -t memory; echo; echo "=== LSPCI ==="; lspci -vvv; echo; echo "=== LSUSB ==="; lsusb; echo; echo "=== LSBLK ==="; lsblk -a; echo; echo "=== LSCPU ==="; lscpu; echo; echo "=== SENSORS ==="; sensors; echo; echo "=== UNAME ==="; uname -a'
```

## 安裝指令

### Debian/Ubuntu
```bash
sudo apt install inxi hwinfo lshw dmidecode pciutils usbutils lm-sensors acpi smartmontools nvme-cli cpuid lsscsi hardinfo mesa-utils
```

### Fedora/RHEL
```bash
sudo dnf install inxi hwinfo lshw dmidecode pciutils usbutils lm_sensors acpi smartmontools nvme-cli cpuid lsscsi hardinfo mesa-utils
```

### Arch Linux
```bash
sudo pacman -S inxi hwinfo lshw dmidecode pciutils usbutils lm_sensors acpi smartmontools nvme-cli cpuid lsscsi hardinfo mesa-utils
```

## 總結

若要最完整地擷取 Linux 筆記型電腦的硬體資訊，建議同時使用 `inxi`（快速總覽）、`lshw`（結構化輸出）、`dmidecode`（韌體/SMBIOS 層級資訊）以及 `hwinfo`（最深層探測）。`lshw -json` 可產生機器可解析的 JSON 輸出，便於程式處理。

## 參考文獻

[^inxi]: inxi — Linux 硬體資訊工具. Retrieved 2026-10-03, from https://github.com/smxi/inxi
[^lshw]: lshw — HardWare Lister. Retrieved 2026-10-03, from https://ezix.org/project/hardware/lshw
[^lscpu]: GNU Core Utilities — lscpu. Retrieved 2026-10-03, from https://man7.org/linux/man-pages/man1/lscpu.1.html
[^smartctl]: smartctl — SMART Control. Retrieved 2026-10-03, from https://www.smartmontools.org
[^nvme]: NVMe CLI. Retrieved 2026-10-03, from https://github.com/linux-nvme/nvme-cli
[^dmidecode]: dmidecode — DMI table decoder. Retrieved 2026-10-03, from https://www.nongnu.org/dmidecode/
[^nvidia]: NVIDIA System Management Interface. Retrieved 2026-10-03, from https://developer.nvidia.com/cuda-downloads
[^sensors]: lm-sensors — Hardware monitoring. Retrieved 2026-10-03, from https://github.com/lm-sensors/lm-sensors
[^upower]: UPower — System power monitoring. Retrieved 2026-10-03, from https://upower.freedesktop.org/