# 在 Talos Linux 上安裝與設定 MetalLB 負載平衡器

## 概述

MetalLB 是一個專為裸機（bare-metal）Kubernetes 叢集設計的網路負載平衡器實作[^metallb-site]。在雲端環境（AWS、GCP、Azure）中，當建立 `LoadBalancer` 類型的 Service 時，雲端控制器會自動佈建外部負載平衡器並指派外部 IP。但在裸機環境中，沒有這樣的整合機制，因此 `LoadBalancer` Service 會卡在 `Pending` 狀態。MetalLB 正是為了解決這個問題而誕生。

MetalLB 提供兩項核心功能：

1. **IP 位址分配** — 從管理者定義的 IP 位址池中，為 `LoadBalancer` Service 指派外部 IP。
2. **外部通告** — 透過以下兩種模式讓 IP 可在叢集外存取：
   - **Layer 2 模式** — 透過 ARP（IPv4）或 NDP（IPv6）回應，讓 Service IP 在同一廣播域內可達。簡單、相容性高，但所有流量透過單一節點進入。
   - **BGP 模式** — 與網路路由器進行 BGP 對等連線，將 Service IP 路由資訊通告至路由器，路由器透過 ECMP 將流量分散至多個節點。

## 在 Talos Linux 上的先決條件

在安裝 MetalLB 前，需要確認以下條件[^oneuptime-l2][^metallb-install]：

1. **一個運作中的 Talos Linux 叢集**，並可透過 `kubectl` 存取。
2. **一段可用的 IP 位址範圍**，位於您的區域網路內（L2 模式），或已分配的 BGP AS 號碼（BGP 模式）。
3. **IP 位址池不得與下列項目衝突**：
   - Talos 節點 IP
   - Kubernetes Service CIDR
   - DHCP 動態分配範圍
4. **kube-proxy 嚴格 ARP** — 若使用 IPVS 模式，必須啟用 `strictARP`；iptables 模式（預設）則不需要。
5. **權限命名空間標籤** — MetalLB speaker Pod 需要較高的網路權限，必須在命名空間上標記 Pod Security Admission 為 privileged。
6. **控制平面節點標籤修正** — Talos 會自動在控制平面節點標記 `node.kubernetes.io/exclude-from-external-load-balancers`，MetalLB 0.14+ 會遵守此標籤，導致不含 Worker 節點的叢集無法進行 L2 宣告。

## 安裝步驟

### 方法 A：使用 Helm 安裝（建議）

```bash
# 加入 MetalLB Helm 儲存庫
helm repo add metallb https://metallb.github.io/metallb
helm repo update

# 建立命名空間
kubectl create namespace metallb-system

# 為命名空間標記 privileged Pod 安全權限（必要）
kubectl label namespace metallb-system \
    pod-security.kubernetes.io/enforce=privileged \
    pod-security.kubernetes.io/audit=privileged \
    pod-security.kubernetes.io/warn=privileged

# 安裝 MetalLB（預設使用 FRR-K8s 模式，建議）
helm install metallb metallb/metallb \
    --namespace metallb-system \
    --wait
```

若要明確指定 FRR-K8s 模式（支援延伸 next-hop、BFD 等進階 BGP 功能）：

```bash
helm install metallb metallb/metallb \
    --namespace metallb-system \
    --set frrk8s.enabled=true \
    --set speaker.frr.enabled=false \
    --wait
```

安裝後驗證：

```bash
kubectl get pods -n metallb-system
# 應看到：metallb-controller（Deployment）+ metallb-speaker（DaemonSet，每節點一個）
```

### 方法 B：使用 Kubernetes 原生 Manifest

FRR-K8s 模式（建議）：

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.16.1/config/manifests/metallb-frr-k8s.yaml
```

原生模式（較輕量，不支援 BFD / IPv6 BGP）：

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.16.1/config/manifests/metallb-native.yaml
```

## Talos 專屬組態設定

### 啟用嚴格 ARP（若使用 IPVS kube-proxy）

在 Talos 中，kube-proxy 由機器組態（machine config）管理，**不可**像 kubeadm 那樣直接編輯 ConfigMap[^oneuptime-kubeproxy]。需建立 Patch 檔案並透過 `talosctl patch` 套用：

```yaml
# talos-strict-arp-patch.yaml
cluster:
  proxy:
    config:
      ipvs:
        strictARP: true
```

套用 Patch：

```bash
talosctl patch machineconfig --nodes $CP_NODES --patch @talos-strict-arp-patch.yaml
```

若要切換至 IPVS 模式：

```yaml
cluster:
  proxy:
    extraArgs:
      proxy-mode: ipvs
      ipvs-scheduler: rr
```

### 修正純控制平面叢集（無 Worker 節點）

若叢集僅有控制平面節點而無專用 Worker，需設定 `ignoreExcludeLB: true`[^lelopez-fix]。

方式一：透過 Helm values 設定：

```yaml
speaker:
  ignoreExcludeLB: true
```

方式二：透過 Talos 機器組態移除節點標籤：

```yaml
machine:
  nodeLabels:
    node.kubernetes.io/exclude-from-external-load-balancers: ""
    $patch: delete
```

之後重新啟動 speaker DaemonSet：

```bash
kubectl rollout restart daemonset/metallb-speaker -n metallb-system
```

### 使用 Talos 原生 BGP 整合（進階）

Sidero Labs 官方文件說明了一種更深層的整合方式[^sidero-bgp]，MetalLB 可將 Service VIP 通告至繫結了 VRF 的 Talos BGP 實例。此方案要求：

- 設定 Talos 原生 BGP fabric 與 workload speaker 拓樸
- MetalLB 使用 FRR-K8s 模式，但停用 in-pod FRR
- 基於 IPv6 veth 的對等連線，搭配延伸 next-hop 能力
- 用 `FRRConfiguration` CRD 啟用延伸 next-hop

## 設定 IP 位址池與負載平衡模式

### Layer 2 模式

步驟一：定義 IP 位址池

```yaml
# ip-address-pool.yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.1.200-192.168.1.250   # 範圍格式
  # 或使用 CIDR：
  # - 10.0.0.192/28
  autoAssign: true                  # 自動指派給 Service（預設為 true）
```

步驟二：定義 L2 通告

```yaml
# l2-advertisement.yaml
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default
  namespace: metallb-system
spec:
  ipAddressPools:
  - default-pool
```

套用兩項資源：

```bash
kubectl apply -f ip-address-pool.yaml
kubectl apply -f l2-advertisement.yaml
```

### BGP 模式

步驟一：定義 IP 位址池（同 L2 模式）

步驟二：定義 BGP 對等連線

```yaml
# bgp-peer.yaml
apiVersion: metallb.io/v1beta2
kind: BGPPeer
metadata:
  name: router-1
  namespace: metallb-system
spec:
  myASN: 64512                    # MetalLB 的 AS 號碼
  peerASN: 64513                  # 路由器的 AS 號碼
  peerAddress: 192.168.1.1       # 路由器 IP
  peerPort: 179                   # 標準 BGP 連接埠
  # 選擇性：BFD 快速故障偵測
  bfdProfile: "fast"
  # 選擇性：MD5 認證
  # password: "your-bgp-password"
  # 選擇性：限定特定節點對等連線
  # nodeSelectors:
  # - matchLabels:
  #     rack: rack-1
```

步驟三：定義 BGP 通告

```yaml
# bgp-advertisement.yaml
apiVersion: metallb.io/v1beta1
kind: BGPAdvertisement
metadata:
  name: bgp-advert
  namespace: metallb-system
spec:
  ipAddressPools:
  - bgp-pool
  aggregationLength: 32            # 每個 Service IP 使用 /32
  localPref: 100                   # BGP local preference
  communities:
  - 64512:100                      # 選擇性 BGP communities
```

### Service 層級的精細控制

指定特定 IP 或位址池：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
  annotations:
    metallb.io/address-pool: production    # 指定目標位址池
    metallb.io/loadBalancerIPs: 192.168.1.201
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: my-app
```

多個 Service 共用同一 IP（不同連接埠）：

```yaml
annotations:
  metallb.io/allow-shared-ip: "shared-web"
  metallb.io/loadBalancerIPs: 192.168.1.200
```

## 在 Proxmox 虛擬機上的注意事項

若 Talos 叢集運作於 Proxmox VE 虛擬機上，需注意[^tazlab-proxmox]：

- Proxmox 的 MAC/IP 防火牆過濾器會阻擋 MetalLB 的 ARP 偽裝行為
- 解決方案：在 Proxmox 中停用 MAC 過濾器，或為 MetalLB IP 位址池建立 IPSet
- 設定路徑：Datacenter → Proxmox VE Firewall → IPSet → 新增 MetalLB 範圍規則

## 常見問題診斷

| 症狀 | 可能原因 | 解決方法 |
|---|---|---|
| Service 卡在 Pending | 未設定 IPAddressPool 或位址池已耗盡 | 檢查 `kubectl get ipaddresspools -n metallb-system` 與 controller 日誌 |
| 已指派外部 IP 但無法連線 | Pod Security 標籤遺失，或 Proxmox MAC 過濾器阻擋 ARP | 驗證命名空間標籤；停用 Proxmox MAC 過濾器或設定 IPSet |
| L2 Status 資源為空 | 控制平面節點被排除於 LB 宣告之外 | 設定 `speaker.ignoreExcludeLB: true` 並重新啟動 speaker |
| `kubectl get servicel2statuses` 回傳空值 | 所有節點皆為控制平面，帶有 `exclude-from-external-load-balancers` 標籤 | 參見上方修正方法 |
| ARP 顯示 `(incomplete)` | MetalLB speaker 未在任何節點上進行宣告 | 檢查 speaker 日誌；驗證節點標籤 |
| BGP 連線無法建立 | 連接埠 179 被封鎖、ASN 不匹配或 MTU 問題 | 使用 `talosctl netstat` 驗證連接埠 179；檢查 speaker 日誌 |
| 流量到達節點但未到達 Pod | CNI 衝突或 `externalTrafficPolicy` 設定錯誤 | 檢查 kube-proxy 規則與 Service endpoints |
| Speaker DaemonSet 無法啟動 | Pod Security Admission 阻擋了 privileged Pod | 為命名空間標記 privileged Pod Security 標準 |

快速診斷指令：

```bash
# 檢查 MetalLB controller 日誌
kubectl logs -n metallb-system -l app.kubernetes.io/component=controller --tail=50

# 檢查 speaker 日誌
kubectl logs -n metallb-system -l app.kubernetes.io/component=speaker --tail=50

# 檢查 L2 狀態
kubectl get servicel2statuses.metallb.io -n metallb-system

# 從同一區域網路的主機測試 ARP
arping <loadbalancer-ip>

# 確認節點 IP 與位址池無衝突
kubectl get nodes -o wide
```

## Talos 環境重點提醒

| 項目 | 說明 |
|---|---|
| **CNI 相容性** | MetalLB 在 L2 模式下可與 Cilium、Calico、Flannel 搭配使用。若使用 Cilium，須確認 Cilium 的 L2 Announcement 功能未啟用，否則會與 MetalLB 衝突。Cilium 亦可完全取代 kube-proxy，此時若僅需 L2 負載平衡，可使用 Cilium 內建功能，無需 MetalLB。 |
| **無 SSH / 傳統除錯方式** | Talos 為不可變基礎架構，無 SSH 亦無 shell。透過 `talosctl logs`、`kubectl logs`、`kubectl describe` 進行除錯。 |
| **無內建防火牆** | Talos 不執行傳統 Linux 防火牆。若已設定 Talos ingress firewall 或上游 ACL，請允許 MetalLB 相關流量。BGP 模式需確保 TCP 連接埠 179 開放。 |
| **kube-proxy 由 Talos 管理** | 不同於 kubeadm 直接編輯 ConfigMap 的方式，Talos 必須透過機器組態的 `cluster.proxy.config.ipvs.strictARP` 設定。變更後需執行 `talosctl upgrade-k8s` 以完整更新 kube-proxy。 |
| **無 systemd** | Talos 使用自有 init 系統。標準 Linux sysctl 調校應使用 `machine.sysctls` 組態區段，而非 init 指令稿。 |

## 授權模式比較

### Layer 2 模式

- **優點**：設定簡單，無需網路設備支援，任何 Ethernet 網路皆可運作
- **限制**：
  - 所有入口流量通過單一 Leader 節點，頻寬受限於該節點
  - 故障轉移仰賴用戶端 ARP 快取更新，通常需數秒時間
  - 不支援真正的跨節點負載分散[^metallb-l2]

### BGP 模式

- **優點**：
  - 路由器透過 ECMP 將流量分散至多個節點
  - 故障轉移快速（與 BGP 收斂時間一致）
  - 可搭配 BFD 實現次秒級故障偵測
- **限制**：
  - 需要網路設備支援 BGP 協定
  - 設定較為複雜
  - 需管理 BGP AS 號碼與對等連線

## 參考文獻

[^metallb-site]: MetalLB. (n.d.). MetalLB — Bare-metal load balancer for Kubernetes. Retrieved 2026-09-26, from https://metallb.io/
[^metallb-l2]: MetalLB. (n.d.). Layer 2 mode. Retrieved 2026-09-26, from https://metallb.io/concepts/layer2/
[^metallb-install]: MetalLB. (n.d.). Installation. Retrieved 2026-09-26, from https://metallb.universe.tf/installation/
[^metallb-config]: MetalLB. (n.d.). Configuration. Retrieved 2026-09-26, from https://metallb.io/configuration/
[^metallb-troubleshoot]: MetalLB. (n.d.). Troubleshooting. Retrieved 2026-09-26, from https://metallb.universe.tf/troubleshooting/
[^oneuptime-l2]: OneUptime. (2026-03-03). Set Up MetalLB Load Balancer on Talos Linux. Retrieved 2026-09-26, from https://oneuptime.com/blog/post/2026-03-03-set-up-metallb-load-balancer-on-talos-linux/view
[^oneuptime-bgp]: OneUptime. (2026-03-03). Set Up BGP Load Balancing with MetalLB on Talos Linux. Retrieved 2026-09-26, from https://oneuptime.com/blog/post/2026-03-03-set-up-bgp-load-balancing-with-metallb-on-talos/view
[^oneuptime-kubeproxy]: OneUptime. (2026-03-03). Configure kube-proxy on Talos Linux. Retrieved 2026-09-26, from https://oneuptime.com/blog/post/2026-03-03-configure-kube-proxy-on-talos-linux/view
[^sidero-bgp]: Sidero Labs. (n.d.). MetalLB BGP on Talos. Retrieved 2026-09-26, from https://docs.siderolabs.com/talos/v1.14/networking/bgp/metallb
[^sidero-patching]: Sidero Labs. (n.d.). Talos Configuration Patching. Retrieved 2026-09-26, from https://docs.siderolabs.com/talos/v1.14/configure-your-talos-cluster/system-configuration/patching
[^tazlab-proxmox]: Taz's Blog. (n.d.). Guide: TalOS with Proxmox and MetalLB. Retrieved 2026-09-26, from https://blog.tazlab.net/guides/talos-proxmox-metallb/
[^lelopez-fix]: LeLopez.io. (n.d.). Homelab v2 — MetalLB & Talos L2 Fix. Retrieved 2026-09-26, from https://lelopez.io/blog/homelab-v2-08a-metallb-talos-l2-fix/
[^simoncor]: Simon Cor. (n.d.). TalOS & MetalLB. Retrieved 2026-09-26, from https://docs.simoncor.net/talos/lb/