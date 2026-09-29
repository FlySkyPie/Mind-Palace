# Kubernetes 多叢集管理完整指南：DevOps 實戰視角

## 目錄

1. [為何需要多叢集](#1-為何需要多叢集)
2. [叢集管理工具](#2-叢集管理工具)
3. [多叢集編排與基礎設施即程式碼](#3-多叢集編排與基礎設施即程式碼)
4. [GitOps 多叢集策略](#4-gitops-多叢集策略)
5. [多叢集服務網格](#5-多叢集服務網格)
6. [叢集聯邦（Federation）的演進](#6-叢集聯邦federation的演進)
7. [多叢集配置管理](#7-多叢集配置管理)
8. [監控與可觀測性](#8-監控與可觀測性)
9. [網路與連線](#9-網路與連線)
10. [安全性](#10-安全性)
11. [災難復原](#11-災難復原)
12. [總結：如何選擇多叢集策略](#12-總結如何選擇多叢集策略)

---

## 1. 為何需要多叢集

在 DevOps 實務中，組織採用多個 Kubernetes 叢集的原因通常包括：

- **環境隔離**：開發、測試、正式環境各自獨立，避免互相干擾
- **地理分佈**：服務部署於多個區域以降低延遲、符合資料主權法規
- **高可用與災難復原**：跨區域備援，單一區域故障時自動切換
- **多租戶隔離**：不同團隊或客戶使用各自獨立的叢集
- **合規要求**：特定工作負載必須部署於特定地理位置或基礎設施

多叢集策略的關鍵在於：**工具選擇應匹配團隊成熟度和運維能力**，不要為了多叢集而多叢集，先釐清「為什麼需要多個叢集」再選擇方案[^why-multi-cluster]。

---

## 2. 叢集管理工具

### 2.1 kubectl Contexts

Kubernetes 原生支援透過 kubeconfig 檔案管理多個叢集。每個 context 由叢集、使用者、namespace 組合而成，定義在 `~/.kube/config` 中[^kubectl-contexts]。

```bash
# 列出所有 contexts
kubectl config get-contexts

# 切換 context
kubectl config use-context production-east

# 查看目前 context
kubectl config current-context

# 合併多個 kubeconfig
export KUBECONFIG=~/.kube/config:~/kubeconfig-cluster2.yaml
kubectl config view --flatten > ~/.kube/config
```

**實戰建議**：
- 為每個 context 設定清晰的命名慣例，例如 `company-cluster-region-env`
- 避免手動編輯 kubeconfig，改用 `kubectl config set-cluster` 等命令
- 在 CI/CD pipeline 中動態切換 context 時，善用 `KUBECONFIG` 環境變數而非 `use-context`

### 2.2 kubectx / kubens

由 Ahmet Alp Balkan 開發的命令列工具，提供快速叢集與 namespace 切換[^kubectx]：

```bash
# 安裝
sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx
sudo ln -s /opt/kubectx/kubectx /usr/local/bin/kubectx
sudo ln -s /opt/kubectx/kubens /usr/local/bin/kubens

# 使用 kubectx
kubectx                  # 列出所有 context
kubectx production-east  # 切換到 production-east
kubectx -                # 切回上一個 context

# 使用 kubens
kubens                   # 列出所有 namespace
kubens kube-system       # 切換到 kube-system
```

**DevOps 實戰**：在自動化腳本中，建議使用 `kubectx <context>` 而非 `kubectl config use-context`，因為前者支援模糊匹配與互動選擇，更適合人工介入的運維場景。

### 2.3 K9s

K9s 是終端機介面的 Kubernetes 管理工具，提供即時監控和導航能力[^k9s]：

```bash
# 安裝
curl -sS https://webinstall.dev/k9s | bash

# 啟動（使用目前 context）
k9s

# 指定 context
k9s --context production-east
```

**常用 K9s 快捷鍵**：
- `:` + 資源類型 — 快速切換視圖（如 `:pod`, `:deploy`, `:svc`）
- `/` — 搜尋資源
- `d` — 描述資源詳細資訊
- `y` — 檢視資源 YAML
- `l` — 檢視 Pod 日誌
- `ctrl+d` — 刪除資源
- `shift+f` — 過濾器

**DevOps 實戰**：K9s 是故障排除的首選工具，因為它同時提供即時日誌串流、資源編輯、Pod 進入執行等功能，無需在終端機之間切換。

### 2.4 Lens

Lens 是圖形化 Kubernetes IDE，由 Mirantis 開發與維護[^lens]：

- **多叢集儀表板**：單一視窗管理所有註冊叢集
- **即時監控**：Pod 日誌、指標、事件即時串流
- **Helm 整合**：內建 Helm Chart 瀏覽器與部署
- **Terminal 整合**：內建終端機可直接執行 kubectl 命令
- **擴充生態系統**：支援熱載入擴充功能

**DevOps 實戰**：Lens 適合平台團隊日常巡檢，但**不建議**在生產環境故障演練時依賴圖形介面——K9s 或純 CLI 在網路延遲高時更可靠。

### 2.5 工具比較

| 工具 | 類型 | 適合場景 | 優點 | 侷限 |
|------|------|---------|------|------|
| kubectx/kubens | CLI | 日常開發者 | 輕量快速，指令明確 | 無圖形介面，無法顯示叢集狀態 |
| K9s | TUI | 運維工程師 | 快速導航、即時日誌、資源編輯 | 學習曲線中等，TUI 限制 |
| Lens | GUI | 平台團隊 | 完整視覺化、Helm 整合 | 資源消耗較大，依賴 Electron |
| Octant | GUI | 除錯分析 | 資源拓撲視覺化 | 專案維護較不活躍 |

---

## 3. 多叢集編排與基礎設施即程式碼

### 3.1 Cluster API（CAPI）

Cluster API 是 Kubernetes SIG Cluster Lifecycle 的官方子專案，提供宣告式 API 來管理叢集生命週期——包含建立、擴縮、升級、銷毀[^capi]。

**核心架構**：
- **Management Cluster**：管理叢集，執行 CAPI controllers
- **Workload Clusters**：被管理的 worker 叢集
- **Infrastructure Provider**：AWS、Azure、GCP、vSphere 等
- **Bootstrap Provider**：kubeadm、EKS、AKS 等

```yaml
# 實戰：使用 Cluster API 宣告 EKS 叢集
apiVersion: cluster.x-k8s.io/v1beta1
kind: Cluster
metadata:
  name: production-east
spec:
  clusterNetwork:
    services:
      cidrBlocks: ["10.96.0.0/12"]
    pods:
      cidrBlocks: ["10.32.0.0/12"]
    serviceDomain: "cluster.local"
  infrastructureRef:
    apiVersion: infrastructure.cluster.x-k8s.io/v1beta2
    kind: AWSCluster
    name: production-east
  controlPlaneRef:
    apiVersion: controlplane.cluster.x-k8s.io/v1beta1
    kind: KubeadmControlPlane
    name: production-east
```

**DevOps 適用場景**：
- 需要一致性的叢集部署流程（重複建立相同配置的叢集）
- 跨多個基礎設施提供者的混合雲環境
- 想要 GitOps 整合叢集生命週期管理（將 Cluster 資源放入 Git 倉庫）
- 管理叢集數量 > 5 個時，手動管理開始產生問題

**DevOps 實戰建議**：
- Management Cluster 本身應採用高可用部署，最好跨可用區域
- 定期升級 CAPI 版本以跟上 Provider API 變更
- 使用 ClusterClass 抽象化叢集模板，減少重複定義

### 3.2 Crossplane

Crossplane 將 Kubernetes 擴展為通用控制平面，用於編排雲端基礎設施——不只用於 Kubernetes 叢集，也涵蓋 S3、RDS、VPC 等雲資源[^crossplane]。

```yaml
# Crossplane 範例：宣告 S3 Bucket
apiVersion: s3.aws.crossplane.io/v1beta1
kind: Bucket
metadata:
  name: crossplane-bucket
spec:
  forProvider:
    locationConstraint: us-east-1
  providerConfigRef:
    name: aws-provider
```

**CAPI vs Crossplane 比較**：

| 面向 | Cluster API | Crossplane |
|------|------------|------------|
| 焦點 | 叢集生命週期管理 | 通用基礎設施編排 |
| 管理對象 | Kubernetes 叢集 | 任何雲端資源（S3、RDS、VPC...） |
| 部署模式 | 自託管 Management Cluster | 自託管或雲端 |
| GitOps 整合 | 原生 CRD | 原生 CRD |
| 主要用例 | 多叢集部署與升級 | 多雲基礎設施即程式碼 |
| 社群規模 | ~4,200 GitHub Stars | ~11,700 GitHub Stars |

**選擇建議**：
- 僅需管理叢集生命週期 → **Cluster API**
- 需要同時管理叢集與雲端基礎設施 → **Crossplane** 或兩者搭配

### 3.3 企業管理平台

#### Rancher

SUSE 的 Rancher 提供企業級多叢集管理儀表板[^rancher]：

- **多叢集管理**：集中管理 EKS、AKS、GKE、自建叢集
- **Fleet GitOps**：內建 GitOps 代理程式，類似 ArgoCD
- **RBAC 整合**：統一使用者權限設定
- **應用商店**：認證過的 Helm Chart 目錄

```bash
# 安裝 Rancher
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
kubectl create namespace cattle-system
helm install rancher rancher-latest/rancher \
  --namespace cattle-system \
  --set hostname=rancher.example.com \
  --set bootstrapPassword=admin
```

#### Azure Arc、Google Anthos、AWS EKS Anywhere

| 平台 | 核心功能 | 適合場景 |
|------|---------|---------|
| Azure Arc | 將非 Azure 叢集註冊至 Azure 管理平面 | 混合雲、Azure 生態系 |
| Google Anthos（GKE Enterprise） | 跨雲叢集管理、Config Management、Service Mesh | 多雲策略、GCP 生態系 |
| AWS EKS Anywhere | 內部部署 EKS，裸機或虛擬化 | AWS 延伸至地端 |

**實戰建議**：如果組織已經深度使用特定雲端生態系，優先選擇該雲端的管理方案。否則開源方案（CAPI + Rancher）更具靈活性與廠商中立性[^cloud-platforms]。

---

## 4. GitOps 多叢集策略

### 4.1 ArgoCD ApplicationSets

ApplicationSets 是 ArgoCD 內建的多叢集 GitOps 方案，透過 Generator 動態產生 Application 資源[^argocd-appsets]。

**核心 Generator 類型**：

| Generator | 適用場景 | 侷限 |
|-----------|---------|------|
| Cluster | 基於叢集標籤動態選擇目標 | 需要每個叢集註冊準確的 metadata |
| List | 明確、可審計的目標清單 | 手動維護，無法自動發現新叢集 |
| Matrix | 笛卡爾積組合（region × env） | 可能產生 N×M 個 Application |
| Merge | 基礎配置 + 各叢集覆蓋 | 先後順序規則較複雜 |
| Git | 配置與應用程式碼共存於同一倉庫 | 耦合應用與基礎設施 |

```yaml
# 實戰 ApplicationSet 範例：平台基礎設施
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: platform-services
spec:
  generators:
    - clusters:
        selector:
          matchLabels:
            platform.company.com/tier: critical
  strategy:
    type: RollingSync
    rollingSync:
      steps:
        - matchExpressions:
            - key: env
              operator: In
              values: [staging]
        - matchExpressions:
            - key: env
              operator: In
              values: [production]
  preserveResourcesOnDeletion: true
  template:
    metadata:
      name: 'platform-{{name}}'
    spec:
      project: platform
      source:
        repoURL: https://github.com/company/platform-charts.git
        targetRevision: HEAD
        path: charts/monitoring-stack
        helm:
          valueFiles:
            - values.yaml
            - 'values-{{metadata.labels.env}}.yaml'
      destination:
        server: '{{server}}'
        namespace: platform-system
      syncPolicy:
        automated:
          prune: false
          selfHeal: true
```

**關鍵實戰技巧**：

1. **RollingSync 安全策略**：分階段部署（staging → production），控制爆炸半徑。永遠不要讓 production 叢集與 staging 叢集在同一個 RollingSync step 中更新。
2. **preserveResourcesOnDeletion**：設定為 `true`，防止叢集從 ApplicationSet 匹配中移除時意外刪除 production 資源。
3. **prune: false**：預設關閉 pruning，待觀察確認無誤後再手動啟用。

### 4.2 Flux CD 多叢集方案

Flux 使用 bootstrap per cluster 的方式，每個叢集獨立指向 Git 倉庫中的對應目錄[^flux-multi-cluster]。

**目錄結構**：

```
fleet-infra/
├── clusters/
│   ├── production-east/
│   │   ├── flux-system/
│   │   ├── infrastructure.yaml
│   │   └── apps.yaml
│   ├── production-west/
│   └── staging/
├── infrastructure/
│   ├── base/
│   │   ├── ingress-nginx/
│   │   └── monitoring/
│   └── overlays/
│       ├── production/
│       └── staging/
└── apps/
    ├── base/
    │   ├── api/
    │   └── frontend/
    └── overlays/
        ├── production-east/
        ├── production-west/
        └── staging/
```

```bash
# 為每個叢集執行 bootstrap
flux bootstrap github \
  --owner=${GITHUB_USER} \
  --repository=fleet-infra \
  --branch=main \
  --path=clusters/production-east \
  --context=production-east
```

**dependsOn 確保依賴順序**：

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: apps
  namespace: flux-system
spec:
  dependsOn:
    - name: infrastructure  # 先確保基礎設施就緒
  interval: 10m
  path: ./apps/overlays/production
  prune: true
```

### 4.3 ArgoCD vs Flux 多叢集策略比較

| 面向 | ArgoCD | Flux |
|------|--------|------|
| 多叢集工具 | ApplicationSets（動態產生） | Bootstrap per cluster（靜態註冊） |
| 配置覆蓋 | Cluster generator + Merge generator | Kustomize overlays / patches |
| 依賴管理 | Sync waves（資源層級） | dependsOn（Kustomization 層級） |
| 安全推送 | RollingSync（內建階段控制） | 需自行設計階段部署 |
| 學習曲線 | 中等（Generator 概念較複雜） | 中等偏低 |

**選擇建議**：
- 叢集艦隊動態增減、叢集數量多 → **ArgoCD + ApplicationSets**
- 叢集配置穩定且各自獨立 → **Flux bootstrap per cluster**

---

## 5. 多叢集服務網格

### 5.1 Istio 多叢集網格

Istio 支援三種多叢集模式，最常見的是 **Multi-Primary on Separate Networks**——每個叢集有獨立控制平面，透過 Istio Ingress/Egress Gateways 以 SNI + mTLS 連通[^service-mesh-comparison]。

**架構**：

```
Cluster A (istiod) ←→ mTLS SNI Gateway ←→ Cluster B (istiod)
     ↓                         ↓                        ↓
   Envoy                    Gateway                  Envoy
```

```yaml
# 跨叢集 ServiceEntry
apiVersion: networking.istio.io/v1beta1
kind: ServiceEntry
metadata:
  name: cross-cluster-backend
spec:
  hosts:
  - backend.namespace.svc.clusterset.local
  ports:
  - number: 80
    name: http
    protocol: HTTP
  resolution: DNS
  endpoints:
  - address: backend-istio-ingress.external-dns.com
    ports:
      http: 15443
```

**運作機制**：
- DNS 解析被 sidecar proxy 攔截
- Envoy 將流量封裝進 SNI + mTLS
- 透過 Ingress Gateway 轉發至目標叢集
- 位元組層級不經解密直接代理

### 5.2 Linkerd 多叢集

Linkerd 的多叢集功能較 Istio 簡潔，使用 ServiceExport CRD 宣告要跨叢集曝光的服務[^service-mesh-comparison]：

```bash
# 啟用 Linkerd 多叢集
linkerd multicluster install \
  --gateway-already-exists=false \
  | kubectl apply -f -

# 連結叢集
linkerd multicluster link \
  --cluster-name cluster-b \
  --link-name cluster-b-link \
  --gateway=false
```

```yaml
# 將服務匯出至其他叢集
apiVersion: multicluster.linkerd.io/v1
kind: ServiceExport
metadata:
  name: web-svc
  namespace: default
spec:
  service: web-svc
```

**Linkerd 特點**：
- 零配置 mTLS（自動啟用）
- Rust 原生代理（極輕量，~10MB）
- API 簡潔，學習曲線低

### 5.3 服務網格比較

| 面向 | Istio | Linkerd | Consul Connect |
|------|-------|---------|----------------|
| Proxy | Envoy（~50MB） | Rust Linkerd2-proxy（~10MB） | Envoy |
| 功能豐富度 | 最高 | 中等 | 高 |
| 運維複雜度 | 高 | 低 | 中等 |
| 多叢集模式 | 三種模式（Multi-Primary, Primary-Remote） | ServiceExport（簡潔） | Mesh Gateway |
| VM 支援 | 需額外 Gateway 設定 | 有限 | 原生支援 |
| 適合場景 | 大型企業、複雜路由、嚴格安全需求 | 輕量、簡潔、快速導入 | 混合 K8s + VM 環境 |

---

## 6. 叢集聯邦（Federation）的演進

### 6.1 KubeFed（Kubernetes Federation v2）

KubeFed 採用 hub-and-spoke 模型：Host Cluster 執行 federation controller，Member Clusters 是獨立 Kubernetes 控制平面[^kubefed]。

**核心 CRD**：

| CRD | 用途 |
|-----|------|
| FederatedTypeConfig | 啟用/禁用特定資源的聯邦 |
| FederatedDeployment | 包裝標準 Deployment + placement + overrides |
| ReplicaSchedulingPreference | 跨叢集權重分配複本 |

**重要現狀**：KubeFed 已於 **2023 年 4 月歸檔**，不再維護。現有使用者需要計畫遷移[^kubefed-archived]。

### 6.2 繼承方案：Karmada 與 KubeAdmiral

**Karmada**（CNCF 孵化專案，推薦繼承方案）[^kubefed]：

```yaml
# Karmada PropagationPolicy
apiVersion: policy.karmada.io/v1alpha1
kind: PropagationPolicy
metadata:
  name: nginx-propagation
spec:
  resourceSelectors:
    - apiVersion: apps/v1
      kind: Deployment
  placement:
    clusterAffinity:
      clusterNames:
        - production-east
        - production-west
    replicasScheduling:
      replicaDivisionPreference: Weighted
      replicaScheduling:
        - clusterName: production-east
          maxReplicas: 10
          weight: 7
        - clusterName: production-west
          replicas: 3
```

| 特性 | KubeFed | Karmada | KubeAdmiral |
|------|---------|---------|-------------|
| 狀態 | 已歸檔 | 活躍開發（CNCF 孵化） | 活躍開發 |
| Push/Pull | 僅 Push | Push + Pull | Push + Pull |
| 排程 | RSP | PropagationPolicy | 拓撲感知排程 |
| 學習曲線 | 中等 | 中等 | 較高 |

### 6.3 原生方法 vs 聯邦框架

現今實務趨勢是從聯邦框架轉向量更輕量的原生方法：

- **ArgoCD ApplicationSets + Cluster API** 已涵蓋大多數聯邦用例
- **原生方法優勢**：不需要包裝資源、不引入特殊 CRD、與現有工具鏈整合更順暢
- **Karmada** 是仍需要「控制平面複製資源到多叢集」場景的最佳選擇

---

## 7. 多叢集配置管理

### 7.1 Helm / Kustomize Overlays

**實戰目錄結構（Flux + Kustomize）**：

```
clusters/
├── production-east/
│   ├── flux-system/
│   ├── infrastructure.yaml
│   └── apps.yaml
├── production-west/
└── staging/
```

**Kustomize Overlay 管理不同環境**：

```yaml
# apps/overlays/production-east/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- ../../base/api
- ../../base/frontend

namespace: production

commonLabels:
  environment: production
  region: us-east-1

replicas:
- name: api
  count: 10
- name: frontend
  count: 5

images:
- name: myregistry.io/api
  newTag: v1.2.3
```

**Helm Value 差異管理**：

```yaml
# values-production-east.yaml
global:
  environment: production
  region: us-east-1
ingress:
  replicaCount: 5
  resources:
    requests:
      cpu: 500m
      memory: 512Mi

# values-staging.yaml
global:
  environment: staging
  region: us-east-1
ingress:
  replicaCount: 2
  resources:
    requests:
      cpu: 100m
      memory: 128Mi
```

### 7.2 跨叢集配置策略

| 策略 | 工具 | 適合場景 |
|------|------|---------|
| Base + Overlays | Kustomize | 多環境、多 region |
| Helm values 差異 | Helm | 參數化配置，功能開關 |
| ApplicationSet Merge Generator | ArgoCD | 動態叢集艦隊 |
| ClusterClass variables | Cluster API | 叢集創建時的模板化配置 |

**DevOps 實戰建議**：

1. **單一 monorepo** 管理所有叢集配置，讓 PR 審查可以同時看到所有叢集的變更
2. **Base + Overlay** 模式避免配置重複——每個叢集的 overlay 只紀錄與 base 的差異
3. **配置驗證自動化**：使用 `conftest`（OPA policy）、`kustomize validate` 在 CI 中檢查
4. **避免複製整個目錄**：不要為每個叢集複製完整的 base 目錄，用 overlay 取代

---

## 8. 監控與可觀測性

### 8.1 Thanos

Thanos 採用 Sidecar 附加 Prometheus 的模型，提供全球查詢視圖與長期儲存[^thanos-vs-mimir]：

```
Cluster A                    Cluster B
Prometheus ─Thanos Sidecar─  Prometheus ─Thanos Sidecar─
                  \\                /
               Thanos Query (Global View)
                       |
                   Object Storage
```

```yaml
# Thanos 配置
thanos:
  querier:
    replicas: 3
    stores:
      - thanos-store-gateway:10901
      - cluster-b-thanos-store-gateway:10901
  compactor:
    retentionResolutionRaw: 30d
    retentionResolution5m: 180d
    retentionResolution1h: 10y
```

### 8.2 Grafana Mimir

Grafana Mimir 採用集中式 Remote Write 模型，Prometheus 以 Agent 模式運作（不儲存本地資料），直接寫入 Mimir 叢集[^mimir-multi-cluster]：

```
Cluster A ──Prometheus Agent──┐
Cluster B ──Prometheus Agent──┼─→ Mimir Gateway ─→ Distributor ─→ Ingester ─→ Object Storage
Cluster C ──Prometheus Agent──┘                                           ↓
                                                                     Query Frontend
```

```yaml
# Mimir 多叢集配置：Prometheus Agent
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: agent
  namespace: monitoring
spec:
  mode: Agent  # Agent 模式不儲存本地資料
  externalLabels:
    cluster: production-east
    region: us-east-1
  remoteWrite:
    - url: http://mimir-gateway.mimir.svc:8080/api/v1/push
      headers:
        X-Scope-OrgID: production-east  # 租戶隔離
```

### 8.3 Thanos vs Mimir 比較

| 特性 | Thanos | Mimir / Cortex |
|------|--------|----------------|
| 架構 | Sidecar 附加 Prometheus | 集中式 Remote Write |
| 部署複雜度 | 中等 | 較高（多個 microservice component） |
| 多租戶 | 有限（需多組 Query） | 原生支援（X-Scope-OrgID） |
| 高基數處理 | 取決於 Prometheus | 原生最佳化 |
| 儲存 | 共用 object storage | 共用 object storage |
| 查詢延遲 | 較高（合併多個 Prometheus） | 較低（單一後端） |
| 適合場景 | 已有大量 Prometheus、逐步導入 | 新建架構、多租戶、大規模 |

### 8.4 跨叢集 Tracing

Jaeger 跨叢集 tracing 的關鍵：

1. **統一 Tracer 配置**：所有叢集的應用程式使用相同的 service name 與 sampler 設定
2. **共享儲存後端**：將所有叢集的 span 寫入同一個 Elasticsearch / Cassandra 叢集
3. **優先使用 gRPC 收集**：比 HTTP 收集更高效、更低延遲

---

## 9. 網路與連線

### 9.1 Cilium Cluster Mesh

Cilium 基於 eBPF 的多叢集網路方案，透過 KVStore（etcd）同步叢集間的服務端點資訊[^cilium-clustermesh]：

```
Cluster A (Cilium + etcd) ←→ KVStore State Sync ←→ Cluster B (Cilium + etcd)
      ↓ eBPF maps                                      ↓ eBPF maps
  Pod A ── Geneve/VXLAN 或 Direct Routing ──→ Pod B
```

```bash
# 啟用 Cilium Cluster Mesh
helm install cilium ... \
  --set cluster.id=1 \
  --set cluster.name=production-east

cilium clustermesh enable --context production-east
cilium clustermesh connect \
  --context production-east \
  --destination-context production-west
```

**路由模式選擇**：
- **Direct Routing**：叢集間已有 VPC Peering / Transit Gateway，無封裝 overhead，效能最佳
- **Encapsulated Routing**：非 routable 網路環境，使用 Geneve/VXLAN 封裝

### 9.2 Submariner

CNCF 專案，提供 CNI 無關的多叢集網路連通方案[^submariner]：

```
Cluster A                          Cluster B
  Pod A                              Pod B
    |                                  |
    v                                  v
Gateway Node A ── IPSec/WireGuard ── Gateway Node B
    |                                  |
Lighthouse (Service Discovery)     Lighthouse
```

**Submariner 關鍵功能**：
- **Globalnet**：動態 SNAT/DNAT，解決叢集間 Pod/Service CIDR 重疊問題
- **Lighthouse**：MCS API 實作，跨叢集 DNS 解析（`.clusterset.local`）
- **IPSec / WireGuard**：支援跨叢集加密隧道

### 9.3 網路方案比較

| 方案 | 層級 | CIDR 重疊處理 | 加密 | 服務發現 | 複雜度 |
|------|------|-----------|------|---------|--------|
| Cilium Cluster Mesh | L3/L4（eBPF） | 需 VPC Peering 或 Geneve | mTLS | 原生 Global Service | 中 |
| Submariner | L3（Gateway） | Globalnet 解決 | IPSec/WireGuard | Lighthouse（MCS API） | 中 |
| Istio Multi-Primary | L7（Envoy） | SNI 路由 | mTLS | Istio DNS | 高 |
| Skupper | L7（應用層） | 不影響 | AMQP + TLS | Skupper DNS | 低 |

**選擇建議**：
- 叢集間已有高速網路連線 → **Cilium Cluster Mesh**
- 叢集位於不同 VPC/資料中心，需要加密 → **Submariner**
- 需要服務網格層級的流量管理 → **Istio 的多叢集模式**
- 只想簡單暴露少數服務 → **Skupper**

---

## 10. 安全性

### 10.1 跨叢集 RBAC 策略

統一 RBAC 的最佳實踐是透過 **OIDC 群組**管理而非個別使用者帳號[^workload-identity]：

```yaml
# 使用 OIDC 群組綁定 ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: platform-admins
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: Group
  name: platform-admins@company.com  # OIDC 群組
  apiGroup: rbac.authorization.k8s.io
```

**DevOps 實戰建議**：
1. 每個叢集使用**相同的 ClusterRole 定義**（透過 GitOps 同步到所有叢集）
2. 透過集中式 **OIDC provider**（Okta、Keycloak、Azure AD）管理使用者，而非每個叢集各自管理
3. 使用 `kubectl --as=user@company.com get pods` 測試權限

### 10.2 跨雲 Workload Identity

| 雲端 | 機制 | OIDC 提供者 | 憑證生命週期 |
|------|------|-------------|-------------|
| AWS | IRSA / EKS Pod Identity | 是（IRSA）/ 否（Pod Identity） | 短暫 AWS 憑證（~1hr） |
| Azure | Workload ID（Entra ID） | 原生 AKS | 投影 Token → Entra Token |
| GCP | Workload Identity Federation | 是 | K8s SA ↔ GCP SA 綁定 |

```yaml
# AWS IRSA 範例
apiVersion: v1
kind: ServiceAccount
metadata:
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789:role/my-app-role
  name: my-app
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    spec:
      serviceAccountName: my-app
```

### 10.3 Service Account Token 管理

跨叢集遷移時，Service Account 的 token 必須可攜帶。使用 TokenRequest API（K8s 1.21+）而非靜態 Secret：

```yaml
# 使用 TokenRequest API（投影 token）
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: my-app
  automountServiceAccountToken: true
  containers:
  - name: app
    volumeMounts:
    - mountPath: /var/run/secrets/tokens
      name: vault-token
  volumes:
  - name: vault-token
    projected:
      sources:
      - serviceAccountToken:
          path: vault-token
          audience: vault
          expirationSeconds: 3600
```

---

## 11. 災難復原

### 11.1 Velero 跨叢集備份/復原

Velero 的跨叢集 DR 架構依賴共享 Object Storage，讓備份叢集寫入、DR 叢集讀取[^velero-dr]：

```
Primary Cluster (us-east-1)          DR Cluster (us-west-2)
       ↓                                      ↓
  Velero Backup ──→ S3 ──→ S3 Replication ──→ Velero (read-only)
```

**安裝 Velero（主要叢集）**：

```bash
velero install \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.9.0 \
  --bucket disaster-recovery-backups \
  --prefix primary-cluster \
  --backup-location-config region=us-east-1 \
  --secret-file ./credentials-velero \
  --use-node-agent
```

**備份排程**：

```yaml
apiVersion: velero.io/v1
kind: Schedule
metadata:
  name: dr-backup
  namespace: velero
spec:
  schedule: "0 */6 * * *"   # 每 6 小時
  template:
    ttl: 720h               # 保留 30 天
    includedNamespaces:
    - '*'
    excludedNamespaces:
    - kube-system
    - velero
    snapshotVolumes: true
    defaultVolumesToFsBackup: true
```

**DR 叢集配置（read-only）**：

```yaml
apiVersion: velero.io/v1
kind: BackupStorageLocation
metadata:
  name: primary-cluster-backups
  namespace: velero
spec:
  provider: aws
  objectStorage:
    bucket: disaster-recovery-backups
    prefix: primary-cluster
  config:
    region: us-east-1
  accessMode: ReadOnly  # 防止 DR 叢集寫入主要備份
```

**跨叢集復原命令**：

```bash
# 檢查可用備份
velero backup get

# 執行復原（DR 叢集）
velero restore create dr-restore \
  --from-backup dr-backup-20240209020000 \
  --wait

# 優先復原關鍵服務
velero restore create critical-restore \
  --from-backup dr-backup-20240209020000 \
  --include-namespaces production,database \
  --wait
```

**處理 StorageClass 差異**：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: change-storage-class-config
  namespace: velero
  labels:
    velero.io/plugin-config: ""
    velero.io/change-storage-class: RestoreItemAction
data:
  gp3: standard        # AWS gp3 → DR 叢集 standard
  io2: premium
  ebs-sc: azure-disk   # 跨雲端遷移
```

### 11.2 跨區域災難復原策略

| 模式 | 描述 | RTO | RPO | 成本 | 適合場景 |
|------|------|-----|-----|------|---------|
| Active-Passive | 主要叢集運作，DR 叢集待命 | 15-30 min | <1 hr | 中 | 多數企業標準 |
| Active-Active | 所有叢集運作，Global Load Balancer | <5 min | 接近即時 | 高 | 延遲敏感、高 SLA |
| Backup & Restore | Velero 備份，按需復原 | 30-60 min | 6 hr | 低 | 非關鍵工作負載 |

**自動化故障轉移腳本示例**：

```bash
#!/bin/bash
# automated-dr-failover.sh

PRIMARY_CLUSTER="primary-context"
DR_CLUSTER="dr-context"
BACKUP_NAME=$1

kubectl config use-context $DR_CLUSTER

# 復原關鍵服務
velero restore create critical-restore-$(date +%s) \
  --from-backup $BACKUP_NAME \
  --include-namespaces production,database,cache \
  --wait

# 驗證 Pods 就緒
kubectl wait --for=condition=ready pod -n production -l tier=critical --timeout=5m

# 復原其餘服務
velero restore create full-restore-$(date +%s) \
  --from-backup $BACKUP_NAME \
  --exclude-namespaces production,database,cache,kube-system,velero \
  --wait

echo "DR failover complete"
```

**DevOps 實戰建議**：
- **每月執行 DR 演練**：撰寫自動化測試驗證復原流程
- **監控備份新鮮度**：透過 Prometheus 告警，當備份超過設定時間未更新時發出警告
- **撰寫 DR Runbook**：確保團隊所有成員都熟悉故障轉移流程
- **Velero + ArgoCD 組合**：Velero 備份狀態資料（PersistentVolume），ArgoCD 同步配置（manifest）

---

## 12. 總結：如何選擇多叢集策略

| 情境 | 推薦方案 | 核心考量 |
|------|---------|---------|
| 小型團隊（< 5 叢集） | kubectx + K9s + 簡單 GitOps | 不要過度工程化，簡單有效 |
| 中型團隊（5-20 叢集） | ArgoCD ApplicationSets + Cluster API + Mimir | 開始需要自動化叢集創建與 GitOps |
| 大型企業（20+ 叢集） | Karmada/Fleet + Rancher + Cilium Cluster Mesh | 需要正式的多叢集平台 |
| 多雲/混合雲 | Crossplane + Azure Arc/Anthos + Submariner | 需要供應商中立的基礎設施層 |
| 法規/安全敏感 | OIDC Federation + Workload Identity + Velero | 稽核與合規是首要驅動力 |

**核心原則**：工具選擇應匹配你的**團隊成熟度**和**運維能力**。不要為了多叢集而多叢集，先釐清「為什麼需要多個叢集」再選擇方案。

始終記住：多叢集增加了運維複雜度，每增加一個叢集，都需要投入更多心力在配置同步、監控覆蓋、安全一致性與備份管理上。

---

[^why-multi-cluster]: Kubernetes Documentation. (n.d.). Configure Access to Multiple Clusters. Retrieved 2026-09-26, from https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/
[^kubectl-contexts]: ComputingForGeeks. (n.d.). Manage Kubernetes Clusters with kubectl Kubectx. Retrieved 2026-09-26, from https://computingforgeeks.com/manage-kubernetes-clusters-kubectl-kubectx/
[^kubectx]: kubectx GitHub Repository. (n.d.). Retrieved 2026-09-26, from https://github.com/ahmetb/kubectx
[^k9s]: K9s CLI Documentation. (n.d.). Retrieved 2026-09-26, from https://k9scli.io/
[^lens]: Lens IDE. (n.d.). Retrieved 2026-09-26, from https://k8slens.dev/
[^capi]: Cluster API Documentation. (n.d.). SIG Cluster Lifecycle. Retrieved 2026-09-26, from https://cluster-api.sigs.k8s.io/
[^crossplane]: Crossplane Documentation. (n.d.). What is Crossplane?. Retrieved 2026-09-26, from https://docs.crossplane.io/latest/whats-crossplane/
[^rancher]: Rancher Enterprise Management. (n.d.). Retrieved 2026-09-26, from https://www.rancher.com/products/cluster-management
[^cloud-platforms]: CloudOptimo. (n.d.). Azure Arc vs Google Anthos vs AWS Outposts: Hybrid Cloud Comparison. Retrieved 2026-09-26, from https://www.cloudoptimo.com/blog/azure-arc-vs-google-anthos-vs-aws-outposts-a-comprehensive-hybrid-cloud-comparison/
[^argocd-appsets]: ArgoCD Documentation. (n.d.). ApplicationSet. Retrieved 2026-09-26, from https://argo-cd.readthedocs.io/en/latest/user-guide/application-set/
[^flux-multi-cluster]: FluxCD Documentation. (n.d.). Kustomization. Retrieved 2026-09-26, from https://fluxcd.io/flux/components/kustomize/kustomizations/
[^service-mesh-comparison]: Multi-Cluster Networking Comparison. (n.d.). Cilium Cluster Mesh vs Submariner vs Istio Multi-Primary. Retrieved 2026-09-26, from https://wantsvibes.online/article/multi-cluster-kubernetes-networking-cilium-clustermesh-vs-submariner-vs-istio-multi-primary
[^kubefed]: Trilio. (n.d.). KubeFed Overview. Retrieved 2026-09-26, from https://trilio.io/resources/kubefed/
[^kubefed-archived]: Kubernetes Federation (kubefed) Repository. (n.d.). Archived. Retrieved 2026-09-26, from https://github.com/kubernetes-retired/kubefed
[^mimir-multi-cluster]: Grafana Mimir Multi-Cluster Metrics. (n.d.). Retrieved 2026-09-26, from https://grafana.com/docs/mimir/latest/
[^thanos-vs-mimir]: Thanos vs Mimir Comparison. (n.d.). Retrieved 2026-09-26, from https://thanos.io/
[^cilium-clustermesh]: Cilium Documentation. (n.d.). Cluster Mesh. Retrieved 2026-09-26, from https://docs.cilium.io/en/stable/network/clustermesh/
[^submariner]: Submariner Architecture Documentation. (n.d.). Retrieved 2026-09-26, from https://submariner.io/getting-started/architecture/
[^workload-identity]: Plural Blog. (n.d.). Kubernetes Workload Identity. Retrieved 2026-09-26, from https://www.plural.sh/blog/kubernetes-workload-identity/
[^velero-dr]: Velero Cross-Cluster Disaster Recovery. (n.d.). Retrieved 2026-09-26, from https://velero.io/docs/