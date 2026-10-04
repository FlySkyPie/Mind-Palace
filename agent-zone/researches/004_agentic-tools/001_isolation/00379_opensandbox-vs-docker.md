# OpenSandbox 與 Docker 的差異分析：為何 AI Agent 需要專用沙箱

## 摘要

OpenSandbox 是一套為 AI Agent 工作負載設計的開源沙箱執行平臺，提供從 Docker（本機開發）到 Kubernetes / Firecracker 微虛擬機器（生產環境）的統一 API。本文比較 OpenSandbox 與 Docker 在隔離模型、效能、安全預設值、Agent 原生功能及營運複雜度上的關鍵差異，並解釋為何純 Docker 容器不足以勝任 AI Agent 沙箱的角色。

---

## 1. 核心問題：威脅模型的錯配

Docker 容器是為封裝與部署**可信賴的應用程式**而設計的。容器與宿主機共用核心（shared kernel）——所有系統呼叫都直接通往宿主機核心，命名空間（namespaces）與 cgroups 僅提供程序層級的隔離[^sitepoint]。

然而 AI Agent（如 Claude Code、Cursor、Codex CLI 等）的行為是**不可預測且來自不可信來源**的——它們執行的程式碼來自 LLM 生成，可能因 prompt injection 而執行惡意指令[^ralphloop]。給予 Agent 一個 shell「在功能上等同於把 root 權限交給不可信的第三方」[^sitepoint]。

Docker 的預設安全姿態是**寬鬆的（permissive）**——開箱即用的容器擁有完整的網路存取權與大量系統權限，必須手動關閉（`--cap-drop=ALL`、`--security-opt=no-new-privileges`、`--network=none`、自訂 seccomp profile）。OpenSandbox 的預設姿態則是**限制性的（restrictive-by-default）**——最小 syscall allowlist、拒絕所有網路連線、唯讀根目錄、暫存空間為 ephemeral，使用者需要**選擇性開放**功能而非關閉它們[^architecture]。

---

## 2. 隔離模型與沙箱技術

OpenSandbox 支援**多種沙箱後端**，可在伺服器層級設定，使用者端零程式碼修改[^architecture]：

| 後端 | 隔離機制 | 啟動開銷 | 記憶體開銷 | 安全性層級 |
|---|---|---|---|---|
| **runc**（預設） | 容器（cgroups + namespaces） | ~0ms | 極低 | 共享核心，低 |
| **gVisor (runsc)** | 使用者空間核心（User-space kernel） | ~10-50ms | ~50MB | 系統呼叫層級，中 |
| **Kata (Firecracker)** | 微虛擬機器（microVM），獨立客體核心 | ~125ms | ~5MB | 硬體虛擬化，高 |
| **Kata (QEMU)** | 完整虛擬機器 | ~500ms | ~20-50MB | 硬體虛擬化，最高 |

**Docker**：僅支援 runc 容器模式，無內建切換選項[^docker]。

Firecracker 微虛擬機器路徑（FastSandbox）是 OpenSandbox 的核心技術——每個沙箱執行在獨立的 KVM 虛擬機器（dedicated Linux kernel）中，即使一個沙箱的核心被攻陷，也不會影響宿主機或其他沙箱[^firecracker]。

---

## 3. 冷啟動延遲與效能

| 指標 | Docker（已硬化） | OpenSandbox FastSandbox (Firecracker) |
|---|---|---|
| 冷啟動延遲 | ~500ms–2s（需預先拉取映像檔） | **P50 97ms（序列）/ P99 308ms（10 並發）**[^performance] |
| 系統呼叫開銷 | 無（直接核心） | 無（微虛擬機器，獨立核心） |
| 資源密度 | ~100-200 容器/宿主機 | 單宿主機每秒可建立 150 個微虛擬機器，每個約 5MB |

Docker 的冷啟動包括通過 dockerd 的容器建立流程與 seccomp profile 載入。OpenSandbox 的 FastSandbox 路徑透過**樣板快照還原**（template-backed snapshot restore）而非從零開機核心來達到次 100ms 的建立時間[^performance]。

---

## 4. Agent 原生功能對照表

這是「為何不直接用 Docker」的關鍵答案——Docker 缺少以下所有專為 Agent 設計的功能[^readme][^bridgers]：

| 功能 | Docker | OpenSandbox |
|---|---|---|
| **Session 生命週期管理** | DIY（手動 volume、命名、清理） | 原生 SDK API：`Sandbox.create()` → `commands.run()` → `sandbox.destroy()` |
| **In-sandbox 執行程序** | 裸 shell 存取 | **execd**——Go 精靈行程，提供 SSE 串流、檔案操作、PTY WebSocket、Jupyter 程式碼執行、資源指標 |
| **多語言 SDK** | 僅 dockerode / docker CLI | Python、Java/Kotlin、TypeScript/JS、C#/.NET、Go——同一 API |
| **網路出口政策** | `--network=none` 或全開放 | 每個沙箱獨立出口規則：FQDN allow/deny、DNS 過濾、nftables 強制執行[^egress] |
| **憑證倉庫（Credential Vault）** | 無 | TLS MITM 代理，在出站 HTTPS 請求中注入 API 金鑰，**憑證永不進入沙箱工作負載**[^vault] |
| **暫停/恢復（Pause/Resume）** | `docker pause` 僅停止排程 | 完整 checkpoint/restore，保留記憶體與磁碟狀態[^fastsandbox] |
| **程式碼直譯器（Code Interpreter）** | 需手動設定 Jupyter | 內建 Jupyter 支援，有狀態 session |
| **瀏覽器/桌面自動化** | 需手動設定 VNC | 內建 Chrome、Playwright、VNC 桌面沙箱 |
| **MCP 整合** | 需自建 MCP server | 原生 MCP server，直接整合 Claude Code、Cursor 等 |
| **批次建立** | 無 | BatchSandbox CRD，支援 RL 訓練與 Agent 評測工作負載 |

---

## 5. 本機到叢集的可遷移性

OpenSandbox 的核心設計原則之一是 **「start local, deploy on Kubernetes」** ——開發者在筆電上使用 Docker 後端撰寫的 Python SDK 程式碼，部署到生產環境時只需切換伺服器設定為 FastSandbox/Kubernetes 後端，**不需修改任何應用程式碼**[^readme]。

```python
# 這段程式碼在 Docker 後端與 Firecracker 後端執行結果完全一致
sandbox = await Sandbox.create("alpine")
execution = await sandbox.commands.run("echo 'Hello OpenSandbox!'")
await sandbox.destroy()
```

Docker Compose 與 Kubernetes 之間則需要撰寫完全不同的 YAML。

---

## 6. 營運複雜度比較

**使用 Docker 建構 AI Agent 沙箱平台：**
- 每個安全強化措施都是手工的（seccomp profile、capability drop、網路隔離）
- 無內建清理機制——孤兒容器不斷累積
- 無 session 或 agent 感知的資源池概念
- 多輪對話需要手動管理 Volume 與狀態
- 等同於在 Docker 之上**重建一個沙箱平台**

**使用 OpenSandbox：**
- 單一 `~/.sandbox.toml` 設定檔控制所有安全設定
- 伺服器啟動時驗證執行時期可用性，設定錯誤則拒絕啟動[^architecture]
- Agent 導向的生命週期：建立 session → 執行 → 銷毀（支援 `try/finally` 清理）
- 可觀測的沙箱狀態：`state`、`reason`、`message`、時間戳

---

## 7. 何時使用什麼

| 情境 | Docker | OpenSandbox |
|---|---|---|
| 可信賴、人工審查的程式碼 | ✅ 最佳選擇 | 過度設計 |
| 內部開發工具、小型團隊 | ✅ 足夠使用 | 加分項（SDK 便利性） |
| 多租戶 Agent 平台 | ❌ 共享核心風險 | ✅ Firecracker/Kata 硬體隔離 |
| 生產環境 Agent 含程式碼執行 | ❌ 需手動強化 | ✅ 專用設計、審計日誌、政策控制 |
| Agent 評測不可信程式碼 | ❌ 隔離不足 | ✅ gVisor 或 Firecracker |
| 大規模 RL 訓練 | ❌ 無批次編排 | ✅ BatchSandbox + Pool CRD |
| 企業合規需求 | ❌ 無內建審計/憑證管理 | ✅ Credential Vault、出口政策、安全執行時期 |
| 大型 Agent 叢集（1000+） | ❌ 密度受限於核心信任 | ✅ FastSandbox（150 VMs/sec, 5MB each） |

---

## 結論

Docker 是包裝與部署**可信軟體**的工具，而 AI Agent 執行的程式碼本質上**不可信**。OpenSandbox 並非 Docker 的替代品——它是專為 AI Agent 工作負載設計的沙箱平臺，在隔離層級（微虛擬機器）、冷啟動速度（<100ms）、安全預設值（restrictive-by-default）、Agent 原生功能（execd、Credential Vault、MCP）以及本機到叢集的 API 一致性上，解決了 Docker 無法直接回答的問題。

如 ralphloop.sh 所述：「容器是給你信任的軟體用的，微虛擬機器是給你明確不信任的軟體用的。以 bypass-permissions 模式運作的自主 Agent，恰恰就是後者。」[^ralphloop]

---

## 參考資料

[^sitepoint]: SitePoint. (2026). AI Agent Sandboxing Guide: Docker vs Dedicated Sandboxes. Retrieved 2026-10-03, from https://www.sitepoint.com/ai-agent-sandboxing-guide/

[^ralphloop]: ralphloop.sh. (2026). Docker Sandboxes vs Containers for AI Agents. Retrieved 2026-10-03, from https://ralphloop.sh/blog/docker-sandboxes-vs-containers-for-agents/

[^architecture]: OpenSandbox Group. (2026). OpenSandbox Architecture — Control Plane, Data Plane, FastSandbox. Retrieved 2026-10-03, from https://open-sandbox.ai/architecture/

[^readme]: OpenSandbox Group. (2026). OpenSandbox — Secure, Fast, and Extensible Sandbox Runtime for AI Agents (GitHub Repository). Retrieved 2026-10-03, from https://github.com/opensandbox-group/OpenSandbox

[^performance]: OpenSandbox Group. (2026). FastSandbox Performance — P50 97ms Serial / P99 308ms at 10 Concurrent. Retrieved 2026-10-03, from https://github.com/opensandbox-group/OpenSandbox/blob/main/docs/architecture/fast-sandbox/performance.md

[^firecracker]: OpenSandbox Group. (2026). Firecracker Sandbox Architecture. Retrieved 2026-10-03, from https://open-sandbox.ai/architecture/fast-sandbox/firecracker

[^egress]: OpenSandbox Group. (2026). Egress Policy — Per-Sandbox Outbound Network Controls. Retrieved 2026-10-03, from https://open-sandbox.ai/architecture/fast-sandbox/egress

[^vault]: OpenSandbox Group. (2026). Credential Vault — TLS MITM Credential Injection. Retrieved 2026-10-03, from https://open-sandbox.ai/guides/credential-vault

[^docker]: Docker Inc. (2026). Docker Container Security — Seccomp, Capabilities, Namespaces. Retrieved 2026-10-03, from https://www.docker.com/blog/comparing-sandboxing-approaches-ai-agents/

[^bridgers]: Bridgers Agency. (2026). OpenSandbox AI Agents Guide — Comparison with E2B, Modal, Daytona. Retrieved 2026-10-03, from https://bridgers.agency/en/blog/opensandbox-ai-agents-guide

[^fastsandbox]: OpenSandbox Group. (2026). FastSandbox — Template-backed MicroVM Architecture. Retrieved 2026-10-03, from https://github.com/opensandbox-group/OpenSandbox/blob/main/docs/architecture/fast-sandbox/index.md