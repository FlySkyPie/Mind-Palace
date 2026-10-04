# RackPeek — 以程式碼管理機房基礎設施的文件開源工具

## 概述

RackPeek 是一套專為**家庭實驗室 (Home Lab)** 與小型 IT 基礎設施設計的開源 CLI 與 Web UI 工具，由 Timmoth 與 Aptacode[^aptacode] 開發，託管於 GitHub[^github]，採用 **AGPL-3.0** 授權，當前版本 **2.1.0**，擁有約 1,800 顆星與 83 個分支。

其核心理念是將基礎設施文件化視為程式碼（**Infrastructure as Code — IaC**）——不依賴 GUI 繪圖後產出過時的 PNG，而是以結構化 YAML 定義所有硬體、系統與服務，讓它們可以放進 Git 進行版本控制、變更歷史追蹤與 Pull Request 審查。

## 設計哲學

RackPeek 的開發方針[^readme]包含：

- **簡潔性** — 範圍明確，不加入企業級 CMDB 功能或功能蔓延
- **易部署** — 安裝直接，日常使用摩擦低
- **開放性** — 採用開放 YAML 格式，使用者完全擁有自己的資料
- **隱私與安全** — 無遙測、無廣告、無追蹤
- **Dogfooding** — 功能必須解決維護者實際遇到的問題
- **Opinionated** — 針對家庭實驗室與自架環境最佳化，而非企業工作流程

## 技術架構

| 層級 | 技術 |
|---|---|
| 語言 | C# (.NET 10.0) |
| CLI 框架 | Spectre.Console.Cli |
| Web UI | Blazor Server + Blazor WebAssembly |
| 持久化 | YAML (YamlDotNet, DocMigrator.Yaml) |
| Git 整合 | LibGit2Sharp（可選） |
| CI/CD | GitHub Actions |
| 容器 | `mcr.microsoft.com/dotnet/aspnet:10.0`，連接埠 8080 |
| 測試 | xUnit + Playwright (E2E) + Testcontainers |

所有狀態儲存在單一 YAML 檔案 `config/config.yaml` 中，無需資料庫。同一個領域模型同時驅動 CLI 二進位檔 (`rpk`) 與 Blazor Server Web UI[^github]。

## 資料模型

YAML 結構如下[^datamodel]：

```yaml
resources:
  - kind: Server | Switch | Firewall | Router | Accesspoint | Desktop | Laptop | Ups | Other | System | Service
    name: <在該 kind 內唯一的名稱>
    tags: [...]
    labels: { key: value }
    notes: |     # 支援 Markdown
    runsOn: [<parent-resource-name>, ...]
    # kind-specific fields 如下
```

三種主要資源分類：

1. **硬體** — Server、Switch、Firewall、Router、Accesspoint、Desktop、Laptop、UPS、Other
2. **系統** — 運行於硬體之上的邏輯分層
3. **服務** — 運行於系統之上的應用程式或服務

每個資源的 `name` 是 `kind` 內的唯一識別（無數字 ID）。`runsOn` 欄位用以追蹤資源之間的依賴關係。

## 功能特色

### 庫存管理
可追蹤每項硬體的詳細規格：CPU、RAM、磁碟、GPU、NIC、網路埠口等。

### CLI 指令 (`rpk`)

| 指令群 | 用途 |
|---|---|
| `rpk servers / switches / routers / ...` | CRUD 操作及元件管理（cpu、drive、gpu、nic、port） |
| `rpk systems` | 管理系統與依賴樹 |
| `rpk services` | 管理服務與子網路列舉 |
| `rpk summary` | 全域資源總覽 |
| `rpk graph topology` | 輸出 Mermaid 實體拓樸圖 |
| `rpk graph logical` | 輸出 Mermaid 邏輯拓樸圖（依子網路與主機分組） |
| `rpk ansible inventory` | 產生 Ansible 庫存檔案 |
| `rpk ssh export` | 產生 SSH config 檔案 |
| `rpk hosts export` | 產生 `/etc/hosts` 相容檔案 |
| `rpk tags list / show` | 探索標籤及其使用次數 |
| `rpk connections add / remove` | 管理實體/邏輯埠口連線 |

### Web UI
Blazor Server 介面，提供 GUI 資料輸入、內建 Web CLI 模擬器、YAML 檔案檢視器及儀表板。

### 圖形視覺化
產生 Mermaid.js 流程圖，涵蓋實體拓樸（硬體 + 連線）與邏輯拓樸（依子網路與主機分組的服務/系統）。

### IaC 產生器
- Ansible inventory
- SSH config
- `/etc/hosts` 檔案
- 連線對映（實體與邏輯埠口）

### 標籤系統
每個資源支援 tags 與 labels，便於彈性分類與篩選。

## 安裝方式

### Docker（建議）

```bash
docker volume create rackpeek-config
docker run -d \
  --name rackpeek \
  -p 8080:8080 \
  -v rackpeek-config:/app/config \
  aptacode/rackpeek:latest
```

容器以 UID/GID **1654:1654** 執行，bind mount 的主機目錄需為此使用者可寫。SELinux 系統（Fedora、RHEL、CentOS）需加上 `:Z` 後綴。

### 獨立 CLI 二進位檔
可透過 `dotnet publish` 產出各平台（linux-x64、linux-arm64、osx-arm64）的單檔二進位檔。

## 社群的迴響與相關資源

RackPeek 在家庭實驗室社群中獲得不少關注：

- **DB Tech** 與 **Brandon Lee** 分別在 YouTube 上介紹[^yt-dbtech][^yt-brandon]
- **Brandon Lee** 也在部落格發表詳細文章[^article-brandon]
- **Jared Heinrichs** 撰寫了使用指南[^article-jared]
- 官方 Live Demo：<https://timmoth.github.io/RackPeek/>
- Docker Hub：`aptacode/rackpeek`[^dockerhub]
- Discord 社群：<https://discord.gg/egXRPdesee>

## 相關連結

- GitHub 倉庫：<https://github.com/timmoth/rackpeek>
- 官方文件（Commands）：<https://github.com/timmoth/rackpeek/blob/main/docs/Commands.md>

---

[^aptacode]: Aptacode. (n.d.). _Aptacode — Software Development_. Retrieved 2026-10-03, from https://aptacode.com/
[^github]: Timmoth. (2026). _RackPeek — Home Lab Infrastructure Documentation Tool_ (Version 2.1.0). GitHub. Retrieved 2026-10-03, from https://github.com/timmoth/rackpeek
[^readme]: Timmoth. (2026). _README — RackPeek_. GitHub. Retrieved 2026-10-03, from https://github.com/timmoth/rackpeek?tab=readme-ov-file#readme
[^datamodel]: Timmoth. (2026). _Commands — RackPeek Data Model_. GitHub. Retrieved 2026-10-03, from https://github.com/timmoth/rackpeek/blob/main/docs/Commands.md
[^yt-dbtech]: DB Tech. (2026). _Finally Document Your Home Lab the Easy Way_ [Video]. YouTube. Retrieved 2026-10-03, from https://www.youtube.com/watch?v=RJtMO8kIsqU
[^yt-brandon]: Brandon Lee. (2026). _I'm Documenting My Entire Home Lab as Code_ [Video]. YouTube. Retrieved 2026-10-03, from https://www.youtube.com/watch?v=wY1DgT3GD6U
[^article-brandon]: Lee, B. (2026, February). _I'm Documenting My Entire Home Lab as Code with RackPeek_. Virtualization Howto. Retrieved 2026-10-03, from https://www.virtualizationhowto.com/2026/02/im-documenting-my-entire-home-lab-as-code-with-rackpeek/
[^article-jared]: Heinrichs, J. (2026). _How to Document Your Entire Homelab_. Substack. Retrieved 2026-10-03, from https://jaredheinrichs.substack.com/p/how-to-document-your-entire-homelab
[^dockerhub]: Aptacode. (2026). _rackpeek_ [Docker Image]. Docker Hub. Retrieved 2026-10-03, from https://hub.docker.com/r/aptacode/rackpeek/