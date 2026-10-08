# Bugzilla 為何沒有官方 Docker 映像

## 概要

Bugzilla 並**沒有官方生產環境用的 Docker 映像**。目前存在的只有：(1) Bugzilla 5.2+ 內附的**僅供評估/示範用** Docker Compose 設定檔；(2) Docker Hub 上的基礎映像（只包含 Perl 相依套件，不含 Bugzilla 本體）；以及 (3) 多個已棄用或封存的社群 Docker 映像。Bugzilla 團隊已明確表示**尚未產出可用於生產環境的 Docker container**。

## 官方說法

### 文件中的明確聲明

Bugzilla 官方文件（Harmony 文件，目前仍有效）明確指出：

> **「At this time the Bugzilla team has not produced a production Docker container for running the server in production. These instructions cover running Bugzilla in a docker container for evaluation.」**[^harmony-docker]

這個聲明自 Harmony（5.9.x）文件開始便存在，至今仍是官方立場。

### Bugzilla 5.2 發行備註

> **「Bugzilla now ships with a Docker Compose configuration which provides an out-of-the-box Bugzilla with a default configuration to test with. This configuration is not suitable for production use.」**[^release-52]

此處明確稱之為「Demo Docker Configuration」，並強調**不適合生產環境**。

### GitHub README

Bugzilla 5.2 分支的 README 指示：

> **「If you have Docker installed and just want to take a quick look around Bugzilla you can cd into the bugzilla directory and type `docker compose up`.」**[^github-readme]

同樣定位為快速評估工具，而非正式部署方式。

## Docker Hub 上的官方映像現狀

Bugzilla 在 Docker Hub 上的組織（[hub.docker.com/u/bugzilla/](https://hub.docker.com/u/bugzilla/)）[^docker-hub-org] 共有 17 個倉儲，**絕大多數已標示為「DEPRECATED - DO NOT USE」**[^docker-hub-dev]：

| 倉儲 | 狀態 |
|---|---|
| `bugzilla/bugzilla-dev` | DEPRECATED - DO NOT USE |
| `bugzilla/bugzilla-ci` | DEPRECATED - DO NOT USE |
| `bugzilla/bugzilla-base` | DEPRECATED - DO NOT USE |
| `bugzilla/harmony` | DEPRECATED - DO NOT USE |
| `bugzilla/harmony-slim` | DEPRECATED - DO NOT USE |
| `bugzilla/caddy` | DEPRECATED - DO NOT USE |

**唯一目前活躍的**是基礎相依性映像（`bugzilla-perl-slim*`、`bugzilla-perl-full*`）[^docker-hub-slim]，這些是**入門用的起點映像**——它們**不含 Bugzilla 程式碼**，僅預先安裝 Perl 模組。

## 社群嘗試

多個社群 Docker 映像存在，但無一被官方採納[^docker-bugzilla-original][^docker-bugzilla-itspoma][^docker-bugzilla-gameldar][^docker-bugzilla-eckford]：

| 倉儲 | 說明 | 狀態 |
|---|---|---|
| `dklawren/docker-bugzilla` | 最早的社群 Docker 努力，CentOS + Apache2 + MySQL 5.6 | ~2015 年後未更新 |
| `itspoma/docker-bugzilla` | 標榜「production-ready」 | 僅 8 次提交，未維護 |
| `gameldar/bugzilla` | Bugzilla 5.0 + PostgreSQL | 未維護 |
| `Eckford-Solutions/bugzilla52-container` | Bugzilla 5.2 + AlmaLinux 8 + 所有相依套件 | 2025 年有更新 |

## 技術挑戰

### a) 複雜的 Perl 相依套件樹

Bugzilla 有大量的 Perl 相依套件。5.2 分支的 Dockerfile 安裝了 **50 個以上的系統套件**用來滿足 Perl 模組需求，再從 CPAN 編譯額外模組[^dockerfile]：

```dockerfile
RUN apt-get -y install \
    apache2 graphviz libapache2-mod-perl2 libapache2-mod-perl2-dev \
    libappconfig-perl libauthen-radius-perl libauthen-sasl-perl \
    libcache-memcached-perl libcgi-pm-perl libchart-perl \
    libdaemon-generic-perl libdate-calc-perl libdatetime-perl \
    libdatetime-timezone-perl libdbi-perl libdbix-connector-perl \
    ...（50+ 套件）
```

以及後續的 CPAN 安裝：

```dockerfile
RUN cpan install Template::Toolkit DBD::MariaDB PatchReader
```

### b) 多種資料庫驅動

Bugzilla 支援 MySQL、MariaDB、PostgreSQL、Oracle 與 SQLite。生產映像需要處理所有這些驅動程式，或為每種資料庫分別建立映像——這也是他們已在基礎映像中部分採取的方式（`bugzilla-perl-slim-mysql`、`bugzilla-perl-slim-pg`、`bugzilla-perl-slim-mariadb` 等）。

### c) Harmony（5.9.x / 6.0）的架構變革

Harmony 版本大幅改變了相依性管理方式——改用 `carton` 將模組安裝為本地 Perl 模組，而非系統套件[^harmony-quickstart]。這項主要轉型使得在此時期產出穩定的生產 Docker 映像變得複雜。

### d) 志工專案的資源限制

Bugzilla 由小型志工團隊維護。為多種資料庫後端建立並維護具生產品質的 Docker 映像、處理版本發佈、安全性修補，以及保持基礎映像更新，需要大量的持續投入，而團隊並未承諾此項工作。

## Bugzilla 本身的追蹤狀態

Bugzilla 的 Bug 追蹤系統（bugzilla.mozilla.org）內，Bugzilla 產品下已設有 **「Docker」** 正式元件[^docker-component]，表示 Docker 相關議題被視為一級關注事項。然而，其範圍僅限於示範設定檔與開發用映像，而非正式部署。

關鍵的相關 Bug [#1888068](https://bugzilla.mozilla.org/show_bug.cgi?id=1888068) 明確將 Docker Compose 設定定位為示範用途[^bug-1888068]：

> **「Need a working Docker config that runs out-of-the-box to demo Bugzilla」**

## 原因總覽

1. **政策明確**：Bugzilla 團隊選擇不產出生產用 Docker container，官方文件已明述。
2. **Perl 相依套件複雜**：Bugzilla 需要龐大且持續演變的 Perl 模組，維護穩定映像耗費資源。
3. **多資料庫後端**：支援五種資料庫導致映像生態碎片化。
4. **志工資源限制**：團隊未將生產 Docker 映像維護列為優先事項。
5. **架構轉型期**：Harmony/6.0 重寫改變相依管理方式（系統套件 → carton），此時非穩定化生產映像的良好時機。
6. **歷史不穩定**：過去多個 Docker 映像已被棄用，顯示團隊難以持續維護。
7. **僅供示範**：既有 Docker 支援定位為評估/測試。

## 參考資料

[^harmony-docker]: Bugzilla Harmony Documentation. (n.d.). Evaluating with Docker. Retrieved 2026-10-03, from https://bugzilla.readthedocs.io/projects/harmony/en/latest/installing/docker.html
[^release-52]: Bugzilla Project. (2023). Bugzilla 5.2 Release Notes. Retrieved 2026-10-03, from https://www.bugzilla.org/releases/5.2/
[^github-readme]: Bugzilla Project. (n.d.). Bugzilla README (branch 5.2). Retrieved 2026-10-03, from https://github.com/bugzilla/bugzilla
[^docker-hub-org]: Bugzilla Project. (n.d.). Bugzilla Docker Hub Organization. Retrieved 2026-10-03, from https://hub.docker.com/u/bugzilla/
[^docker-hub-dev]: Bugzilla Project. (n.d.). bugzilla/bugzilla-dev on Docker Hub. Retrieved 2026-10-03, from https://hub.docker.com/r/bugzilla/bugzilla-dev/
[^docker-hub-slim]: Bugzilla Project. (n.d.). bugzilla/bugzilla-perl-slim on Docker Hub. Retrieved 2026-10-03, from https://hub.docker.com/r/bugzilla/bugzilla-perl-slim
[^docker-bugzilla-original]: Miller, D. (2015). dklawren/docker-bugzilla. Retrieved 2026-10-03, from https://github.com/dklawren/docker-bugzilla
[^docker-bugzilla-itspoma]: itspoma. (n.d.). itspoma/docker-bugzilla. Retrieved 2026-10-03, from https://github.com/itspoma/docker-bugzilla
[^docker-bugzilla-gameldar]: gameldar. (n.d.). gameldar/bugzilla. Retrieved 2026-10-03, from https://github.com/gameldar/bugzilla
[^docker-bugzilla-eckford]: Eckford Solutions. (2025). Eckford-Solutions/bugzilla52-container. Retrieved 2026-10-03, from https://github.com/Eckford-Solutions/bugzilla52-container
[^dockerfile]: Bugzilla Project. (n.d.). Bugzilla Dockerfile (branch 5.2). Retrieved 2026-10-03, from https://github.com/bugzilla/bugzilla/blob/5.2/Dockerfile
[^harmony-quickstart]: Bugzilla Harmony Documentation. (n.d.). Quick Start. Retrieved 2026-10-03, from https://bugzilla.readthedocs.io/projects/harmony/en/latest/installing/quick-start.html
[^docker-component]: Bugzilla Project. (n.d.). Bugzilla Docker Component — bugzilla.mozilla.org. Retrieved 2026-10-03, from https://bugzilla.mozilla.org/describecomponents.cgi?product=Bugzilla&component=Docker
[^bug-1888068]: Miller, D. (2024). Bug 1888068 — Need a working Docker config that runs out-of-the-box to demo Bugzilla. Retrieved 2026-10-03, from https://bugzilla.mozilla.org/show_bug.cgi?id=1888068
[^bmo-devbox]: Mozilla Wiki. (n.d.). BMO/DeveloperBox. Retrieved 2026-10-03, from https://wiki.mozilla.org/BMO/DeveloperBox
[^docs]: Bugzilla Project. (n.d.). Bugzilla Documentation. Retrieved 2026-10-03, from https://www.bugzilla.org/docs/