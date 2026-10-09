# K3s 內建的 Flannel 是什麼？

K3s 是輕量級 Kubernetes 發行版，其設計目標之一就是「開箱即用」。要讓 Pod 之間（尤其是跨節點）能互相通訊，必須提供一個**容器網路介面（CNI）**外掛。K3s 挑選的**預設 CNI 外掛就是 Flannel**，並且將它直接編譯進單一 `k3s` 二進位檔，而非以獨立程序或額外容器執行[^k3s-net][^flannel-readme]。

## Flannel 簡介

Flannel 是一個**輕量級第 3 層（Layer 3）網路矩陣**，實現了 Kubernetes 的容器網路介面（CNI），因此它被稱為「CNI Plugin」[^k3s-net]。它的核心運作原理是：

- 從一個預先設定的較大位址空間中，為每個節點分配一個**子網路租約（subnet lease）**
- 節點之間的封包使用可設定的**後端（backend）**機制來封裝轉送，例如 VXLAN、WireGuard 或 host-gw[^flannel-readme][^k3s-net]

## K3s 與 Flannel 的整合方式

K3s **不是**把 Flannel 當成獨立的 `flanneld` 程序、也不是部署成 DaemonSet Pod，而是以 **Go 函式庫**的方式，將上游 `github.com/flannel-io/flannel` 直接作為依賴編譯進去[^k3s-gomod][^k3s-flannelgo]：

1. K3s 的 `go.mod` 宣告 `github.com/flannel-io/flannel v0.28.4` 為直接依賴[^k3s-gomod]
2. K3s 在 `pkg/agent/flannel/setup.go` 中以 **goroutine** 在**同一個程序內**啟動 Flannel[^k3s-setup]
3. K3s 啟動時會產生 CNI 設定檔與 Flannel 的 JSON 設定檔（如 `/var/lib/rancher/k3s/agent/etc/flannel/net-conf.json`），並依叢集設定替換其中的佔位符（`%CIDR%`、`%backend%` 等）[^k3s-setup]

這樣做的好處是：**沒有額外的 `flanneld` 程序、沒有額外容器映像要拉取、單一二進位檔全部包含**，完全符合 K3s 的部署哲學[^k3s-setup]。

## Flannel 的後端選項

K3s 透過 `--flannel-backend` 參數選擇 Flannel 的封裝後端[^k3s-net]：

| Backend | 說明 |
|---|---|
| `vxlan`（**預設**） | 使用 VXLAN 封裝節點間封包[^k3s-net] |
| `host-gw` | 直接透過節點 IP 建立到 Pod 子網的路由，**要求所有節點之間有第 2 層直接連通**[^k3s-net] |
| `wireguard-native` | 使用 WireGuard 封裝並**加密**流量，要求每個節點有 WireGuard 核心模組[^k3s-net] |
| `ipsec` | 使用 strongSwan IPSec；**已棄用**，將於 v1.27.0 移除[^k3s-net] |
| `none` | 停用 Flannel，改用其他 CNI 外掛（如 Calico、Cilium 等）[^k3s-net] |

K3s 另外支援 `tailscale` 擴充後端，雖然未列於文件表格，但可見於原始碼[^k3s-sc]。

Agent 層級還有 `--flannel-iface`、`--flannel-conf`、`--flannel-cni-conf` 可覆蓋預設網路介面與設定檔路徑[^k3s-net]。

## K3s 沒有維護自己的 Flannel fork

K3s **並不**自建或維護一套 K3s 專用的 Flannel。它直接依賴上游 `flannel-io/flannel`：原始碼中的匯入路徑全部來自 `github.com/flannel-io/flannel/...`[^k3s-flannelgo]。過去存在於 `github.com/rancher/flannel` 的舊 fork 已於 2026 年 3 月 27 日被歸檔、不再使用，其 README 僅會重導到上游專案[^rancher-flannel]。

此外，Flannel 官方 README 也明確提到 K3s 的情況：「例如 K3s 會利用開源的 K3s Installer 來驅動設定流程，在它所建立的 Kubernetes 叢集中設定 flannel」[^flannel-readme]。

## 總結

- K3s 內建的 Flannel 是指 K3s **預設、嵌入式**的第 3 層網路解決方案
- 它以**函式庫**形式直接編譯進 `k3s` 單一二進位檔，無獨立的 `flanneld` 或額外容器
- 預設使用 **VXLAN** 後端，也可切換為 host-gw、WireGuard 等
- 使用的是**上游** `flannel-io/flannel`，K3s 沒有專屬 fork

## 參考資料

[^k3s-net]: K3s Project. (n.d.). *Networking / Basic Network Options — Flannel Options*. Retrieved 2026-10-09, from https://docs.k3s.io/networking/basic-network-options

[^flannel-readme]: flannel-io. (n.d.). *flannel README*. Retrieved 2026-10-09, from https://github.com/flannel-io/flannel

[^k3s-gomod]: Rancher/K3s. (n.d.). *go.mod*（宣告 `github.com/flannel-io/flannel v0.28.4` 為直接依賴）. Retrieved 2026-10-09, from https://github.com/k3s-io/k3s/blob/main/go.mod

[^k3s-flannelgo]: Rancher/K3s. (n.d.). *pkg/agent/flannel/flannel.go*（匯入上游 flannel-io/flannel 套件）. Retrieved 2026-10-09, from https://github.com/k3s-io/k3s/blob/main/pkg/agent/flannel/flannel.go

[^k3s-setup]: Rancher/K3s. (n.d.). *pkg/agent/flannel/setup.go*（後端選擇與程序內啟動 Flannel）. Retrieved 2026-10-09, from https://github.com/k3s-io/k3s/blob/main/pkg/agent/flannel/setup.go

[^k3s-sc]: Rancher/K3s. (n.d.). *pkg/agent/flannel/setup.go（tailscale 擴充後端）*. Retrieved 2026-10-09, from https://github.com/k3s-io/k3s/blob/main/pkg/agent/flannel/setup.go

[^rancher-flannel]: rancher. (2026, March 27). *rancher/flannel（已歸檔的 fork）*. Retrieved 2026-10-09, from https://github.com/rancher/flannel
