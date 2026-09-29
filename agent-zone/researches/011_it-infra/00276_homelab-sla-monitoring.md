# Homelab SLA 監控：服務可用性與效能追蹤指南

## 什麼是 Homelab 的 SLA 監控？

在 Homelab 環境中，SLA 監控並非出於合約罰則需求，而是為了**量化自家自架服務的可用性、回應速度與可靠度**。核心監控指標包括：

| 指標 | 說明 | 為何重要 |
|---|---|---|
| **Uptime %** | 服務可達到的時間百分比 | SLA 核心指標；公式為 `(總時間 - 停機時間) / 總時間 × 100` |
| **Response time** | 服務回應速度（例如低於 200ms） | 可在完全斷線前抓到效能退化；回應 4 秒的服務雖然「上線了」但已故障 |
| **SSL 憑證到期日** | TLS 憑證剩餘天數 | 預防自己造成的斷線（忘記更新憑證） |
| **Heartbeat / Dead-man's-switch** | 排程任務（備份、cron）是否如期執行 | 最常被忽略的失敗模式——「你以為在跑的備份其實早就停了」 |
| **Error rate** | 請求失敗的比例 | 追蹤服務健康度，不只 up/down 二元判斷 |

## 推薦工具

### Uptime Kuma — 社群首選

- **定位：** 自託管、美觀 Web UI、90+ 通知提供者、15+ 監控類型（HTTP、TCP、Ping、DNS、Docker、gRPC、資料庫等）[^kuma-gh]
- **SLA 功能：** 自動計算 24h/7d/30d/自訂區間 uptime %、內建狀態頁面、事件時間軸、憑證到期監控
- **資源用量：** ~80-100MB RAM，SQLite 儲存
- **部署：** 一行 Docker Compose

### Gatus — 以設定檔為代碼

- **定位：** 開發者導向，YAML 定義監控規則，支援豐富條件語法（狀態碼 + 回應時間 + JSONPath 內文驗證 + 憑證到期）[^gatus-gh]
- **SLA 功能：** 自動 SLA 評分、回應時間追蹤、uptime 徽章、可設定 failure-threshold/success-threshold 防誤報、Prometheus `/metrics` 端點
- **資源用量：** ~30MB RAM（非常輕量）
- **部署：** 單一容器，基本使用不需資料庫
- **適合：** GitOps 工作流程——設定檔放在 git，新主機 `git clone && docker compose up` 就完成

### Prometheus + Grafana — 完整觀測堆疊

- **定位：** Prometheus 從各種 exporter 抓取指標，Grafana 製作儀表板與警報
- **SLA 功能：** 儀表板 uptime 面板、回應時間趨勢圖、Alertmanager 警報路由
- **關鍵 exporter：** `node_exporter`（主機指標）、`blackbox_exporter`（外部 HTTP/HTTPS/ICMP 探測）
- **資源用量：** 完整堆疊約 1-2GB RAM

### 其他工具

| 工具 | 定位 | 適合場景 |
|---|---|---|
| **Checkmk** | 企業級自動探索監控，大量外掛 | 混合 fleet（VM、容器、實體機） |
| **Zabbix** | Agentless SNMP 探索，範本式設定 | 網路設備監控 |
| **Healthchecks.io** | Heartbeat / cron 監控 | 備份與排程任務監控 |
| **Statping-ng** | 單一二進位檔，內建監控+狀態頁 | 追求極簡 |
| **UptimeRobot** | 外部免費服務（50 個監控器） | 從外部監控 Homelab |
| **Better Stack** | 外部免費服務（10 個監控器） | 從外部監控 Homelab |

## SLA 儀表板與狀態頁設定

### Uptime Kuma — 5 分鐘完成狀態頁

1. 透過 Docker Compose 部署
2. 為關鍵服務建立監控器（建議間隔：關鍵 60s、重要 2-3min、次要 5min）
3. 前往 **Status Pages → New Status Page** → 選擇監控器 → 設定 slug → 分享 URL
4. 可透過 SQLite 匯出 SLA 資料

### Gatus — YAML 定義 SLA

```yaml
endpoints:
  - name: nextcloud
    group: core-services
    url: "https://nextcloud.example.com/status.php"
    interval: 1m
    conditions:
      - "[STATUS] == 200"
      - "[RESPONSE_TIME] < 800"
    alerts:
      - type: ntfy
        failure-threshold: 3
        success-threshold: 2
        send-on-resolved: true
```

Gatus 支援從 SQLite 直接查詢 SLA 百分比，且每個 endpoint 可獨立設定條件式，不只檢查 up/down。[^gatus-sql]

### 狀態頁方案比較

| 方案 | 類型 | 適合 |
|---|---|---|
| **Uptime Kuma status mode** | 內建狀態頁 | 多數 Homelab——無痛，已在 Kuma 內 |
| **Gatus badges** | Per-endpoint uptime 徽章 | 嵌入 README 或內部 Wiki |
| **Cachet** | 獨立事故管理 | 有真正使用者、正式 SLA 需求 |

## 最佳實踐

### 內外兼顧：外部探測的必要性

如果監控工具和你監控的服務在同一台機器或同一個網路，當整個 Homelab 斷線時，監控工具也跟著掛了。**至少需要一個從外部網路執行的監控**（如便宜的 VPS 或免費雲端服務）。[^ensju]

| 檢查類型 | 能抓到什麼 | 執行位置 |
|---|---|---|
| **內部**（LAN 內） | 反向代理設定錯誤、容器當掉 | Homelab 內 |
| **外部**（VPS/雲端） | ISP 斷線、停電、家用網路故障 | 外部 VPS 或免費方案 |

### 合成監控 (Synthetic Monitoring)

不要只檢查「服務有回應」，要檢查「服務是否正常運作」。Gatus 的條件語言在這方面特別強：

```yaml
conditions:
  - "[STATUS] == 200"
  - "[RESPONSE_TIME] < 800"
  - "[BODY].installed == true"
  - "[CERTIFICATE_EXPIRATION] > 336h"
```

### 避免警示疲勞

- **關鍵服務：** 連續 2-3 次失敗才發警報
- **非關鍵服務：** 連續 3-5 次失敗
- **效能退化警報：** 較低閾值（2 次失敗），因為退化問題很少自行恢復

### 分層警報

| 優先級 | 類型 | 發送方式 | 範例 |
|---|---|---|---|
| **Critical** | 叫醒我 | 手機推播（ntfy、Gotify、Pushover） | 磁碟全滿、主機掛掉、備份連續兩晚沒跑 |
| **Warning** | 明天再看 | Email 或儀表板 | 磁碟 > 80%、憑證 14 天內到期 |
| **Informational** | 僅記錄 | 儀表板或日誌 | 例行重啟、正常事件 |

### 追蹤回應時間，不只 up/down

純 up/down 檢查對效能退化完全無效。回應 4 秒的資料庫是「上線」但故障。應在每個 endpoint 加上 `[RESPONSE_TIME]` 條件，在真正斷線前數小時就抓到緩慢故障。

### 現實的檢查間隔

- **30 秒：** 面向使用者的 Web 應用，flapping 算事故
- **2-5 分鐘：** 內部基礎設施合成檢查
- **6 小時：** 憑證到期檢查（失敗模式以天為單位）

### 備份監控設定

- **Uptime Kuma / Statping / Healthchecks：** 備份 SQLite 資料庫 volume
- **Gatus：** 將 YAML 設定檔加入 Git 版本控制
- **Grafana：** 匯出儀表板為 JSON，將警報規則納入版本控制

## 推薦入門堆疊

對於 2026 年的多數 Homelab 管理者，建議的 SLA 監控堆疊：

1. **Uptime Kuma** — 自託管 uptime 檢查 + 狀態頁（30 分鐘設定）
2. **ntfy 或 Gotify** — 手機推播警報（15 分鐘設定）
3. **一個外部服務**（UptimeRobot 或 Better Stack 免費方案）— 抓整個 Homelab 斷線
4. **Healthchecks.io** — 備份與 cron 任務的心跳監控
5. **Prometheus + Grafana**（需要更深層指標時）— 主機與應用層級指標
6. **Scrutiny** — 磁碟健康 (SMART) 監控

**成本：$0**（皆為開源軟體）。**設定時間：約 2 小時**。**涵蓋約 90% 的 Homelab 故障情境**。

## 參考資源

### 綜合指南

- [The 2026 Homelab Observability Stack: Every Layer](https://www.bigiron.cc/guides/the-2026-homelab-observability-stack-every-layer) — 完整 10 層觀測堆疊指南
- [Homelab Monitoring in 2026 — Visual Sentinel](https://visualsentinel.com/blog/how-to-monitor-homelab-websites-downtime-changes) — 指標、工具與整合步驟
- [Homelab Monitoring: A Practical Low-Cost Guide — Ensju](https://ensju.com/resources/homelab-monitoring-guide) — NAT/CGNAT 限制、最小入門堆疊

### 工具特定資源

- [Uptime Kuma GitHub](https://github.com/louislam/uptime-kuma)[^kuma-gh]
- [Gatus GitHub](https://github.com/TwiN/gatus)[^gatus-gh]
- [Gatus: Config-as-code in git](https://www.bigiron.cc/guides/gatus-config-as-code-uptime-checks-in-git)[^gatus-sql]
- [Prometheus + Grafana 觀測堆疊](https://www.bigiron.cc/guides/grafana-prometheus-influxdb-loki-the-observability-stack)

### 警報與合成監控

- [Designing alerts that do not get ignored](https://www.bigiron.cc/guides/designing-alerts-that-do-not-get-ignored)
- [Synthetic monitoring and the false-positive problem](https://www.bigiron.cc/guides/synthetic-monitoring-and-the-false-positive-problem)
- [The minimum monitoring a household actually needs](https://www.bigiron.cc/guides/the-minimum-monitoring-a-household-actually-needs)

[^kuma-gh]: louislam. (n.d.). Uptime Kuma — A self-hosted monitoring tool. Retrieved 2026-09-26, from https://github.com/louislam/uptime-kuma
[^gatus-gh]: TwiN. (n.d.). Gatus — Automated service health dashboard. Retrieved 2026-09-26, from https://github.com/TwiN/gatus
[^gatus-sql]: Big Iron. (2026). Gatus: Config-as-code uptime checks in git. Retrieved 2026-09-26, from https://www.bigiron.cc/guides/gatus-config-as-code-uptime-checks-in-git
[^ensju]: Ensju. (2026). Homelab Monitoring: A Practical Low-Cost Guide. Retrieved 2026-09-26, from https://ensju.com/resources/homelab-monitoring-guide