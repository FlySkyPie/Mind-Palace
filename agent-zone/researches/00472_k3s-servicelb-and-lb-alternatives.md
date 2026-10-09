# k3s 內建 ServiceLB 介紹與無預裝 ServiceLB 之替代方案

## 摘要

本文探討兩個主題：(1) k3s 內建的 ServiceLB（舊稱 Klipper LoadBalancer）是什麼、如何運作及其限制；(2) 若使用其他風味的 Kubernetes 發行版而沒有預裝 ServiceLB 時，有哪些替代方案可以讓 `LoadBalancer` 型態的 Service 正常運作。報告基於官方文件與專案原始碼，以 zh_TW 撰寫，所有事實皆有註腳標記來源。

---

## 1. 背景：為什麼需要 LoadBalancer Controller

在標準 Kubernetes 中，`Service` 物件的 `type: LoadBalancer` 若無任何 cloud controller manager 或負載平衡器控制器介入，其狀態會一直停留在 `<pending>`，無法取得外部 IP[^k8s-lb]。雲端供應商（如 AWS、GCP、Azure）會透過各自的 CCM 對應到商用的雲端負載平衡器；但在裸機（bare-metal）、VM、邊緣裝置等沒有雲端負載平衡器的環境，就需要自行安裝一個 LB 控制器來填補這個空缺。k3s 與 RKE2 正是透過內建的 ServiceLB 來解決此問題[^k3s-lb]。

---

## 2. k3s 內建 ServiceLB 是什麼

### 2.1 定位

ServiceLB（Service Load Balancer，舊稱 Klipper LoadBalancer）是隨 k3s 出廠、內嵌的負載平衡器控制器，是 k3s 內建 Cloud Controller Manager（CCM）的一部分[^k3s-lb][^klipper-repo]。它讓 k3s（與 RKE2）可以在**沒有雲端供應商、也不需外部 LB 方案**的情況下，直接提供 `LoadBalancer` 型態的 Service[^k3s-lb]。

官方文件描述：

> "By default, K3s provides a load balancer known as ServiceLB (formerly Klipper LoadBalancer) that uses available host ports."（預設情況下，K3s 提供一個名為 ServiceLB 的負載平衡器，它使用可用的 host port）[^k3s-lb]

其主要元件：
- 運運時映像：`rancher/klipper-lb`，原始碼在 [github.com/k3s-io/klipper-lb](https://github.com/k3s-io/klipper-lb)[^klipper-repo]
- 控制器程式碼：k3s 原始碼樹中的 `pkg/cloudprovider/servicelb.go`[^k3s-servicelb-go]

GitHub 星數：klipper-lb 約 437 顆星[^stars-klipper]，k3s 約 3.4 萬顆星[^stars-k3s]。

### 2.2 運作機制

ServiceLB 的運作流程如下[^klipper-readme][^k3s-servicelb-go][^agent1]：

1. **偵測 Service**：控制器監看 `type: LoadBalancer` 的 Service，同時監看節點、Pod 與 EndpointSlice 以反應變更。
2. **自動建立 DaemonSet**：對每個 LoadBalancer Service，在 `kube-system` 命名空間建立一個命名為 `svclb-<service-name>-<uid前8碼>` 的 DaemonSet。
3. **Pod 設定**：每個 klipper-lb Pod 的容器命名為 `lb-<protocol>-<port>`，使用 `hostPort` 綁定節點上的 Service port，並具備 `NET_ADMIN` capability 以操作 iptables，同時設定 `net.ipv4.ip_forward` 等 sysctl，並容忍（tolerate）control-plane 節點以便在上面運作。
4. **iptables 轉送規則**：klipper-lb 容器內的 shell script 建立 DNAT/MASQUERADE 規則，把抵達節點 `hostPort` 的流量轉送至 Service 的 ClusterIP（預設 `externalTrafficPolicy: Cluster`）或本地 NodePort（`externalTrafficPolicy: Local`）。
5. **填充狀態**：控制器把所有已就緒 ServiceLB Pod 所在節點的 IP 填入 Service 的 `status.loadBalancer.ingress`。若節點有 ExternalIP 則優先列出，否則使用 InternalIP；rootless 模式則回傳 `127.0.0.1`[^agent1]。

簡言之，ServiceLB 本質上是一個**零設定、以 host port + iptables DNAT 為基礎的輕量級代理**，讓使用者建立 `LoadBalancer` Service 即可取得對外可連線的節點 IP。

### 2.3 限制與已知問題

ServiceLB 並未提供完整負載平衡功能，主要限制如下[^k3s-lb][^agent1]：

- **使用 host port**：每個 LoadBalancer Service 會佔用每個符合條件的節點上一個 port。若兩個 Service 需要相同 port（例如 80），在相同節點上就無法共存；若無可用 host port，ServiceLB 狀態會停留在 pending[^agent1]。
- **沒有專屬虛擬 IP**：不同於 MetalLB 會配置真正的專用 IP，ServiceLB 直接暴露節點 IP。若某一節點掛掉，使用該節點 IP 的 Service 對早已快取該 IP 的用戶端而言將無法連線。
- **非真正的流量負載平衡**：ServiceLB 本身不做流量分配，而是依賴用戶端在多節點 IP 間分散請求，或依賴 kube-proxy（iptables/ipvs）在流量到達 ClusterIP 後進行後端分配。
- **無進階功能**：沒有後端健康檢查、流量加權/切分、黏著性 session、TLS 終止或速率限制。
- **與其他 LB 互斥**：若要使用功能更完整的方案（MetalLB、kube-vip 等），須以 `--disable=servicelb` 停用 ServiceLB[^k3s-disable][^agent1]。
- **iptables 版本敏感**：進入點 script 會偵測 nft/legacy 模式，但某些舊版 iptables 有已知問題。
- **rootless 限制**：rootless 模式僅回傳 `127.0.0.1`，且 1024 以下的 port 會偏移 +10000。

---

## 3. 無預裝 ServiceLB 的替代方案

若使用其他風味的 Kubernetes（例如原生 K8s、kubeadm、MicroK8s、KubeKey 等）而未預裝 ServiceLB 時，有以下主要替代方案可讓 `LoadBalancer` Service 運作[^agent2]。

### 3.1 MetalLB

**狀態：** 成熟、廣為採用，CNCF graduated 專案（v0.16.x），約 8.4k 顆星[^stars-metallb]。

**運作方式：** 由兩部分組成：`controller` 負責從 IP 池分配 IP，`speaker`（DaemonSet）負責向網路宣告這些 IP。支援兩種模式[^metallb-concepts]：

- **Layer 2 模式（ARP/NDP）**：每個 Service 由一個節點競選為 leader，該節點回應服務 IP 的 ARP（IPv4）或 NDP（IPv6）查詢。所有流量先進該節點，再由 kube-proxy 分散至後端。leader 掛掉時透過 memberlist 重新競選並發送 gratuitous ARP 更新網路[^metallb-l2]。
- **BGP 模式：** 每個節點與上游路由器建立 BGP session 並宣告服務 IP，路由器用 ECMP（等價多路徑）在節點間分散流量，達到真正的多節點負載平衡。支援 FRR-K8s（推薦，含 BFD 與 IPv6）、Native 與已棄用的 FRR 三種 BGP 後端[^metallb-bgp]。

**優點：** 成熟可靠；Layer 2 模式在任何 Ethernet 網路皆可用、無需特殊路由器；BGP 模式提供真正的多節點平衡；支援雙棧（IPv4+IPv6）；IP 池管理豐富；支援 v1.24+ 的 `loadBalancerClass` 可與其他 LB 共存[^metallb-concepts]。

**缺點：** Layer 2 模式單節點瓶頸且容錯依賴用戶端 ARP cache 更新；BGP 模式需要支援 BGP/ECMP 的路由器，且節點掛掉時既有 TCP 連線會中斷（ECMP 重新雜湊的固有限制）；使用 IPVS 時需在 kube-proxy 開啟 `strictARP`[^metallb-concepts][^agent2]。

### 3.2 kube-vip

**狀態：** 活躍、成熟（v0.8.x），約 3.0k 顆星[^stars-kube-vip]。

**運作方式：** 可同時扮演兩個角色：(a) 為 Kubernetes **控制平面**提供 HA 的 VIP（API server 高可用），(b) 提供 `LoadBalancer` Service IP。IP 管理需搭配 kube-vip-cloud-provider[^kube-vip][^kv-cloud-provider]。支援模式[^agent2]：

- **ARP（Layer 2）**：透過 leader election 由單一 leader 持有 VIP 回應 ARP 查詢；可開啟 `svc_election=true` 實現「每個 Service」各自的 leader，把不同 Service 分散到不同節點。
- **BGP（Layer 3）**：所有節點透過 BGP 向路由器宣告 VIP。
- **Routing Table 模式**：管理核心路由表位址，配合 ECMP 使用。
- **WireGuard 模式**：透過 WireGuard interface 廣告 Service（多用於多叢集）。

**優點：** 單一專案同時解決控制平面 HA 與 Service 負載平衡；「每 Service」leader election 可分散流量；支援 `externalTrafficPolicy: Local`（啟用每 Service election 時）以保留來源 IP；獨特的 DHCP IP 分配、UPnP 埠轉發、WireGuard 模式；支援 DDNS/DNS；輕量且支援多架構（x86、ARM、ppc64le），適合肥岸邊/樹莓派等邊緣環境[^agent2]。

**缺點：** 完整 IPAM 需額外部署 kube-vip-cloud-provider；ARP 模式對單一 Service 仍有單節點瓶頸；BGP 模式同樣有 ECMP 重新雜湊問題；社群規模小於 MetalLB；文件分散於主專案、cloud-provider 與部落格。

### 3.3 Cilium（eBPF 內建 LB）

**狀態：** 非常活躍、CNCF graduated（v1.20.x），約 2.56 萬顆星[^stars-cilium]。Cilium 是完整的 CNI + 網路安全 + 負載平衡方案，若叢集已用或可改用 Cilium，其內建功能可直接取代 ServiceLB[^agent2]：

- **Kube-Proxy Replacement**：Cilium 可完全取代 kube-proxy，用 eBPF 處理 ClusterIP、NodePort 與 LoadBalancer Service，不再需要 iptables/IPVS，效能更佳[^cilium-k8s]。
- **L2 Announcements（Beta）**：透過 Kubernetes leader election（lease）回應本機網路上 LoadBalancer IP 的 ARP/NDP 查詢，每個服務 IP 同時僅由一個節點宣告。以 `CiliumL2AnnouncementPolicy` CRD 設定，並支援細緻的 service/node selector 與介面比對[^cilium-l2]。
- **BGP Control Plane**：Cilium 原生內建 BGP 控制平面（無需另外的路由軟體），可向上游路由器宣告 Pod CIDR 與 LoadBalancer IP，以 `CiliumBGPClusterConfig` 與 `CiliumBGPPeerConfig` CRD 設定[^cilium-bgp]。
- **LB IPAM**：內建的 LoadBalancer IP 位址管理，以 `CiliumLoadBalancerIPPool` CRD 從 CIDR 或範圍分配 IP，支援 service selector、雙棧、IP 共享與衝突偵測[^cilium-lb-ipam]。

**優點：** 一站式方案；eBPF 效能優於 iptables 方案；政策粒度細；雙棧內建；支援 IP 共享；BGP 完全 CRD 驅動；內建 Hubble 可觀測性[^agent2]。

**缺點：** 專案龐大，若只需要 LB 是較重的依賴；L2 Announcements 仍為 Beta；BGP Control Plane 較新、未如 MetalLB BGP 久經考驗；部分功能需 kernel 5.10+；學習曲線較陡；不易替換已運行其他 CNI 的叢集。

### 3.4 Cloud Controller Manager（CCM）／外部負載平衡器整合

此類並非獨立的 LB 實作，而是實作 Kubernetes cloud-provider 介面的控制器：它監看 `LoadBalancer` Service，並設定外部的硬體或軟體負載平衡器將流量路由到叢集節點，再填入 `status.loadBalancer.ingress`[^agent2]。例子包括：

- **kube-vip-cloud-provider**（與 kube-vip 搭配，負責 IPAM）[^kv-cloud-provider]
- **MetalLB** 同時扮演 controller 與 speaker，本身不需額外 CCM
- **Keepalived + HAProxy / NGINX**：在專用 LB 節點上以 VRRP 與 HAProxy/NGINX 提供 HA
- **OpenELB**（前身 PorterLB）：CNCF sandbox 專案，提供 LB IP 分配並可整合 BGP 路由器，約 1.8k 顆星[^stars-openelb]
- **PureLB**：較簡單的 MetalLB 替代，採 CCM 模型搭配自己的 LB 節點 agent，約 132 顆星[^stars-purelb]
- **各家硬體 / 軟體供應商 CCM**：例如 F5 BIG-IP CCM、VMware AVI CCM、Citrix CCM

**優點：** 可獲得企業級外部 LB 的完整流量控制、健康檢查、session 保持；能處理叢集內方案無法應付的大規模；可與叢集內方案並存不同 Service[^agent2]。

**缺點：** 需要維護額外硬體或軟體；零件更多、部署與維運較複雜；外部 LB 若非 HA 設定會成為單點故障；多數 CCM 為供應商綁定。

---

## 4. 結論與選型建議

### 4.1 ServiceLB 定位

ServiceLB 最大的價值是**免設定**——在 k3s/RKE2 上建立 `LoadBalancer` Service 即可取得對外 IP，非常適合開發、邊緣與小規模部署。其限制（無專屬虛擬 IP、使用 host port、無進階功能）使其在需要穩定 VIP、多叢集 ingress 或進階流量管理的正式生產環境中可能不足[^agent1]。

### 4.2 選型建議

選項綜合對照如下[^agent2]：

| 方案 | L2 ARP/NDP | BGP | 每 Service 選舉 | eBPF | 控制平面 HA | DHCP/UPnP | 雙棧 | 成熟度 | 複雜度 |
|---|---|---|---|---|---|---|---|---|---|
| MetalLB | ✅ | ✅ | 否（memberlist） | 否 | 否 | 否 | ✅ | 非常成熟 | 低-中 |
| kube-vip | ✅ | ✅ | ✅（svc_election） | 否 | ✅ | ✅ | ✅ | 成熟 | 中 |
| Cilium | ✅（Beta） | ✅（內建） | ✅（lease） | ✅ | 否 | 否 | ✅ | 成熟（LB 功能較新） | 高 |
| CCM/外部 LB | 依外部 LB | 依外部 LB | 否 | 否 | 依外部設定 | 依外部設定 | 依外部 | 視供應商 | 中-高 |

具體建議：

- **想要直接替換 ServiceLB、低複雜度** → **MetalLB** 的 Layer 2 模式。最成熟、文件最完整、任何網路皆可用。
- **同時需要控制平面 HA（API server 的 VIP）** → **kube-vip**。可同時解決兩問題，並具備每 Service 選舉、DHCP、UPnP 等特色。
- **已使用（或願改為）Cilium 作為 CNI** → 直接使用 **Cilium 內建的 L2 Announcements + LB IPAM + BGP Control Plane**，一站式且 eBPF 效能佳。
- **想與現有路由器 BGP 整合、追求真正的多節點負載平衡** → **MetalLB BGP 模式（FRR-K8s）** 或 **Cilium BGP Control Plane**。
- **已有企業級 LB 硬體（F5、AVI 等）** → 使用該**供應商的 CCM**進行完整整合。

---

## 參考文獻

[^k8s-lb]: Kubernetes. (n.d.). Service — Load balancing. Retrieved 2026-10-09, from https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer
[^k3s-lb]: K3s Documentation. (n.d.). Service Load Balancer. Retrieved 2026-10-09, from https://docs.k3s.io/networking/networking-services#service-load-balancer
[^k3s-disable]: K3s Documentation. (n.d.). Agent flags — --disable. Retrieved 2026-10-09, from https://docs.k3s.io/cli/agent#disable
[^klipper-repo]: k3s-io. (n.d.). klipper-lb. Retrieved 2026-10-09, from https://github.com/k3s-io/klipper-lb
[^klipper-readme]: k3s-io. (n.d.). klipper-lb — README. Retrieved 2026-10-09, from https://github.com/k3s-io/klipper-lb/blob/master/README.md
[^k3s-servicelb-go]: k3s-io. (n.d.). k3s — pkg/cloudprovider/servicelb.go. Retrieved 2026-10-09, from https://github.com/k3s-io/k3s/blob/main/pkg/cloudprovider/servicelb.go
[^metallb-concepts]: MetalLB. (n.d.). MetalLB concepts. Retrieved 2026-10-09, from https://metallb.universe.tf/concepts/
[^metallb-l2]: MetalLB. (n.d.). Layer 2 mode. Retrieved 2026-10-09, from https://metallb.universe.tf/concepts/layer2/
[^metallb-bgp]: MetalLB. (n.d.). BGP mode. Retrieved 2026-10-09, from https://metallb.universe.tf/concepts/bgp/
[^kube-vip]: kube-vip. (n.d.). kube-vip — Load balancer for Kubernetes. Retrieved 2026-10-09, from https://kube-vip.io/
[^kv-cloud-provider]: kube-vip. (n.d.). kube-vip-cloud-provider. Retrieved 2026-10-09, from https://github.com/kube-vip/kube-vip-cloud-provider
[^cilium-k8s]: Cilium. (n.d.). Kubernetes networking. Retrieved 2026-10-09, from https://docs.cilium.io/en/stable/network/kubernetes/
[^cilium-l2]: Cilium. (n.d.). L2 announcements / L2 aware load balancing. Retrieved 2026-10-09, from https://docs.cilium.io/en/stable/network/l2-announcements/
[^cilium-bgp]: Cilium. (n.d.). BGP Control Plane. Retrieved 2026-10-09, from https://docs.cilium.io/en/stable/network/bgp-control-plane/bgp-control-plane/
[^cilium-lb-ipam]: Cilium. (n.d.). LoadBalancer IP Address Management (LB IPAM). Retrieved 2026-10-09, from https://docs.cilium.io/en/stable/network/lb-ipam/
[^agent1]: K3s-io. (2026). k3s 內建 ServiceLB 機制整理（Web research synthesis）. Retrieved 2026-10-09, from https://github.com/k3s-io/klipper-lb/blob/master/entry
[^agent2]: MetalLB, kube-vip, & Cilium. (2026). 無預裝 ServiceLB 之替代方案整理（Web research synthesis）. Retrieved 2026-10-09, from https://metallb.universe.tf/concepts/
[^stars-klipper]: GitHub. (2026). k3s-io/klipper-lb. Retrieved 2026-10-09, from https://github.com/k3s-io/klipper-lb
[^stars-k3s]: GitHub. (2026). k3s-io/k3s. Retrieved 2026-10-09, from https://github.com/k3s-io/k3s
[^stars-cilium]: GitHub. (2026). cilium/cilium. Retrieved 2026-10-09, from https://github.com/cilium/cilium
[^stars-metallb]: GitHub. (2026). metallb/metallb. Retrieved 2026-10-09, from https://github.com/metallb/metallb
[^stars-kube-vip]: GitHub. (2026). kube-vip/kube-vip. Retrieved 2026-10-09, from https://github.com/kube-vip/kube-vip
[^stars-openelb]: GitHub. (2026). openelb/openelb. Retrieved 2026-10-09, from https://github.com/openelb/openelb
[^stars-purelb]: GitHub. (2026). purelb/purelb. Retrieved 2026-10-09, from https://github.com/purelb/purelb
