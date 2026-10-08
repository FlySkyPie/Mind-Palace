# Egress Mode（出口模式）在反向代理與 API Gateway 中的意義

## 概述

**Egress Mode（出口模式／Egress Gateway）** 是一種反向代理／正向代理的部署模式，將一個專用的閘道放置在受管理環境（如 Kubernetes 叢集、服務網格、私有網路）的邊界上，作為所有**對外流量**的受控出口點。它與更常見的 **Ingress Gateway（入口閘道，管控進入流量）** 是對稱的概念[^istio-egress]。

在現代雲端原生架構中，每個微服務個別直接存取外部服務（第三方 API、SaaS、公共網際網路）會帶來安全、監管、IP 管理上的挑戰。Egress Mode 的核心思維是：**不讓每個服務各自向外連線，而是將所有對外流量集中到一個統一的閘道層**，統一進行路由、安全檢查、監控與策略執行[^proxidize]。

## 工作原理

### 典型流量路徑

```
內部服務 (Pod) → 服務網格 Sidecar → Egress Gateway Pod → 外部服務 (如 httpbin.org、AWS API)
```

流程說明[^istio-egress][^adhdecode]：

1. **內部服務**嘗試連線至某個外部主機。
2. **服務網格的 sidecar 代理**（例如 Istio 的 Envoy sidecar）攔截到對外流量。
3. **路由規則**（如 Istio 的 `VirtualService` + `Gateway` 資源）將目的地改寫為指向 egress gateway 的位址。
4. **Egress gateway pod** 收到流量後，查詢實際的外部目的地，並向該外部服務發起新連線。
5. **外部服務**看到的來源 IP 是 egress gateway 的 IP，而非內部 pod 的 IP。

### 核心設定元件（以 Istio 為例）

Istio 提供了最成熟的 egress gateway 模型，其設定包含以下關鍵資源[^istio-egress]：

- **`ServiceEntry`** — 將外部主機註冊到服務註冊表，標記為 `MESH_EXTERNAL`。
- **`Gateway`** — 定義 egress gateway 的 listener（連接埠、協定、負責的主機）。
- **`VirtualService` / `HTTPRoute`** — 設定流量如何從 mesh sidecar 經由 egress gateway 到達外部服務。
- **`DestinationRule`** — 將目的地映射到特定的 gateway backend 子集。

### 強制執行的重要性

單純定義 egress gateway 並**不能保證所有流量都經過它**。Istio 文件明確指出，如果內部 workload 被入侵而 bypass 了 sidecar，就可以繞過 egress gateway。真正的強制執行需要搭配[^istio-control]：

- Kubernetes `NetworkPolicy` 規則
- 防火牆規則（非 gateway 的流量全部阻擋）
- NAT 設定防止 pod 直接取得公共 IP
- 雲端層級的安全控制

## 實作此模式的產品

### 1. Istio（最重要的參考實作）

Istio 擁有最成熟、文件最完整的 egress gateway 模式。支援兩種方式[^istio-egress]：

- **Sidecar 式 egress**：使用 `ServiceEntry` + `Gateway` + `VirtualService` + `DestinationRule`
- **Kubernetes Gateway API 式 egress**：使用 `Gateway` + `HTTPRoute` / `TLSRoute`，搭配 `parentRefs`

> *「Egress gateway 是 ingress 的對稱概念；它定義了從 mesh 出去的出口點。Egress gateway 讓你可以將 Istio 的功能（例如監控和路由規則）應用於離開 mesh 的流量。」*

### 2. Envoy Proxy

Envoy 常被用作獨立（standalone）的 egress gateway，不需要完整 Istio 服務網格即可運作[^devopstales]：

- 支援**正向代理模式**，透過 `listeners` 和 `clusters` 設定
- 提供進階 L7 功能：rate limiting、mTLS、circuit breaking、JWT 驗證、IP whitelisting
- 可整合 Prometheus、Jaeger、Zipkin 等觀測工具

### 3. Kong API Gateway

Kong 主要用於 ingress，但也能作為 egress gateway。具體案例[^kong-egress]：

- **Checkr 公司**的工程師發表了題為 *「This Way Out: The Benefits of Building an Egress Gateway Pattern」* 的議程，分享他們將 90% 的 egress 流量遷移到 Kong 上。
- Kong Mesh 支援**委託閘道（delegated gateway）**模式，讓外部 API gateway 處理 ingress，Kong Mesh 管理 egress。

### 4. Apache APISIX

APISIX 本身並未在標準部署模式（傳統、解耦、獨立）中提供專屬的「egress mode」。但[^apisix-guide]：

- APISIX 有一個名為 **Amesh** 的服務網格實作，可搭配 Istio 控制層（取代 Envoy 作為 sidecar）。
- APISIX Ingress Controller 可作為 **Istio Egress Gateway** 的 drop-in 替代方案。
- 文件明確描述 APISIX 在服務網格中扮演 ingress/egress 角色的方式。

> *「服務網格也有基本的 ingress/egress gateway 來處理 north-south 流量…egress gateway 讓 mesh 內部的服務可以存取外部服務。」*

### 5. Nginx

Nginx 可透過以下方式實現類似 egress 的模式[^nginx-forward]：

- **HTTP CONNECT forward proxy**（Nginx Plus R36+）：使用 `tunnel_pass` 指令啟用正向代理模式。
- **與 Istio sidecar 搭配**：由 Nginx Ingress Controller 搭配 Istio Proxy sidecar 管理 egress 流量。
- 支援 mTLS 認證、連接埠／主機限制、存取記錄。

### 6. Cilium（eBPF 式 egress）

Cilium 提供基於 eBPF 的 egress gateway，無需 sidecar，以高效能進行對外流量控制。它在網路層應用 **SNAT（Source Network Address Translation）**，提供可預測的來源 IP 管理[^cncf-envoy]。

### 7. Monzo Egress Operator（Kubernetes-native）

Monzo 銀行開源的 Kubernetes Operator，建立 **Envoy 為底層的 egress gateway pod**，並透過 Kubernetes CRD 和網路政策控制存取。它使用 CoreDNS plugin 改寫 DNS 查詢，將 egress 流量導向 gateway pod[^monzo]。

### 8. 其他解決方案

- **Squid Proxy**：傳統 HTTP forward proxy，具備快取和 ACL 功能。
- **Cloud NAT gateway**：Aws NAT Gateway、GCP Cloud NAT、Azure Firewall/NAT — 網路層的 egress 解決方案。
- **Tyk**：主要為 ingress API gateway，但社群有提出 outbound mTLS 的功能請求。
- **Traefik**：主要聚焦於 ingress／反向代理。

## 主要使用場景

| 場景 | 說明 | 常見產品 |
|------|------|----------|
| **安全與合規** | 所有對外流量必須通過專用、可審計的節點。防止資料外洩。 | Istio、Envoy + NetworkPolicy |
| **來源 IP 管理** | 提供外部合作夥伴一個可預測的來源 IP，便於防火牆 allowlist 設定。 | Cilium (SNAT)、Istio |
| **Pod 無公共 IP** | 應用節點不具備公共 IP，僅 egress gateway 節點配置公共 IP。 | Istio、Envoy |
| **集中審計** | 在單一操作層收集對外流量的 log、metrics 與 traces。 | Envoy (Prometheus + Jaeger)、Istio |
| **TLS 起始** | 應用發起 HTTP，egress gateway 建立 TLS 到外部服務。 | Istio（TLS origination at egress） |
| **對外 Rate Limiting** | 防止某個內部服務對外部 API 造成過度請求。 | Envoy (local rate limit)、Istio |
| **出口策略強制執行** | 只允許核准的目的地，其餘全部阻擋。 | Monzo Egress Operator、Istio + NetworkPolicy |
| **對外 mTLS** | 在連接第三方 API 時出示客戶端憑證。 | Envoy、Nginx Plus |
| **AI/LLM API 存取控制** | 控制哪些內部服務可存取外部 AI/LLM API，並進行審計和 rate limiting。 | Kong AI Gateway、Envoy |
| **多雲／混合架構** | 跨雲端提供一致的 egress 策略。 | Custom Envoy、Istio |
| **SaaS／金流呼叫** | 保護微服務對 Stripe、Twilio、AWS 等外部服務的呼叫。 | 任意 egress gateway |

## Egress Gateway vs. 正向代理 vs. 反向代理 vs. API Gateway

以下比較表整理這四種模式的核心差異[^proxidize]：

| 問題 | 正向代理 (Forward Proxy) | 反向代理 (Reverse Proxy) | API Gateway | Egress Gateway |
|------|------------------------|------------------------|-------------|----------------|
| **代表誰？** | 請求者（客戶端） | 被請求的服務 | API 產品 | 受管理的 workload 環境 |
| **由誰控制？** | 客戶端／網路團隊 | 服務／基礎設施團隊 | API 平台團隊 | 平台／安全團隊 |
| **主要任務？** | 將請求透過另一台伺服器轉發 | 接收並保護 inbound 流量 | 發布和管理 API | 集中管控對外路由、策略、身分、遙測 |
| **目的地看到什麼？** | 代理的 IP | N/A | Gateway 作為服務端點 | Gateway 或下游 NAT 的 IP |
| **典型範例** | Squid、Proxidize | Nginx、HAProxy、Envoy | Kong、APISIX、AWS API Gateway | Istio Egress Gateway、Cilium Egress Gateway |

## 總結

**Egress Mode（出口模式）** 是反向代理／API Gateway 領域中的一個重要架構模式，其核心是將一個專用閘道部署在受管理環境的邊界上，統一管控所有**對外流量**。這個模式在 Kubernetes 服務網格（尤其是 Istio）中最為成熟，但 Kong、APISIX（透過 Amesh）、Nginx Plus、Cilium 等產品也能以不同方式參與 egress 場景。

它不僅僅是「反向代理做正向代理的事」—— egress gateway 代表的是**受管理 workload 環境的出口**，其關注點在於安全合規、來源 IP 管理、集中審計和策略執行，而非單純的請求轉發。在現代雲端原生與服務網格架構中，Egress Mode 已成為不可或缺的安全與治理基礎設施。

---

## 參考文獻

[^istio-egress]: Istio. (n.d.). *Egress Gateway*. Retrieved 2026-10-01, from https://istio.io/latest/docs/tasks/traffic-management/egress/egress-gateway/

[^istio-control]: Istio. (n.d.). *Egress Control*. Retrieved 2026-10-01, from https://istio.io/latest/docs/tasks/traffic-management/egress/egress-control/

[^proxidize]: Proxidize. (n.d.). *Forward Proxy vs. Reverse Proxy vs. API Gateway vs. Egress Gateway — Definitions, Comparisons and When to Use Each*. Retrieved 2026-10-01, from https://proxidize.com/blog/forward-proxy-vs-reverse-proxy-vs-api-gateway-vs-egress-gateway/

[^apisix-guide]: APISIX. (n.d.). *A Comprehensive Guide to API Gateways, Kubernetes Gateways, and Service Meshes*. Retrieved 2026-10-01, from https://dev.to/apisix/a-comprehensive-guide-to-api-gateways-kubernetes-gateways-and-service-meshes-34d4

[^devopstales]: DevOps Tales. (n.d.). *Custom Envoy Egress Proxy*. Retrieved 2026-10-01, from https://devopstales.github.io/kubernetes/custom-envoy-egress-proxy/

[^kong-egress]: Kong. (n.d.). *This Way Out: The Benefits of Building an Egress Gateway Pattern*. Retrieved 2026-10-01, from https://konghq.com/resources/videos/way-benefits-building-egress-gateway-pattern

[^adhdecode]: adhdecode. (n.d.). *Istio Egress Gateway Configuration*. Retrieved 2026-10-01, from https://adhdecode.com/articles/istio/istio-egress-gateway-configuration/

[^cncf-envoy]: CNCF. (2025-08-26). *Use Envoy Gateway as the Unified Ingress Gateway and Waypoint Proxy for Ambient Mesh*. Retrieved 2026-10-01, from https://www.cncf.io/blog/2025/08/26/use-envoy-gateway-as-the-unified-ingress-gateway-and-waypoint-proxy-for-ambient-mesh/

[^nginx-forward]: Nginx. (n.d.). *NGINX as an HTTP CONNECT Forward Proxy*. Retrieved 2026-10-01, from https://docs.nginx.com/nginx/admin-guide/web-server/http-connect-proxy/

[^monzo]: Monzo. (n.d.). *Egress Operator*. Retrieved 2026-10-01, from https://github.com/monzo/egress-operator