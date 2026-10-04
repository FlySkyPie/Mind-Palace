# Docker AI Sandboxes 的 FOSS 替代方案

Docker AI Sandboxes 是 Docker 公司在 2025 年推出的產品，讓 AI 代理（AI agents）在隔離的沙盒環境中執行程式碼，每個沙盒擁有自己的 Docker daemon、檔案系統與網路[^docker]。然而 Docker AI Sandboxes 是封閉原始碼的託管服務，需要綁定 Docker Desktop/Cloud 生態。以下整理各類開放原始碼（FOSS）替代方案，依隔離模型分層討論。

## 隔離層級總覽

| 隔離層級 | 技術 | 強度 | 適用情境 |
|---|---|---|---|
| OS 層級 Jail | nsjail、bubblewrap、Firejail | 最弱（共用 host kernel） | 執行單一二進位檔、可信任度較高的程式碼、最低開銷 |
| 容器強化 | Sysbox、Podman | 中等（user namespace、無 VM） | Docker-in-Docker、系統容器、比純 Docker 更強的隔離 |
| 使用者空間核心 | gVisor | 中高（系統呼叫攔截） | 已有容器基礎架構，想升級隔離但保持相容性 |
| 微型 VM | Firecracker、Kata Containers、OpenSandbox、microsandbox、E2B、Daytona、SmolVM | 最強（硬體虛擬化、KVM） | 完全不信任的程式碼、AI 代理、多租戶大規模部署 |

## 基礎隔離元件（較低階，需自行整合）

這類工具提供隔離的底層機制，使用者需自行搭建上層的沙盒管理系統。

### 1. Firecracker ⭐ 37,133

- **授權：** Apache 2.0
- **團隊：** Amazon
- **GitHub：** https://github.com/firecracker-microvm/firecracker
- **說明：** 以 Rust 撰寫的開源虛擬機器監控器（VMM），透過 KVM 建立輕量的 microVM。每個 microVM 執行獨立核心，冷啟動低於 200ms，耗費小於 5 MiB 記憶體，是 AWS Lambda 與 Fargate 的底層技術。目前許多沙盒平台皆建構於此之上[^firecracker]。


### 2. gVisor ⭐ 19,494

- **授權：** Apache 2.0
- **團隊：** Google
- **GitHub：** https://github.com/google/gvisor
- **說明：** 一種「應用層核心」，在使用者空間攔截沙盒程式的系統呼叫，而非傳遞給 host kernel。可作為 OCI runtime（runsc）整合進 Docker/Kubernetes。隔離強度介於純容器與完整 VM 之間，此模型會降低特定工作負載的相容性[^gvisor]。


### 3. nsjail ⭐ 4,134

- **授權：** Apache 2.0
- **團隊：** Google
- **GitHub：** https://github.com/google/nsjail
- **說明：** 輕量行程隔離工具，使用 Linux namespaces（mount、PID、IPC、NET、USER、UTS、cgroups）、rlimits 以及 seccomp-bpf 系統呼叫過濾。常用於 CTF 解題、fuzzing 以及需要在嚴謹限制下執行單一不受信任的二進位檔。無 VM 開銷，但共用 host kernel[^nsjail]。


### 4. bubblewrap ⭐ 8,914

- **授權：** LGPL
- **團隊：** containers/freedesktop
- **GitHub：** https://github.com/containers/bubblewrap
- **說明：** 低階的非特權沙盒工具，使用 Linux user namespaces，也是 Flatpak 的基礎。在 tmpfs 上建立全新的空 mount namespace，除非明確綁定否則隱藏所有 host 檔案系統。支援 PID/IPC/UTS/network namespaces、seccomp 過濾與能力卸除。不需 setuid 模式[^bwrap]。


### 5. Sysbox ⭐ 3,890

- **授權：** Apache 2.0
- **團隊：** Nestybox（已被 Docker 收購）
- **GitHub：** https://github.com/nestybox/sysbox
- **說明：** 專用的 OCI container runtime（可直接取代 runc），透過 Linux user-namespaces（容器內 root = host 非特權使用者）強化容器隔離，並虛擬化 procfs/sysfs、隱藏 host 資訊。允許在容器內執行 systemd、Docker、Kubernetes，不需特權容器或 host socket 掛載。隔離強度介於容器與 VM 之間[^sysbox]。


### 6. Kata Containers ⭐ 8,870

- **授權：** Apache 2.0
- **團隊：** CNCF 專案
- **GitHub：** https://github.com/kata-containers/kata-containers
- **網站：** https://katacontainers.org
- **說明：** 將每個容器放在輕量 VM（Firecracker、QEMU 或 Cloud Hypervisor）中執行，同時提供標準 OCI/container 介面。可透過 CRI 與 Kubernetes 整合。提供硬體虛擬化等級的隔離，同時保留容器化的開發者體驗，屬於建構模塊而非完整沙盒平台[^kata]。


## 完整沙盒平台（提供 API/SDK，可自建託管）

這類專案提供接近 Docker AI Sandboxes 的完整體驗，包含沙盒生命週期管理、網路隔離、檔案系統等。

### 7. OpenSandbox ⭐ 15,654

- **授權：** Apache 2.0
- **團隊：** Alibaba
- **GitHub：** https://github.com/opensandbox-group/OpenSandbox
- **網站：** https://open-sandbox.ai
- **說明：** 生產級 AI 代理沙盒 runtime，底層使用 Firecracker。本地使用 Docker，生產環境使用 Kubernetes。提供統一的沙盒 API，支援程式碼代理、GUI 代理、代理評估、程式碼執行及強化學習（RL）訓練。具備 97ms P50 的快速沙盒啟動、網路存取控制、憑證庫、暫停/恢復功能。提供 Python、Java、TypeScript、C#、Go 的 SDK，並內建 MCP server 整合。功能最全面的 Docker AI Sandboxes FOSS 替代方案之一[^opensandbox]。


### 8. microsandbox ⭐ 8,546

- **授權：** Apache 2.0
- **團隊：** Super Rad Company
- **GitHub：** https://github.com/superradcompany/microsandbox
- **網站：** https://microsandbox.dev
- **說明：** 開源自建 microVM 平台，使用 libkrun 提供硬體隔離的 microVM。可將 OCI 容器映像檔在 microVM 內執行。啟動低於 100ms、支援 snapshot/fork、每個沙盒獨立網路、Docker-in-VM、MCP 整合。支援 Linux、macOS（Hypervisor.framework）、Windows（WSL2）。提供 Rust、Python、TypeScript、Go 的 SDK[^microsandbox]。
### 9. E2B ⭐ 14,140

- **授權：** Apache 2.0
- **GitHub：** https://github.com/e2b-dev/E2B
- **網站：** https://e2b.dev
- **說明：** 以 Firecracker 建構的雲端沙盒，專為 AI 代理設計。提供程式碼直譯器 SDK、自訂沙盒模板、桌面沙盒（電腦使用）、低於 200ms 冷啟動、單次連線最長 24 小時。支援自帶雲端（BYOC）/自建部署。提供 Python 與 JavaScript/TypeScript SDK。Hugging Face、Manus、Groq 等公司使用[^e2b]。
### 10. Daytona ⭐ 71,670

- **授權：** Apache 2.0
- **GitHub：** https://github.com/daytonaio/daytona
- **網站：** https://daytona.io
- **說明：** 安全彈性的 AI 程式碼執行基礎設施。低於 90ms 沙盒建立、大量平行化執行、檔案/Git/LSP/執行 API、環境快照、電腦使用（Linux/macOS/Windows）、Docker-in-Docker、GPU 支援（Nvidia H100）。提供 Python 與 TypeScript SDK。LangChain、Turing 等公司使用[^daytona]。
### 11. PandaStack ⭐ 35

- **授權：** Apache 2.0（核心）
- **GitHub：** https://github.com/iam-the-ironman/pandastack（原始上游：https://github.com/pandastack-io/pandastack-ai ⭐ 35）
- **網站：** https://pandastack.ai
- **說明：** 以 Firecracker 為基礎的開源沙盒平台，每個沙盒即一個 Firecracker microVM。每次建立即做 snapshot-restore（179ms P50）、同主機複製寫入（COW forking）約 400ms、每個沙盒獨立網路隔離（NATID）。可在同一套基礎設施上託管 PostgreSQL/git。需在具備 /dev/kvm 的 Linux 主機上自建[^pandastack]。
### 12. SmolVM ⭐ 1,005

- **授權：** Apache 2.0（依各儲存庫）
- **團隊：** Celesto AI
- **GitHub：** https://github.com/CelestoAI/celesto
- **網站：** https://celesto.ai
- **說明：** 開源 AI 沙盒，支援彈性 microVM 後端 ── 內建 QEMU 與 Firecracker 支援。提供完整 VM，包含獨立檔案系統、網路與行程空間。支援 Ubuntu、Windows 或任何作業系統。具備狀態恢復的快照功能、GPU passthrough、網路控制。可透過 pip install smolvm 安裝 Python SDK[^smolvm]。
### 13. Beam ⭐ 1,802

- **授權：** AGPL 3.0
- **GitHub：** https://github.com/beam-cloud/beta9
- **網站：** https://beam.cloud
- **說明：** 開源的 GPU 沙盒平台，具備 checkpoint restore、持久工作佇列與 serverless 推理端點。可使用 gVisor 或 runc（可設定）作為隔離層。支援 GPU checkpoint/restore、Docker-in-Docker、儲存磁碟區、按秒計費。無限連線時間。提供 Python 與 JavaScript/TypeScript SDK[^beam]。
### 14. Superserve ⭐ 465

- **授權：** Apache 2.0
- **GitHub：** https://github.com/superserve-ai/superserve
- **網站：** https://www.superserve.ai
- **說明：** 以 Firecracker microVM 建構的持久安全沙盒，專為 AI 代理設計。低於 200ms 啟動、無限連線時間、版本化檔案系統支援快照與回滾、憑證代理人（API 金鑰不暴露給代理）、每個沙盒獨立出口規則。提供 TypeScript 與 Python SDK[^superserve]。
### 15. minimal ⭐ 141

- **授權：** Apache 2.0
- **GitHub：** https://github.com/gominimal/minimal
- **網站：** https://minimal.dev
- **說明：** 開源 CLI 工具，提供沙盒化的開發環境與 AI 程式碼代理。在 macOS 上使用 libkrun microVM，在 Linux 上使用 namespace 隔離。使用宣告式 `minimal.toml` 定義環境，可將連線綁定至 git worktree 上下文，內建 MCP server 整合與 SLSA Build L3 來源保證[^minimal]。
### 16. cs-sandbox ⭐ 4

- **授權：** 開源
- **團隊：** codesweep
- **GitHub：** https://github.com/codesweep-ai/sandbox
- **說明：** 一次性使用的隔離 Linux 開發沙盒，使用 rootless Podman containers 或 Firecracker microVM，專為執行 AI 程式碼代理設計。預先整合 Claude Code、Codex 與 OpenCode 代理[^cssandbox]。
## 其他特殊用途

### 17. bashkit4j ⭐ 4

- **授權：** MIT
- **GitHub：** https://github.com/tersePrompts/bashkit4j
- **說明：** 在 JVM 內執行的 bash 沙盒 ── 以 Rust 重新實作超過 160 個指令的 POSIX 風格 bash，於記憶體虛擬檔案系統中運作。不受信任的腳本不會產生任何 OS 行程或執行系統呼叫。具備記憶體內 VFS、選擇性 allowlisted host mounts、每次執行超時設定與取消功能。適合 Java 生態系的 AI 代理[^bashkit4j]。
### 18. h5i ⭐ 671

- **授權：** Apache 2.0
- **GitHub：** https://github.com/h5i-dev/h5i
- **說明：** 開源 CLI，可同時執行多個程式碼代理處理同一任務，每個代理各自擁有獨立的 sealed git-worktree 沙盒，防止檔案、分支與埠號衝突。具備沙盒間同儕審查機制與中立驗證者[^h5i]。
## 結論

Docker AI Sandboxes 最直接的 FOSS 替代方案為：

1. **OpenSandbox** ── 功能最全面，Kubernetes 原生，由阿里巴巴開發維護
2. **microsandbox** ── 簡潔的自建 microVM 平台，內建 MCP 支援
3. **E2B** ── Firecracker 基礎，已獲多個大型專案採用
4. **Daytona** ── 最低延遲（<90ms）且具備 GPU 支援

若只需要底層隔離機制自行搭建，則可從 Firecracker、gVisor、nsjail 或 bubblewrap 開始。

---

[^docker]: Docker Inc. (n.d.). Docker AI Sandboxes. Retrieved 2026-10-03, from https://docs.docker.com/ai/sandboxes/
[^firecracker]: Amazon. (n.d.). Firecracker MicroVM. Retrieved 2026-10-03, from https://github.com/firecracker-microvm/firecracker
[^gvisor]: Google. (n.d.). gVisor. Retrieved 2026-10-03, from https://github.com/google/gvisor
[^nsjail]: Google. (n.d.). nsjail. Retrieved 2026-10-03, from https://github.com/google/nsjail
[^bwrap]: containers/freedesktop. (n.d.). bubblewrap. Retrieved 2026-10-03, from https://github.com/containers/bubblewrap
[^sysbox]: Nestybox. (n.d.). Sysbox. Retrieved 2026-10-03, from https://github.com/nestybox/sysbox
[^kata]: Kata Containers Community. (n.d.). Kata Containers. Retrieved 2026-10-03, from https://github.com/kata-containers/kata-containers
[^opensandbox]: Alibaba. (n.d.). OpenSandbox. Retrieved 2026-10-03, from https://github.com/opensandbox-group/OpenSandbox
[^microsandbox]: Super Rad Company. (n.d.). microsandbox. Retrieved 2026-10-03, from https://github.com/superradcompany/microsandbox
[^e2b]: E2B. (n.d.). E2B. Retrieved 2026-10-03, from https://github.com/e2b-dev/E2B
[^daytona]: Daytona. (n.d.). Daytona. Retrieved 2026-10-03, from https://github.com/daytonaio/daytona
[^pandastack]: iam-the-ironman. (n.d.). PandaStack. Retrieved 2026-10-03, from https://github.com/iam-the-ironman/pandastack
[^smolvm]: Celesto AI. (n.d.). SmolVM. Retrieved 2026-10-03, from https://github.com/CelestoAI/SmolVM
[^beam]: Beam Cloud. (n.d.). beam/beta9. Retrieved 2026-10-03, from https://github.com/beam-cloud/beta9
[^superserve]: Superserve AI. (n.d.). superserve. Retrieved 2026-10-03, from https://github.com/superserve-ai/superserve
[^minimal]: minimal. (n.d.). minimal. Retrieved 2026-10-03, from https://github.com/gominimal/minimal
[^cssandbox]: codesweep. (n.d.). cs-sandbox. Retrieved 2026-10-03, from https://github.com/codesweep-ai/sandbox
[^bashkit4j]: tersePrompts. (n.d.). bashkit4j. Retrieved 2026-10-03, from https://github.com/tersePrompts/bashkit4j
[^h5i]: h5i. (n.d.). h5i. Retrieved 2026-10-03, from https://github.com/h5i-dev/h5i