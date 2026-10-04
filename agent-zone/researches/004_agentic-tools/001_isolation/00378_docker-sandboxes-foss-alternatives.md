# Docker Sandboxes 之 FOSS 替代方案調查

## 概述

Docker Sandboxes[^docker-sbx] 是 Docker 官方推出的 AI 程式碼代理（AI coding agent）沙箱環境方案，透過 CLI 工具 `sbx` 提供隔離的 microVM 環境來安全執行 AI 代理。本報告調查其 FOSS（自由及開源軟體）替代方案，聚焦於「開箱即用」的完整工具，而非底層程式庫。

---

## 頂尖替代方案（最接近 Docker Sandboxes / `sbx`）

### 1. OpenSandbox[^opensandbox]（Apache 2.0）

阿里巴巴推出的通用沙箱平台，主打 AI 代理，內建 CLI、六種語言 SDK 與 MCP 整合。

- **授權**：Apache 2.0
- **隔離機制**：Docker（本機）＋ Firecracker microVM（生產/Kubernetes）
- **特色**：憑證保險庫、網路出口控制、暫停/恢復（狀態保留），Firecracker 冷啟動 97ms P50
- **SDK**：Python、Java/Kotlin、TypeScript、C#/.NET、Go、CLI `osb`
- **GitHub 星數**：~15,700 ★

### 2. CubeSandbox[^cubesandbox]（Apache 2.0）

騰訊雲推出的硬體隔離 microVM 沙箱，冷啟動 <60ms，記憶體開銷 <5MB，為效能最優者。

- **授權**：Apache 2.0 — **完全開源**
- **隔離機制**：自訂 RustVMM ＋ KVM microVM（專屬核心）
- **特色**：E2B SDK 相容（更換環境變數即可切換）、CubeCoW 快照引擎（<100ms 檢查點）、eBPF 網路隔離、Web UI 主控台、快照/複製/回滾
- **冷啟動**：<60ms（同級最快）
- **單節點密度**：2,000+ 沙箱
- **GitHub 星數**：~12,800 ★

### 3. Eclipse Enclave[^enclave]（MIT）

一條指令即可啟動隔離的 AI 代理工作階段，但為專案初期階段（2026 年 7 月才建立），社群規模尚小。

- **授權**：MIT
- **隔離機制**：Docker 容器（預設）＋ 實驗性 QEMU microVM
- **支援代理**：Claude Code、Codex CLI、OpenCode、Pi、Mistral Vibe、Theia AI
- **特色**：每工作階段獨立檔案系統掛載、DNS 白名單閘道、憑證保險庫（金鑰永不進入容器）、並行工作階段
- **使用方式**：`enclave` 即啟動
- **平台**：Linux、macOS、Windows（WSL2）
- **GitHub 星數**：~60 ★

### 4. Nono[^nono]（Apache 2.0）

零延遲、零設定的核心層級沙箱——無容器、無 VM、無背景服務。

- **授權**：Apache 2.0
- **隔離機制**：Linux Landlock / macOS Seatbelt（作業系統核心原始語法）
- **特色**：每個工具個別沙箱化（per-tool sandbox），無需 Docker 或 VM
- **使用方式**：`nono run --profile nolabs-ai/opencode -- opencode`
- **平台**：Linux、macOS、Windows（WSL2）
- **GitHub 星數**：~4,300 ★

### 5. AIO Sandbox[^aiosbx]（Apache 2.0）

單一 Docker 容器整合瀏覽器（VNC）、Shell 終端機、檔案操作、VSCode Server、Jupyter 與 MCP 伺服器的全合一沙箱。

- **授權**：Apache 2.0
- **隔離機制**：Docker 單容器
- **使用方式**：`docker run` 即可，瀏覽器存取 `http://localhost:8080`
- **GitHub 星數**：~6,100 ★

### 6. Agents Sandbox（agbox）[^agbox]（Apache 2.0）

專為 Claude Code、Codex、OpenClaw 設計的本機沙箱 CLI，自動掛載專案程式碼、預設代理執行環境，用完即銷毀。

- **授權**：Apache 2.0
- **隔離機制**：Docker 容器
- **使用方式**：`agbox claude` 或 `agbox codex`

---

## 其他值得注意的方案

### Daytona[^daytona]（AGPL-3.0）

開源開發環境管理器，支援持久化工作區與 GPU（H100、RTX PRO 6000）。冷啟動 <90ms，適合長時間運行的 AI 代理工作階段。

### Microsandbox[^microsandbox]（Apache 2.0）

本機優先的 libkrun/KVM microVM，無需雲端。冷啟動 ~200ms，跨平台（Linux、macOS、Windows），rootless 執行。

### OpenComputer[^opencomputer]（開源）

diggerhq 開發的完整 KVM VM，支援即時 CPU/RAM 動態調整（雙向）、持久化儲存、休眠、可複製快照。無工作階段時間限制。

### PandaStack[^pandastack]（Apache 2.0）

Firecracker microVM 開源平台，支援快照恢復與 CoW 分支（MAP_PRIVATE + XFS reflink，400-750ms），適合平行代理分支。

### sandbox-cli[^sandboxcli]（開源）

透用 Docker 容器包裝 Claude Code、Codex、Gemini 等 12+ 種 AI 代理的 CLI 工具，僅掛載專案目錄。

### gVisor / runsc[^gvisor]（Apache 2.0）

Google 開發的使用者空間核心，可作為 Docker OCI runtime 直接使用（`--runtime=runsc`）。隔離強度介於容器與 microVM 之間。

---

## 對比總表

| 工具 | 隔離機制 | 冷啟動時間 | CLI | Web UI | E2B 相容 | 授權 |
|------|---------|-----------|-----|--------|---------|------|
| **Eclipse Enclave** (新) | Docker / QEMU microVM | ~秒 | ✅ | ✅（開發中） | ❌ | MIT |
| **Nono** | Landlock/Seatbelt | 即時 | ✅ | ❌ | ❌ | Apache 2.0 |
| **OpenSandbox** | Docker / Firecracker | 97ms | ✅ | ❌ | ✅ | Apache 2.0 |
| **CubeSandbox** | RustVMM + KVM | <60ms | ❌ | ✅ | ✅ | Apache 2.0 |
| **AIO Sandbox** | Docker 整合容器 | ~秒 | ❌ | ✅（瀏覽器） | 部分(MCP) | Apache 2.0 |
| **Daytona** | Docker / Kata/Sysbox | <90ms | ✅ | ✅ | ❌ | AGPL-3.0 |
| **Microsandbox** | libkrun/KVM | ~200ms | ✅ | ❌ | ❌ | Apache 2.0 |
| **OpenComputer** | QEMU/KVM | ~秒 | ✅ | ✅ | ❌ | 開源 |
| **PandaStack** | Firecracker | 179ms | ✅ | ✅ | ❌ | Apache 2.0 |
| **sandbox-cli** | Docker | ~秒 | ✅ | ❌ | ❌ | 開源 |
| **gVisor/runsc** | 使用者空間核心 | <1秒 | ✅ | ❌ | ❌ | Apache 2.0 |

---

## 選用建議

- **社群與生態系最成熟** → **OpenSandbox**：Apache 2.0，~15,700 ★，Python/TS/Java/C#/Go SDK，K8s 原生支援。
- **效能與隔離最強** → **CubeSandbox**：<60ms 冷啟動、<5MB 開銷、E2B 相容，硬體級 microVM 隔離。
- **最接近 Docker Sandboxes / `sbx` 概念** → **Eclipse Enclave**：MIT 授權，一條 `enclave` 指令即可用，但專案尚新（2026-07），僅 ~60 ★，成熟度待觀察。
- **無 Docker 依賴、零設定** → **Nono**：`brew install nono` 即可使用，無需容器或 VM。
- **全合一 AI 代理工作站** → **AIO Sandbox**：瀏覽器 + VSCode + Shell + Jupyter + MCP 一體化。
- **持久化長期代理工作區** → **Daytona**：GPU 支援，暫停/封存生命週期管理。
- **Firecracker 平台自託管** → **PandaStack**：快照恢復與 CoW 分支。

---

## 資料來源

[^docker-sbx]: Docker Inc. (n.d.). Docker Sandboxes. Retrieved 2026-10-03, from https://docs.docker.com/ai/sandboxes/
[^enclave]: Eclipse Foundation. (2026). Eclipse Enclave. Retrieved 2026-10-03, from https://github.com/eclipse-enclave/enclave
[^nono]: Nolabs AI. (2026). Nono — Zero-latency kernel-level sandbox for AI agents. Retrieved 2026-10-03, from https://github.com/nolabs-ai/nono
[^opensandbox]: OpenSandbox Group. (2026). OpenSandbox. Retrieved 2026-10-03, from https://github.com/opensandbox-group/OpenSandbox
[^cubesandbox]: Tencent Cloud. (2026). CubeSandbox — Hardware-isolated microVM sandbox. Retrieved 2026-10-03, from https://github.com/TencentCloud/CubeSandbox
[^aiosbx]: Agent Infra. (2026). AIO Sandbox. Retrieved 2026-10-03, from https://github.com/agent-infra/sandbox
[^agbox]: 1996fanrui. (2026). Agents Sandbox (agbox). Retrieved 2026-10-03, from https://github.com/1996fanrui/agents-sandbox
[^daytona]: Daytona. (2026). Daytona — Open-source dev environment manager. Retrieved 2026-10-03, from https://github.com/daytonaio/daytona
[^microsandbox]: Microsandbox. (2026). Microsandbox — Local-first microVM. Retrieved 2026-10-03, from https://github.com/microsandbox/microsandbox
[^opencomputer]: Diggerhq. (2026). OpenComputer. Retrieved 2026-10-03, from https://github.com/diggerhq/opencomputer
[^pandastack]: PandaStack. (2026). PandaStack. Retrieved 2026-10-03, from https://www.pandastack.ai/blog/best-open-source-code-sandboxes/
[^sandboxcli]: sandbox-cli. (2026). Retrieved 2026-10-03, from https://sandbox-cli.vercel.app/
[^gvisor]: Google. (n.d.). gVisor — Application kernel for containers. Retrieved 2026-10-03, from https://github.com/google/gvisor