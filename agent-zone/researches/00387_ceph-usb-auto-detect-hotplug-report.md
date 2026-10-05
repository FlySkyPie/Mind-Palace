# Ceph 的「自動偵測/自動格式化」與「熱插拔」能力對 USB 外接硬碟之影響

## 問題背景

分散式儲存系統 Ceph 在企業環境中通常使用內接 SATA/SAS 或 NVMe 硬碟作為 OSD (Object Storage Daemon) 的儲存介質。然而，許多人好奇是否能夠使用 USB 外接硬碟來低成本擴充 Ceph 叢集（例如搭配 Raspberry Pi 的 homelab 環境），以及 Ceph 原生的自動偵測、自動格式化、熱插拔能力是否對 USB 硬碟同樣有效。

---

## 1. Ceph 的 OSD 自動偵測與自動格式化機制

### 1.1 宣告式 OSD 服務規格（Declarative OSD Service Spec）

在 cephadm 管理模式下，Ceph 提供宣告式的 OSD 規格機制，允許管理員定義規則後，讓系統自動將符合條件的磁碟初始化為 OSD[^ceph-osd-spec]：

```yaml
service_type: osd
service_id: default_drive_group
placement:
  host_pattern: '*'
spec:
  data_devices:
    all: true
```

執行 `ceph orch apply osd --all-available-devices` 後，cephadm 會持續掃描所有節點上的可用磁碟，並自動將之加入叢集。根據官方文檔，**此命令的效果是持續性的**——在命令完成後才新增的磁碟（只要符合條件）也會被自動偵測並加入[^ceph-orch-apply]。

### 1.2 cephadm 判定「可用磁碟」的條件

cephadm 在自動掃描時會檢查以下所有條件，全部滿足才視為可用[^ceph-device-reject]：

- 不能有分割區（no partitions）
- 不能有 LVM 狀態
- 不能已被掛載
- 不能包含檔案系統
- 不能包含 Ceph BlueStore OSD 標籤
- 容量必須 ≥ 5 GB

---

## 2. Ceph 的熱插拔（Hot-Swap / Hot-Plug）能力

### 2.1 硬體層面

現代伺服器通常配備 hot-swappable 硬碟槽，允許在不關機的情況下更換硬碟。Ceph 官方與 Red Hat 文檔皆明確提及此能力[^rh-ceph-admin]。

### 2.2 軟體層面——替換故障 OSD 的標準流程

軟體層的 OSD 更換是半自動流程，需要管理員手動執行（步驟節選）[^rh-ceph-admin]：

1. `ceph osd out osd.<num>` — 將 OSD 標記為 out
2. `systemctl stop ceph-osd@<osd-id>` — 停止 OSD 程序
3. 等待 data migration 完成
4. `ceph osd crush remove osd.<num>` — 從 CRUSH 移除
5. `ceph auth del osd.<num>` — 刪除認證
6. `ceph osd rm osd.<num>` — 從叢集移除
7. 實際更換硬碟（若支援 hot-swap 則可抽換）
8. 重新建立 OSD

在 cephadm 模式下有較簡潔的指令：

```bash
ceph orch osd rm <osd_id> --replace   # 保留 OSD ID 替換
ceph orch osd rm <osd_id> --zap       # 移除並抹除資料
```

> **重點：** Ceph 的「熱插拔」僅指硬體層的抽換，OSD 的移除與重建需管理員手動執行，並非完全自動化。不存在「插入新硬碟 → 自動取代故障 OSD」的機制。

---

## 3. USB 外接硬碟 vs 內接 SATA/SAS——是否同等對待？

### 答案：**否，Ceph 有明確的機制將 USB 裝置排除在外**

### 3.1 第一道關卡：ceph-volume 的 `id_bus` 過濾

在 Ceph 原始碼 `src/ceph-volume/ceph_volume/util/device.py` 中，`_check_generic_reject_reasons()` 方法明確檢查[^ceph-source-device]：

```python
reasons = [
    ('id_bus', 'usb', 'id_bus'),   # ← USB 匯流排的裝置被直接拒絕
    ('ro', '1', 'read-only'),
]
```

如果磁碟的 `id_bus` 屬性為 `usb`，它會被加入 reject reasons，**不被視為可用裝置**。

### 3.2 第二道關卡：`removable` 屬性檢查

根據對 Ceph 原始碼的分析，`ceph_volume/util/disk.py` 中亦有下列檢查[^dcaro-blog]：

```python
if get_file_contents(os.path.join(_sys_block_path, dev, 'removable')) == "1":
    continue
```

所有 Linux 核心標記為 `removable=1` 的裝置（通常是 USB 外接硬碟）都會被跳過。

### 3.3 Rook（Kubernetes 上的 Ceph）亦有同現象

在 Rook 的 GitHub Issue #14699 中，使用者回報錯誤訊息 `cephosd: skipping device "sdc": ["id_bus"]`，維護者回應此為預期行為：Ceph 會自動排除 `ID_BUS=usb` 的裝置，以避免伺服器短暫插入 USB 隨身碟時的意外行為[^rook-issue]。

---

## 4. 強制使用 USB 硬碟的方法

### 4.1 修改 ceph-volume 原始碼

進入 `cephadm shell`，編輯 `/usr/lib/python3.6/site-packages/ceph_volume/util/disk.py`，註解掉檢查 `removable` 的程式行，然後手動執行[^dcaro-blog]：

```bash
ceph-volume lvm zap /dev/sda
ceph-volume lvm prepare --data /dev/sda
ceph orch daemon add osd node1:/dev/sda lvm
```

### 4.2 使用 LVM 繞過

根據 Ceph Tracker Bug #38833 的回應：無法從 ceph-volume 停用內部檢查，但可以在 USB 硬碟上先建立 LVM 邏輯卷，再將 LV 路徑提供給 ceph-volume，繞過磁碟層的直接檢查[^ceph-tracker]。

---

## 5. 官方立場與社群實務建議——**強烈不建議使用 USB 作為 OSD**

### 5.1 性能災難

Proxmox 論壇上有真實案例：使用 USB 3.0 連接 NVMe 作為 OSD 後，整體 IOPS 從約 3000 暴跌至 659（讀取）和 219（寫入）。移除 USB OSD 後恢復正常。經驗總結為[^proxmox-forum]：

> *「a single OSD with a very high latency (=low IOPS) might slow down the complete cluster.」*

單一慢速 OSD 即可拖垮全叢集。

### 5.2 可靠性不足

USB 連接缺乏企業級 SATA/SAS 硬碟的 SMART 監控、故障 LED 識別以及 libstoragemanagement 整合。Ceph 依賴可靠的底層儲存，USB 介面不滿足此要求。

### 5.3 社群共識

- **Homelab / 測試環境：** 搭配 Raspberry Pi 使用 USB 外接硬碟在技術上可行，但需手動 hack
- **生產環境：** 應使用內接 SATA/SAS 或 NVMe 硬碟，USB 方案不被支援且不被建議[^reddit-ceph]

---

## 6. 總結

| 能力 | 內接 SATA/SAS | USB 外接 |
|---|---|---|
| 自動偵測並加入 OSD（cephadm） | ✅ 原生支援 | ❌ 被 `id_bus` 和 `removable` 過濾排除 |
| 自動格式化 | ✅ | ❌ 需手動 hack 原始碼或使用 LVM 繞過 |
| 熱插拔（硬體層） | ✅ 支援 | ✅ 可插拔，但 Ceph 會拒絕 |
| 熱插拔（軟體層——自動重建 OSD） | ❌ 需手動操作 | ❌ 同上 |
| 生產環境建議 | ✅ 推薦 | ❌ 強烈不建議 |
| 對整體叢集效能影響 | 正常 | ⚠️ 單一 USB OSD 即可拖垮全叢集 |

結論：**Ceph 的「自動偵測/自動格式化」與「熱插拔」能力對 USB 外接硬碟並無作用**。ceph-volume 原始碼中有兩道針對 USB 裝置的過濾邏輯（`id_bus` 與 `removable`），從根本上排除了 USB 外接硬碟。即使透過修改原始碼或 LVM 繞過，社群與官方均強烈不建議在生產環境中使用 USB 作為 OSD，因為單一 USB OSD 的高延遲可能拖垮整個叢集效能。

---

[^ceph-osd-spec]: Ceph Documentation. (n.d.). Cephadm services: OSD Service Specification. Retrieved 2026-10-03, from https://docs.ceph.com/en/latest/cephadm/services/osd/
[^ceph-orch-apply]: IBM Documentation. (n.d.). Deploying OSDs on all available devices — IBM Ceph Storage 8.1.0. Retrieved 2026-10-03, from https://www.ibm.com/docs/en/storage-ceph/8.1.0?topic=osds-deploying-ceph-all-available-devices
[^ceph-device-reject]: Ceph Documentation. (n.d.). Cephadm design: Storage devices and OSDs. Retrieved 2026-10-03, from https://docs.ceph.com/en/latest/dev/cephadm/design/storage_devices_and_osds/
[^rh-ceph-admin]: Red Hat Documentation. (n.d.). Red Hat Ceph Storage 2 Administration Guide: Changing an OSD Drive. Retrieved 2026-10-03, from https://docs.redhat.com/en/documentation/red_hat_ceph_storage/2/html/administration_guide/changing_an_osd_drive
[^ceph-source-device]: Ceph Source Code. (n.d.). src/ceph-volume/ceph_volume/util/device.py — `_check_generic_reject_reasons()` method. Retrieved 2026-10-03, from https://github.com/ceph/ceph/blob/master/src/ceph-volume/ceph_volume/util/device.py
[^dcaro-blog]: Caro, D. (2024-02-17). Adding USB OSD to Rpi Ceph Cluster. Retrieved 2026-10-03, from https://musings.dcaro.es/posts/2024-02-17-adding-usb-osd-to-rpi-ceph-cluster/
[^rook-issue]: Rook GitHub Issue #14699. (n.d.). OSD creation skips USB devices due to id_bus check. Retrieved 2026-10-03, from https://github.com/rook/rook/issues/14699
[^ceph-tracker]: Ceph Tracker Bug #38833. (n.d.). ceph-volume does not support removable devices. Retrieved 2026-10-03, from https://tracker.ceph.com/issues/38833
[^proxmox-forum]: Proxmox Forum. (n.d.). [SOLVED] FYI: do not extend Ceph with OSDs connected via USB. Retrieved 2026-10-03, from https://forum.proxmox.com/threads/fyi-do-not-extend-ceph-with-osds-connected-via-usb.140975/
[^reddit-ceph]: Reddit r/ceph. (n.d.). Some questions about running Ceph with external drives. Retrieved 2026-10-03, from https://www.reddit.com/r/ceph/comments/sfnbz2/some_questions_about_running_ceph_with_external/