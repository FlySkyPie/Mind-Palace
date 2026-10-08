# Bugzilla 官方 Docker 映像檔調查

## 摘要

Bugzilla 專案並未提供可直接用於生產環境的官方 Docker 映像檔，但提供了兩類 Docker 相關資源：(1) 用於未來版本（5.9.x / 6.0）的 Perl 依賴基礎映像檔（base images），以及 (2) 在原始碼儲存庫中附帶的 Dockerfile 與 docker-compose.yml，可用於建置執行容器化的 Bugzilla 實例。官方立場明確表示這些資源僅供評估與測試之用。

## 資源概覽

### 基礎映像檔（Docker Hub, `bugzilla` 組織）

Bugzilla 在 Docker Hub 上維護一系列基礎映像檔，這些映像檔預先載入 Bugzilla 所需的 Perl 模組，但**不包含 Bugzilla 軟體本身**。使用者需自行將 Bugzilla 原始碼複製入內。[^docker-hub-bugzilla]

| 映像檔名稱 | 說明 | 狀態 |
|---|---|---|
| `bugzilla/bugzilla-perl-slim` | 僅必要模組（不含 DB 驅動） | 活躍（約 2 月前更新） |
| `bugzilla/bugzilla-perl-slim-mysql` | slim + MySQL 8 驅動 | 活躍 |
| `bugzilla/bugzilla-perl-slim-pg` | slim + PostgreSQL 驅動 | 活躍 |
| `bugzilla/bugzilla-perl-slim-mariadb` | slim + MariaDB 驅動 | 活躍 |
| `bugzilla/bugzilla-perl-full` | 所有必要與可選模組 | 活躍 |
| `bugzilla/bugzilla-perl-full-mysql` | full + MySQL 8 驅動 | 活躍 |
| `bugzilla/bugzilla-perl-full-pg` | full + PostgreSQL 驅動 | 活躍 |
| `bugzilla/bugzilla-perl-full-mariadb` | full + MariaDB 驅動 | 活躍 |

已棄用（DEPRECATED）映像檔：`harmony`、`harmony-slim`、`caddy`、`bugzilla-dev`、`bugzilla-ci`、`bugzilla-base` 以及 `bugzilla-perl-slim-mysql8`、`bugzilla-perl-slim-pg9`。

### 官方原始碼儲存庫中的 Docker 支援（5.2 分支）

從原始碼建置完整 Bugzilla 容器的方法位於官方 GitHub 儲存庫的 `5.2` 分支。[^bugzilla-github]

- **Dockerfile** — 77 行，基於 `ubuntu:24.04`，安裝 Apache2、所有 Perl 相依套件，並複製 Bugzilla 原始碼。[^dockerfile]
- **docker-compose.yml** — 定義兩個服務：`bugzilla5.web`（Apache + Bugzilla）與 `bugzilla5.db`（MariaDB），並使用命名磁碟區儲存資料。[^docker-compose]
- **docker/** 目錄 — 包含啟動腳本、Apache 設定、MySQL 設定及安裝回應檔案。[^docker-dir]

使用方式（摘錄自官方 README）：

> If you have Docker installed and just want to take a quick look around Bugzilla you can cd into the bugzilla directory and type `docker compose up`.

## 官方立場與注意事項

Bugzilla 官方文件明確指出：[^harmony-docker-docs]

> At this time the Bugzilla team has not produced a production Docker container for running the server in production. These instructions cover running Bugzilla in a docker container for evaluation.

亦即 Bugzilla 團隊目前**不建議**將 Docker 容器用於生產部署，容器化僅供評估測試。

## 總結

| 面向 | 狀態 |
|---|---|
| 有無官方直接可用的生產級 Docker 映像檔 | **無** |
| 有無官方基礎映像檔（需自行放入 Bugzilla 原始碼） | **有**（8 個活躍的 Perl 依賴基礎映像檔） |
| 有無官方 Dockerfile 與 docker-compose 範例 | **有**（5.2 分支原始碼內附） |
| 官方對生產部署的建議 | 建議傳統安裝方式，容器僅供評估 |
| 未來 6.0 版本的容器支援進展 | 基礎映像檔持續更新中，但尚未正式發布 |

## 參考文獻

[^docker-hub-bugzilla]: Bugzilla Project. (n.d.). Docker Hub Organization: bugzilla. Retrieved 2026-10-03, from https://hub.docker.com/u/bugzilla/
[^bugzilla-github]: Bugzilla Project. (n.d.). bugzilla/bugzilla (5.2 branch). GitHub. Retrieved 2026-10-03, from https://github.com/bugzilla/bugzilla/tree/5.2
[^dockerfile]: Bugzilla Project. (n.d.). Dockerfile (5.2 branch). GitHub. Retrieved 2026-10-03, from https://github.com/bugzilla/bugzilla/blob/5.2/Dockerfile
[^docker-compose]: Bugzilla Project. (n.d.). docker-compose.yml (5.2 branch). GitHub. Retrieved 2026-10-03, from https://github.com/bugzilla/bugzilla/blob/5.2/docker-compose.yml
[^docker-dir]: Bugzilla Project. (n.d.). docker/ directory (5.2 branch). GitHub. Retrieved 2026-10-03, from https://github.com/bugzilla/bugzilla/tree/5.2/docker
[^harmony-docker-docs]: Bugzilla Project. (n.d.). Installing Bugzilla with Docker. Bugzilla Harmony Documentation. Retrieved 2026-10-03, from https://bugzilla.readthedocs.io/projects/harmony/en/latest/installing/docker.html