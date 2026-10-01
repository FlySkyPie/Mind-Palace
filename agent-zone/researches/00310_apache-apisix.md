# Apache APISIX 研究報告

## 概述

**Apache APISIX** 是一個動態、即時、高效能的雲端原生 API 閘道 (API Gateway)，建基於 OpenResty (NGINX + LuaJIT) 之上。它提供負載平衡、動態上游配置、金絲雀發布 (Canary Release)、熔斷 (Circuit Breaking)、認證、可觀測性 (Observability) 等豐富的流量管理功能。自 3.16 版起，APISIX 也演進為 **AI 閘道 (AI Gateway)**，原生支援 LLM 代理、Token 為基準的速率限制、語意模型路由及 AI 快取。

APISIX 是 **Apache 軟體基金會頂級專案** (Top-Level Project)[^apache-top-level]，採用 Apache 2.0 授權條款，截至 2026 年擁有 **17,200+ GitHub 星數**、**3,000+ 分叉**、**600+ 貢獻者** 與 **600 萬+ 下載次數**。

## 歷史與發展

| 時間 | 里程碑 |
|------|--------|
| 2019 年初 | 兩位工程師（隸屬於支流科技，後更名為 API7.ai）在小型會議室中從頭打造 APISIX |
| 2019 年 6 月 6 日 | APISIX 在 GitHub 上開源 |
| 2019 年 10 月 | 捐贈予 Apache 軟體基金會，進入孵化器 |
| **2020 年 7 月 15 日** | **正式畢業為 Apache 頂級專案**，為 ASF 史上最快孵化案例之一[^apache-top-level] |
| 2025 年 | 3.13.0 及 3.14.0 版開始引入 AI 閘道功能 |
| **2026 年 8 月** | **3.18.0 版** — 大幅擴充 AI 閘道功能（AI 快取、語意路由、Lakera Guard）[^release-318] |
| **2026 年 9 月** | **3.19.0 版** — 原生 WebSocket 框架處理、OpenAPI-to-MCP、TLS 穿透、Slow Start[^release-319] |

目前版本週期約 **每兩個月一版**。

## 主要功能

### 核心能力
- **完整動態配置**：熱更新、熱插件—配置變更在 **毫秒級生效**，不需重新啟動程序
- **管理方式**：RESTful Admin API（連接埠 9180）、CLI、儀表板 (Dashboard)
- **配置模式**：etcd 支援、Standalone 檔案驅動 (YAML/JSON)、Standalone API 驅動、分離式控制層/資料層模式

### 路由 (Routing)
- 完整路徑比對、前綴比對、內容為基準路由、地理路由
- 支援所有 NGINX 內建變數作為路由條件（如 cookie、args 等）
- GraphQL 屬性路由
- 使用 **Radix Trie（前綴樹）** 實現超快速路由查閱，即使超過 100,000 條路由仍表現良好

### 多協定支援
- HTTP/1.1、HTTP/2、HTTP/3 (QUIC)
- gRPC、gRPC-Web、gRPC 轉換（HTTP ↔ gRPC）
- WebSocket（3.19.0 起原生框架處理）
- TCP/UDP（串流代理）
- MQTT（物聯網協定）
- Dubbo（RPC 框架）、Proxy Protocol、SSL/TLS、mTLS

### 負載平衡與流量管理
- 加權輪詢 (Weighted Round-robin)、一致性雜湊 (Consistent Hashing)、最少連線 (Least Connections)
- 主動/被動健康檢查
- 熔斷、金絲雀發布、A/B 測試、藍綠部署
- 流量分割（百分比為基準）、請求鏡像、**Slow Start 暖機**（3.19.0）、故障注入

### 安全性與認證（100+ 內建插件）
- **認證**：key-auth、JWT、basic-auth、HMAC-auth、OpenID Connect、OAuth2、LDAP、Casbin、Keycloak、Casdoor
- **速率限制**：limit-req、limit-count、limit-concurrency（Redis 滑動視窗）
- **IP/限制**：IP 白/黑名單、Referer 限制、CORS、CSRF、請求驗證器
- **AI 安全性**：Lakera Guard（3.18.0）、AWS 及阿里雲 AI 內容審核、Prompt Guard

### 可觀測性與監控
- Prometheus 指標（原生端點、Grafana 儀表板）
- OpenTelemetry 追蹤、Apache SkyWalking、Zipkin、Datadog
- **記錄器**：HTTP、TCP、UDP、Kafka、RocketMQ、Syslog、Elasticsearch、ClickHouse、Splunk 等

### 多語言插件開發
- **原生**：Lua（LuaJIT）
- **外部插件執行器**：Go、Java、Python、Node.js（透過 RPC/Sidecar）
- **WebAssembly**（實驗性）

### Kubernetes 整合
- **APISIX Ingress Controller** — 作為 Kubernetes Ingress，支援 Gateway API
- Helm Chart 部署
- 服務探索：Kubernetes、Consul、Nacos、Eureka、DNS

### AI 閘道能力（3.16+）
- 透過統一介面代理多個 LLM 提供者
- LLM 間的負載平衡、重試、備援
- **Token 為基準的速率限制**
- **AI 回應快取**：Redis 為基準的精確 L1 快取 + 語意 L2 快取（餘弦相似度/RediSearch）— 3.18.0
- **語意模型路由**：透過嵌入 (Embedding) 依語意路由提示 — 3.18.0
- **OpenAPI-to-MCP**：將 OpenAPI 文件轉換為 MCP 伺服器 — 3.19.0
- **MCP 橋接插件**：stdio 為基準的 MCP 轉 HTTP SSE

## 架構

APISIX 建基於 **OpenResty (NGINX + LuaJIT)**，利用 LuaJIT 進行程序內請求處理。

### 控制層 / 資料層分離
架構將 **控制層 (Control Plane)** 與 **資料層 (Data Plane)** 完全分離：
- **資料層**：無狀態的 APISIX 節點，處理所有流量，不儲存持久狀態
- **控制層**：配置儲存於 **etcd** 叢集，透過 long-polling watcher 即時同步至 APISIX 節點

**配置流程**：Admin API → etcd → 監聽事件 → APISIX worker 程序（傳播時間 < 100 毫秒）

### 部署模式
1. **傳統模式**：單一 APISIX 角色同時處理流量 + Admin API，etcd 儲存
2. **分離模式**：分離控制層與資料層角色，透過 etcd 共享配置
3. **Standalone 檔案驅動模式**：本機 YAML/JSON 配置，不需 etcd
4. **Standalone API 驅動模式**：全記憶體配置，專用 Standalone Admin API

### 請求生命週期
插件依序執行：**rewrite → access → header_filter → body_filter → log**，可透過 `_meta.priority` 調整優先權。

## 與其他 API 閘道比較

| 面向 | Apache APISIX | Kong | Envoy | NGINX |
|------|:---:|:---:|:---:|:---:|
| **底層語言** | Lua (NGINX+LuaJIT) | Lua (NGINX+LuaJIT) | C++ | C |
| **配置儲存** | **etcd** | PostgreSQL 或 DB-less | 靜態檔案或 xDS | 靜態檔案 |
| **配置傳播** | **毫秒級** (etcd watch) | ~5 秒 DB 輪詢；DB-less 需重載 | xDS 動態 | 需重載 |
| **插件生態** | **100+ 全部開源** | 100+（許多限企業版） | HTTP/網路過濾器 | 有限（第三方） |
| **插件語言** | **Lua、Go、Java、Python、Wasm** | Lua、Go、Python、JS | C++、Wasm | C、Lua |
| **gRPC** | **原生支援** | 支援 | 原生 | 有限 |
| **Kubernetes** | **APISIX Ingress + Gateway API** | Kong Ingress | Envoy + Istio/Gloo | NGINX Ingress |
| **儀表板** | **內建（開源）** | 僅企業版 | 管理介面 | 第三方 |
| **授權** | **Apache 2.0（全部開源）** | Apache 2.0 核心 + 企業付費牆 | Apache 2.0 | BSD-like |
| **治理模式** | **中立 (ASF)** | 單一供應商 (Kong Inc.) | CNCF（中立） | F5（商業） |

### 主要區別

- **vs Kong**：APISIX 使用 etcd（不需資料庫），AI/LLM 插件完全開源（Kong 將多數進階功能設在企業版），配置傳播毫秒級 vs Kong 約 5 秒輪詢。Kong 由單一供應商控制，APISIX 由 ASF 治理。
- **vs NGINX**：APISIX 建於 NGINX 之上，但增加了動態路由、100+ 插件、Admin API、儀表板、etcd 配置。NGINX 需編輯設定檔並重載。
- **vs Envoy**：Envoy 是高效能 C++ 代理，需搭配 Istio/Gloo 等控制層。APISIX 為一站式閘道，內建控制層、Admin API 與儀表板。

### 效能
- APISIX 自報 **每核心約 18,000 QPS**，**平均延遲 0.2 ms**
- 8 核心 AWS：**140,000 QPS**，0.2 ms 延遲[^benchmark]

## 知名生產用戶

**Zoom**（APISIX Ingress Controller，多可用區部署）、**NASA JPL**、**Bilibili**、**騰訊遊戲**、**Lenovo**、**OPPO**、**vivo**、**新浪微博 (Sina Weibo)**、**Airwallex**、**Geely**、**Swisscom**、**Nayuki** 等[^zoom-case]。

## 應用場景

1. **微服務 API 管理**：動態路由數百個後端服務，負載平衡與健康檢查
2. **API 安全與認證**：集中式 JWT/OIDC/OAuth2/LDAP/mTLS 認證，速率限制，IP 限制
3. **Kubernetes Ingress**：自動將 Kubernetes 資源轉譯為 APISIX 路由規則
4. **多協定閘道**：統一處理 HTTP、gRPC、WebSocket、TCP/UDP、MQTT、Dubbo
5. **AI/LLM 閘道**：多 LLM 提供者路由、負載平衡、快取、Token 速率限制、語意路由、內容審核
6. **金絲雀發布與 A/B 測試**：逐步切換上游服務版本流量
7. **API 聚合**：合併多個後端服務呼叫至單一客戶端端點
8. **可觀測性**：集中式指標、追蹤、結構化記錄
9. **流量控制與故障處理**：熔斷、健康檢查、重試、故障注入
10. **混合雲部署**：跨私有雲與公有雲（如 Zoom 的多可用區）[^zoom-case]

## 總結

Apache APISIX 是功能最完整的開源 API 閘道之一，以 **完全動態配置、毫秒級生效、100+ 開源插件、多語言支援** 為核心優勢。2025–2026 年快速演化為 AI 閘道，具備 LLM 代理、語意快取、模型路由等能力。其 Apache 基金會治理模式確保供應商中立性，與 Kong 的單一供應商模式形成對比。

---

[^apache-top-level]: Apache Software Foundation. (2020). Apache APISIX graduates as a Top-Level Project. Retrieved 2026-10-01, from https://apisix.apache.org/
[^release-318]: Apache APISIX. (2026-08-20). Release Apache APISIX 3.18.0. Retrieved 2026-10-01, from https://apisix.apache.org/blog/2026/08/20/release-apache-apisix-3.18.0/
[^release-319]: Apache APISIX. (2026-09-28). Release Apache APISIX 3.19.0. Retrieved 2026-10-01, from https://apisix.apache.org/blog/2026/09/28/release-apache-apisix-3.19.0/
[^benchmark]: Apache APISIX. (n.d.). Performance Benchmarks. Retrieved 2026-10-01, from https://apisix.apache.org/docs/apisix/benchmark/
[^zoom-case]: API7.ai. (2022). Zoom Uses APISIX Ingress Controller. Retrieved 2026-10-01, from https://api7.ai/blog/zoom-uses-apisix-ingress