# K3s HelmChart CRD vs Talos Helm 部署方式對照

## 背景

這個問題通常出現在從 K3s 遷移到 Talos Linux 的場景。K3s 內建了一個 Helm Controller 與 `HelmChart` CRD，可以自動部署 Helm Chart；Talos Linux 則沒有，需要使用者自行解決 Helm Chart 的部署問題。

---

## K3s 的 HelmChart CRD 是什麼？

K3s 隨附一個 **Helm Controller**，它監聽 `helm.cattle.io/v1` 下的兩個自訂資源（CRD）：[^k3s-helm]

1. **`HelmChart`** — 定義一個要安裝/升級/解除安裝的 Helm Chart
2. **`HelmChartConfig`** — 覆寫既有 `HelmChart` 的 values（主要用於自訂 K3s 內建元件如 Traefik）

運作方式：Helm Controller 是一支在 K3s Server 內部執行的控制器，會 **watch `HelmChart` 資源的增刪改**，並自動執行對應的 `helm install / upgrade / uninstall`。[^k3s-helm-controller]

此外，K3s 的 `/var/lib/rancher/k3s/server/manifests/` 目錄會被 **AddOn Controller** 監控，任何放在該目錄的 YAML（包括 `HelmChart` 資源）都會自動 `kubectl apply` 到叢集中。[^k3s-manifests]

### 實例

將以下 YAML 放置於 manifests 目錄下，K3s 會自動安裝 cert-manager：[^oneuptime-k3s]

```yaml
apiVersion: helm.cattle.io/v1
kind: HelmChart
metadata:
  name: cert-manager
  namespace: kube-system
spec:
  repo: https://charts.jetstack.io
  chart: cert-manager
  version: v1.14.4
  targetNamespace: cert-manager
  createNamespace: true
  set:
    installCRDs: "true"
```

### 優點

- 不需手動執行 `helm` 指令
- 宣告式管理（manifest 放在目錄即可）
- 叢集重啟後自動重新部署
- 搭配 GitOps 工具可做到版本控制

---

## Talos Linux 為什麼沒有？

Talos Linux 是一個**以 Kubernetes 為目標的專用作業系統**，它部署的是 **Vanilla（上游原版）Kubernetes**，不附帶任何非標準的 controller 或 CRD。[^talos-vs-k3s]

K3s 屬於 **Kubernetes Distribution**（在既有 OS 上執行），Talos 則是 **將 Kubernetes 作為 OS 的一部分**。架構哲學不同：Talos 盡量保持 Kubernetes 原汁原味，不預裝額外元件。

Talos 只提供 `inlineManifests` 與 `extraManifests` 兩個機器設定欄位，可在叢集啟動時套用**純 YAML**，但它們**沒有 Helm 意識**——無法自動解析、下載或渲染 Helm Chart。[^talos-inline]

---

## Talos 上部署 Helm Chart 的對應方案

### 方案一：標準 Helm CLI（工作站/CI 端執行）

因為 Talos 跑的是標準 Kubernetes API，在工作站上執行 Helm 即可：[^oneuptime-helm]

```bash
talosctl kubeconfig --nodes <control-plane-ip>
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-release bitnami/nginx --namespace web
```

### 方案二：`helm template` + `inlineManifests`（啟動時注入）

適合需要在叢集啟動第一時間就存在的元件（如 CNI）。先在本機渲染 Chart，再嵌入 Talos 機器設定：[^talos-cilium]

```yaml
apiVersion: v1alpha1
kind: KubeInlineManifestConfig
name: cilium
manifest: |
  # Source: cilium/templates/...
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: "cilium"
    namespace: kube-system
  # ...（完整渲染後的 YAML）
```

### 方案三：ArgoCD（GitOps）

在 Talos 上安裝 ArgoCD，再由 ArgoCD 管理 Helm Chart（直接支援 Helm Repository 與 Git 來源）：[^talos-argocd]

- 可用 `extraManifests` 在啟動時自動安裝 ArgoCD 本身
- ArgoCD 的 `Application` CRD 原生支援 Helm Chart 部署

### 方案四：Flux（GitOps）

Flux 提供 `HelmRelease`、`HelmRepository`、`HelmChart` 等 CRD，與 K3s 的 `HelmChart` 概念最接近：[^talos-flux]

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: cert-manager
  namespace: flux-system
spec:
  chart:
    spec:
      chart: cert-manager
      sourceRef:
        kind: HelmRepository
        name: jetstack
      version: v1.14.4
  install:
    createNamespace: true
```

### 方案五：Terraform + Helm Provider（IaC）

適合完全自動化的基礎設施即程式碼工作流程：[^talos-terraform]

```hcl
provider "helm" {
  kubernetes {
    host = "https://${var.cp_ip}:6443"
    # ... kubeconfig 設定
  }
}

resource "helm_release" "nginx" {
  name       = "nginx"
  repository = "https://charts.bitnami.com/bitnami"
  chart      = "nginx"
}
```

---

## 對照表

| K3s 內建功能 | Talos 對應方案 |
|---|---|
| HelmChart CRD（自動部署 Helm） | 無此 CRD，需轉為標準 Helm CLI、ArgoCD、Flux、Terraform 或 `helm template` + `inlineManifests` |
| `/var/lib/rancher/k3s/server/manifests/` 自動 apply | `inlineManifests` / `extraManifests`（僅啟動時生效，非持久 watch） |
| Helm Controller（宣告式 Helm 生命週期管理） | Flux `HelmRelease` CRD 或 ArgoCD `Application` CRD |
| HelmChartConfig（覆寫內建元件 values） | 無對應內建功能，需自行管理 values |

---

## 結論

K3s 的 HelmChart CRD 最大的價值是**讓 Helm Chart 變成 Kubernetes 原生的宣告式資源**，搭配 manifests 目錄可做到「放檔案即部署」。Talos 不提供同等功能，因為其設計哲學是保持 Kubernetes 原版。

最接近的替代方案是 **Flux**，它的 `HelmRelease` CRD 提供類似的宣告式 Helm 管理體驗——但需要額外安裝 Flux，而非像 K3s 那樣開箱即用。對於需要 K3s 這種「自動部署 Helm」體驗的使用者，可在 Talos 上安裝 Flux 或 ArgoCD 來補足，或使用 `helm template` 渲染後透過 `inlineManifests` 在啟動時注入。

---

[^k3s-helm]: K3s Documentation. (n.d.). *Add-ons / Helm*. Retrieved 2026-09-27, from https://docs.k3s.io/add-ons/helm
[^k3s-helm-controller]: k3s-io/helm-controller. (n.d.). *K3s Helm Controller Source Code*. Retrieved 2026-09-27, from https://github.com/k3s-io/helm-controller/
[^k3s-manifests]: Adhdecode. (n.d.). *K3s Helm Controller Auto Deploy*. Retrieved 2026-09-27, from https://adhdecode.com/articles/k3s/k3s-helm-controller-auto-deploy/
[^oneuptime-k3s]: OneUptime. (2026-03-20). *K3s Auto-Deploying Helm Charts*. Retrieved 2026-09-27, from https://oneuptime.com/blog/post/2026-03-20-k3s-auto-deploying-helm-charts/view
[^talos-vs-k3s]: Sidero Labs. (n.d.). *Talos Linux vs K3s*. Retrieved 2026-09-27, from https://www.siderolabs.com/blog/talos-linux-vs-k3s
[^talos-inline]: Sidero Labs Documentation. (n.d.). *inlineManifests / extraManifests*. Retrieved 2026-09-27, from https://docs.siderolabs.com/kubernetes-guides/advanced-guides/inlinemanifests
[^oneuptime-helm]: OneUptime. (2026-03-03). *How to Install Helm on a Talos Linux Cluster*. Retrieved 2026-09-27, from https://oneuptime.com/blog/post/2026-03-03-install-helm-on-a-talos-linux-cluster/view
[^talos-cilium]: Sidero Labs Documentation. (n.d.). *Deploying Cilium CNI*. Retrieved 2026-09-27, from https://docs.siderolabs.com/kubernetes-guides/cni/deploying-cilium
[^talos-argocd]: Sidero Labs Documentation. (n.d.). *Deploy ArgoCD on Talos Linux*. Retrieved 2026-09-27, from https://docs.siderolabs.com/kubernetes-guides/advanced-guides/deploy-argocd
[^talos-flux]: Sidero Labs Documentation. (n.d.). *Deploy Flux on Talos Linux*. Retrieved 2026-09-27, from https://docs.siderolabs.com/kubernetes-guides/advanced-guides/deploy-flux
[^talos-terraform]: Talos Workshop. (n.d.). *Terraform with Talos and Helm*. Retrieved 2026-09-27, from https://qjoly.github.io/workshop-talos/terraform/