# K3s 單節點轉移至三節點 Talos Linux：注意事項與經驗整理

## 背景

本報告針對一個已使用 K3s 單節點的用戶，計劃新增一個三節點 Talos Linux[^talos] 叢集（兩個叢集將共存運作，不涉及服務遷移），提供轉移前應注意的事項與觀念整備。

Talos Linux 是由 Sidero Labs 開發的**以 Kubernetes 為核心的 Linux 發行版**（Kubernetes-optimized OS），而非像 K3s 那樣的**Kubernetes 發行版**[^sidero-vs-k3s]。兩者處於不同抽象層級，理解此差異是轉移的第一個關鍵。

## K3s 與 Talos 的根本差異

### 層級不同

| 面向 | K3s | Talos Linux |
|------|-----|-------------|
| 定義 | 在既有 Linux OS 上執行的 Kubernetes 發行版 | 本身就是安裝並管理上游 Kubernetes 的作業系統 |
| 可比較對象 | K3s + Ubuntu/Debian | Talos + 上游 K8s[^sidero-vs-k3s] |
| SSH | 有（透過底層 OS） | 無，無法 SSH 進入節點 |
| 套件管理 | apt/yum 等（透過底層 OS） | 無套件管理器，系統不可變（immutable）[^talos-arch] |

### 資料儲存：SQLite → etcd

K3s 單節點預設使用 SQLite（透過 Kine 轉換層），而 Talos **只能使用 etcd**[^talos-etcd]：

| 面向 | K3s (SQLite) | Talos (etcd) |
|------|-------------|--------------|
| RAM 使用 | ~50 MB | ~500 MB 以上 |
| 仲裁 (Quorum) | 不需要 | 三節點需 2/3 正常 |
| 備份方式 | 複製檔案即可 | `talosctl etcd snapshot` |
| 多節點 HA | 不支援 | 原生支援（3+ 控制平面）[^etcd-quorum] |

**重要**：etcd 要求**奇數個控制平面節點**（3、5、7），兩節點控制平面不具容錯能力——任一節點故障即失去仲裁（quorum）[^sidero-vs-k3s]。

### K3s 內建但 Talos 不提供的功能

這是 K3s 使用者轉移時最常忽略的部分[^migration-guide][^pitfalls]：

| K3s 內建功能 | Talos 對應方案（需自行安裝） |
|------|---------------------|
| Traefik Ingress Controller | 自行安裝 Traefik、NGINX Ingress 或 Cilium Gateway API |
| ServiceLB (Klipper LoadBalancer) | MetalLB 或 Cilium L2 Announcements |
| Local-Path Provisioner（儲存） | Longhorn、Rook Ceph 或其他 CSI Driver |
| Metrics Server | 自行安裝上游 manifests |
| HelmChart CRD（自動部署 Helm） | 無此 CRD，需轉為標準 Helm 或 GitOps（ArgoCD/Flux） |
| Network Policy Controller | 需透過 Cilium 或 Calico 提供 |
| Spegel（映像檔鏡像） | 無內建，需自行設定 registry mirrors |

這些功能在 Talos 上**全部需手動安裝**。若未事先部署，應用程式將無法取得 Ingress、LoadBalancer IP 與持久儲存。

## 三節點 Talos 叢集的關鍵注意事項

### 1. 控制平面端點（VIP / Load Balancer）

三節點 Talos 叢集需要一個**穩定的控制平面端點**（VIP 或外部 LB），讓 `kubelet` 與 `kubectl` 能夠持續存取 API Server。這需在機器設定檔（machine config）中事先設定，選項包括：

- Talos 內建的 **VIP**（基於 keepalived）
- 外部 HAProxy / NGINX
- 雲端 Load Balancer[^talos-vip]

### 2. 升級與維護

Talos 的升級是**原子化的雙軌系統**[^talos-upgrade]：

- **Talos OS 升級**（`talosctl upgrade`）：替換整個 OS 映像，具備 A/B 分割區自動回滾機制
- **Kubernetes 版本升級**（`talosctl upgrade-k8s`）：不影響 OS 層級

兩者**互相獨立**，升級 Talos OS 不會同時升級 Kubernetes 版本。

三節點升級流程：控制平面一次升級一個節點（Talos 會自動保護 etcd 仲裁），工人節點再逐一升級[^talos-upgrade-strategy]。

### 3. 開機前先規劃 Extension

這是 Talos 新手最容易踩的陷阱[^pitfalls]：如果需要特定核心模組（如 Longhorn 所需的 `iscsi-tools`、`util-linux-tools`），必須在**安裝前**透過 [Talos Image Factory](https://factory.talos.dev/) 自訂映像檔。Talos 不可變的 rootfs 不允許安裝後以 `apt install` 補上。

### 4. 節點可拋棄（Disposable Node）觀念

所有節點透過同一份 YAML 機器設定檔管理。節點故障時，重新安裝 Talos 並套用相同設定檔即可加入叢集。不再有「SSH 進入修補」的作業模式[^talos-arch]。

### 5. 除錯方式轉變

| 傳統作法 | Talos 對應作法 |
|---------|--------------|
| SSH + journalctl | `talosctl logs` / `talosctl dmesg` |
| htop / tcpdump | `talosctl dashboard` / 特權 Debug Pod |
| apt install 除錯工具 | 以 DaemonSet 執行特權容器 (`nsenter`)[^no-ssh] |

### 6. etcd 備份紀律

從 SQLite 的「複製檔案」轉換到 etcd，需要建立排程備份機制：

- 使用 `talosctl etcd snapshot` 定期備份（建議每 4-6 小時）
- 可使用 Sidero 官方 `talos-backup` CronJob 搭配 `age` 加密與 S3 儲存[^talos-backup]
- 升級前務必備份：`talosctl etcd snapshot ./pre-upgrade.snapshot --nodes <cp-ip>`[^etcd-backup-guide]

### 7. 控制平面排程設定

三節點皆為控制平面時，Kubernetes 預設不會在工作負載控制平面節點上。需決定[^sidero-vs-k3s]：

- **選項 A**：允許控制平面排程（`allowSchedulingOnControlPlanes: true`）——硬體需求較低，但升級節點時會影響工作負載
- **選項 B**：控制平面與工人節點分離——真正的高可用性，但需要更多硬體

## 雙叢集共存注意事項

用戶提到兩叢集將同時存在，不需服務遷移，應注意：

1. **網路不衝突**：兩個叢集的 Pod CIDR（預設 `10.42.0.0/16`）與 Service CIDR（預設 `10.43.0.0/16`）不可重疊。在 Talos 機器設定中需明確設定不同的 CIDR[^cilium-talos]。
2. **LoadBalancer IP 池分離**：MetalLB 的 IP 池（Talos 端）不應與 K3s 的 ServiceLB 或其他 IP 分配重疊。
3. **DNS 分流**：兩個叢集各自的 Ingress Controller 需綁定不同網域名稱或 IP。

## 總結

從 K3s 單節點轉換至三節點 Talos 叢集是**作業模式的根本轉變**：

| K3s 使用者習慣 | Talos 新思維 |
|---------------|-------------|
| SSH 進入操作 | talosctl API-only、不可變系統 |
| SQLite 資料檔簡單備份 | etcd 排程備份 + quorum 管理 |
| 內建一大堆功能（Traefik、ServiceLB、StorageClass） | 全部自行安裝 |
| 二元檔級別升級 | OS + K8s 原子化雙軌升級 |
| 各節點各有歷史（snowflake） | 所有節點完全一致，可拋棄重建 |

**不建議將 K3s 節點原地轉換為 Talos**——兩者不可互換。最好的策略是平行建置 Talos 叢集，確認所有基礎元件（Ingress、LB、CSI Storage、Metrics Server、Helm/GitOps）皆已就緒後，再逐步遷移工作負載。

---

[^talos]: Sidero Labs. (n.d.). Talos Linux — A Kubernetes-optimized operating system. Retrieved 2026-09-26, from https://github.com/siderolabs/talos

[^sidero-vs-k3s]: Sidero Labs. (n.d.). Talos Linux vs K3s. Retrieved 2026-09-26, from https://www.siderolabs.com/blog/talos-linux-vs-k3s

[^talos-arch]: Sidero Labs. (n.d.). Talos Linux Architecture. Retrieved 2026-09-26, from https://www.talos.dev/docs/latest/learn-more/architecture/

[^talos-etcd]: Sidero Labs. (n.d.). Talos Control Plane. Retrieved 2026-09-26, from https://www.talos.dev/docs/latest/learn-more/control-plane/

[^etcd-quorum]: K3s Documentation. (n.d.). Embedded etcd High Availability. Retrieved 2026-09-26, from https://docs.k3s.io/datastore/ha-embedded

[^migration-guide]: OneUptime. (2026). Migrate from K3s to Talos Linux. Retrieved 2026-09-26, from https://oneuptime.com/blog/post/2026-03-03-migrate-from-k3s-to-talos-linux/view

[^pitfalls]: Simeon Ivanov. (n.d.). Migrating K3s to Talos Linux. Retrieved 2026-09-26, from https://simeonivanov.org/posts/migrating-k3s-to-talos-linux/

[^talos-vip]: Sidero Labs. (n.d.). Talos Cluster VIP. Retrieved 2026-09-26, from https://www.talos.dev/docs/latest/learn-more/vip/

[^talos-upgrade]: Sidero Labs. (n.d.). Upgrading Talos Linux. Retrieved 2026-09-26, from https://www.talos.dev/docs/latest/learn-more/upgrading-talos/

[^talos-upgrade-strategy]: Daly, M. (n.d.). Kubernetes Talos Upgrades. Retrieved 2026-09-26, from https://blog.dalydays.com/post/kubernetes-talos-upgrades/

[^no-ssh]: Alexandre Vazquez. (2026). Talos Linux Guide. Retrieved 2026-09-26, from https://alexandre-vazquez.com/talos-linux-guide/

[^talos-backup]: Sidero Labs. (n.d.). talos-backup — official etcd backup tool. Retrieved 2026-09-26, from https://github.com/siderolabs/talos-backup

[^etcd-backup-guide]: OneUptime. (2026). Using talosctl etcd snapshot for backups. Retrieved 2026-09-26, from https://oneuptime.com/blog/post/2026-03-03-use-talosctl-etcd-snapshot-for-backups/view

[^cilium-talos]: Widdershoven, P. (n.d.). Networking on Talos Linux with Cilium. Retrieved 2026-09-26, from https://www.pimwiddershoven.nl/entry/networking-on-talos-linux-with-cilium/