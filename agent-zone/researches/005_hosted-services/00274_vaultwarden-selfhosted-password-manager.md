# Vaultwarden 自架密碼管理器 — 完整設定與使用指南

## 1. 概述

**Vaultwarden**（原名 **bitwarden_rs**）是 Bitwarden 伺服器 API 的非官方輕量化開源重新實作，使用 **Rust** 語言撰寫，採用 AGPL-3.0 授權。它並非 Bitwarden Inc. 的官方產品，而是社群維護的第三方伺服器實作[^github]。

由於與官方伺服器共用相同的 API，所有官方 Bitwarden 客戶端（桌面應用程式、瀏覽器擴充功能、iOS/Android App、CLI）均可直接對接 Vaultwarden，無需修改[^github]。此外，付費功能（TOTP 驗證碼、檔案附件、緊急存取、組織功能、安全報告等）在 Vaultwarden 上完全免費開放，因為不存在授權層[^comparison]。

Vaultwarden 最大的優勢是**極度輕量**——官方自架 Bitwarden 需執行多個容器，要求 2 GB 以上 RAM；而 Vaultwarden 僅需**單一二進位檔/容器**，閒置時僅佔 **50–256 MB RAM**，可穩定運作於樹莓派[^hardware]。

目前最新版本為 **v1.37.3**（2026-09-13），在 GitHub 上獲得約 68,000 顆星、3,096+ 次提交[^release]。

## 2. 系統需求

| 項目 | 最低需求 |
|------|---------|
| **RAM** | 128 MB（建議 256 MB+） |
| **CPU** | 任何現代處理器，支援 ARM v6/v7/v8（樹莓派） |
| **儲存** | 極低基礎需求，隨附件/寄送功能成長 |
| **作業系統** | 任何 Linux 發行版（建議 64 位元），亦可透過 Docker 在 macOS/Windows 執行 |
| **支援架構** | `linux/amd64`、`linux/arm64`、`linux/arm/v7`、`linux/arm/v6` |

多數使用者將其部署於 **每月 $5–10 美元的 VPS**、**NAS（Synology、QNAP）**，或 **樹莓派 4（2 GB RAM）**[^hardware]。

## 3. 安裝方法

### 3.1 Docker 命令列（建議）

最簡單且官方推薦的方式是使用 Docker：

```bash
docker pull vaultwarden/server:latest

docker run -d --name vaultwarden \
  --env DOMAIN="https://vw.yourdomain.com" \
  --volume /vw-data/:/data/ \
  --restart unless-stopped \
  --publish 127.0.0.1:8000:80 \
  vaultwarden/server:latest
```

[^deploy]

### 3.2 Docker Compose（生產環境建議）

```yaml
services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: unless-stopped
    environment:
      DOMAIN: "https://vw.yourdomain.com"
      SIGNUPS_ALLOWED: "false"
      ADMIN_TOKEN: "your-secure-admin-token"
    volumes:
      - ./vw-data/:/data/
    ports:
      - 127.0.0.1:8000:80
```

[^deploy]

### 3.3 可取得的容器登錄檔

| 登錄檔 | 映像檔名稱 |
|--------|-----------|
| GitHub Container Registry | `ghcr.io/dani-garcia/vaultwarden` |
| Docker Hub | `vaultwarden/server` |
| Quay.io | `quay.io/vaultwarden/server` |

**映像檔變體：**
- `vaultwarden/server:latest` — Debian 基底（標準 glibc）
- `vaultwarden/server:latest-alpine` — Alpine 基底（較小、musl）

### 3.4 從原始碼手動建置

```bash
git clone https://github.com/dani-garcia/vaultwarden.git
cd vaultwarden
cargo build --release
./target/release/vaultwarden
```

[^github]

### 3.5 反向代理伺服器（HTTPS 必備）

Web Vault 必須使用 HTTPS。建議透過 **Nginx、Caddy 或 Traefik** 設定反向代理：

```nginx
server {
    listen 443 ssl http2;
    server_name vault.example.com;

    ssl_certificate /etc/letsencrypt/live/vault.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/vault.example.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }

    location /notifications/hub {
        proxy_pass http://127.0.0.1:3012;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

[^localtonet]

## 4. 設定選項

Vaultwarden 主要透過**環境變數**進行設定。

### 4.1 核心設定

| 變數 | 說明 | 預設值 | 生產建議值 |
|------|------|--------|-----------|
| `DOMAIN` | Vaultwarden 完整公開網址 | `http://localhost` | `https://vault.example.com` |
| `DATABASE_URL` | 資料庫連線字串 | SQLite `/data/db.sqlite3` | SQLite 或 PostgreSQL URL |
| `ADMIN_TOKEN` | 管理面板驗證金鑰 | 停用 | 32+ 字元安全隨機值 |
| `SIGNUPS_ALLOWED` | 允許公開註冊 | `true` | `false` |

### 4.2 資料庫選項

```bash
# SQLite（預設，建議 ≤100 使用者）
DATABASE_URL=/data/db.sqlite3

# PostgreSQL（建議較大部署）
DATABASE_URL=postgresql://vw_user:secure_pass@localhost:5432/vaultwarden

# MySQL/MariaDB
DATABASE_URL=mysql://vw_user:secure_pass@localhost:3306/vaultwarden
```

[^config]

### 4.3 SMTP / Email 設定

| 變數 | 說明 | 預設值 |
|------|------|--------|
| `SMTP_HOST` | SMTP 伺服器主機名稱 | - |
| `SMTP_FROM` | 寄件者 Email 地址 | - |
| `SMTP_PORT` | SMTP 連接埠 | 587 |
| `SMTP_SECURITY` | 加密方式：`starttls`、`tls`、`none` | `starttls` |
| `SMTP_USERNAME` | SMTP 驗證使用者名稱 | - |
| `SMTP_PASSWORD` | SMTP 驗證密碼 | - |

### 4.4 安全與速率限制

| 變數 | 說明 | 建議值 |
|------|------|--------|
| `LOGIN_RATELIMIT_MAX_BURST` | 登入嘗試上限（觸發速率限制） | 5 |
| `LOGIN_RATELIMIT_SECONDS` | 速率限制時間窗口（秒） | 120 |
| `ADMIN_RATELIMIT_MAX_BURST` | 管理面板嘗試上限 | 3 |
| `ADMIN_RATELIMIT_SECONDS` | 管理面板速率限制窗口（秒） | 300 |
| `IP_HEADER` | 客戶端 IP 偵測標頭 | `X-Forwarded-For` |

### 4.5 功能開關

| 變數 | 預設值 | 說明 |
|------|--------|------|
| `ORGANIZATIONS_ALLOWED` | `true` | 啟用組織功能 |
| `ATTACHMENTS_ALLOWED` | `true` | 允許檔案附件 |
| `SEND_ALLOWED` | `true` | 啟用 Send 功能 |
| `EMERGENCY_ACCESS_ALLOWED` | `true` | 啟用緊急存取 |
| `WEB_VAULT_ENABLED` | `true` | 啟用 Web 介面 |
| `WEBSOCKET_ENABLED` | `false` | 啟用 WebSocket 即時同步（生產環境建議 `true`） |

### 4.6 Admin Token 設定

兩種方式：

```bash
# 純文字（安全性較低，直接存於環境變數）
ADMIN_TOKEN=your-32-char-secure-token

# Argon2 預先雜湊（較安全）
# 產生方式：
#   docker run --rm -it vaultwarden/server:latest /vaultwarden hash
ADMIN_TOKEN='$argon2id$v=19$m=65540,t=3,p=4$...'
```

[^config]

## 5. 使用方式（客戶端應用程式）

由於 Vaultwarden 實作**與官方相同的 API**，使用者可直接使用 **官方 Bitwarden 客戶端**[^comparison]。

### 5.1 客戶端設定步驟

1. 從各平台商店或 [bitwarden.com/download](https://bitwarden.com/download) 下載官方 Bitwarden App/擴充功能
2. 開啟應用程式 → 點選 **「Self-hosted」** 或 **「使用自架伺服器登入」**
3. 輸入伺服器網址：`https://vw.yourdomain.com`（你的 Vaultwarden 實例）
4. 使用 Email 與主密碼登入
5. 完成雙因素驗證（如有啟用）
6. 密碼庫將自動同步

### 5.2 相容客戶端

| 平台 | 客戶端 |
|------|--------|
| **瀏覽器** | Chrome、Firefox、Edge、Safari、Brave、Opera、Vivaldi 的 Bitwarden 擴充功能 |
| **桌面** | Bitwarden 桌面應用程式（Windows、macOS、Linux） |
| **行動裝置** | iOS 與 Android 的 Bitwarden App |
| **命令列** | `bitwarden-cli`（終端機工具） |

### 5.3 遷移注意事項

- 一個客戶端同時僅能對接一個 Bitwarden 伺服器
- 遷移方式：從舊伺服器**匯出**密碼庫（加密密碼保護 JSON），再於新伺服器**匯入**
- Vaultwarden 問題請使用 Vaultwarden 的 GitHub、Matrix 或 Discourse 社群，**勿使用 Bitwarden 官方支援**
- 客戶端更新可能造成相容性問題——大規模升級前請先測試[^github]

## 6. 安全考量與最佳實踐

### 6.1 架構：客戶端加密

所有加密均在**客戶端**進行，資料到達伺服器前已完成加密。Vaultwarden 儲存無法讀取的資料：

| 層級 | 技術 |
|------|------|
| **客戶端加密** | AES-256-CBC |
| **金鑰衍生** | PBKDF2-SHA256 / Argon2id |
| **傳輸加密** | TLS 1.3（經反向代理） |
| **伺服器端** | 可選磁碟加密[^security] |

### 6.2 強化檢查清單

**部署前：**

- ✅ 透過反向代理（Nginx/Caddy/Traefik）設定 **TLS 1.3 的 HTTPS**
- ✅ 使用 32+ 字元隨機值設定 **ADMIN_TOKEN**（可使用 `openssl rand -base64 48`）
- ✅ 設定 **SIGNUPS_ALLOWED=false**（建立帳號後關閉公開註冊）
- ✅ 資料庫檔案權限限制（`chmod 640`，由容器使用者擁有）
- ✅ 容器以**非 root 使用者**執行（`user: "1000:1000"`）
- ✅ 反向代理設定安全標頭（HSTS、X-Frame-Options、CSP 等）
- ✅ 啟用速率限制（登入與管理面板）
- ✅ 若不需持續管理，在設定完成後**停用管理面板**

**部署後：**

- ✅ 設定 **Fail2ban** 封鎖暴力破解嘗試
- ✅ 管理面板**網路層級限制**（僅允許內部 IP 存取 `/admin`）
- ✅ 容器**移除所有權限**（`cap_drop: ALL`）
- ✅ 容器**唯讀根檔案系統**
- ✅ WebSocket 透過反向代理保護（WSS）
- ✅ 透過 **Cloudflare Tunnel、Tailscale 或 VPN** 暴露服務——**避免直接暴露連接埠**[^security]

### 6.3 Docker 容器安全設定

```yaml
services:
  vaultwarden:
    image: vaultwarden/server:latest
    user: "1000:1000"
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    read_only: true
    tmpfs:
      - /tmp:noexec,nosuid,size=100m
```

[^security]

### 6.4 安全性稽核

Vaultwarden 已接受外部安全性稽核，詳見 [Vaultwarden Wiki](https://github.com/dani-garcia/vaultwarden/wiki/Vaultwarden-Audits)。自 v1.35.0 起，釋出採用**不可變釋出搭配簽署驗證**（cosign）[^github]。

## 7. 備份與還原

### 7.1 需備份的元件

| 元件 | 位置 | 優先級 |
|------|------|--------|
| **資料庫** | `data/db.sqlite3`（SQLite）或外部資料庫 | 🔴 關鍵 |
| **附件** | `data/attachments/` | 🔴 關鍵 |
| **寄送檔案** | `data/sends/` | 🔴 關鍵 |
| **設定檔** | `data/config.json` | 🟡 建議 |
| **簽署金鑰** | `data/rsa_key.*` | 🟡 建議 |
| **圖示快取** | `data/icon_cache/` | 🟢 可選 |
| **環境設定** | `.env`、`compose.yaml` | 🔴 關鍵（加密儲存） |

### 7.2 資料目錄結構

```
data/
├── attachments/          # 密碼庫項目中的檔案附件
├── config.json           # 管理頁面設定
├── db.sqlite3            # 主要 SQLite 資料庫
├── db.sqlite3-shm        # SQLite 共享記憶體（非必現）
├── db.sqlite3-wal        # SQLite 日誌檔（非必現）
├── icon_cache/           # 網站 favicon 快取
├── rsa_key.der           # JWT 簽署金鑰
├── rsa_key.pem
├── rsa_key.pub.der
└── sends/                # Send 功能附件
```

[^backup]

### 7.3 備份方法

**方法 1：SQLite `.backup` 命令（執行中備份，建議）**

```bash
sqlite3 data/db.sqlite3 ".backup '/path/to/backups/db-$(date '+%Y%m%d-%H%M').sqlite3'"
```

[^backup]

**方法 2：內建備份命令（v1.32.1+）**

```bash
# 直接執行
/vaultwarden backup

# 透過 Docker
docker exec -it vaultwarden /vaultwarden backup
```

[^backup]

**方法 3：自動化備份腳本**

```bash
#!/bin/bash
set -eu
umask 077

DATA_DIR="/opt/vaultwarden/vw-data"
BACKUP_ROOT="/backup/vaultwarden"
STAMP=$(date -u +%Y%m%dT%H%M%SZ)
DEST="$BACKUP_ROOT/$STAMP"

mkdir -p "$DEST"

# SQLite 一致性快照
sqlite3 "$DATA_DIR/db.sqlite3" ".timeout 10000" ".backup '$DEST/db.sqlite3'"

# 附件與寄送
for dir in attachments sends; do
  [ -d "$DATA_DIR/$dir" ] && rsync -aH "$DATA_DIR/$dir/" "$DEST/$dir/"
done

# 設定檔與金鑰
[ -f "$DATA_DIR/config.json" ] && cp -p "$DATA_DIR/config.json" "$DEST/"
for key in "$DATA_DIR"/rsa_key*; do
  [ -e "$key" ] && cp -p "$key" "$DEST/"
done

# 驗證完整性
sqlite3 "$DEST/db.sqlite3" "PRAGMA integrity_check;"
```

[^backup]

### 7.4 第三方備份工具

- [ttionya/vaultwarden-backup](https://github.com/ttionya/vaultwarden-backup) — Docker 化，支援 rclone、加密、通知
- [jjlin/vaultwarden-backup](https://github.com/jjlin/vaultwarden-backup) — 簡潔的自動化 SQLite 備份

### 7.5 還原步驟

1. **停止** Vaultwarden：`docker compose down`
2. **取代**資料目錄為備份內容
3. **刪除**過期的 `db.sqlite3-wal` 檔案（如使用 `.backup` 還原）
4. **確保正確權限**（擁有者需與容器使用者一致）
5. **啟動** Vaultwarden：`docker compose up -d`
6. **驗證**——登入、檢查項目、附件與同步狀態
7. **務必定期測試還原流程！**[^backup]

## 8. 更新與維護

### 8.1 Docker 更新步驟

```bash
cd /path/to/vaultwarden
docker compose pull
docker compose up -d
```

[^update]

### 8.2 更新前檢查清單

1. 閱讀 GitHub **Release Notes**——檢查重大變更或資料庫遷移
2. 更新前**建立完整備份**
3. 記錄目前映像檔摘要：`docker compose images`
4. 規劃**回滾方案**——保留先前映像檔。資料庫遷移可能導致無法直接降版
5. **安排維護時段**進行更新

### 8.3 版本驗證（v1.35.0+）

自 v1.35.0 起，釋出採用**不可變釋出搭配簽署驗證**：

```bash
cosign verify \
  --certificate-identity-regexp "https://github.com/dani-garcia/vaultwarden" \
  --certificate-oidc-issuer "https://token.actions.githubusercontent.com" \
  ghcr.io/dani-garcia/vaultwarden:1.35.3
```

[^github]

### 8.4 例行維護

**每日/每週：**
- 容器正常執行且日誌無錯誤
- 公開網址可正常存取且憑證有效
- 備份完成並送達異地儲存
- 磁碟容量充足
- 公開註冊保持關閉

**每月：**
- 檢視 Vaultwarden 新版本
- 測試從備份還原
- 審查安全性日誌，檢查異常登入嘗試

**安全性更新：**
- 監控 GitHub Security Advisories：https://github.com/dani-garcia/vaultwarden/security/advisories
- 重大修補立即套用
- 訂閱 Vaultwarden 釋出通知（RSS）

### 8.5 故障復原

| 故障 | 症狀 | 復原方式 |
|------|------|---------|
| 容器停止 | 通道正常但無回應 | `docker compose up -d`，檢查日誌 |
| 儲存卸載 | 出現空密碼庫 | 修正掛載，還原資料目錄 |
| TLS 憑證過期 | 憑證警告 | 更新憑證（Let's Encrypt） |
| 資料庫損毀 | 應用程式無法啟動 | 從驗證備份還原 |
| 客戶端不相容 | 無法同步/連線 | 更新 Vaultwarden 至最新版[^backup] |

## 9. 總結

Vaultwarden 是一個極具成本效益的密碼管理自架方案。相較於官方 Bitwarden 自架版本，它具備以下優勢：

- **極低資源消耗**：可運行於樹莓派或低階 VPS
- **零授權費用**：內建 Bitwarden 所有付費功能
- **完整客戶端相容性**：直接使用所有官方 Bitwarden 客戶端
- **活躍社群**：GitHub 68k+ 星、持續維護中

不過也需注意其限制：
- **非官方產品**，無法獲得 Bitwarden Inc. 的支援
- **需自行管理**更新、備份與安全性
- **可能與新客戶端版本暫時不相容**

對於熟悉 Docker 與基本 Linux 管理的使用者而言，Vaultwarden 是目前最輕量、高效的自架密碼管理方案。

---

[^github]: dani-garcia. (n.d.). Vaultwarden — Unofficial Bitwarden compatible server written in Rust. GitHub. Retrieved 2026-09-26, from https://github.com/dani-garcia/vaultwarden

[^comparison]: WunderTech. (2026). Vaultwarden vs Bitwarden: Which should you choose? Retrieved 2026-09-26, from https://www.wundertech.net/vaultwarden-vs-bitwarden/

[^hardware]: dani-garcia. (n.d.). Vaultwarden hardware requirements discussion (#2903). GitHub Discussions. Retrieved 2026-09-26, from https://github.com/dani-garcia/vaultwarden/discussions/2903

[^deploy]: DeepWiki. (n.d.). Vaultwarden Docker deployment. Retrieved 2026-09-26, from https://deepwiki.com/dani-garcia/vaultwarden/6.1-docker-deployment

[^config]: Linux Server Admin Wiki. (n.d.). Vaultwarden configuration reference v1.35.x. Retrieved 2026-09-26, from https://wiki.linux-server-admin.com/web-apps/vault-secret-management/vaultwarden/configuration

[^security]: Linux Server Admin Wiki. (n.d.). Vaultwarden security hardening guide. Retrieved 2026-09-26, from https://wiki.linux-server-admin.com/web-apps/vault-secret-management/vaultwarden/security

[^localtonet]: LocalToNet. (2026). How to self-host Vaultwarden. Retrieved 2026-09-26, from https://localtonet.com/blog/how-to-self-host-vaultwarden

[^backup]: dani-garcia. (n.d.). Backing up your vault — Vaultwarden Wiki. Retrieved 2026-09-26, from https://github-wiki-see.page/m/dani-garcia/vaultwarden/wiki/Backing-up-your-vault

[^update]: dani-garcia. (n.d.). Updating the Vaultwarden image — Vaultwarden Wiki. Retrieved 2026-09-26, from https://github-wiki-see.page/m/dani-garcia/vaultwarden/wiki/Updating-the-vaultwarden-image

[^release]: ReleaseAlert. (2026). Vaultwarden release tracking v1.37.3. Retrieved 2026-09-26, from https://releasealert.dev/github/dani-garcia/vaultwarden