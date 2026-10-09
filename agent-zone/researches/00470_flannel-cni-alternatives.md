# k3s 內建的 Flannel 是什麼？非 k3s 發行版的替代方案

## 摘要

Flannel 是 k3s 預設內建的網路解決方案。本文說明 Flannel 的本質、運作原理，並列出其他 Kubernetes（K8s）發行版若未預裝 Flannel 時可用的 CNI（Container Network Interface）替代方案，包括 Calico、Cilium、Antrea、Canal、kube-router 等，並比較各自特性。

---

## 一、Flannel 是什麼？

Flannel 是由 CoreOS 最初開發的輕量級層 3（Layer 3）網路 fabric，實作 Kubernetes 的 CNI 介面，是廣義上所謂的「CNI 外掛（CNI Plugin）」。其目標是為叢集中每個 Pod 提供一個跨節點皆可路由的獨立 IP 位址，方式是在節點間建立 overlay 網路並進行封包封裝與轉送[^k3s_net][^flannel]。

### 運作原理

- 一個名為 `flanneld` 的代理程式（agent）在叢集的每個節點上執行[^calico_canal]。
- 它會從一個較大的預設網段（例如 `10.244.0.0/16`）為每個主機分配一個子網路租約（subnet lease）[^k8s_cni_alt]。
- 網路設定、子網分配等狀態可存放於 Kubernetes API（所謂「kube subnet manager」模式）或獨立的 etcd 叢集[^k8s_cni_alt]。
- 封包在不同主機間轉送時，依後端機制（backend）不同而異，常見為 **VXLAN**（透過 UDP 封裝第二層畫框）與 **host-gw**（直接在主機建立路由，無封裝、效能較好，但要求所有節點在同一第二層網路上）[^k3s_net][^k8s_cni_alt]。

### 重要特性

- **單純且輕量**，以 `kubectl apply` 即可部署[^k8s_cni_alt]。
- **本身不原生支援 Network Policy**（網路政策）；它專注於連通性。若要政策強制，需搭配 Calico（見下方 Canal）或部署 Kubernetes SIGs 的 network policy controller[^k8s_cni_alt]。
- 除了 VXLAN 與 host-gw，k3s 也可設定使用 `wireguard-native`（加密）或 `ipsec` 後端[^k3s_net]。

---

## 二、k3s 中的 Flannel

k3s 預設即隨附並執行 Flannel，這是其「開箱即用（batteries-included）」設計的一部分。透過下列 CLI 旗標可切換其行為[^k3s_net]：

- `--flannel-backend=vxlan`（預設）、`host-gw`、`wireguard-native`、`ipsec`（已標記棄用）。
- `--flannel-backend=none` 可停用 Flannel，改用其他自訂 CNI。
- `--flannel-iface`、`--flannel-conf` 等可覆寫各節點上的 Flannel 介面或設定檔。

當使用 `--flannel-backend=none` 並安裝自訂 CNI 時，由於多數 CNI 自帶網路政策引擎，官方建議一併 `--disable-network-policy` 以避免衝突[^k3s_net]。

> 補充：k3s 的文件雖偏向以 k3s 為出發點，但 CNI 的替代選項在「其他口味的 K8s」上同樣適用，因為 CNI 是 Kubernetes 開放且標準化的介面。

---

## 三、非 k3s 發行版（未預裝 Flannel）的替代方案

Kubernetes 本身不內建任何特定 CNI；若發行版未預裝 Flannel，可自行新增 CNI 外掛。以下為常見替代方案（並附 GitHub star 數，資料抓取時間 2026-10-09）[^k8s_addons][^cni_stars]。

### 1. Calico（Project Calico）

- 最廣泛採用的 Kubernetes CNI 之一，提供容器網路、網路安全與可觀測性的統一平台[^calico_site][^k8s_cni_alt]。
- 使用 BGP 做直接（非 overlay）路由，也支援 VXLAN/IPIP overlay；以 iptables、nftables 或 eBPF 強制 Network Policy（L3–L4）[^k8s_cni_alt]。
- 可支援多種資料面（eBPF、iptables、nftables、VPP、Windows）[^k8s_cni_alt]。
- GitHub star 約 7.4k[^cni_stars]。

### 2. Cilium

- 以 eBPF（extended Berkeley Packet Filter）做為基礎的網路、可觀測性與安全方案[^k8s_cni_alt]。
- 支援 overlay（VXLAN、Geneve）或原生命中轉路由模式；可完全取代 kube-proxy 進行負載平衡[^k8s_cni_alt]。
- 強制 L3–L7 Network Policy（HTTP、gRPC、DNS aware），內建 Hubble 可觀測平台，支援 Cluster Mesh 跨叢集連通，並可做 IPsec、WireGuard 透明加密[^k8s_cni_alt]。
- CNCF **Graduated**（畢業）專案，GitHub star 約 25.6k（上述選項中最高）[^k8s_cni_alt][^cni_stars]。

### 3. Canal

- 並非獨立 CNI，而是 **Flannel（負責網路）+ Calico（負責 Network Policy）** 的組合，又稱「Calico for policy and Flannel for networking」[^k8s_cni_alt][^calico_canal]。
- 使用 Flannel 的 VXLAN overlay 做 Pod 對 Pod 連通，同時以 Calico 的政策引擎做安全強制[^k8s_cni_alt]。
- 適合想要 Flannel 的簡單性又需要 Network Policy 的團隊[^k8s_cni_alt]。

### 4. Antrea

- 以 **Open vSwitch（OVS）** 做為資料面的 Kubernetes 原生 CNI[^k8s_cni_alt]。
- VXLAN/Geneve overlay 網路；以 OVS 強制 Kubernetes Network Policy（規模上比 iptables 更快）；支援 IPsec 加密與硬體 offload（如 NVIDIA SmartNIC）[^k8s_cni_alt]。
- CNCF Sandbox 專案，GitHub star 約 1.8k[^k8s_cni_alt][^cni_stars]。

### 5. kube-router

- 單一二進位/DaemonSet 同時處理 pod network、service proxy、network policy 與 load balancer IP 分配[^k8s_cni_alt]。
- 以 BGP（GoBGP）做直接 Pod 路由（無 overlay）；以 IPVS/LVS 取代 kube-proxy；以 iptables/ipsets 強制 Network Policy[^k8s_cni_alt]。
- 曾被提及用於 k0s 與 k3s 環境，GitHub star 約 2.5k[^k8s_cni_alt][^cni_stars]。

### 6. 其他（已歸檔或較小眾）

- **Weave Net**：以 VXLAN 與 OVS fast data path 做 overlay，曾很流行（star 約 6.6k），但專案已在 2024 年 6 月 20 日被擁有者歸檔（read-only），不再積極維護，實際上已算棄用[^k8s_cni_alt][^cni_stars]。
- **OVN-Kubernetes**：以 OVN（Open Virtual Network）與 Open vSwitch 為基礎，由 OpenShift 使用[^k8s_cni_alt]。
- **Multus**：meta-plugin（其他 CNI 的「多網路介面」附加層），可讓單一 Pod 掛載多個網路介面[^k8s_cni_alt]。
- **CNI-Genie**、**Romana**、**Contiv**、**Terway**（Alibaba VPC）、**NSX-T**、**Spiderpool** 等另有列出但偏小眾或特定雲端情境[^k8s_cni_alt]。

---

## 四、比較摘要

| CNI | 網路模式 | Network Policy | 資料面 | 現況 |
|---|---|---|---|---|
| Flannel | Overlay（VXLAN, host-gw） | 原生不支援 | VXLAN/UDP | 廣泛使用；k3s 預設 |
| Calico | BGP、VXLAN、IPIP | 有（L3–L4） | eBPF、iptables、nftables | 最廣泛採用 |
| Cilium | Overlay、原生路由 | 有（L3–L7） | eBPF | CNCF 畢業、快速成長 |
| Canal | Flannel overlay | 有（經 Calico） | VXLAN + iptables | 常見組合 |
| Antrea | Overlay（VXLAN/Geneve） | 有 | Open vSwitch | CNCF Sandbox |
| kube-router | BGP（直接路由） | 有 | IPVS + iptables | 用於 k0s/k3s |
| Weave Net | Overlay | 有（基本） | VXLAN/OVS | 已歸檔（2024） |

---

## 五、結論與建議

Flannel 是 k3s 預設內建的輕量 overlay CNI，適合追求簡單與低資源消耗的場景，但缺點是原生不支援 Network Policy。若你使用的是其他「口味」的 Kubernetes 發行版（如 kubeadm、k0s、RKE、MicroK8s 等），CNI 並非內建元件，可自行選擇：

- 需要**簡單**：沿用 Flannel。
- 需要 **Network Policy** 又不想放棄 Flannel 的簡單性：**Canal**。
- 需要**完整安全與政策、eBPF 效能**：**Cilium**（功能最豐富）。
- 需要**成熟、最廣泛採用、BGP 直接路由**：**Calico**。
- 已在 **VMware/OVS** 生態：**Antrea**。

---

## 參考資料（來源反思）

本報告事實均來自 2026-10-09 當日抓取的官方文件與專案 README，優先採用第一手來源（k3s 官方文件、Kubernetes 官方文件、各專案 GitHub 頁面）。GitHub star 數為近似值並隨時間變動，僅供相對熱度參考；Weave Net 的「已歸檔」狀態與 k3s 的 `--flannel-backend` 選項以官方文件/專案網站為準。網路政策支援與資料面描述屬各專案官方自述，可能帶有宣傳傾向，建議部署前再依實際版本驗證。

[^k3s_net]: K3s Project Authors. (2026). Basic Network Options. Retrieved 2026-10-09, from https://docs.k3s.io/networking/basic-network-options
[^flannel]: flannel-io. (n.d.). flannel: a network fabric for containers, designed for Kubernetes. Retrieved 2026-10-09, from https://github.com/flannel-io/flannel
[^k8s_cni_alt]: Flannel CNI and alternatives overview (聚合自各專案官方 README / Kubernetes addons 頁面), cross-referenced 2026-10-09. Retrieved from https://github.com/flannel-io/flannel, https://github.com/cilium/cilium, https://github.com/weaveworks/weave, https://github.com/cloudnativelabs/kube-router, https://antrea.io/, https://kubernetes.io/docs/concepts/cluster-administration/networking/
[^k8s_addons]: Kubernetes. (2026). Installing Addons. Retrieved 2026-10-09, from https://kubernetes.io/docs/concepts/cluster-administration/addons/
[^calico_canal]: Tigera. (n.d.). Canal install for Flannel (Calico for policy and Flannel for networking). Retrieved 2026-10-09, from https://docs.tigera.io/calico/latest/getting-started/kubernetes/flannel/install-for-flannel
[^calico_site]: Tigera. (n.d.). Project Calico overview. Retrieved 2026-10-09, from https://www.tigera.io/project-calico/
[^cni_stars]: GitHub star 數（近似值，2026-10-09 抓取）。Cilium ≈25.6k、Flannel ≈9.5k、Calico ≈7.4k、Weave ≈6.6k、kube-router ≈2.5k、Antrea ≈1.8k，取自各專案 GitHub 首頁。
