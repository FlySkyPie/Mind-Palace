# VPC（AWS/GCP）與 CNI（Kubernetes）抽象層之間的關聯

## 問題核心

AWS / GCP 中的 **VPC（Virtual Private Cloud）** 與 Kubernetes 中的 **CNI（Container Network Interface）** 兩者都是「網路抽象層」，但它們作用在不同的層級。本報告探討兩者之間的層級關聯、實作映射方式以及選擇取捨。

---

## 1. VPC 作為抽象層

VPC 是雲端環境中**邏輯隔離的虛擬網路邊界**，定義了 IP 位址空間（CIDR）、路由表、防火牆規則等網路基元（primitives）。

| 面向 | AWS VPC | GCP VPC |
|------|---------|---------|
| 範圍 | 區域級（regional） | 全域級（global） |
| 子網 | 可用區（AZ）層級 | 區域（region）層級 |
| 計算單元 | Elastic Network Interface（ENI）— 虛擬網卡，附屬於子網 | Alias IP Ranges — 子網上的次要 IP 範圍 |
| 路由控制 | 路由表 + 安全群組（Security Groups） | VPC 防火牆規則（分散於 VM） |
| 底層實作 | AWS 自有硬體 + 虛擬化 | Andromeda（Google 自有 SDN） |

資料來源：[AWS VPC 路由表說明][^aws-routes] 、 [GCP VPC 說明文件][^gcp-vpc]。

---

## 2. CNI 作為抽象層

**CNI（Container Network Interface）** 是 CNCF 規範，定義容器運行時與網路插件之間的合約。當 Pod 被排程到節點上時，工作流程如下：

1. kubelet 呼叫容器運行時（containerd / CRI-O）建立 Pod 沙箱
2. 運行時為 Pod 建立**獨立網路命名空間（network namespace）**
3. 運行時根據 `/etc/cni/net.d/` 設定，執行對應的 CNI 插件（`CNI_COMMAND=ADD`）
4. CNI 插件建立一對 veth —— 一端在 Pod netns（eth0），另一端在主機端 namespace
5. 插件分配 IP 位址、設定路由與 DNS
6. 回傳 JSON 結果給運行時，kubelet 確認 Pod 就緒

資料來源：[k8s.info CNI 深入探討][^cni-deepdive] 、 [Kubernetes 官方網路插件文件][^k8s-cni] 。

---

## 3. CNI 如何映射到 VPC 基元

雲端原生的 CNI 插件（AWS VPC CNI、GKE Dataplane V2、Azure CNI）本質上是**一層轉譯層**，將 Kubernetes Pod 的網路需求映射到雲端 VPC 的基元上。

### AWS VPC CNI

`aws-node` DaemonSet 透過 EC2 API 管理節點上的 ENI 生命週期：

- 為 EC2 實例附加**次要 ENI**
- 從 VPC 子網分配次要 IP 位址給這些 ENI
- 每個 Pod 取得**原生 VPC IP**——可在 VPC、VPC Peering、VPN、Direct Connect 間直接路由

```
EC2 節點
 ├── 主要 ENI（節點 IP: 10.0.1.10）
 └── 次要 ENI（由 vpc-cni 附加）
        ├── 10.0.1.20 → Pod A
        ├── 10.0.1.21 → Pod B
        └── 10.0.1.22 → Pod C
```

資料來源：[AWS VPC CNI 最佳實踐][^aws-cni] 、 [AWS EKS 管理 VPC CNI][^aws-cni-mgmt]。

### GKE VPC-native 叢集（Dataplane V2）

Pod 從子網的**次要 IP 範圍（alias IP ranges）** 取得 IP：

- Pod IP 在 VPC 原生路由、VPC Peering 中可直接路由
- Dataplane V2 基於 **Cilium 的 eBPF 引擎**，用 eBPF 取代 iptables 實現網路策略
- Google 的 Andromeda SDN 處理底層路由

資料來源：[GCP Alias IP 說明][^gcp-alias] 、 [GKE Dataplane V2][^gcp-dpv2]。

---

## 4. Overlay vs VPC-integrated CNI 架構差異

| 維度 | Overlay CNI（Flannel、Calico VXLAN） | VPC 原生 CNI（AWS VPC CNI、GKE Dataplane V2） |
|------|------|------|
| **Pod IP 來源** | 叢集管理私有 CIDR（如 10.244.0.0/16） | 雲端 VPC 子網原生 IP |
| **封包路徑** | Pod → veth → Linux bridge → VXLAN 封裝 → 節點 NIC → 目標節點 → 解封裝 | Pod → veth → ENI → VPC 路由 → 目標節點 ENI |
| **VPC Flow Logs** | 僅看到節點間隧道 IP（overlay），看不到 Pod IP | 看到每個 Pod 的 IP |
| **子網壓力** | 低——僅節點消耗 VPC IP | 高——每個 Pod 消耗一個 VPC IP |
| **效能** | 5–10% 封裝損失 | 接近裸機（1-2% 差異） |
| **MTU** | 減少（1500 基礎上 VXLAN 約為 1450） | 完整 MTU（如 AWS 9001 jumbo frames） |
| **可攜性** | 跨雲、跨地端 | 綁定特定雲端供應商 |
| **安全群組** | VPC SG 看不到 Pod IP（使用 K8s NetworkPolicy） | Pod 繼承 VPC SG 規則 |
| **LB 整合** | 透過 NodePort / proxy | 可直接導向 Pod IP |

資料來源：[Dev.to CNI 完整指南][^cni-guide] 、 [KubeDojo VPC 拓撲模組][^kubedojo] 、 [Tigera CNI 指南][^tigera]。

### 本質差異

Overlay CNI **在 VPC 之上建構一個新的 L2/L3 網路**，使用封裝（encapsulation）將 Pod 流量包裹在節點間隧道中。Pod 與 VPC 之間是**解耦**的。

VPC 原生 CNI 則賦予 Pod **真實的 VPC 身分**，Pod 是雲端網路的一級公民，但代價是消耗 VPC IP 位址空間。

---

## 5. 抽象層級架構

```
實體網路（AWS 全球基礎設施 / Google Andromeda）
        │
        ▼
        VPC（虛擬私有雲）
        │  - 邏輯隔離邊界
        │  - 定義 IP 空間（CIDR）
        │  - 路由表、防火牆規則
        │
        ▼
    ┌───┴────────────────────┐
    │  AWS                   │  GCP
    │  Subnets（AZ 層級）     │  Subnets（區域層級）
    │  ENIs                  │  Alias IP Ranges
    │  （彈性網路介面）       │  （子網次要 IP 範圍）
    └───┬────────────────────┘
        │
        ▼
        CNI 轉譯層
        │
        ├── AWS VPC CNI:  Pod → ENI 次要 IP（ipamd 管理）
        ├── GKE Dataplane V2: Pod → Alias IP（Cilium eBPF）
        ├── Calico BGP:   Pod → BGP 通告 CIDR（Underlay）
        ├── Calico VXLAN: Pod → overlay 隧道（忽略 VPC）
        └── Flannel:      Pod → VXLAN overlay（忽略 VPC）
                │
                ▼
        Pod 網路（eth0）
        │  - veth pair 連接 Pod netns 與主機
        │  - IP、路由、DNS 由 CNI 插件設定
        │  - NetworkPolicy 選擇性啟用
        │
        ▼
        容器工作負載
```

---

## 6. 關鍵結論

1. **VPC 和 CNI 作用在不同的抽象層級**：VPC 是雲端供應商層級的虛擬網路邊界；CNI 是 Kubernetes 層級的容器網路介面標準，規範 Pod 網路命名空間的配接方式。

2. **CNI 是決策點，不只是轉譯層**：選擇不同的 CNI 插件決定了 Pod 網路與 VPC 的整合深度——從**完全穿透 VPC**（AWS VPC CNI）到**完全忽略 VPC**（Flannel）。

3. **VPC 原生 CNI = Pod 獲得 VPC 身分**：代價是 IP 消耗高；好處是效能最佳、可觀測性強、安全群組覆蓋、Load Balancer 可直接導向 Pod。

4. **Overlay CNI = Pod 與 VPC 解耦**：網路策略轉由 K8s NetworkPolicy 負責；可攜性與 IP 靈活度高；代價是 5-10% 的封裝效能損失與故障排除難度增加。

5. **CNI 插件的選擇是架構性決策**，需要權衡 IP 位址空間、效能、安全模型、營運複雜度與可攜性需求。

---

## 參考資料

[^aws-routes]: Amazon Web Services. (n.d.). VPC route tables. Retrieved 2026-10-09, from https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html

[^gcp-vpc]: Google Cloud. (n.d.). VPC networks. Retrieved 2026-10-09, from https://docs.cloud.google.com/vpc/docs/vpc

[^cni-deepdive]: k8s.info. (n.d.). CNI deep dive. Retrieved 2026-10-09, from https://k8s.info/docs/expert/networking-cni

[^k8s-cni]: Kubernetes Authors. (n.d.). Network plugins. Retrieved 2026-10-09, from https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/

[^aws-cni]: Amazon Web Services. (n.d.). Amazon EKS best practices: Amazon VPC CNI. Retrieved 2026-10-09, from https://docs.aws.amazon.com/eks/latest/best-practices/vpc-cni.html

[^aws-cni-mgmt]: Amazon Web Services. (n.d.). Managing the Amazon VPC CNI plugin for Kubernetes. Retrieved 2026-10-09, from https://docs.aws.amazon.com/eks/latest/userguide/managing-vpc-cni.html

[^gcp-alias]: Google Cloud. (n.d.). VPC-native clusters using alias IPs. Retrieved 2026-10-09, from https://docs.cloud.google.com/kubernetes-engine/docs/concepts/alias-ips

[^gcp-dpv2]: Google Cloud. (n.d.). Dataplane V2. Retrieved 2026-10-09, from https://docs.cloud.google.com/kubernetes-engine/docs/concepts/dataplane-v2

[^cni-guide]: Pendela, B. S. (2025). Kubernetes CNI complete guide: Flannel vs Cilium vs Calico and cloud provider CNIs. Retrieved 2026-10-09, from https://dev.to/pendelabhargavasai/kubernetes-cni-complete-guide-flannel-vs-cilium-vs-calico-cloud-provider-cnis-5c6c

[^kubedojo]: KubeDojo. (n.d.). Cloud-native networking and VPC topologies (Module 4.4). Retrieved 2026-10-09, from https://kube-dojo.github.io/cloud/architecture-patterns/module-4.4-vpc-topologies/

[^tigera]: Tigera. (n.d.). Kubernetes CNI: The basics and 4 network models explained. Retrieved 2026-10-09, from https://www.tigera.io/learn/guides/kubernetes-networking/kubernetes-cni/