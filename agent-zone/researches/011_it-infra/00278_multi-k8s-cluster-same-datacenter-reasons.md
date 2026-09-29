# 同機房多 K8s Cluster 而非單一 Cluster 的理由

## 問題

在同一個機房或實體地點，為什麼要用多個 Kubernetes Cluster 而不是把所有資源放在單一 Cluster 內管理？以下從隔離性、爆炸半徑、控制面風險、資源衝突、法規遵循、團隊自治、升級風險、API Server 擴展限制等面向分析。

---

## 1. 多租戶隔離：Namespace 不是安全邊界

Kubernetes Namespace 是 API 範圍限定的原始工具（API-scoping primitive），並非安全邊界（security boundary）。多個租戶共享同一個 Cluster 時，Namespace 層級的隔離在多個面向均不足[^climstech-multitenancy]。

| 面向 | Namespace 隔離 | Cluster 隔離 |
|------|---------------|-------------|
| **網路** | 跨 Namespace 的 Pod 可自由通訊，除非用 NetworkPolicy 明確封鎖（預設無政策） | 完全隔離的網路平面；跨 Cluster 流量需透過明確、可稽核的路徑（如 VPN、Service Mesh、API Gateway） |
| **核心（Kernel）** | 同一 Node 上的所有 Pod 共用同一個 Linux 核心。容器逃逸（如 CVE-2019-5736、CVE-2022-0185）可以觸及底層 Node，不受 Namespace 限制 | 獨立的 Node 池可用不同硬體甚至實體隔離 |
| **API Server** | 所有租戶共享同一個控制面。失控的 Admission Webhook 或 List-Watch 風暴會影響所有人 | 各自獨立的控制面——一個 Cluster 的故障不會影響另一個 |
| **Secret** | 一個 ClusterRoleBinding 綁定 `view` 即可讀取整個 Cluster 的所有 Secret | Secret 完全限定在單一 Cluster 的 etcd 內 |

**結論：** Namespace 隔離適合互信的團隊；Cluster 隔離適合不互信或不能要求互信的租戶。

---

## 2. 爆炸半徑（Blast Radius）控制

單一大型 Cluster 中，錯誤的爆炸半徑是**整個 Cluster**：

- 某個 Namespace 的 Ingress 設定錯誤可能導致共用的 NGINX Controller 每 8 秒重新載入，對**所有租戶**造成封包丟失。[^climstech-multitenancy]
- 一個吵鬧的租戶對大型資源進行大量的 List-Watch 操作，會降低**整個 Cluster 的 API Server 效能**。
- 一個失效的 Admission Webhook 套用於 Cluster 範圍時，會同時阻擋**所有 Namespace** 的准入請求。
- 某個 Node 上發生容器逃逸後，可透過 Kernel 漏洞危害該 Node 上**所有租戶**的工作負載。

多 Cluster 能將爆炸半徑**限制在該 Cluster 範圍內**。一個 Cluster 的故障、設定錯誤或安全漏洞無法傳播到其他 Cluster。

---

## 3. 控制面單點故障

Kubernetes 原生只支援單一 Cluster 模型——每個 Cluster 只有一個控制面（即使用 HA 複寫）。[^devoteam-single-vs-multi] 控制面故障影響的範圍包括：

- Pod 排程（新 Pod 無法被放置）
- Pod 故障偵測與修正（Reconciliation Loop 停止）
- 應用更新（Deployment 卡住）
- Cluster 設定變更（`kubectl apply` 失效）
- Kubernetes 版本升級

在 Multi-Cluster 架構下，每個 Cluster 擁有自己的控制面。一個 Cluster 的控制面故障**不會影響**其他 Cluster 的工作負載，其他 Cluster 仍然可以正常進行部署、更新和排程。

---

## 4. Cluster 層級資源的衝突

有幾類關鍵資源是 **Cluster-Scoped**（全 Cluster 層級），無法用 Namespace 隔離：

### Custom Resource Definition（CRD）

CRD 本身是 Cluster 層級物件。即使自訂資源本身可以是 Namespaced，但 CRD 定義卻是全 Cluster 唯一的。[^vcluster-crds] 若兩個租戶需要不同版本的 CRD（例如不同版本的 cert-manager），Namespace 隔離無法解決——一個 Cluster 只能有一個 CRD 定義。

### PriorityClass

PriorityClass 是 Cluster-Scoped。惡意或設定錯誤的租戶可以建立最高優先權的 Pod，搶佔整個 Cluster 的關鍵系統工作負載。[^k8s-priorityclass]

### ClusterRole / ClusterRoleBinding

這些資源若綁定在 Cluster 層級，則授予跨所有 Namespace 的權限。一個「暫時」留下的 `cluster-admin` ClusterRoleBinding 可以存在數月而不被察覺，成為真實世界的已知風險。[^climstech-multitenancy]

### PersistentVolume（PV）

PV 是 Cluster-Scoped。Namespace A 的租戶若 RBAC 有漂移，可能影響 Namespace B 使用的 PV。

**Multi-Cluster 架構下**，每個 Cluster 擁有自己獨立的 Cluster-Scoped 資源，徹底消除這類衝突。團隊可以安裝不同的 CRD 版本、定義不同的 PriorityClass、獨立管理 ClusterRole。

---

## 5. 法規遵循（PCI-DSS、HIPAA、SOC 2）

以 PCI DSS 為例，一個支付平台業者最初嘗試單一 Cluster 搭配網路分段，雖然技術上符合 PCI DSS Requirement 1，但長期授權與稽核需求仍使其轉向 Multi-Cluster 架構[^valensas-pci]：

- 一個專用 Cluster 給 PCI 範圍內的服務（卡號資料）
- 另一個 Cluster 給非 PCI 的服務
- Service Mesh 搭配 mTLS 加密跨 Cluster 通訊
- Zero Trust 模型——不信任任何 in-cluster 流量

各法規要求的關鍵點：

- **PCI DSS Requirement 1**——安全網路分段。分離的 Cluster 提供硬邊界；基於 Namespace 的隔離較難稽核與驗證。
- **HIPAA**——PHI（受保護健康資訊）與非 PHI 環境之間需要加密與網路邊界。專用 Cluster 能乾淨地定義稽核範圍。
- **SOC 2**——需要控制範圍的邊界。獨立 Cluster 提供清晰的稽核邊界。
- **GDPR 資料主權**——Multi-Cluster 允許將 Cluster 部署在特定地理區域以確保資料落地。

---

## 6. 團隊自治與獨立生命週期

多個 Cluster 賦予團隊獨立性，單一 Cluster 無法提供：

- **獨立的升級排程**——Team A 可以跑 Kubernetes 1.29（因第三方 Operator 需要），Team B 跑 1.31（因需要某個新功能）。單一 Cluster 下所有人被迫同步升級。
- **獨立設定**——每個團隊可以根據特定工作負載需求調整 Node 大小、CNI 外掛、StorageClass 和安全政策。
- **獨立事故應變**——Team A 的 Cluster 控制面出問題時，他們可以自行修復，不須與 Team B 協調變更凍結。
- **獨立成本追蹤**——每個 Cluster 可以乾淨地對應到成本中心或 Chargeback 模型。

**真實案例：** Spotify 運行約 200 個 Cluster，服務 6 億以上的月活躍用戶與 4,000 多個微服務，正是為了支援團隊自治。[^spotify-clusters]

---

## 7. 升級風險

單一大型 Cluster 的升級是**高風險、全有或全無**的事件：

- 若控制面升級失敗，**所有團隊的所有工作負載**都會受影響。
- 若新版本棄用了某個團隊依賴的 API，**所有團隊**必須同時遷移，否則會面臨服務中斷。
- 若 Node 升級同時 Draining 多個團隊的 Pod，排程或資源問題的爆炸半徑最大。[^devoteam-single-vs-multi]

Multi-Cluster 架構下：

- **Canary 升級**——先升級一個 Cluster，驗證 30 天後再升級其他 Cluster。
- **分階段升級**——不同團隊/Cluster 可以不同節奏升級。
- **復原隔離**——若某個 Cluster 升級失敗，其他 Cluster 維持穩定。

---

## 8. API Server 負載與擴展限制

Kubernetes 官方文件中定義了硬限制[^k8s-large-cluster]：

- 最多 **5,000 個 Node**
- 最多 **150,000 個 Pod**
- 最多 **300,000 個 Container**

但在此之前，GKE 等雲端平台的文件指出軟限制（soft limits）就會導致效能衰退[^gke-limits]：

- etcd 大小上限 6 GB（達 80% 時觸發警報）
- 每個資源類型在 etcd 中的物件總大小 800 MB
- 每個 Cluster 10,000 個 Service（不使用 Dataplane V2 時 kube-proxy 的 iptables 會衰退）
- 每個 Namespace 5,000 個 Service（Shell 環境變數限制）
- 所有 Service 合計最多 260,000 個 Endpoint（Dataplane V2 的 eBPF Map 限制）
- 最多 5,000 個 HPA 物件（超過後 HPA 重處理線性衰退）
- 每個 Cluster 最多 200,000 個 Watch（超過後控制面初始化異常）
- 啟用加密時最多 30,000 個 Secret（超過後 Cluster 啟動不穩定）
- Admission Webhook 延遲應平均低於 **10ms**（50-100ms 的一致延遲會顯著降低效能）

**Multi-Cluster 的幫助：** 將工作負載分散到多個 Cluster，使每個 Cluster 維持在穩定的營運甜蜜點，避免逼近這些軟硬限制。

---

## 9. 成本權衡

Multi-Cluster 有確定的成本溢價。以 ClimsTech 提供的試算為例[^climstech-multitenancy]：

| 情境 | 月成本 |
|------|--------|
| 單一 Cluster：10 個團隊、Namespace 隔離、8 × m5.2xlarge Node | ~2,300 美元 |
| 10 個獨立 Cluster：每個 Cluster 3 × m5.xlarge 最小 HA Node | ~4,900 美元 |
| 倍率 | ~2.1 倍 |

**是否值得取決於風險承受度，以及安全或合規失效的成本，而非只看絕對數字。**

Cast.ai 的 2026 年報告指出平均 Cluster CPU 利用率僅 **8%**、記憶體 **20%**[^k8s-cost-analysis]，意味著獨立 Cluster 中過度佈署的備用資源浪費必須與共享 Cluster 的風險一起權衡。

---

## 10. 折衷模型——多數組織不選極端

最常見的生產模式是混合做法：

- **Production 客戶 Cluster**——每個客戶層級（或每個大型客戶）一個 Cluster，嚴格 PSS 強制執行，合規範圍的工作負載使用專用 Node 池。
- **內部工程 Cluster**——所有團隊放在 Namespace 內，搭配五層防護設定（RBAC + ResourceQuota + NetworkPolicy + PSS + Audit Logging）。
- **非 Production Cluster**——所有團隊的 Staging 與 Dev 共用一個 Cluster，及早發現政策漂移。

業界常說的一句話：**「每個環境（dev/stage/prod）一個 Cluster，Cluster 內再用 Namespace 做團隊隔離。」**

---

## 11. 真實案例

| 公司 | 做法 | 規模 | 關鍵原因 |
|------|------|------|---------|
| Spotify | ~200 個 Cluster | 6 億+ 用戶、4,000+ 微服務、4,500 Node | 團隊自治、獨立生命週期管理、地理分佈[^spotify-clusters] |
| Adidas | Multi-Cluster（Giant Swarm 託管） | 一年內 40% 關鍵系統上 K8s | 由外部夥伴營運以降低營運複雜度 |
| 支付平台（Valensas 案例） | PCI 範圍 Cluster + 非 PCI Cluster | 地端、Multi-Cluster + Service Mesh | PCI DSS 遵循、稽核範圍邊界、授權需求[^valensas-pci] |

---

## 結論與決策框架

| 適用單一 Cluster | 適用多個 Cluster |
|-----------------|-----------------|
| 內部團隊且互信 | 外部客戶或不互信的租戶 |
| 低法規需求 | PCI-DSS、HIPAA、FedRAMP、SOC 2 範圍環境 |
| 可接受單一 K8s 版本 | 團隊需要不同 K8s 版本 |
| 對成本溢價敏感 | 零容忍租戶之間的爆炸半徑 |
| <1,000 Node、<50,000 Pod | 接近 K8s 硬限制（5,000 Node、150K Pod） |
| 專屬平台團隊統一管理 | 團隊需要獨立升級/事故生命週期 |

---

[^climstech-multitenancy]: ClimsTech Engineering. (n.d.). The definitive guide to Kubernetes multi-tenancy. Retrieved 2026-09-27, from https://climstech.com/blog/kubernetes-multi-tenancy

[^devoteam-single-vs-multi]: Devoteam. (n.d.). Kubernetes cluster strategy — single or multi? (Part 1). Retrieved 2026-09-27, from https://www.devoteam.com/expert-view/kubernetes-cluster-strategy-single-or-multi-part-1/

[^vcluster-crds]: vCluster. (n.d.). Kubernetes CRDs — a huge pain in multi-tenant clusters. Retrieved 2026-09-27, from https://www.vcluster.com/blog/kubernetes-crds-huge-pain-in-multi-tenant-clusters/

[^k8s-priorityclass]: Kubernetes. (n.d.). Pod Priority and Preemption. Retrieved 2026-09-27, from https://kubernetes.io/docs/concepts/scheduling-eviction/pod-priority-preemption/

[^valensas-pci]: Valensas. (n.d.). Multi-cluster Kubernetes architecture on the PCI DSS journey. Retrieved 2026-09-27, from https://valensas.com/blog/multi-cluster-kubernetes-architecture-on-the-pci-dss-journey/

[^spotify-clusters]: Matheus, R. (2026). Companies using Kubernetes in 2026 — who runs K8s and how they scale. Retrieved 2026-09-27, from https://dev.to/matheus_releaserun/companies-using-kubernetes-in-2026-who-runs-k8s-and-how-they-scale-2log

[^k8s-large-cluster]: Kubernetes. (n.d.). Considerations for large clusters. Retrieved 2026-09-27, from https://kubernetes.io/docs/setup/best-practices/cluster-large/

[^gke-limits]: Google Cloud. (n.d.). Planning a large cluster. Retrieved 2026-09-27, from https://docs.cloud.google.com/kubernetes-engine/docs/concepts/planning-large-clusters

[^k8s-cost-analysis]: Cast.ai. (2026). State of Kubernetes Optimization 2026. Retrieved 2026-09-27, from https://cast.ai