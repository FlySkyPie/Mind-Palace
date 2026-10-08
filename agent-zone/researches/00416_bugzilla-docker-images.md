# Bugzilla 第三方 Docker 映像檔調查

Bugzilla 是知名的開源缺陷追蹤系統。本報告調查 Bugzilla 是否有著名、可信賴且廣為使用的第三方 Docker 映像檔，以及 Bugzilla 專案官方提供的 Docker 映像檔現狀。

## 官方 Docker 映像檔（bugzilla/ 組織）

Bugzilla 專案在 Docker Hub 上擁有 `bugzilla/` 組織，**但並非 Docker Official Images 專案的一部分**（即沒有 `bugzilla:latest` 這種官方頂層映像檔）。官方提供的是「基礎映像檔」（base images），僅包含 Bugzilla 所需的 Perl 依賴，不包含 Bugzilla 應用程式本身，使用者需以此為基底自行建置[^bugzilla-org]。

以下為官方目前維護的活躍映像檔：

| 映像檔名稱 | 說明 | 大小 | pull 次數 | 最近更新 |
|---|---|---|---|---|
| `bugzilla/bugzilla-perl-slim` | Debian Bullseye + Bugzilla 5.9.x 所需的必要 Perl 依賴（無 DB 驅動、無 Bugzilla 本體） | 198 MB | ~4.3K | ~2 個月前 |
| `bugzilla/bugzilla-perl-slim-mysql` | Slim + MySQL 8 客戶端驅動 | 280 MB | **10K+** | ~2 個月前 |
| `bugzilla/bugzilla-perl-slim-mariadb` | Slim + MariaDB 客戶端驅動 | 290 MB | ~1.5K | ~2 個月前 |
| `bugzilla/bugzilla-perl-slim-pg` | Slim + PostgreSQL 客戶端驅動 | 258 MB | ~1.6K | ~2 個月前 |
| `bugzilla/bugzilla-perl-full` | Slim + 所有可選 Perl 依賴 | 285 MB | ~120 | ~2 個月前 |
| `bugzilla/bugzilla-perl-full-mysql` | Full + MySQL 驅動 | 374 MB | ~95 | ~2 個月前 |
| `bugzilla/bugzilla-perl-full-mariadb` | Full + MariaDB 驅動 | 384 MB | ~120 | ~2 個月前 |
| `bugzilla/bugzilla-perl-full-pg` | Full + PostgreSQL 驅動 | 356 MB | ~92 | ~2 個月前 |

官方還曾維護 `bugzilla/bugzilla-dev`（曾經 50K+ pull，已棄用）、`bugzilla/harmony`（10K+ pull，已棄用）等映像檔，但均已停止更新[^bugzilla-org]。

## 第三方 Docker 映像檔

以下是第三方維護、直接可用（包含 Bugzilla 完整應用）的映像檔，依 pull 次數排序：

| 映像檔名稱 | 維護者 | pull 次數 | 最後更新 |
|---|---|---|---|
| `nasqueron/bugzilla` | Nasqueron 組織 | **10K+** | ~8 年前 |
| `achild/bugzilla` | achild | **10K+** | ~11 年前 |
| `moravianlibrary/bugzilla` | Moravian Library | **10K+** | ~10 年前 |
| `chardek/bugzilla` | chardek | ~9.9K | ~7 年前 |
| `mojab/bugzilla` | mojab | ~5.1K | ~7 年前 |
| `yangzhaofengsteven/bugzilla` | yangzhaofengsteven | ~3.1K | ~2 年前 |
| `herzcthu/bugzilla` | herzcthu | ~3.1K | ~7 年前 |
| `turnkeylinux/bugzilla` | TurnKey Linux | ~1.4K | ~7 年前 |
| `dshap/bugzilla` | dshap | ~1.1K | **約 13 天前**（近期有更新） |

其他較小 pull 數的映像檔包括：`bxwill/bugzilla`（631）、`rekiba/bugzilla`（402）、`neo1975/bugzilla`（2.1K）、`cssdata/bugzilla`（1.0K）、`x2store/bugzilla`（1.1K）等，共計約 20 多個第三方維護的版本[^docker-hub-search]。

## Bugzilla GitHub 官方倉庫的自建選項

Bugzilla 官方 GitHub 倉庫（bugzilla/bugzilla）包含了 `Dockerfile`、`Dockerfile.mariadb` 與 `docker-compose.yml`，使用者可以透過 `docker compose up` 一鍵啟動完整的 Bugzilla 實例[^github-bugzilla]。

## 分析與建議

**重要發現：** 沒有任何一個 Docker 映像檔是「Docker Official Images」，也沒有單一「公認最權威的第三方映像檔」。第三方映像檔普遍已多年未更新（5–11 年），安全性與 Bugzilla 版本均已老舊。

| 使用情境 | 最佳選擇 |
|---|---|
| 需要最權威、可信賴的基礎 | `bugzilla/bugzilla-perl-slim-mysql`（官方維護、近期更新、10K+ pull） |
| 需要一個可直接執行的 Bugzilla | 使用官方 GitHub 倉庫的 `Dockerfile` 自建 |
| 需要一個最近有更新的第三方映像檔 | `dshap/bugzilla`（約 13 天前更新） |
| 知名的第三方映像檔 | `turnkeylinux/bugzilla`（但已 7 年未更新） |

**總結：** Bugzilla **沒有**一個被廣泛公認的「著名第三方 Docker 映像檔」。最可靠的做法是使用官方提供的基礎映像檔（`bugzilla/bugzilla-perl-slim-mysql`）搭配官方 GitHub 倉庫的 Dockerfile 自建。第三方映像檔雖多，但因長期未更新，不建議用於生產環境。

[^bugzilla-org]: Bugzilla Project. (n.d.). Docker Hub — bugzilla organization images. Retrieved 2026-10-01, from https://hub.docker.com/u/bugzilla

[^docker-hub-search]: Docker Hub. (n.d.). Search results for "bugzilla" on Docker Hub. Retrieved 2026-10-01, from https://hub.docker.com/search?q=bugzilla

[^github-bugzilla]: Bugzilla Project. (n.d.). bugzilla/bugzilla — GitHub repository. Retrieved 2026-10-01, from https://github.com/bugzilla/bugzilla

[^nasqueron]: Nasqueron. (n.d.). nasqueron/bugzilla Docker image. Retrieved 2026-10-01, from https://hub.docker.com/r/nasqueron/bugzilla

[^achild]: achild. (n.d.). achild/bugzilla Docker image. Retrieved 2026-10-01, from https://hub.docker.com/r/achild/bugzilla

[^turnkeylinux]: TurnKey Linux. (n.d.). turnkeylinux/bugzilla Docker image. Retrieved 2026-10-01, from https://hub.docker.com/r/turnkeylinux/bugzilla

[^dshap]: dshap. (n.d.). dshap/bugzilla Docker image. Retrieved 2026-10-01, from https://hub.docker.com/r/dshap/bugzilla