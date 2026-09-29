# K3s 內建的 Metrics Server 與 Talos 需自行安裝的差異

## 什麼是 Metrics Server？

**Metrics Server** 是 Kubernetes 集群中一個輕量級的資源指標收集元件，主要功能是從每個節點（Node）上的 Kubelet 收集 CPU 與記憶體的使用量數據，並透過標準化的 Metrics API（`metrics.k8s.io`）在 Kubernetes API Server 中公開[^metrics-server]。

這些指標數據主要用於：

- **Horizontal Pod Autoscaler（HPA）** — 橫向自動擴縮 Pod 副本數量
- **Vertical Pod Autoscaler（VPA）** — 自動調整 Pod 的資源請求
- **`kubectl top` 指令** — 查看節點與 Pod 的即時 CPU / 記憶體使用量

Metrics Server 僅暫存指標於記憶體中，不做長期儲存，專為自動擴縮用途設計，不適合當作完整監控方案[^k8s-metrics-pipeline]。

## K3s：內建 Metrics Server

K3s **預設內建 Metrics Server** 作為其「packaged component」（內建套件）之一。K3s 內建的其他套件還包括 CoreDNS、Traefik 與 local-storage。這些套件通過 K3s 的 AddOn 機制自動部署——K3s 會將 `/var/lib/rancher/k3s/server/manifests/` 目錄下的 manifest 檔案在啟動時自動 apply 到集群中[^k3s-packaged-components]。

Metrics Server 在 K3s 中：

- **預設啟用**：安裝 K3s 後無需額外設定即可使用 `kubectl top` 與 HPA
- **可透過 `--disable=metrics-server` 旗標關閉**
- 使用 `rancher/mirrored-metrics-server` 映像檔，以 15 秒為間隔收集指標[^k3s-metrics-ref]

## Talos Linux：需自行安裝

Talos Linux **不內建 Metrics Server**。Talos 的設計哲學是極簡主義——僅包含 CoreDNS 與 kubelet bootstrap token 等必要元件，其餘所有 addons 皆由使用者自行安裝[^talos-metrics-server]。

在 Talos 上部署 Metrics Server 需要自行提供上游 manifests，主要方式有：

1. **手動 `kubectl apply`**：直接套用 upstream 的 `components.yaml`
2. **`extraManifests` / `KubeExternalManifestConfig`**：在 Talos machine config 中指定外部 manifest URL，讓 Talos 在 bootstrap 時自動套用
3. **`inlineManifests` / `KubeInlineManifestConfig`**：將 manifest 內容直接嵌入 machine config
4. **Helm chart**：透過 Helm 安裝

此外，Talos 還需額外處理 TLS 憑證問題——因為 Talos 的 kubelet 憑證不包含 IP 位址在 SAN 中，但 Metrics Server 透過 IP 連接 kubelet，導致 TLS 驗證失敗。解決方式有二：使用 `--kubelet-insecure-tls`（僅測試用），或啟用 kubelet 憑證輪換並安裝 Kubelet Serving Certificate Approver（正式環境）[^talos-tls]。

## 比較總結

| 面向 | K3s | Talos Linux |
|------|-----|-------------|
| Metrics Server 是否內建 | 是，預設啟用 | 否，完全需自行安裝 |
| 部署方式 | 自動透過 AddOn 機制部署 | 需手動或透過 machine config 提供 upstream manifests |
| 額外設定 | 無需任何設定即可使用 | 需處理 kubelet TLS 憑證問題 |
| 關閉方式 | `--disable=metrics-server` | 不部署即可（預設就不存在） |

## 對照表的含義

原始對照表中的「K3s 內建功能」與「Talos 對應方案（需自行安裝）」的差別反映了兩者完全不同的設計哲學：

- **K3s** 追求「開箱即用」，將常用元件（含 Metrics Server）預設打包在內，使用者裝好就能直接使用 `kubectl top` 和 HPA
- **Talos Linux** 追求「極簡安全」，預設不安裝任何非必要元件，Metrics Server 這類 addon 需要使用者依自身需求自行部署 upstream manifests，並處理衍生的 TLS 設定

---

[^k8s-metrics-pipeline]: Kubernetes. (n.d.). Resource Metrics Pipeline. Retrieved 2026-09-26, from https://kubernetes.io/docs/tasks/debug/debug-cluster/resource-metrics-pipeline/
[^k3s-packaged-components]: K3s. (n.d.). Managing Packaged Components. Retrieved 2026-09-26, from https://docs.k3s.io/installation/packaged-components
[^k3s-metrics-ref]: K3s. (n.d.). Metrics Reference. Retrieved 2026-09-26, from https://docs.k3s.io/reference/metrics
[^talos-metrics-server]: Sidero. (n.d.). Deploy Metrics Server on Talos. Retrieved 2026-09-26, from https://docs.siderolabs.com/kubernetes-guides/monitoring-and-observability/deploy-metrics-server
[^talos-tls]: Kubito. (n.d.). Talos Linux Kubernetes Metrics Server. Retrieved 2026-09-26, from https://kubito.dev/posts/talos-linux-kubernetes-metrics-server/
[^metrics-server]: Kubernetes SIGs. (n.d.). Metrics Server GitHub Repository. Retrieved 2026-09-26, from https://github.com/kubernetes-sigs/metrics-server