# Docker AI Sandboxes（sbx）的 FOSS 替代方案

## 摘要

Docker AI Sandboxes（`sbx` CLI）是 Docker 推出的商業產品，用於在隔離的 microVM 中執行 AI 程式碼代理（coding agent）：每個沙箱擁有獨立的 Docker daemon、檔案系統與網路，代理可在其中建置容器、安裝套件、修改檔案，而不會接觸主機資源[^sbx-docs]。其架構核心為 microVM 隔離、主機側代理（proxy）管控網路與注入憑證、檔案系統 passthrough 與快照[^sbx-arch]。本文調查 2024–2026 年間可作為此產品替代品的開源（FOSS）方案，分為三層：(1) 專為 AI 代理沙箱打造的平台（E2B、Daytona、Coder Agents）；(2) 可作沙箱後端的代理平台（OpenHands、DevPod）；(3) 底層隔離建構塊（Firecracker、gVisor、Kata Containers）。重點發現：Daytona 已於 2026-06 轉閉源並封存公開倉庫；E2B 是當前最接近、可自架（Apache-2.0、Firecracker microVM）的替代品；其餘方案多為容器層級隔離而非 microVM。

## 1. 背景：Docker AI Sandboxes 是什麼

Docker Sandboxes 讓 AI 程式碼代理在隔離的 microVM 沙箱中執行。每個沙箱各有自己的 Docker daemon、檔案系統與網路，代理可以建置容器、安裝套件、修改檔案，無法存取主機資源（除了使用者明確共享的部分）。`sbx` CLI 免費使用（含商業用途），組織治理功能需付費訂閱[^sbx-docs]。

架構重點[^sbx-arch]：

- **microVM 隔離**：每個沙箱為獨立虛擬機，有自己的 Docker daemon 狀態、image cache 與套件安裝，沙箱之間不共享 image 或層。
- **工作區儲存**：可透過檔案系統 passthrough 直接掛載主機目錄（即時雙向同步，無同步程序）；亦可無掛載運作（資料存在 VM 內並跨重啟保留）；Clone 模式則以唯讀掛載來源並在沙箱內建立私人 clone。
- **網路與憑證**：沙箱所有對外 TCP 流量經由主機側代理，代理強制網路政策，並在請求離開 microVM 後才將哨兵（sentinel）憑證替換為真實值——真實憑證全程不進入沙箱。
- **效能**：直接掛載的工作區預設啟用 virtiofs 快取，減少檔案讀寫繞行主機的往返。

因此，功能完備的 FOSS 替代品至少應具備：受隔離的執行環境、程式化 API/CLI 可建立與控制沙箱、網路與資源管控，且最好可自架。

## 2. 專為 AI 代理沙箱打造的平台

### 2.1 E2B（最接近的替代品）

- **授權**：Apache-2.0[^e2b]
- **隔離機制**：每個沙箱一個 Firecracker microVM（KVM），具備 snapshot 續跑、userfaultfd 懶載入記憶體、CoW 根檔案系統、nftables egress firewall、cgroup 與 network namespace 隔離[^e2b-serve]。
- **功能**：Python/JS/TS SDK、Code Interpreter、Desktop（computer use：滑鼠、鍵盤、截圖）、檔案系統操作、URL 路由、暫停/續跑、從執行中的沙箱 fork、快照[^e2b]。
- **自架**：**可**。E2B 提供「E2B Embed」模式，可在單一 Linux 主機（需 KVM，x86-64 或 arm64）以 `docker compose up -d --wait` 啟動完整堆疊（PostgreSQL、Redis、ClickHouse、控制平面、orchestrator、dashboard）；亦提供 Terraform on GCP 與 Kubernetes 安裝方式；API 金鑰於首次啟動自動產生[^e2b-embed]。亦可透過 Terraform 部署於 AWS/GCP/Azure[^e2b-serve]。注意：Embed 是單機評估計畫，生產部署需企業方案[^e2b-embed]。
- **缺點/限制**：主機僅支援 Linux（不支援 macOS/Windows host）；單機模式非生產規模；E2B 公司同時提供代管雲端服務，部分商業考量存在[^e2b-embed]。
- **現況**：活躍開發，runtime 平台版本持續釋出（如 2026-09-17 的 API 版本），並使用 AWS 的 Firecracker v1.14[^e2b-serve]。

### 2.2 Daytona（已封存，歷史參考）

- **授權**：公開倉庫保留原授權，但**已不再維護**[^daytona]
- **隔離機制**：宣稱「完整可組合電腦（full composable computers）」——獨立核心、檔案系統、網路堆疊與分配的 vCPU/RAM/磁碟，OCI/Docker 相容[^daytona]。
- **歷史**：曾是星星數最高（71.7k）的開源 AI 程式碼執行基礎設施，主打 <90ms 沙箱啟動、多語言 SDK、快照、computer use、MCP server[^daytona]。
- **現況（重要）**：**2026-06 起核心開發移往私有 codebase，公開倉庫不再更新**。公開倉庫仍可自由使用與 fork（依授權條款，as-is 無支援）[^daytona]。因此它不再是可長期依賴的 FOSS 選項。

### 2.3 Coder（企業級代理基礎設施）

- **授權**：AGPL-3.0[^coder]
- **隔離機制**：以 Terraform 定義環境，可為 Docker container、Kubernetes Pod、EC2 VM 等；透過 WireGuard 安全隧道存取[^coder]。
- **功能**：「Coder Agents」支援 AI 程式碼代理跑在自己的基礎設施上；代理迴圈執行於控制平面，工作區不存放 API 金鑰；集中式模型治理與稽核日誌；可帶入任何模型[^coder]。
- **自架**：**可**。安裝腳本或二進位，需 PostgreSQL[^coder]。
- **優缺點**：企業級治理與稽核是強項，但資源需求較重，且非以「每代理一個 microVM」為預設模型；隔離程度取決於 Terraform 設定的後端[^coder]。

## 3. 可作為沙箱後端的代理平台

### 3.1 OpenHands

- **授權**：MIT[^openhands]
- **隔離機制**：代理預設在 Docker 沙箱執行（也可在主機、VM 或 OpenHands Cloud 執行）；可執行 OpenHands 自家代理或 Claude Code、Codex、Gemini 等任何 ACP 相容代理[^openhands]。
- **自架**：**可**。`npm install -g @openhands/agent-canvas`、Docker image 或原始碼[^openhands]。
- **優缺點**：生態系大（89k stars）、易於自架；但隔離為 **Docker container 層級**（與主機共享核心），非 microVM——若需求是執行不受信任程式碼，需外加 gVisor/Kata 等強隔離 runtime。
- **相關**：OpenHands 另抽出獨立的 `sandbox-server` 倉庫（Docker 後端沙箱控制平面，早期階段），以及使用 Helm/K8s 的 OpenHands Cloud——後者採 Polyform Free Trial License，**非開源**，限制每年 30 天試用[^sandbox-server][^openhands-cloud]。

### 3.2 DevPod

- **授權**：MPL-2.0[^devpod]
- **隔離機制**：以 devcontainer.json 定義可重現開發環境，每環境跑在 container 中；透過 provider 可部署於本機 Docker、Kubernetes、遠端機器或雲端 VM[^devpod]。
- **自架**：**可**。純 client 工具，無需伺服器後端[^devpod]。
- **優缺點**：非專為 AI 代理設計，但可作為建立沙箱環境的基礎；隔離為 container 層級。

## 4. 底層隔離建構塊（可自行組裝）

若需求是自行建置類似 Docker Sandboxes 的平台，以下是可組裝的建構塊：

| 工具 | 授權 | 隔離機制 | 特性 |
|---|---|---|---|
| Firecracker | Apache-2.0 | microVM（KVM） | AWS Lambda 底層，開機 <125ms，記憶體開銷 <5MiB；非容器 runtime，需搭配管理器[^firecracker] |
| gVisor（runsc） | Apache-2.0 | 使用者空間核心（syscall 攔截） | OCI runtime，可 drop-in 替換 Docker runtime；無需硬體虛擬化，但系統呼叫相容性有缺口[^gvisor] |
| Kata Containers | Apache-2.0 | 每容器輕量 VM | 支援 QEMU、Cloud-Hypervisor、Firecracker、Dragonball；需 KVM[^kata] |

## 5. 其他相關（背景與排除）

- **Windmill**（AGPLv3 + 商業授權）：開發者平台，以 nsjail 提供行程層級沙箱執行不受信任程式碼，可自架（Docker Compose/Helm），但**非 VM 隔離、非專為 AI 代理設計**，沙箱預設關閉需手動設定[^windmill]。
- **Modal**：支援 AI 代理 Sandboxes，但平台為**封閉原始碼的雲端服務，不提供自架**（僅 SDK 開源）[^modal]。
- **Daytona 資源站** `github.com/daytona`：Daytona 的後續（閉源）資源所在[^daytona]。

## 6. 比較總表

| 方案 | 授權 | 隔離技術 | 自架 | 程式化 API/SDK | 專為 AI 代理 | 現況 |
|---|---|---|---|---|---|---|
| Docker Sandboxes（基準） | 商業（sbx 免費）[^sbx-docs] | microVM（Firecracker 類） | 否（本機 CLI） | `sbx` CLI + 整合 | 是 | 2026 活躍 |
| E2B | Apache-2.0 | Firecracker microVM | **是**（Embed/Compose/K8s/Terraform） | Python/JS/TS SDK | 是 | 活躍 |
| Daytona | 原開源授權 | 完整隔離電腦 | 曾是 | SDK（Python/TS/Ruby/Go/Java） | 是 | **封存（2026-06）** |
| Coder | AGPL-3.0 | Container/VM（Terraform） | 是 | API + CLI | 是（Coder Agents） | 活躍 |
| OpenHands | MIT | Docker container | 是 | SDK + CLI | 是 | 活躍 |
| OpenHands Cloud | Polyform（非自由） | Kubernetes | 受限 | SDK | 是 | 活躍 |
| DevPod | MPL-2.0 | Container（devcontainer） | 是 | CLI | 否（通用） | 活躍 |
| Firecracker | Apache-2.0 | microVM | 是 | API（低階） | 否（建構塊） | 活躍 |
| gVisor | Apache-2.0 | 使用者空間核心 | 是（runtime） | 無 | 否（建構塊） | 活躍 |
| Kata Containers | Apache-2.0 | 輕量 VM | 是（runtime） | 無 | 否（建構塊） | 活躍 |
| Windmill | AGPLv3+ | nsjail（行程） | 是 | API | 否 | 活躍 |
| Modal | 封閉（SDK 開源） | container/隔離雲 | **否** | SDK | 是 | 活躍 |

## 7. 選型建議

- **需要最接近 Docker Sandboxes 的完整 FOSS 替代（microVM、自架、API）**：**E2B**——Apache-2.0、Firecracker 隔離、Embed 模式單機自架、完整 SDK，是唯一在產品定位上直接對應的開源專案[^e2b][^e2b-embed]。
- **已有 Kubernetes 且偏企業治理**：Coder Agents，但注意其隔離等級取決於後端設定，非預設 microVM[^coder]。
- **僅需容器層級隔離、想要最成熟代理生態**：OpenHands（MIT、易自架）；若工作負載含不受信任程式碼，再疊加 gVisor 或 Kata 強化[^openhands][^gvisor][^kata]。
- **想從零自建平台**：以 Firecracker 為 VMM 基底，搭配容器管理器與自己的控制平面（E2B 即此路線）[^firecracker]。
- **避免**：不要再以 Daytona 作為新專案依賴（已封存、轉閉源）；Modal 無法自架；OpenHands Cloud 授權非自由[^daytona][^modal][^openhands-cloud]。

## 8. 結論

在 2026 年，Docker AI Sandboxes 的直接 FOSS 替代品選項不多但明確：**E2B** 是目前唯一兼具「專為 AI 代理設計、microVM 級隔離、Apache-2.0、可自架」條件的專案；Daytona 曾是最熱門選擇但已轉閉源。若可接受較弱的容器級隔離，OpenHands 與 DevPod 提供成熟且易自架的基礎；而 Firecracker、gVisor、Kata Containers 則是自行組裝平台的底層建構塊。選擇時應以隔離強度需求、是否需 OCI/容器生態相容、硬體虛擬化（KVM）可用性與自架維運成本為主要權衡。

## 參考來源

[^sbx-docs]: Docker Inc. (n.d.). Docker Sandboxes. Retrieved 2026-09-24, from https://docs.docker.com/ai/sandboxes/

[^sbx-arch]: Docker Inc. (n.d.). Architecture – Docker Sandboxes. Retrieved 2026-09-24, from https://docs.docker.com/ai/sandboxes/architecture/

[^e2b]: e2b-dev. (n.d.). E2B. Retrieved 2026-09-24, from https://github.com/e2b-dev/E2B

[^e2b-serve]: E2B. (n.d.). Self-hosting E2B. Retrieved 2026-09-24, from https://github.com/e2b-dev/runtime

[^e2b-embed]: E2B. (n.d.). E2B Embed. Retrieved 2026-09-24, from https://github.com/e2b-dev/runtime/blob/main/embed/README.md

[^daytona]: Daytona. (n.d.). Daytona – Secure and Elastic Infrastructure for Running AI-Generated Code. Retrieved 2026-09-24, from https://github.com/daytonaio/daytona

[^coder]: Coder. (n.d.). Coder. Retrieved 2026-09-24, from https://github.com/coder/coder

[^openhands]: OpenHands. (n.d.). OpenHands: AI-Driven Development. Retrieved 2026-09-24, from https://github.com/All-Hands-AI/OpenHands

[^sandbox-server]: OpenHands. (n.d.). sandbox-server. Retrieved 2026-09-24, from https://github.com/OpenHands/sandbox-server

[^openhands-cloud]: OpenHands. (n.d.). OpenHands Cloud. Retrieved 2026-09-24, from https://github.com/OpenHands/OpenHands-Cloud

[^devpod]: Loft Labs. (n.d.). DevPod. Retrieved 2026-09-24, from https://github.com/loft-sh/devpod

[^firecracker]: Firecracker. (n.d.). Firecracker. Retrieved 2026-09-24, from https://github.com/firecracker-microvm/firecracker

[^gvisor]: Google. (n.d.). gVisor. Retrieved 2026-09-24, from https://github.com/google/gvisor

[^kata]: Kata Containers. (n.d.). Kata Containers. Retrieved 2026-09-24, from https://github.com/kata-containers/kata-containers

[^windmill]: Windmill Labs. (n.d.). Windmill. Retrieved 2026-09-24, from https://github.com/windmill-labs/windmill

[^modal]: Modal Labs. (n.d.). Modal. Retrieved 2026-09-24, from https://modal.com