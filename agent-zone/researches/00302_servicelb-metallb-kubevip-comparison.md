# ServiceLB (Klipper LoadBalancer)、MetalLB 與 kube-vip 比較

## 概述

Kubernetes 在裸機（bare-metal）或地端（on-premise）環境中，缺少原生的 `type: LoadBalancer` Service 實作，因此 `LoadBalancer` 類型的 Service 會永遠停留在 `Pending` 狀態。ServiceLB（Klipper LoadBalancer）、MetalLB 與 kube-vip 都是為了解決這個問題而誕生的工具，但它們的設計理念、功能範圍與適用場景各不相同。

---

## 1. ServiceLB（Klipper LoadBalancer）

### 是什麼

ServiceLB 是 **K3s**（以及 RKE2）內建附帶的負載平衡器控制器，前稱 Klipper LoadBalancer。其運行時映像檔來自 [k3s-io/klipper-lb](https://github.com/k3s-io/klipper-lb) 倉庫。[^servicelb-k3s]

### 運作方式

ServiceLB 控制器會監聽 `type: LoadBalancer` 的 Service，針對每一個 LoadBalancer Service 在 `kube-system` 命名空間中建立一個 **DaemonSet**。這個 DaemonSet 的 Pod 使用 `hostPort` 直接綁定到節點的通訊埠，且僅會在有**可用通訊埠**的節點上執行。Pod 內部使用 **iptables** 將流量轉發到 Service 的 ClusterIP。如果沒有任何節點擁有該通訊埠，LoadBalancer 會維持 `Pending` 狀態。[^servicelb-k3s]

### 關鍵特性

- **無 IP 池管理**：不分配獨立 IP，直接使用節點現有的 IP。
- **通訊埠層級競爭**：多個 Service 只有在使用不同通訊埠時才能共存於同一節點。
- **節點選擇**：可透過 `svccontroller.k3s.cattle.io/enablelb=true` 標籤控制，也支援 `lbpool` 標籤來區分節點池。
- **無真正負載平衡**：流量只會落在單一節點的通訊埠上，再透過 iptables/kube-proxy 轉發到 ClusterIP。
- **K3s 專屬**：非通用 Kubernetes 解決方案，僅內建於 K3s / RKE2。

### 優缺點

| 優點 | 缺點 |
|---|---|
| **零配置** — 在 K3s 開箱即用，無需 CRD、IP 池或路由器設定 | **無獨立外部 IP** — 使用節點 IP，而非分配專屬 VIP |
| **與 K3s 深度整合** — 屬於內建雲端控制器管理器的一部分 | **通訊埠衝突** — 使用 `hostPort`，多個 Service 會競爭通訊埠 |
| **簡單輕量** — 無需部署額外元件 | **單節點瓶頸** — 所有流量經由單一節點進入 |
| **節點池支援** — 可透過標籤限制 Service 執行的節點 | **無 BGP / Layer 2 協定** — 無法與現有網路基礎設施整合 |
| | **不適合生產規模使用** — 較偏向小型 K3s 叢集的便利功能 |

---

## 2. MetalLB

### 是什麼

MetalLB 是最廣泛使用的裸機 LoadBalancer 解決方案，屬於 **CNCF 沙盒專案**，以 Go 語言撰寫，由社群積極維護。[^metallb-site]

### 運作方式

MetalLB 由兩個元件組成：[^metallb-concepts]

1. **Controller**（Deployment）— 監聽 `LoadBalancer` Service，從設定的位址池中分配 IP。
2. **Speaker**（DaemonSet）— 在每個節點上執行，負責廣播分配的 IP。

MetalLB 支援兩種操作模式：[^metallb-layer2][^metallb-bgp]

**Layer 2 模式（ARP/NDP）：**
- 單一節點（領導者）取得 Service IP 的所有權，回應區域網路上的 ARP（IPv4）或 NDP（IPv6）請求。
- 領導者選舉是**無狀態的**——每個 speaker 計算 `node+VIP` 的排序雜湊來決定領導者，無需持久化狀態。
- 領導者失效時，memberlist 會偵測到並選出新領導者。
- **限制**：所有流量流向單一節點，頻寬受限於該節點。故障轉移仰賴客戶端接受無償 ARP 封包（通常有 1-3 秒黑洞期）。

**BGP 模式：**
- 每個節點與上游路由器建立 BGP peer 連線，廣播 Service IP。
- 路由器使用 **ECMP**（等價多路徑）將流量分散到所有節點。
- 預設使用 **FRR-K8s**（Free Range Routing）作為 BGP 後端，支援 **BFD**（雙向轉送偵測）以實現快速故障偵測。
- **限制**：BGP 是無狀態的——當節點下線時，該節點上所有進行中的連線都會中斷（Connection reset by peer）。緩解策略包括彈性 ECMP 雜湊、入口緩衝或客戶端重試邏輯。

### 配置方式

從 v0.13.0 開始，完全使用 Kubernetes CRD（`IPAddressPool`、`L2Advertisement`、`BGPPeer`、`BGPAdvertisement`），舊的 ConfigMap 方式已棄用。[^metallb-site]

### 優缺點

| 優點 | 缺點 |
|---|---|
| **成熟且經實戰驗證** — CNCF 沙盒專案，最廣泛使用，社群龐大 | **不含控制平面 HA** — 僅處理 Service LoadBalancer，API Server VIP 需另尋方案（kube-vip 或 keepalived） |
| **CNI 無關** — 支援 Flannel、Calico、Cilium 等任何 CNI | **Layer 2 模式 = 單節點瓶頸** — 每個 Service 頻寬受限於單一節點 |
| **Layer 2 模式** — 任何 Ethernet 網路皆可運作，不需要特殊硬體 | **BGP 模式 = 節點故障時連線中斷** — 無狀態 ECMP，後端變動時連線重置 |
| **BGP 模式** — 透過 ECMP 實現真正的跨節點負載平衡，可與現有網路基礎設施整合 | **需要路由器支援 BGP** — 並非所有地端網路都具備 BGP 能力 |
| **專屬 IP 池** — 從可設定的範圍分配實際的外部 IP | **設定較 ServiceLB 複雜** — 需要 IP 池規劃、CRD 定義，以及可能的 BGP 配置 |
| **支援 IPv4 與 IPv6** — 完整雙堆疊支援 | |
| **CRD 宣告式配置** — 適合 GitOps 工作流程 | |
| **FRR-K8s 整合 BGP 與 BFD 支援** — 快速故障偵測 | |
| **無通訊埠衝突** — 每個 Service 有獨立 IP | |

---

## 3. kube-vip

### 是什麼

kube-vip 是一個**雙功能工具**，同時提供**控制平面高可用性**（API Server VIP）和 **LoadBalancer Service** 能力。以 Go 撰寫。[^kubevip-site]

### 運作方式

kube-vip 可以作為 **static pod**（用於控制平面 VIP）或 **DaemonSet**（用於 LoadBalancer Service）執行。同一個二進位檔即可處理兩種角色。[^kubevip-site]

支援的模式：[^kubevip-site]
- **ARP/NDP（Layer 2）** — Service IP 的領導者選舉，類似 MetalLB L2。
- **BGP** — 向上游路由器廣播路由。
- **Routing Table** — 用於 ECMP 設定，管理路由表中位址的新增與刪除。
- **WireGuard** — 通過 WireGuard 介面廣播 Service（適用於多叢集或分散式設定）。

配置方式使用環境變數與 CLI 旗標，而非 CRD。透過 Docker 指令產生 manifest。

### 優缺點

| 優點 | 缺點 |
|---|---|
| **一魚兩吃** — 單一工具同時提供控制平面 VIP 與 Service LoadBalancer | **Service LB 功能較 MetalLB 不成熟** — 功能較少 |
| **輕量** — 單一二進位檔，記憶體佔用低於 MetalLB（總計約 144 MiB vs MetalLB 約 250 MiB） | **無 CRD 配置** — 較不適合 GitOps 流程 |
| **多種模式** — ARP、BGP、Routing Table、WireGuard | **IP 池管理較粗略** — 不如 MetalLB 的 IPAddressPool CRD 靈活 |
| **無需 CRD** — 透過旗標與環境變數配置 | **BGP 功能較基本** — 不如 MetalLB 的 FRR-K8s 整合 |
| **內建控制平面 HA** — 完全取代 keepalived | **社群較小** — Service LB 功能的實戰驗證較少 |
| **DHCP 支援** — 可從現有 DHCP 基礎設施取得 IP | |

---

## 總結比較表

| 特性 | ServiceLB (Klipper LB) | MetalLB | kube-vip |
|---|---|---|---|
| **主要角色** | 僅 Service LB | 僅 Service LB | 控制平面 VIP + Service LB |
| **適用範圍** | 僅 K3s/RKE2 | 任何 Kubernetes | 任何 Kubernetes |
| **成熟度** | 便利功能 | CNCF 沙盒，經實戰驗證 | 廣泛使用，較 MetalLB 年輕 |
| **Layer 2 模式** | 無（僅 hostPort） | 有（ARP/NDP） | 有（ARP/NDP） |
| **BGP 模式** | 無 | 有（FRR-K8s + BFD） | 有（原生實作） |
| **ECMP 支援** | 無 | 有（BGP） | 有（BGP + routing table） |
| **CRD 配置** | 無（僅標籤） | 有 | 無（環境變數/旗標） |
| **專屬 IP 池** | 無（使用節點 IP） | 有 | 有 |
| **控制平面 VIP** | 無 | 無 | 有（內建） |
| **WireGuard 模式** | 無 | 無 | 有 |
| **CNI 無關** | 是 | 是 | 是 |
| **資源使用** | 非常低 | 中等（~250 MiB） | 低（~144 MiB 總計） |
| **設定複雜度** | 非常低 | 低-中 | 低-中 |
| **通訊埠衝突** | 有（hostPort） | 無 | 無 |

---

## 適用場景建議

### 使用 ServiceLB（Klipper LB）當：
- 你正在執行 **K3s** 且需求單純（homelab、開發環境、小型部署）
- 你想要**零配置**——建立 LoadBalancer Service 就能用
- 你不需要專屬外部 IP 或多節點流量分配
- 你能夠接受潛在通訊埠衝突

### 使用 MetalLB 當：
- 你想要 **最成熟、最廣泛使用、經實戰驗證** 的解決方案
- 你需要 **Layer 2 模式**（適用於任何網路，無需路由器變更）
- 你需要 **BGP 模式**來實現真正的跨節點 ECMP 負載平衡
- 你想要 **CRD 宣告式配置**（適合 GitOps）
- 你擁有 **具備 BGP 能力的網路基礎設施**
- 你的控制平面 HA 已有其他方案處理（kube-vip、keepalived 等）

### 使用 kube-vip 當：
- 你需要 **同時解決控制平面 HA 和 Service LoadBalancer**
- 你想要 **最小化元件數量**（用一個工具取代 keepalived + MetalLB）
- 你正在建置**全新叢集**，本來就需要 API Server VIP
- 你想要 **WireGuard 模式**用於多叢集或分散式設定
- 你偏好簡單的**環境變數 / 旗標配置**勝於 CRD

### 常見部署組合

1. **簡單 K3s 叢集**：ServiceLB（內建，零配置）
2. **生產環境裸機 + 完整網路**：kube-vip 做控制平面 VIP + MetalLB 做 Service LB（或 kube-vip 兩者全包）
3. **最少元件**：kube-vip 同時處理控制平面 HA 和 Service LB
4. **企業環境 + BGP 基礎設施**：MetalLB（BGP 模式）做 Service，kube-vip 或 keepalived 做控制平面

---

[^servicelb-k3s]: K3s Documentation. (n.d.). *Networking — Networking Services*. Retrieved 2026-09-26, from https://docs.k3s.io/networking/networking-services

[^metallb-site]: MetalLB. (n.d.). *MetalLB — Bare-metal load balancer for Kubernetes*. Retrieved 2026-09-26, from https://metallb.universe.tf/

[^metallb-concepts]: MetalLB. (n.d.). *Concepts — Overview*. Retrieved 2026-09-26, from https://metallb.universe.tf/concepts/

[^metallb-layer2]: MetalLB. (n.d.). *Concepts — Layer 2*. Retrieved 2026-09-26, from https://metallb.universe.tf/concepts/layer2/

[^metallb-bgp]: MetalLB. (n.d.). *Concepts — BGP*. Retrieved 2026-09-26, from https://metallb.universe.tf/concepts/bgp/

[^kubevip-site]: kube-vip. (n.d.). *kube-vip — A Kubernetes control-plane and LoadBalancer VIP*. Retrieved 2026-09-26, from https://kube-vip.io/