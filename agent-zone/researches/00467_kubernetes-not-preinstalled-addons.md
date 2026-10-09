# 原生 Kubernetes 未預裝、但其他 flavor 常預裝的元件

## 摘要

在原生（vanilla / upstream）Kubernetes 中，透過 `kubeadm init` 建立的叢集**只會**預裝兩個 addon：`CoreDNS` 與 `kube-proxy`[^kubeadm]；再加上控制平面核心元件（kube-apiserver、kube-controller-manager、kube-scheduler、etcd）與節點元件（kubelet、kubectl）[^comp][^kubeadm]。除此之外的所有「實用功能」（Ingress、Load Balancer、儲存、監控、自動擴縮等）皆**不預裝**，需自行安裝[^kubeadm][^add]——即使是在雲端管理叢集上，也只有部分項目會被代管。這份報告整理這些「不會預裝、但在其他 flavor 常預裝」的元件，以及各家發行版（EKS、GKE、AKS、k3s、k0s、MicroK8s）的差異。

## 一、原生 kubeadm 預裝的極小集合

kubeadm 官方文件明確表示，其 `addon` 階段只安裝通過 conformance 測試所需的元件[^kubeadm]：

```text
addon                Install required addons for passing conformance tests
  /coredns             Install the CoreDNS addon
  /kube-proxy          Install the kube-proxy addon
```

kubeadm 維護者亦在 issue 中說明：除了 kube-proxy 與 kube-dns/CoreDNS 之外，不提供其他 addon，一切皆屬其範疇外[^issue62]。原生 Kubernetes 對「addon」的定義即為「透過套件或其他機制部署，以擴充 Kubernetes 功能的資源」[^add]，而 kubeadm 只負責「啟動最小可運作的叢集」，佈署 Dashboard、監控、雲端 addon 等皆「不在其範疇內」[^kubeadm]。

## 二、原生不預裝、但其他 flavor 常預裝的清單

下列元件在原生 kubeadm 叢集**全部不會預裝**，需自行部署；但在雲端管理或輕量發行版中常被代管或內建[^add][^comp]。

| 類別 | 元件 | 原生 kubeadm | 為何需要 |
|------|------|:---:|------|
| **CNI / Pod 網路** | Calico、Cilium、Flannel、Weave 等 | ❌ | 原生不含任何 CNI，未安裝前 Pod 無法排程[^kubeadm] |
| **Ingress Controller** | NGINX、Traefik、Contour 等 | ❌ | Ingress 要運作必須有對應的 controller，原生一個都沒有[^ingress] |
| **Load Balancer** | MetalLB、kube-vip、雲端 LB | ❌ | 原生只內建「呼叫 IaaS 平台」的 LB glue（GCP/AWS/Azure）；裸機上 `type: LoadBalancer` 會卡在 Pending，需裝 MetalLB[^metallb] |
| **CSI 儲存驅動** | EBS CSI、GCE PD、Azure Disk、Longhorn、Rook/Ceph 等 | ❌ | 原生**沒有**任何儲存 provisioner，也沒有預設 StorageClass；不裝則 PVC 卡在 Pending[^metallb] |
| **metrics-server** | 資源指標（for `kubectl top`、HPA/VPA） | ❌ | 原生不預裝，需自行部署才能用 Metrics API[^metrics] |
| **Dashboard** | Kubernetes Dashboard 網頁介面 | ❌ | 「不預裝」，官方明確如此聲明[^dash] |
| **cluster-autoscaler** | 節點自動擴縮 | ❌ | 獨立專案，原生不含[^add] |
| **cert-manager** | TLS 憑證自動簽發與續期 | ❌ | 第三方專案，非 Kubernetes 一部分[^add] |
| **external-dns** | Service/Ingress ↔ DNS 記錄同步 | ❌ | 第三方專案[^add] |
| **node-problem-detector** | 節點異常偵測 | ❌ | 屬於「基礎設施 addon」，可選安裝[^add] |
| **Konnectivity** | 控制平面 → 節點連線代理 | ❌ | 原生不裝；GKE、k0s 使用[^gke][^k0s] |
| **監控/日誌** | Prometheus/Grafana、日誌聚合 | ❌ | 原生不含任何完整可觀測堆疊[^metallb] |
| **預設 StorageClass** | 預設儲存類別 | ❌ | 原生不建立任何 StorageClass[^metallb] |

> 註：`ingress-nginx` 專案已於 2026-03-24 封存，Kubernetes 社群現建議改用 Gateway API[^ingress][^metallb]。

## 三、雲端管理發行版（EKS / GKE / AKS）的差異

### Amazon EKS

每個叢集**自動安裝**三個 addon：Amazon VPC CNI（CNI）、kube-proxy、CoreDNS；而 `eksctl >= 0.184.0` 也會自動裝 **metrics-server**[^eks]。另有大量**可選**的受管 addon 目錄，包含 metrics-server、EBS/EFS CSI Driver、AWS Load Balancer Controller（ingress）、CloudWatch Observability 等，但**非自動**安裝[^eks]。EKS Auto Mode 則由內建控制器接管 Pod/Service 網路、叢集 DNS、自動擴縮、區塊儲存、Load Balancer controller 等[^eks]。

### Google GKE

提供完全受管的控制平面，**內建**：Cluster Autoscaler、Node Auto-Provisioning、node-problem-detector、Workload Identity、Konnectivity 與受管的監控/日誌[^gke]。`type: LoadBalancer` 在 GCP 上透過 cloud-controller-manager 開箱即用，不需 MetalLB[^gke][^metallb]。但 GKE **不自動**安裝 metrics-server、Dashboard、cert-manager、external-dns 或任何 ingress controller[^gke]。

### Azure AKS

AKS Standard 的核心 addon（跑在 `kube-system`）包含 CoreDNS、metrics-server、konnectivity-agent、kube-proxy（依網路設定可能改用 Cilium）[^aks]。AKS **Automatic**（全受管模式）更具侵入性，預先配置：Azure CNI Overlay（Cilium）、受管 NGINX ingress（application routing）、Node Auto-Provisioning、受管 Prometheus + Grafana、HPA/KEDA/VPA 等[^aksauto]。

## 四、輕量 / 社群發行版的差異

### k3s（Rancher / SUSE）——「batteries-included」

比 kubeadm 塞入更多預裝元件[^k3s]：

| 元件 | 類別 | 原生 kubeadm 比較 |
|------|------|------|
| containerd / cri-dockerd | 容器執行時 | 原生可自行選擇 |
| **Flannel** | CNI / Pod 網路 | 原生：手動安裝 |
| CoreDNS | DNS | 相同 |
| **Traefik** | Ingress controller | 原生：手動安裝 |
| **ServiceLB** | 裸機 Load Balancer | 原生：手動安裝（MetalLB 等） |
| Kube-router | 網路政策 controller | 原生：視 CNI 而定 |
| **local-path-provisioner** | Local PV 儲存 | 原生：手動安裝 |

### k0s（Mirantis）——「minimal add-ons」哲學

預裝 containerd（預設 CRI）、Kube-router（預設 CNI，Calico 亦可）、CoreDNS、**Konnectivity**、**Metrics Server**；並可用 etcd/SQLite 做資料儲存[^k0s]。但 k0s **不**內建任何 ingress controller、CSI driver、dashboard、cert-manager 或監控——官方明言「盡量少塞 addon」[^k0s]。

### MicroK8s（Canonical）——modular addon 系統

snp 式發行版，**唯一預設啟用**的 addon 是 dns（CoreDNS）；其餘（dashboard、storage、ingress、metallb、metrics-server、registry、prometheus、rbac、cert-manager、gpu、rook-ceph 等）全部以 `microk8s enable <addon>` 選擇性啟用[^microk8s]。

## 五、比較總表

| 元件 | 原生 kubeadm | EKS | GKE | AKS Automatic | k3s | k0s | MicroK8s |
|------|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| CoreDNS | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅（預設） |
| kube-proxy | ✅ | ✅ | ✅ | ✅（Cilium?） | ✅ | ✅ | ✅ |
| **CNI 插件** | ❌ | ✅ VPC CNI | ✅ | ✅ Cilium | ✅ Flannel | ✅ Kube-router | ❌（選用） |
| **metrics-server** | ❌ | ✅（eksctl） | ❌（選用） | ✅ | ❌ | ✅ | ❌（選用） |
| **Ingress Controller** | ❌ | ❌（選用） | ❌（選用） | ✅ NGINX | ✅ Traefik | ❌ | ❌（選用） |
| **Load Balancer** | ❌ | ✅ AWS ELB | ✅ GCP LB | ✅ Azure LB | ✅ ServiceLB | ❌ | ❌（選用） |
| **儲存 / CSI** | ❌ | ✅ EBS | ✅ GCE PD | ✅ Azure Disk | ✅ local-path | ❌ | ❌（選用） |
| **cluster-autoscaler** | ❌ | ❌（選用） | ✅ 內建 | ✅ 內建 | ❌ | ❌ | ❌ |
| **Konnectivity** | ❌ | ❌ | ✅ | ✅ | ❌ | ✅ | ❌ |
| **Dashboard** | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌（選用） |
| **cert-manager** | ❌ | ❌（選用） | ❌ | ❌（選用） | ❌ | ❌ | ❌（選用） |
| **external-dns** | ❌ | ❌（選用） | ❌ | ❌ | ❌ | ❌ | ❌ |
| **node-problem-detector** | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ |
| **監控** | ❌ | ❌（選用） | ✅ 受管 | ✅ Auto | ❌ | ❌ | ❌（選用） |

（✅ = 預裝 / 內建；❌ = 不預裝，需手動或選擇性啟用。）[^metallb]

## 六、結論

原始 Kubernetes 的設計目標是「最小核心 + 自選 addon」，因此 `kubeadm` 只裝 CoreDNS 與 kube-proxy。**Ingress、Load Balancer、儲存/CSI、metrics-server、Dashboard、自動擴縮、cert-manager、監控**等，幾乎全部都是「其他 flavor 會預裝、原生不預裝」的元件。差異最明顯的是：

- **雲端受管叢集**（EKS/GKE/AKS）── LoadBalancer 開箱即用（內建雲端 LB），並代管 CNI 與常見儲存/監控。
- **輕量發行版（k3s）**── 走「batteries-included」，預裝 Traefik、ServiceLB、local-path-provisioner、Flannel。
- **k0s / MicroK8s**── 介於中間，各自以不同哲學取捨哪些 addon 內建。

---

[^kubeadm]: Kubernetes Authors. (n.d.). kubeadm init. Retrieved 2026-10-09, from https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/
[^comp]: Kubernetes Authors. (n.d.). Kubernetes Components. Retrieved 2026-10-09, from https://kubernetes.io/docs/concepts/overview/components/
[^add]: Kubernetes Authors. (n.d.). Installing Addons. Retrieved 2026-10-09, from https://kubernetes.io/docs/concepts/cluster-administration/addons/
[^issue62]: Kubernetes. (2015). kubeadm issue #62. Retrieved 2026-10-09, from https://github.com/kubernetes/kubeadm/issues/62
[^ingress]: Kubernetes Authors. (n.d.). Ingress Controllers. Retrieved 2026-10-09, from https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/
[^metallb]: MetalLB. (n.d.). MetalLB documentation. Retrieved 2026-10-09, from https://metallb.universe.tf/
[^metrics]: Kubernetes Authors. (n.d.). Resource metrics pipeline. Retrieved 2026-10-09, from https://kubernetes.io/docs/tasks/debug/debug-cluster/resource-metrics-pipeline/
[^dash]: Kubernetes Authors. (n.d.). Web UI (Dashboard). Retrieved 2026-10-09, from https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/
[^gke]: Google Cloud. (n.d.). GKE cluster architecture. Retrieved 2026-10-09, from https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-architecture
[^k0s]: k0s Project. (n.d.). k0s documentation. Retrieved 2026-10-09, from https://docs.k0sproject.io/stable/
[^eks]: Amazon Web Services. (n.d.). Amazon EKS add-ons. Retrieved 2026-10-09, from https://docs.aws.amazon.com/eks/latest/userguide/eks-add-ons.html
[^aks]: Microsoft. (n.d.). AKS core concepts. Retrieved 2026-10-09, from https://learn.microsoft.com/en-us/azure/aks/core-aks-concepts
[^aksauto]: Microsoft. (n.d.). AKS Automatic overview. Retrieved 2026-10-09, from https://learn.microsoft.com/en-us/azure/aks/intro-aks-automatic
[^k3s]: SUSE Rancher. (n.d.). K3s documentation. Retrieved 2026-10-09, from https://docs.k3s.io/
[^microk8s]: Canonical. (n.d.). MicroK8s addons. Retrieved 2026-10-09, from https://microk8s.io/docs/addons
