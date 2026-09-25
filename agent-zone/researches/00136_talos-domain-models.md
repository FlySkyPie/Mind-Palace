# Talos Linux 領域模型與概念知識

## 概述

Talos Linux 是由 Sidero Labs 開發的現代 Linux 發行版，專門設計用於執行 Kubernetes。不同於通用 Linux 發行版，Talos 的系統管理完全透過 gRPC API 進行——沒有 shell、SSH、套件管理器或互動式主控台。整個作業系統從頭以 Go 語言建構，從 PID 1 (`machined`) 到所有元件皆為自製，不衍生自其他發行版[^what-is-talos]。

本文整理 Talos 使用者所需理解的關鍵領域模型與核心概念。

---

## 1. 核心設計哲學（六大支柱）

Talos 的架構建立在六大設計原則之上，理解這些原則是掌握所有後續概念的基礎[^philosophy]：

| 原則 | 說明 |
|------|------|
| **分散式 (Distributed)** | 以高可用資料層為優先設計；etcd 以 ad-hoc 方式形成；無單點故障 |
| **不可變 (Immutable)** | 根檔案系統為唯讀 SquashFS；映像檔本身從不被修改；經簽署且版本化 |
| **極簡 (Minimal)** | 無 shell、SSH、GNU 工具組、busybox；SquashFS 映像檔小於 80 MB |
| **暫時性 (Ephemeral)** | 所有寫入磁碟的資料要不是可複寫的，就是可重建的；可寫分割區刻意命名為「ephemeral」 |
| **安全 (Secure)** | 所有通訊以 mTLS 加密，使用短生命週期自動輪換憑證；核心依 KSPP 建議強化 |
| **宣告式 (Declarative)** | 一切由單一 YAML 清單（machine configuration）定義——無腳本或程序式步驟 |

---

## 2. 節點與角色

Talos **節點 (node)** 是執行 Talos Linux 並參與 Kubernetes 叢集的機器。節點分為兩種角色[^components]：

- **Controlplane（控制平面節點）**：同時執行 Talos 控制平面與 Kubernetes 控制平面元件（etcd、kube-apiserver、kube-controller-manager、kube-scheduler）。
- **Worker（工作節點）**：透過 kubelet 與 containerd 執行使用者工作負載。

節點無 SSH 存取、無 shell、無套件管理器，完全透過 Talos gRPC API 經由 `talosctl` 管理。

---

## 3. 叢集

Talos **叢集 (cluster)** 是一群或多個 Talos 節點共同執行 Kubernetes 的集合。關鍵特性[^architecture]：

- etcd 叢集以 ad-hoc 方式建立，每個指定節點自行加入
- 操作模型類似生物系統：「如果有元件行為異常，切除它並讓替代元件成長」
- 叢集身份由 `clusterID`（base64 編碼的 32 位元組隨機值）和 `clusterSecret`（從不經網路傳送的共享秘密）共同定義

---

## 4. Machine Configuration（機器設定檔）

**Machine configuration** 是定義 Talos 節點所有狀態的單一宣告式 YAML 檔案，也是唯一的設定機制[^config-overview]。

### 4.1 `v1alpha1` 文件結構

設定檔為多文件 YAML（以 `---` 分隔）。`v1alpha1` 文件是唯一強制要求的文件，包含兩個頂層區段：

**`machine:`** 區段——節點特定設定：
- `machine.type`：`controlplane` 或 `worker`
- `machine.install`：安裝磁碟、映像檔、核心參數
- `machine.network`：主機名稱、網路介面、DNS
- `machine.kubelet`：kubelet 映像檔、額外參數、nodeIP
- `machine.disks`、`machine.files`、`machine.sysctls` 等

**`cluster:`** 區段——叢集全域設定：
- `cluster.controlPlane.endpoint`：標準 Kubernetes API 端點
- `cluster.network`：Pod/Service 子網路、DNS 領域、CNI
- `cluster.etcd`：etcd 設定
- `cluster.discovery`：叢集成員發現
- `cluster.token`：節點加入用的 bootstrap token

### 4.2 設定檔管理

設定檔可透過**修補 (patch)**（使用 strategic merge patch）以及**即時編輯**（立即生效或排程至下次重啟）來修改[^talosctl]。

---

## 5. talosctl CLI

`talosctl` 是管理 Talos 叢集的唯一 CLI 工具[^talosctl]。

### 5.1 端點 (Endpoints) vs 節點 (Nodes)——最重要的心智模型

這是 `talosctl` 中最重要的概念，兩者服務完全不同的用途：

- **端點 (Endpoints)**（`-e`/`--endpoints`）：工作站實際連線的位置。應為控制平面節點或前方的負載平衡器/VIP。指定多個端點時，`talosctl` 會自動進行負載平衡與容錯轉移。
- **節點 (Nodes)**（`-n`/`--nodes`）：指令實際要操作的目標機器。

**代理模型**：`talosctl` 連線至端點，端點再將請求代理轉送至目標節點。這表示你只需要直接網路存取端點（控制平面節點），而工作節點可位於私人網路中仍可被管理。

```bash
# 連線至控制平面 192.168.1.101（端點），目標工作節點 192.168.1.110（節點）
talosctl logs kubelet --endpoints 192.168.1.101 --nodes 192.168.1.110
```

注意：指定節點時，其 IP/主機名稱是**從端點伺服器的視角**，而非從客戶端視角——因為所有連線都透過端點代理。

### 5.2 設定檔與 Context（上下文）

與 `kubectl` 類似，`talosctl` 使用 **context** 來管理多個叢集：

- 設定檔位置：`$HOME/.talos/config`（`TALOSCONFIG` 環境變數可覆蓋）
- 每個 context 儲存：`endpoints`、`nodes`、`ca`（CA 憑證）、`crt`/`key`（客戶端憑證與金鑰）

### 5.3 常用指令模式

| 類別 | 範例指令 |
|------|---------|
| 叢集啟動 | `apply-config`、`bootstrap` |
| 節點檢查 | `dmesg`、`logs`、`processes`、`version`、`memory` |
| 資源管理 | `get`、`edit`、`list`、`create`、`delete` |
| Kubernetes 操作 | `kubeconfig`、`cluster`、`health` |
| 節點生命週期 | `reboot`、`reset`、`shutdown`、`upgrade` |
| 設定管理 | `config endpoint`、`config context`、`config merge` |
| 診斷 | `service`、`stats`、`copy`、`inspect` |

### 5.4 初始化佈建（維護模式）

當節點處於**維護模式**（從 ISO 開機，尚無設定檔）時，使用 `--insecure` 旗標。此時 `--nodes` 同時作為端點與節點——直接連線至機器連接埠 50000，無代理：

```bash
talosctl apply-config --insecure --nodes 192.168.1.101 --file controlplane.yaml
```

---

## 6. 控制器與資源（Controllers & Resources）

Talos 內部使用類似 Kubernetes 的 **controller-resource** 模式，命名為 COSI（Controller OSI）[^controllers-resources]。

### 6.1 資源 (Resources)

資源以 `(namespace, type, id)` 三元組唯一識別。所有資源均為**節點本機**且儲存於記憶體中——每次重啟時重建（除了 `MachineConfig`）[^controllers-resources]。

| 資源類型 | Namespace | 說明 |
|---------|-----------|------|
| `MachineConfigs.config.talos.dev` | config | 機器設定檔 |
| `MachineTypes.config.talos.dev` | config | 節點角色（controlplane/worker） |
| `StaticPods.kubernetes.talos.dev` | controlplane | 靜態 Pod 定義 |
| `Services.v1alpha1.talos.dev` | runtime | Talos 內部服務狀態 |
| `TimeStatuses.v1alpha1.talos.dev` | runtime | 時間同步狀態 |
| `Secrets.*.talos.dev` | secrets | PKI/憑證秘密 |

查詢方式：

```bash
talosctl get <resource-type>
talosctl get <resource-type> -o yaml    # YAML 輸出
talosctl get <resource-type> --watch    # 即時變更監控
```

### 6.2 控制器 (Controllers)

控制器是在 `machined` 內部執行的**獨立輕量執行緒**（goroutine）[^controllers-resources]：

- 目標：根據輸入**調和 (reconcile)** 狀態並更新輸出
- 監看指定的資源變更並在變更發生時調和
- 也可進行排程調和或監看 etcd 鍵值
- 每個控制器管理單一（namespace, type）組合——避免衝突
- 相依性圖可透過 `talosctl inspect dependencies` 檢視（輸出 Graphviz 圖形）

### 6.3 與 Kubernetes 模式比較

| 面向 | Kubernetes | Talos |
|------|-----------|-------|
| 資源儲存 | etcd（叢集層級） | 記憶體（節點本機） |
| 持久性 | 持久 | 重啟時重建（除 MachineConfig） |
| 範圍 | 叢集範圍 | 節點本機 |
| 存取方式 | kubectl | talosctl |

---

## 7. 開機階段（Phases of Boot）

Talos 的開機流程包含以下階段[^architecture]：

1. **核心啟動** — 依 KSPP 建議設定的 Linux 核心載入 initramfs
2. **`machined` 以 PID 1 啟動** — Talos 自己的 init 程序（非 systemd）
3. **載入設定檔** — 若無設定檔則進入維護模式
4. **維護模式** — 在 RAM 中執行（ISO 開機時），等待 `talosctl apply-config --insecure`
5. **設定檔套用** — 如從 ISO 開機，Talos 此時安裝至磁碟（寫入六個分割區）
6. **重新開機** — 從磁碟安裝開機，根檔案系統（squashfs）以唯讀掛載
7. **叢集形成** — 節點透過發現服務互相發現；控制平面節點形成 etcd 叢集
8. **Bootstrap** — 在單一控制平面節點上執行 `talosctl bootstrap`，初始化 etcd 並啟動靜態 Pod
9. **Kubernetes 叢集就緒** — 工作節點加入、kubelet 註冊、CNI 安裝

---

## 8. 磁碟分割區與檔案系統

### 8.1 六個標籤分割區

Talos 在磁碟上使用六個標籤分割區[^disk-layout]：

| 分割區 | 標籤 | 用途 |
|---------|------|------|
| **EFI** | `EFI` | 儲存 EFI 開機資料（UKI、systemd-boot） |
| **BIOS** | `BIOS` | GRUB 第二階段開機（舊式 BIOS） |
| **BOOT** | `BOOT` | 開機載入程式、initramfs、核心資料（GRUB 安裝） |
| **META** | `META` | 節點中繼資料（節點 ID、升級狀態、開機嘗試計數器） |
| **STATE** | `STATE` | 機器設定檔、節點身份（叢集發現）、KubeSpan 資訊 |
| **EPHEMERAL** | `EPHEMERAL` | 暫時狀態，掛載於 `/var`（etcd、kubelet、containerd 資料） |

### 8.2 三層根檔案系統

1. **唯讀基底 (SquashFS)** — 以 loop 裝置掛載至記憶體的唯讀基底（約 80 MB）
2. **tmpfs 層** — `/dev`、`/proc`、`/run`、`/sys`、`/tmp` 以及特殊 `/system` 目錄；`/system` 下的所有檔案在每次開機時完全重新建立
3. **overlayfs 層** — 需要在開機間持久的檔案（如 `/etc/kubernetes`），由 `/var` 上的 XFS 支援

`/var` 目錄為可寫入，由 etcd、kubelet、containerd 使用。其內容在重啟與升級後保留，但 `talosctl reset` 時清除。

---

## 9. 叢集發現（Cluster Discovery）

Talos 內建多後端的叢集發現系統，自動化節點尋找、識別、確認彼此為叢集成員的流程[^discovery]。

### 9.1 三層資源模型

發現機制建模為透過三種資源類型的**進程 (progression)**：

**身份 (Identities)**：
每個節點有唯一的**身份**——base62 編碼的 32 位元組隨機值，作為節點的 `Affiliate` 識別符。儲存於 `STATE` 分割區的 `node-identity.yaml`，在重啟與升級後保留（但重置時重新產生）[^discovery]。

```bash
talosctl get identities -o yaml
# spec:
#   nodeId: Utoh3O0ZneV0kT2IUBrh7TgdouRcUW2yzaaMl4VXnCd
```

**關聯體 (Affiliates)**：
**Affiliate** 是與目前節點共享相同 `clusterID` 和 `clusterSecret` 的節點——即可以交換加密發現資料的**潛在**叢集成員。不同叢集的 Affiliate 互相不可見[^discovery]。

```bash
talosctl get affiliates
# 顯示所有註冊表知道的節點（主機名稱、機器類型、位址）
```

**成員 (Members)**：
**Member** 是已被**確認並核准**加入叢集的 Affiliate。這是叢集成員資格的最終權威視圖[^discovery]。

```bash
talosctl get members
# 顯示已確認的叢集成員
```

### 9.2 發現服務 (Service Registry)

預設的外部發現服務位於 `https://discovery.talos.dev/`[^discovery]：

- **最小信任設計** — 服務本身是**不受信任的**。它儲存和轉發的只有加密資料，無法讀取節點資訊。
- **暫時性操作** — 資料保留於記憶體中，TTL 為 **30 分鐘**。節點在 TTL 到期前定期重新整理註冊。
- **加密模型** — Affiliate 資料以 AES-GCM 加密；端點資料以 AES-ECB 加密（允許伺服器端去重）。
- 發現服務**能看到**的明文：叢集 ID（單向雜湊，不可逆）、Affiliate ID、用戶端版本、Affiliate 數量。
- 發現服務**看不到**的內容：節點 IP、KubeSpan WireGuard 公鑰、節點主機名稱。

### 9.3 Kubernetes 註冊表（已棄用）

**Kubernetes 註冊表**將發現資料儲存為 Kubernetes `Node` 資源的註解（annotations），但已於目前版本中**預設禁用且已棄用**[^discovery]。

### 9.4 發現被禁用的影響

若關閉發現功能，以下功能將受影響[^discovery]：
- KubeSpan 和 KubePrism 將**無法運作**
- 初始叢集啟動與復原時間延長
- 工作節點在故障期間更依賴控制平面可用性

---

## 10. KubeSpan（WireGuard 網狀網路）

KubeSpan 是 Talos 內建的功能，能**自動化在叢集所有節點間建立與維護完整網狀 (full-mesh) WireGuard 網路**[^kubespan]。使用 **UDP 連接埠 51820**。

### 10.1 資源模型

KubeSpan 透過四種主要資源類型實現（namespace: `kubespan`）：

**KubeSpanIdentities**（`KubeSpanIdentities.kubespan.talos.dev`）：
代表本機節點的 WireGuard 身份，ID 為 `"local"`。包含節點的 KubeSpan IPv6 位址、子網路、WireGuard 金鑰對。私鑰永不離開節點。儲存於 STATE 分割區的 `kubespan-identity.yaml`（重啟與升級後保留）[^kubespan]。

**KubeSpanPeerSpecs**（`KubeSpanPeerSpecs.kubespan.talos.dev`）：
代表 WireGuard 對等規格——每個發現的叢集成員一個。由 *Peer Spec Controller* 從叢集 Affiliate 資料產生。包含對等的位址、允許的子網路、端點、標籤。ID 為 base64 編碼的 WireGuard 公鑰[^kubespan]。

**KubeSpanPeerStatuses**（`KubeSpanPeerStatuses.kubespan.talos.dev`）：
代表 WireGuard 對等連線的**即時狀態**。狀態機為 `unknown → up → down`。每 **30 秒**更新一次。包含目前端點、傳輸位元組、最後握手時間[^kubespan]。

**KubeSpanEndpoints**（`KubeSpanEndpoints.kubespan.talos.dev`）：
代表**觀察到的可用端點**，對映回其 Affiliate。由 *Endpoint Controller* 產生——當對等透過某端點連線時，該端點被收集並重新發佈至發現服務，讓其他節點可嘗試[^kubespan]。

### 10.2 控制器

| 控制器 | 職責 |
|---------|------|
| **Identity Controller** | 產生/管理 WireGuard 金鑰對，發佈 Identity 資源 |
| **Peer Spec Controller** | 監看發現的 Affiliate，建立 PeerSpec 資源 |
| **Manager Controller** | 讀取 PeerSpecs，設定實際 WireGuard 介面/對等 |
| **Endpoint Controller** | 從 PeerStatuses 收集可用端點，回饋至發現服務 |

### 10.3 封包路由（三階段系統）

KubeSpan 使用三階段路由系統以隔離於 CNI 之外[^kubespan]：

1. **NFTables**（table: `talos_kubespan`）— 定義「叢集內」目的地的 IP 集合，標記相符封包（firewall mark `0x40` within `0x60`）
2. **IP Route Rules** — 若封包標記 KubeSpan firewall mark，送至路由表 **180**
3. **路由表 180** — 兩條預設路由：IPv4 → `kubespan` 介面，IPv6 → `kubespan` 介面

此設計與 Calico（`0xffff0000`）、Cilium（`0xf00`）、Flannel（`0x4000`）相容。

### 10.4 設定選項

```yaml
apiVersion: v1alpha1
kind: KubeSpanConfig
enabled: true
```

| 選項 | 預設值 | 說明 |
|------|--------|------|
| `advertiseKubernetesNetworks` | false | 若啟用，Pod/Service CIDR 會透過 KubeSpan 公告 |
| `allowDownPeerBypass` | false | 允許流量繞過故障對等（較不安全） |
| `mtu` | 1420 | WireGuard 介面 MTU |
| `filters.endpoints` | `["0.0.0.0/0", "::/0"]` | 公告端點的 CIDR 過濾器 |
| `harvestExtraEndpoints` | false | 是否從對等狀態收集端點回饋至發現服務 |
| `extraAnnouncedEndpoints` | [] | 手動指定的額外端點（如 NAT 對映） |

---

## 11. Image Factory 與 Schematics

**Image Factory**（`https://factory.talos.dev`）是產生自訂 Talos 開機資產的服務[^image-factory]。

### 11.1 Schematics（示意圖）

**Schematic** 是宣告 Talos 映像檔自訂項的 YAML 文件[^image-factory]：

```yaml
customization:
  extraKernelArgs:
    - vga=791
  systemExtensions:
    officialExtensions:
      - siderolabs/gvisor
      - siderolabs/amd-ucode
overlay:
  name: rpi_generic
  image: siderolabs/sbc-raspberry-pi
```

**內容可定址 (Content-Addressable)**：Schematic 是內容可定址的——YAML 內容的 SHA-256 雜湊成為其唯一 ID。相同內容 → 相同 ID（冪等）。不可修改，必須建立新的。

「原始」(vanilla) schematic ID：`376567988ad370138ad8b2698212367b8edcb69b5fd68c80be1f2ec7d603b4ba`

### 11.2 Model（模型）

**Model** 是具體的 Talos 開機資產。產生模型所需的輸入[^image-factory]：
- Schematic ID
- Talos 版本（如 `v1.9.0`）
- 模型類型（ISO、UKI、磁碟映像、installer 映像等）
- 架構（`amd64` 或 `arm64`）

### 11.3 System Extensions（系統擴充）

**系統擴充**是在映像檔建置時注入不可變根檔案系統的容器映像檔。以 OCI 容器映像檔格式發佈，包含特定目錄結構（`manifest.yaml` + `rootfs`）[^image-factory]。

支援層級：`core`（Sidero 完全支援）、`extra`（最佳 effort）、`contrib`（社群）。

範例：`siderolabs/gvisor`、`siderolabs/amd-ucode`、`siderolabs/nvidia-container-toolkit`。

### 11.4 Overlays（覆疊層）

**Overlay** 用於自訂安裝流程本身（而非根檔案系統內容），主要用於**單板電腦 (SBC)** 如 Raspberry Pi。提供板專用的韌體、開機載入程式、裝置樹 (DTB)、安裝程式[^image-factory]。

### 11.5 工作流程

1. **定義** — 撰寫 schematic YAML
2. **上傳** — POST 至 `https://factory.talos.dev/schematics`
3. **雜湊** — Image Factory 對 YAML 進行 SHA-256 雜湊 → 唯一 ID
4. **產生** — 使用 ID + 版本 + 模型類型請求模型（`/image/{schematic_id}/{version}/{model_type}`）
5. **簽署** — 產生的成品經 ECDSA P-256 簽署後提供

---

## 12. 升級模型

### 12.1 A-B 映像檔方案

Talos 使用 **A-B 映像檔方案** 使升級安全且可復原——每次升級後保留先前的核心與 OS 映像檔，因此任何時刻磁碟上都有兩個可開機版本[^upgrade]。

| 開機槽 | 角色 |
|--------|------|
| **Slot A** | 使用中（目前執行） |
| **Slot B** | 非使用中（先前版本/備用） |

UKI（UEFI 系統）的開機資產儲存於 EFI 分割區；舊式 GRUB 安裝則儲存於 BOOT 分割區。META 分割區儲存「大腦」——節點 UUID、升級狀態、開機嘗試計數器[^upgrade]。

### 12.2 升級流程詳解

透過 `talosctl upgrade --image <installer-image>` 觸發的 API 呼叫，執行以下序列[^upgrade]：

1. **Cordon** — 節點封裝自身，防止新工作負載排程
2. **Drain** — 既有工作負載從節點排除（尊重 `preStop` hooks）
3. **關閉服務** — Talos 關閉所有內部程序並**卸載所有檔案系統**
4. **磁碟驗證** — 檢查並準備磁碟
5. **寫入映像檔** — 新安裝程式映像檔寫入**非使用中開機槽**；開機載入程式設定為以新映像檔**一次性**開機（"staged" 升級）
6. **重新開機** — 節點透過 `kexec` 重新開機（極快）
7. **驗證** — 開機進入新版本後，Talos 驗證自身
8. **提交 (Commit)** — 開機載入程式變更設為**永久**
9. **重新加入並取消封裝** — 節點重新加入叢集並取消 cordon

### 12.3 復原機制

| 類型 | 觸發 | 結果 |
|------|------|------|
| **自動復原** | 新版本開機失敗（核心恐慌、當機等） | 開機載入程式自動退回前一槽，無需手動干預 |
| **手動復原** | `talosctl rollback` | 將 META 中的開機參照更新回前一版本，然後重新開機 |

### 12.4 升級限制

- 設定遷移僅在**相鄰次要版本**之間經過測試，因此建議逐一升級每個次要版本至其最新修補版本
- 控制平面節點升級時，Talos 會檢查 etcd 法定人數——若會造成法定人數遺失則拒絕升級
- 若多個控制平面節點同時被要求升級，Talos 透過檢查 etcd 法定人數確保**一次僅一個升級**

---

## 13. SideroLink（點對點管理覆疊網路）

SideroLink 是 Talos 叢集的**點對點管理覆疊網路**（由 Sidero Omni 使用）[^siderolink]：

- 建立從每台機器到 SideroLink API 伺服器的安全 WireGuard 隧道
- 使用 ULA IPv6 位址進行管理——即使無法直接 IP 存取也可運作
- 維護模式 API 僅在 SideroLink 網路上監聽
- 透過核心參數 `siderolink.api` 或 `SideroLinkConfig` 文件設定

---

## 14. 關鍵的領域概念對照表

| 概念 | 與傳統 Linux 的差異 | 使用者須知 |
|------|-------------------|-----------|
| 系統管理 | 無 SSH/Shell | 僅透過 `talosctl` API |
| 套件管理 | 無套件管理器 | 改以系統擴充 (System Extensions) 在映像檔建置時注入 |
| 設定 | 無互動式設定 | 單一宣告式 YAML，可修補與即時編輯 |
| 檔案系統 | 唯讀 SquashFS + 暫時性 `/var` | 不可變基底，系統檔案不可直接編輯 |
| 升級 | 非套件升級 | A-B 方案 API 升級，具自動復原能力 |
| 網路 | 無 iptables 管理 | KubeSpan 自動 WireGuard 網狀網路 |
| 叢集發現 | 無內建發現機制 | 自動發現服務，零手動節點註冊 |

---

## 參考來源

[^what-is-talos]: Sidero Labs. (n.d.). What is Talos Linux? Retrieved 2026-09-25, from https://www.talos.dev/v1.9/introduction/what-is-talos/

[^philosophy]: Sidero Labs. (n.d.). Philosophy. Retrieved 2026-09-25, from https://www.talos.dev/v1.9/learn-more/philosophy/

[^architecture]: Sidero Labs. (n.d.). Architecture. Retrieved 2026-09-25, from https://www.talos.dev/v1.9/learn-more/architecture/

[^components]: Sidero Labs. (n.d.). Components. Retrieved 2026-09-25, from https://www.talos.dev/v1.9/learn-more/components/

[^controllers-resources]: Sidero Labs. (n.d.). Controllers and Resources. Retrieved 2026-09-25, from https://www.talos.dev/v1.9/learn-more/controllers-resources/

[^talosctl]: Sidero Labs. (n.d.). talosctl. Retrieved 2026-09-25, from https://www.talos.dev/v1.9/learn-more/talosctl/

[^config-overview]: Sidero Labs. (n.d.). Machine Configuration Overview. Retrieved 2026-09-25, from https://www.talos.dev/v1.9/reference/configuration/

[^kubespan]: Sidero Labs. (n.d.). KubeSpan. Retrieved 2026-09-25, from https://www.talos.dev/v1.9/learn-more/kubespan/

[^image-factory]: Sidero Labs. (n.d.). Image Factory. Retrieved 2026-09-25, from https://www.talos.dev/v1.9/learn-more/image-factory/

[^discovery]: Sidero Labs. (n.d.). Discovery Service. Retrieved 2026-09-25, from https://docs.siderolabs.com/talos/v1.13/configure-your-talos-cluster/system-configuration/discovery

[^upgrade]: Sidero Labs. (n.d.). Upgrading Talos Linux. Retrieved 2026-09-25, from https://docs.siderolabs.com/talos/v1.14/configure-your-talos-cluster/lifecycle-management/upgrading-talos

[^disk-layout]: Sidero Labs. (n.d.). Disk Layout. Retrieved 2026-09-25, from https://docs.siderolabs.com/talos/v1.13/configure-your-talos-cluster/storage-and-disk-management/disk-management/layout

[^siderolink]: Sidero Labs. (n.d.). SideroLink. Retrieved 2026-09-25, from https://docs.siderolabs.com/talos/v1.14/networking/siderolink