# Linux 上如何取得磁碟／磁區的 UUID

## 摘要

在 Linux 上，「UUID」不是單一概念。當使用者問「磁碟的 UUID」時，實際上可能指的是三種不同的識別碼：**檔案系統 UUID（Filesystem UUID）**、**磁區 UUID（PARTUUID）**，以及**磁碟硬體序號（Disk Serial）**。三者產生的時機、儲存位置與生命週期都不同。本文說明如何在 Linux 取得這些識別碼，並比較其差異與適用情境。

---

## 一、三種識別碼的差異

在動手查詢之前，必須先釐清要取得的是哪一種 UUID，因為不同指令回傳的欄位意義不同。

| 識別碼 | 作用範圍 | 何時改變 | 由誰產生 | 範例 |
|---|---|---|---|---|
| 檔案系統 UUID | 磁區**內部**的檔案系統 | 每次 `mkfs` 重新格式化都會改變 | 格式化工具（`mkfs.*`） | `0a3407de-014b-458b-b5c1-848e92a327a3` |
| PARTUUID | 分割表中的**磁區項目** | 重新格式化不變；改寫分割表或磁區編號變動才會變 | 分割工具（`gdisk`、`parted`） | `d0d0d110-0a71-4ed6-936a-304969ea36af`（GPT） |
| 磁碟序號（by-id） | **實體磁碟硬體**本身 | 裝置存在期間固定不變 | 硬體製造商 | `ata-WDC_WD2500BEVT-22ZCT0_WD-WXE908VF0470` |

關鍵區別如下[^persist]：

1. **檔案系統 UUID 每次重新格式化都會變動**；PARTUUID 則存放在分割表而非檔案系統內，因此重新格式化不會改變。
2. **PARTUUID 屬於「磁區」而非「檔案系統」**。在 GPT 上，若刪除後以相同邊界重建磁區，通常會得到新的 PARTUUID（因為 GPT 的每個磁區項目 UUID 是隨機生成）。
3. **MBR 的 PARTUUID 是「偽 UUID」**，格式為 `SSSSSSSS-PP`，其中 `SSSSSSSS` 是 MBR 磁碟簽章（補零），`PP` 是磁區編號（十六進位）。因此磁區編號改變時，這個偽 UUID 也會跟著變。
4. **磁碟序號（by-id）識別的是實體磁碟本身**，最為持久，但無法指出是哪個磁區或檔案系統。
5. **FAT／exFAT／NTFS 沒有真正的 UUID**，它們使用較短的 UID（例如 `CBB6-24F2`），同樣由格式化時產生，且仍會出現在 `/dev/disk/by-uuid/` 之下。

---

## 二、取得 UUID 的常用指令

### 1. `blkid` — 通用工具

`blkid` 是 `util-linux` 套件中操作 `libblkid` 函式庫的命令列介面，可判斷區塊裝置承載的內容類型（例如檔案系統或 swap），並讀取其詮釋資料（例如 LABEL 或 UUID 欄位）[^blkidman]。

```bash
# 列出所有裝置的完整屬性
blkid

# 以可直接解析的 export 格式輸出
blkid -o export

# 只取特定裝置的檔案系統 UUID
blkid -s UUID -o value /dev/sda1

# 只取 PARTUUID
blkid -s PARTUUID -o value /dev/sda1

# 反向查詢：由 UUID 找出對應裝置
blkid -U "0a3407de-014b-458b-b5c1-848e92a327a3"

# JSON 輸出
blkid -o json /dev/sda1
```

`blkid -o export` 的輸出範例[^persist]：

```
DEVNAME=/dev/sda1
UUID="CBB6-24F2"
TYPE="vfat"
PARTLABEL="EFI system partition"
PARTUUID="d0d0d110-0a71-4ed6-936a-304969ea36af"

DEVNAME=/dev/sda2
LABEL="Arch Linux"
UUID="0a3407de-014b-458b-b5c1-848e92a327a3"
TYPE="ext4"
PARTLABEL="GNU/Linux"
PARTUUID="98a81274-10f7-40db-872a-03df048df366"
```

> **注意**：以一般使用者身分執行 `blkid` 時，可能取得快取中的舊資料。`blkid` 直接讀取裝置，對非特權使用者它回傳的是未經驗證的快取資訊；官方手冊建議多數情境改用 `lsblk`[^blkidman]。

### 2. `lsblk -f` — 樹狀、人類可讀（官方推薦）

`lsblk` 讀取 sysfs 與 udev 資料庫來收集資訊；`blkid` 的官方手冊明確建議：使用 `lsblk` 取得區塊裝置資訊、或 `lsblk --fs` 取得檔案系統概觀[^blkidman][^lsblkman]。

```bash
# 檔案系統概觀（含 UUID、LABEL、FSTYPE、MOUNTPOINTS）
lsblk --fs

# 等同於
lsblk -o NAME,FSTYPE,FSVER,LABEL,UUID,FSAVAIL,FSUSE%,MOUNTPOINTS

# 只取單一欄位
lsblk -dno UUID /dev/sda1        # 檔案系統 UUID
lsblk -dno PARTUUID /dev/sda1    # 磁區 UUID
lsblk -dno LABEL /dev/sda2       # 標籤

# JSON 輸出（適合自動化）
lsblk -J -o NAME,UUID,FSTYPE /dev/sda1
```

輸出範例[^persist]：

```
NAME        FSTYPE LABEL      UUID                                 MOUNTPOINT
sda
├─sda1      vfat              CBB6-24F2                            /boot
├─sda2      ext4   Arch Linux 0a3407de-014b-458b-b5c1-848e92a327a3 /
├─sda3      ext4   Data       b411dc99-f0a0-4c87-9e05-184977be8539 /home
└─sda4      swap              f9fe0b69-a280-415d-a03a-a32752370dee [SWAP]
```

> **注意**：`lsblk` 只顯示檔案系統 UUID（`UUID` 欄）與 PARTUUID，**不顯示磁碟序號**。`lsblk` 的 `UUID` 欄位在沒有 udev 資料庫或未以 udev 支援編譯時，會改由直接讀取裝置取得，此時需要 root 權限[^lsblkman]。

### 3. `/dev/disk/by-uuid/` — udev 建立的符號連結

```bash
ls -l /dev/disk/by-uuid/
```

輸出範例[^persist]：

```
lrwxrwxrwx 1 root root 10  0a3407de-014b-458b-b5c1-848e92a327a3 -> ../../sda2
lrwxrwxrwx 1 root root 10  CBB6-24F2 -> ../../sda1
```

`/dev/disk/` 下相關目錄的用途[^persist]：

| 目錄 | 內容 |
|---|---|
| `/dev/disk/by-uuid/` | 檔案系統 UUID |
| `/dev/disk/by-partuuid/` | 磁區 UUID（GPT 為真 UUID；MBR 為偽 UUID） |
| `/dev/disk/by-id/` | 硬體序號、WWN |
| `/dev/disk/by-path/` | 實體 PCI／連接埠路徑 |
| `/dev/disk/by-label/` | 檔案系統標籤 |
| `/dev/disk/by-partlabel/` | GPT 磁區標籤 |

> **注意**：這些符號連結由 udev 動態建立，裝置未連接或沒有檔案系統時可能不存在。

### 4. `udevadm` — 底層裝置資訊

`udevadm info` 可查詢 udev 資料庫中的裝置屬性。`--query=property` 會輸出所有 `KEY=value` 屬性（含 `ID_FS_UUID`），`--property` 可限制只輸出特定屬性，`--value` 則只印出值[^udevadmman]。

```bash
# 查詢某裝置的所有屬性
udevadm info --query=all --name=sda1

# 上列指令等價於
udevadm info /dev/sda1

# 只取檔案系統 UUID，且只印值
udevadm info --query=property --name=sda1 --property=ID_FS_UUID --value

# 以 KEY='value' 形式匯出
udevadm info --query=property --name=sda1 --export
```

> **注意**：`udevadm` 顯示的是原始 udev 屬性，較冗長、對使用者較不友善，但適合除錯與撰寫 udev 規則。

### 5. `tune2fs` / `dumpe2fs` — 僅限 ext2/3/4

`tune2fs` 用於調整 **ext2/ext3/ext4** 檔案系統的參數，其中 `-l` 選項會列出檔案系統 superblock 內容（含 UUID 等目前值）[^tune2fsman]。

```bash
# 列出 superblock，其中包含 UUID
tune2fs -l /dev/sda2 | grep UUID

# dumpe2fs 完整傾印（輸出極多，建議搭配 grep）
dumpe2fs -h /dev/sda2 | grep UUID
```

`tune2fs -l` 是唯讀的（只列出），相對地 `tune2fs -U UUID` 則是**設定** UUID，其參數也可為特殊關鍵字 `clear`（清除）、`random`（隨機產生）、`time`（時間基底 UUID）[^tune2fsman]。

> **注意**：這兩者只適用於 ext2/3/4，無法用於 btrfs、xfs、vfat、swap 等檔案系統。

---

## 三、方法比較

| 面向 | `blkid` | `lsblk` |
|---|---|---|
| 主要用途 | 底層區塊裝置屬性探測 | 列出區塊裝置、輸出人類可讀 |
| 需要 root？ | 是（非特權僅得快取資料） | 否（讀 sysfs／udev） |
| 快取 | 使用 `/run/blkid/blkid.tab` | 直接讀 sysfs／udev |
| 適合腳本 | 是（`export`、`value` 格式） | 高度推薦（`--pairs`、`--json`、`--raw`、`--noheadings`） |
| 樹狀檢視 | 否 | 是（預設） |
| 由 UUID 反查裝置 | `blkid -U <uuid>` | 無直接對應，需自行過濾 |
| 官方是否推薦 | 否，建議改用 `lsblk` | 是 |

（整理自 `blkid(8)` 與 `lsblk(8)` 手冊[^blkidman][^lsblkman]）

---

## 四、使用情境對照

| 想要識別 | 建議寫法 |
|---|---|
| 不隨分割表變動的檔案系統 | `UUID=...`（重新格式化後會變） |
| 不隨檔案系統重新格式化而變的磁區 | `PARTUUID=...`（僅 GPT 為真 UUID） |
| 不隨連接埠變動的實體磁碟 | `/dev/disk/by-id/...` 或 `wwn-...` |
| 以人類可讀標籤識別 | `LABEL=...`（須唯一，最長 16 字元） |
| 以 GPT 磁區標籤識別 | `PARTLABEL=...`（僅 GPT，最長 72 字元） |

（整理自 Arch Linux Wiki 的持久化區塊裝置命名頁面[^persist]）

---

## 五、常見陷阱

1. **磁碟複製（clone）** 會產生兩顆 UUID 相同的磁碟，同系統上同時掛載會衝突。
2. **未以 root 執行 `blkid`** 可能顯示過期快取資料。
3. **NVMe 的 by-id 連結**若不含 namespace 識別碼（NSID）已屬過時，建議優先使用帶 `_1` 後綴的版本[^persist]。
4. **`by-path` 連結**在硬體移到不同 PCI 埠／連接器時會改變。
5. **MBR 的 PARTUUID** 是偽 UUID，分割表改寫或磁區編號位移時會改變。
6. **`tune2fs`／`dumpe2fs`** 僅支援 ext2/3/4，對其他檔案系統無效。
7. **加密卷（LUKS）**：容器內的檔案系統 UUID 只有在解鎖容器後才可見[^persist]。

---

## 結論

若只是要快速查看磁碟 UUID，`lsblk --fs` 是最單純且官方推薦的方式[^blkidman]；需要腳本化或由 UUID 反查裝置時，`blkid -s UUID -o value` 與 `blkid -U` 更為便利；而在 `/etc/fstab` 中選擇 `UUID=`、`PARTUUID=` 或 `/dev/disk/by-id/`，則取決於希望識別碼在「重新格式化」「重建分割表」「更換連接埠」等情況下是否保持穩定。

---

[^persist]: Arch Linux. (n.d.). Persistent block device naming. Retrieved 2026-10-24, from https://wiki.archlinux.org/title/Persistent_block_device_naming
[^blkidman]: util-linux. (n.d.). blkid(8) - Linux manual page. Retrieved 2026-10-24, from https://man7.org/linux/man-pages/man8/blkid.8.html
[^lsblkman]: util-linux. (n.d.). lsblk(8) - Linux manual page. Retrieved 2026-10-24, from https://man7.org/linux/man-pages/man8/lsblk.8.html
[^tune2fsman]: e2fsprogs. (n.d.). tune2fs(8) - Linux manual page. Retrieved 2026-10-24, from https://man7.org/linux/man-pages/man8/tune2fs.8.html
[^udevadmman]: systemd. (n.d.). udevadm(8) - Linux manual page. Retrieved 2026-10-24, from https://man7.org/linux/man-pages/man8/udevadm.8.html
