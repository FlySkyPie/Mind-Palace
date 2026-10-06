# MergerFS 基於副本冗餘之建構方法

## 概述

MergerFS 為一 **union filesystem**（JBOFS — Just a Bunch of FileSystems），將多個磁碟路徑（branch）合併為單一掛載點。官方文件明確將「RAID 式冗餘」列為非功能[^mergerfs-docs]。**檔案不經分割、條帶化或即時鏡像**，每個檔案完整存放在單一分支上。

因此，基於副本的冗餘（即 RAID1 式的檔案層級鏡像）**必須在 MergerFS 之外**透過額外工具實現。

---

## 方法一：`mergerfs.dup`（官方工具，離線批次）

`mergerfs-tools` 套件提供 `mergerfs.dup` 腳本，為官方支援的副本冗餘工具[^mergerfs-tools]。

### 原理

- 走訪 MergerFS 掛載點目錄樹
- 透過 xattr（`user.mergerfs.basepath`、`user.mergerfs.allpaths`、`user.mergerfs.relpath`）判斷檔案分佈
- 選取一份「來源」檔案，`rsync` 至其他分支
- 優先複製至剩餘空間最大的分支
- 支援 `--prune` 移除超出指定份數的副本

### 基本用法

```bash
# 維護 2 份複本，選擇最新的檔案為來源，真正執行 rsync
mergerfs.dup -d newest -c 2 -e /mnt/pool/data

# 加入包含/排除過濾
mergerfs.dup -c 2 -d newest -I '*.mkv' -I '*.mp4' -e /mnt/pool/data

# 排除特定目錄
mergerfs.dup -c 2 -d newest -E '/temp/*' -e /mnt/pool
```

### 參數說明

| 參數 | 說明 |
|------|------|
| `-c, --count=N` | 目標副本數量（預設 2） |
| `-d, --dup=` | 多份副本時選取來源的策略：`newest`、`oldest`、`smallest`、`largest`、`mergerfs` |
| `-p, --prune` | 移除超出 `--count` 的過多副本 |
| `-e, --execute` | 實際執行 rsync/rm（預設為乾燥模式） |
| `-I, --include=` | fnmatch 包含過濾（可重覆） |
| `-E, --exclude=` | fnmatch 排除過濾（可重覆） |

### 排程方式（cron / systemd timer）

```bash
# cron — 每日凌晨 3 點
0 3 * * * /usr/local/bin/mergerfs.dup -d newest -c 2 -e /mnt/storage
```

systemd timer 版本：

```systemd
# /etc/systemd/system/mergerfs-dup.service
[Unit]
Description=Duplicate files across mergerfs pool branches
Requires=mergerfs.service
After=mergerfs.service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/mergerfs.dup -c 2 -d newest -e /mnt/pool
```

```systemd
# /etc/systemd/system/mergerfs-dup.timer
[Unit]
Description=Run mergerfs.dup nightly

[Timer]
OnCalendar=daily
Persistent=true
RandomizedDelaySec=30min

[Install]
WantedBy=timers.target
```

啟用：

```bash
systemctl daemon-reload
systemctl enable --now mergerfs-dup.timer
```

### 已知限制

- **非即時**：排程間隔內有資料視窗（如 cron daily 則最多遺失 24 小時變更）
- **掃描耗時**：GitHub issue #140 回報 273GB / 147,148 個檔案需 ~45 分鐘，即使無需複製亦然，因 rsync 對每個檔案執行 mtime 比對[^gh-issue-140]
- **非交易性**：無原子多分支寫入保證

---

## 方法二：跨分支 rsync 排程（DIY 最簡方式）

直接對 MergerFS 底層分支執行 rsync，繞過 MergerFS 層以減少開銷。

### 基本腳本

```bash
#!/bin/bash
# 將 diskA 的資料完整鏡像至 diskB
rsync -avxhHAXWE --numeric-ids --info=progress2 /mnt/diskA/ /mnt/diskB/
```

### 多分支鏡像

```bash
#!/bin/bash
SRC="/mnt/diskA"
DESTINATIONS=("/mnt/diskB" "/mnt/diskC" "/mnt/diskD")

for dest in "${DESTINATIONS[@]}"; do
    rsync -avxhHAXWE --numeric-ids "$SRC/" "$dest/"
done
```

### ⚠️ 關鍵效能注意

**不要使用 `-S`（`--sparse`）旗標。** MergerFS 維護者 trapexit 指出 `-S` 使 rsync 寫入 1KB 區塊而非 256KB，吞吐量從 ~150MB/s 暴跌至 <5MB/s[^gh-rsync-sparse]。

---

## 方法三：LSyncd（近即時事件驅動）

LSyncd 使用 `inotify` 監控目錄變更，累積事件數秒後喚起 rsync 同步[^lsyncd]。

### 設定範例

```lua
-- /etc/lsyncd/lsyncd.conf.lua
settings {
    logfile = "/var/log/lsyncd/lsyncd.log",
    statusFile = "/var/log/lsyncd/lsyncd.status",
    statusInterval = 20,
}

sync {
    default.rsync,
    source = "/mnt/diskA/data",
    target = "/mnt/diskB/data",
    delay = 5,
    rsync = {
        archive = true,
        compress = false,
        whole_file = false,
    }
}
```

### 適用場景

- 需要較短復原點（RPO）的目錄（如資料庫匯出、即時產生的檔案）
- 非大量隨機寫入的工作負載（inotify 可能在高事件率下丟失）

### 限制

- 對 MergerFS 掛載點直接監控可行（MergerFS 支援 inotify），但官方建議偏好直接操作底層分支[^mergerfs-inotify]
- 不處理衝突——最後寫入者取勝

---

## 方法四：Unison（雙向同步）

Unison 支援兩條分支的雙向同步，具備衝突偵測與處理機制。

```bash
unison /mnt/diskA/ /mnt/diskB/ -auto -batch
```

適合需要雙向同步且重視衝突處理的場景，但非即時，需定時調用。

---

## 方法五：合併策略（選購性保護）

許多使用者將 SnapRAID（奇偶校驗）與 mergerfs.dup（選購性副本）結合[^reddit-selective]：

- SnapRAID 為大量媒體檔案提供空間高效的奇偶保護
- 對關鍵目錄（文件、相片、設定檔）額外以 `mergerfs.dup -c 2 -e /mnt/pool/docs` 維持多份實體副本

亦可在寫入原則上選用 `category.create=epmfs` 使相關目錄留在同磁碟，減少跨磁碟碎片。

---

## 方法比較

| 方法 | 即時性 | 複雜度 | 掃描負擔 | 適合場景 |
|------|--------|--------|----------|----------|
| `mergerfs.dup` + cron | 排程（離線） | 低 | 高（完整掃描） | 週期性批次副本維護 |
| rsync 跨分支排程 | 排程（離線） | 低 | 中（差異傳輸） | 簡單雙向鏡像 |
| LSyncd | 近即時（秒級） | 中 | 低（事件驅動） | 活躍目錄的近即時複製 |
| Unison | 排程（離線） | 中 | 中（雙向差異） | 雙向同步需求 |
| SnapRAID（對照組） | 排程（離線） | 中 | 低（奇偶校驗） | 空間高效的錯誤保護 |

---

## 技術限制與注意事項

1. **無原子多分支寫入**：所有外部複製方式都存在時間視窗，其中寫入存在於一個分支但尚未擴散。若在此視窗內發生磁碟故障，可能遺失未複製的資料。
2. **MergerFS 不追蹤副本狀態**：`mergerfs.dup` 依賴 xattr 與每次掃描來發現現有副本，無持續的資料庫記錄哪些檔案已鏡像。刪除或移動檔案時需注意手動清理孤立副本。
3. **`epall` ≠ 副本**：MergerFS 的 `epall` 建立原則對 `mkdir`/`mknod`/`symlink` 套用至所有分支，但對檔案的 `create` 操作仍僅寫入一個分支，並非真正鏡像[^mergerfs-policies]。
4. **檔案鎖定**：若應用程式在 rsync 執行期間持有寫入鎖，可能導致不一致狀態。部分自訂腳本使用 `lsof`/`fuser` 跳過開啟檔案（如 `mover` 腳本的 `--skip-open` 功能[^mover-script]）。

---

## 參考資料

[^mergerfs-docs]: trapexit. (n.d.). mergerfs — A union filesystem. Retrieved 2026-10-03, from https://github.com/trapexit/mergerfs

[^mergerfs-tools]: trapexit. (n.d.). mergerfs-tools. Retrieved 2026-10-03, from https://github.com/trapexit/mergerfs-tools

[^mergerfs-policies]: trapexit. (n.d.). mergerfs — Functions, Categories & Policies. Retrieved 2026-10-03, from https://trapexit.github.io/mergerfs/latest/config/functions_categories_policies/

[^mergerfs-inotify]: trapexit. (n.d.). mergerfs — FAQ: Compatibility and Integration (inotify). Retrieved 2026-10-03, from https://trapexit.github.io/mergerfs/latest/faq/compatibility_and_integration/

[^gh-issue-140]: GitHub. (2023). mergerfs-tools Issue #140 — mergerfs.dup performance enhancement. Retrieved 2026-10-03, from https://github.com/trapexit/mergerfs-tools/issues/140

[^gh-rsync-sparse]: GitHub. (2023). mergerfs Discussion #1287 — Best MergerFS Options for Rsync. Retrieved 2026-10-03, from https://github.com/trapexit/mergerfs/discussions/1287

[^lsyncd]: LSyncd. (n.d.). LSyncd — Live Syncing Daemon. Retrieved 2026-10-03, from https://github.com/lsyncd/lsyncd

[^reddit-selective]: Reddit r/DataHoarder. (n.d.). Configuring mergerfs to keep copies of some files? Retrieved 2026-10-03, from https://www.reddit.com/r/DataHoarder/comments/w98dd9/configuring_mergerfs_to_keep_copies_of_some_files/

[^mover-script]: thiscantbeserious. (n.d.). mover — Bidirectional sync and retention-mode move for mergerfs. Retrieved 2026-10-03, from https://gist.github.com/thiscantbeserious/4400a7dc0fdc71ef8f94caf1415d2fb9