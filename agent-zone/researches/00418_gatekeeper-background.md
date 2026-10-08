# Gatekeeper（gogatekeeper/gatekeeper）專案背景調查

## 概要

Gatekeeper 是一個輕量級反向代理，用於驗證 HTTP 請求和授權 API 請求，主要作為 OpenID Connect（OIDC）驗證代理使用。本報告調查其背後的公司、資金、團隊、社群及歷史背景。

## 家系：從 Red Hat 專案到社群分支

Gatekeeper 並非起源於 gogatekeeper，而是歷經三次轉手：

| 階段 | 時間 | 說明 |
|------|------|------|
| Keycloak Gatekeeper | ~2015–2020 | 由 Red Hat 旗下 Keycloak 團隊開發的原版專案 |
| louketo/louketo-proxy | 2020 年初–2020-12-07 | Keycloak 團隊將專案重組並更名為 Louketo Proxy |
| gogatekeeper/gatekeeper | 2020-11-26 至今 | 社群分支，由上流 louketo-proxy 於 2020-11-26 分支出來 |

2020 年 8 月 21 日，Keycloak 團隊的 Bruno Oliveira 宣佈 Louketo 終止（EOL 為 2020-11-21），原因是與 OAuth2 Proxy 合併的努力失敗，並建議使用者遷移至 [oauth2-proxy/oauth2-proxy][^eol]。2020 年 12 月 7 日，louketo-proxy 倉庫正式封存（read-only）。[^eol]

## 公司與資金

**沒有任何公司、商業實體或創投支援此專案。** Gatekeeper 是完全由社群維護的開源專案，採用 Apache 2.0 授權。[^repo]

- ❌ 無註冊公司（LLC / Inc. / GmbH 等）
- ❌ 無創投（VC）投資
- ❌ 無已知投資者或加速器參與
- ❌ 無 GitHub Sponsors、Open Collective、Patreon 或其他贊助管道
- ❌ 無公司贊助

Docker 映像托管於 Red Hat 的 quay.io（quay.io/gogatekeeper/gatekeeper），但這僅為公共映像托管，並非商業合作關係。[^quay]

## 團隊成員

專案實際由 **單一核心維護者** 運作：[^contributors][^repo]

| 貢獻者（GitHub） | 提交數 | 角色 |
|---|---|---|
| **p53**（Pavol Ipoth） | 515 | 唯一活躍維護者 |
| **gambol99** | 409 | 上流 louketo-proxy 原始作者（繼承提交） |
| **jangaraj** | 5 | 小型貢獻 |
| **stianst**（Stian Thorgersen, Red Hat Keycloak 負責人） | 5 | 上流貢獻（繼承） |
| 其他約 16 人 | 1–3 每人 | 小型社群貢獻 |

### Pavol Ipoth（p53）

- 公開聯絡：`pavol.ipoth@protonmail.com`（ProtonMail，暗示獨立、非公司身份）[^repo]
- Medium 部落格：`@pavol.ipoth`，撰寫關於 Gatekeeper 用於轉發代理認證的文章
- RocketReach 將其潛在雇主列為 "Telekom"，但未經驗證
- 公開背景細節極少

### 組織成員

gogatekeeper GitHub 組織沒有公開成員，頁面顯示：「This organization has no public members. You must be a member to see its members.」[^org]

## 社群規模與健康度

| 指標 | 數值 | 評估 |
|---|---|---|
| GitHub Stars | 319 | 小型但穩定 |
| Forks | 56 | 中等 |
| Open Issues | 4 | 維護者及時處理 |
| Open PRs | 0 | 表明維護者積極 |
| 總提交數 | 1,083 | |
| 貢獻者總數 | 20 人 | 1 位核心活躍 + 19 位零散貢獻 |
| 最後提交 | 2026-10-07 | 持續活躍開發中 |
| 最新版本 | 5.0.0（2026-09-13） | 重大重構版本 |
| 語言 | Go（Go 1.25+） | |
| 授權 | Apache 2.0 | |
| 社群管道 | Discord（discord.com/invite/zRqVXXTMCv） | 活躍程度不明 |

**健康度評估：** 小型但健康的社群。一位非常活躍的維護者（p53）。Stars 從 2020 年 11 月的 0 成長到 319，穩步成長但不爆炸性。Issues 僅 4 個開放，顯示維護者會處理通報。2026 年仍有活躍提交和版本發布，顯示專案生命力。

作為對照，上流已封存的 louketo/louketo-proxy 有 948 顆星和 352 個 forks。[^louketo]

## 版本歷史與路線圖

### 重要時間線

| 日期 | 事件 |
|------|------|
| ~2015 | Keycloak Gatekeeper 原始專案啟動 |
| 2020-08-21 | Bruno Oliveira（Red Hat）宣佈 Louketo 終止 |
| 2020-11-21 | Louketo EOL |
| 2020-11-26 | Pavol Ipoth（p53）從 louketo-proxy 分支建立 gogatekeeper/gatekeeper |
| 2020-12-07 | louketo-proxy 倉庫封存（read-only） |
| 2026-09-13 | v5.0.0 發布 — 重大重構、strict-deny 預設啟用、路徑正規化、安全改進 |
| 2026-10-07 | 最後一次提交（截至本報告） |

### 路線圖

Milestones 頁面顯示 2 個已關閉的里程碑（5.0.0、4.12.0），沒有公開的未來路線圖。專案似乎以「隨需演進」方式進行，而非計劃性發布節奏。[^milestones]

## 結論

Gatekeeper（gogatekeeper/gatekeeper）完全是一個**純社群維護的開源分支專案**，沒有任何公司、資金或商業實體支撐。它由 Pavol Ipoth 單人獨立維護，從已終止的 Red Hat/Keycloak 專案分支出來，以小而健康的方式持續運作。對潛在使用者而言，應注意這是**無 SLA、無公司背書、依賴單一維護者**的專案。

## 參考文獻

[^repo]: gogatekeeper/gatekeeper. (n.d.). GitHub repository. Retrieved 2026-10-03, from https://github.com/gogatekeeper/gatekeeper

[^org]: gogatekeeper. (n.d.). GitHub organization. Retrieved 2026-10-03, from https://github.com/gogatekeeper

[^eol]: Keycloak Team / Bruno Oliveira. (2020-08-21). Sunsetting the Louketo project. Keycloak Blog. Retrieved 2026-10-03, from https://www.keycloak.org/2020/08/sunsetting-louketo-project.adoc

[^louketo]: louketo/louketo-proxy. (n.d.). GitHub repository. Retrieved 2026-10-03, from https://github.com/louketo/louketo-proxy

[^contributors]: gogatekeeper/gatekeeper contributors. (n.d.). GitHub API. Retrieved 2026-10-03, from https://api.github.com/repos/gogatekeeper/gatekeeper/contributors?per_page=20

[^milestones]: gogatekeeper/gatekeeper milestones. (n.d.). GitHub. Retrieved 2026-10-03, from https://github.com/gogatekeeper/gatekeeper/milestones

[^quay]: quay.io/gogatekeeper/gatekeeper. (n.d.). Red Hat Quay Container Registry. Retrieved 2026-10-03, from https://quay.io/repository/gogatekeeper/gatekeeper?tab=tags