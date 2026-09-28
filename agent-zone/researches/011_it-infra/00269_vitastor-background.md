# Vitastor 專案背景調查

Vitastor 是一個高效能的開源分散式儲存系統，定位為 Ceph 的輕量化替代品。本文調查其團隊、商業模式、社群規模與資金來源。

## 創作者與團隊

Vitastor 由俄羅斯開發者 **Vitaliy Filippov**（GitHub: `vitalif`）於 2019 年左右從零開始撰寫[^author]。他是一位資深的 C++/Node.js 系統開發者，此前最知名的專案是 **grive2**（Google Drive Linux 客戶端，GitHub 1500+ stars）[^github-vitalif]。

專案核心為一人開發，截至目前（2026 年 9 月）累積約 **3,122 次提交**，程式碼規模約 **60,000 行 C++**（對比 Ceph 約一百萬行）[^github-repo]。在釋出日誌中提及的外部貢獻者包括 Changwei Ye（新儲存層）、Dmitry Naumov（VitastorFS NFS 修復）、Stepan Rabotkin（OpenAPI 規範）、Vincent Schweiger（serve 模式修復），以及 MIND Software 的 Yury Luneff[^release-v321][^release-v320]。

## 背後公司與資金

**Vitastor 沒有任何公司、創投或組織支援**[^author]。專案完全由 Vitaliy Filippov 個人資源驅動，版權歸屬於個人（Filippov Vitaliy Vladimirovich），並於 2021 年 5 月向俄羅斯聯邦智財局（Rospatent）登記證書 #2021617829[^cla]。唯一提及的第三方機構是資料中心 Contell.ru，在其演講投影片中獲得感謝。

無任何已知的外部投資或募資紀錄。

## 授權與商業模式

Vitastor 採用作者自創的 **VNPL 1.1（Vitastor Network Public License）**，基於 GPLv3.0 修改，比 AGPL 更嚴格的 Copyleft 許可證[^author][^license]。

| 組件 | 授權 |
|------|------|
| 伺服端程式碼（OSD、Monitor 等） | VNPL 1.1 |
| 客戶端函式庫 | VNPL 1.1 + GPL 2.0+ 雙重授權 |

VNPL 的核心設計是「網路互動條款」——任何專門設計用於與 Vitastor 配合、並透過網路與之互動的中間程式（「Proxy Programs」）都必須開源。作者認為 AGPL 對基礎設施軟體無效，因為終端用戶看不見底層儲存，無法觸發 Copyleft 效力[^author]。

商業模式為「**開源核心 + 商業許可證**」：
- 若組織不願開源中間組件，可向作者購買商業許可證
- 作者直接提供技術與架構支援
- 具體價格未公開，需個別聯繫[^author]

## 社群規模

| 指標 | 數據 |
|------|------|
| GitHub Stars（鏡像倉庫） | 243 |
| GitHub Forks | 40 |
| 提交數 | 3,122 |
| Telegram 群組成員 | ~708 人 |
| Kubernetes Operator（社群維護） | 13 stars |

主要程式碼庫託管於作者自建的 Gitea 實例（git.yourcmc.ru），GitHub 上的倉庫為唯讀鏡像[^github-repo]。Telegram 群組 [@vitastor](https://t.me/vitastor) 是主要的社群交流渠道[^website]。

社群屬於**較小但活躍**，2026 年幾乎每月有新版本釋出。

## 專案時間線

| 時間 | 里程碑 |
|------|--------|
| 2019+ | 專案啟動 |
| 2020-09 | v0.4.0，首次公開效能對比測試（vs Ceph 15.2.4） |
| 2021 | DevOpsConf 2021 首次公開演講 |
| 2021-05 | Rospatent 著作權登記 |
| 2022 | Highload 2022 演講 |
| 2025 | KuberConf 2025、Highload 2025 演講 |
| 2026-07 | v3.1.0 安全性大版本（TLS、AES-XTS 加密、Vault 整合） |
| 2026-08 | v3.2.0 引入混沌測試 CI |
| 2026-09 | v3.2.2 最新版本 |

## 競爭定位

Vitastor 的定位是 **Ceph 的直接替代品**，目標是「更快 + 更簡單」：

- 4KB Q1 延遲約 **0.1ms**（宣稱比 Ceph 快 10 倍）
- CPU 使用約 **1 核心/NVMe 磁碟**（Ceph 需數十倍）
- 使用 **etcd** 儲存 metadata（Ceph 使用 RocksDB）
- PG 映射使用數學最佳化（lp_solve）而非 CRUSH 演算法[^architecture]

## 參考資料

[^author]: Filippov, V. (n.d.). Author and License. Retrieved 2026-09-27, from https://github.com/vitalif/vitastor/blob/master/docs/intro/author.en.md
[^github-vitalif]: Filippov, V. (n.d.). GitHub Profile. Retrieved 2026-09-27, from https://github.com/vitalif
[^github-repo]: Filippov, V. (n.d.). Vitastor - Fast Software-Defined Block Storage. Retrieved 2026-09-27, from https://github.com/vitalif/vitastor
[^release-v321]: Filippov, V. (2026-09-13). Vitastor 3.2.1 Release Notes. Retrieved 2026-09-27, from https://vitastor.io/en/blog/2026-09-13-v3.2.1.html
[^release-v320]: Filippov, V. (2026-08-31). Vitastor 3.2.0 Release Notes. Retrieved 2026-09-27, from https://vitastor.io/en/blog/2026-08-31-v3.2.0.html
[^cla]: Filippov, V. (n.d.). Contributor License Agreement. Retrieved 2026-09-27, from https://github.com/vitalif/vitastor/blob/master/CLA-en.md
[^license]: Vitastor Network Public License 1.1. (n.d.). Retrieved 2026-09-27, from https://github.com/vitalif/vitastor/blob/master/LICENSE
[^website]: Vitastor Official Website. (n.d.). Retrieved 2026-09-27, from https://vitastor.io
[^architecture]: Filippov, V. (n.d.). Vitastor Architecture. Retrieved 2026-09-27, from https://github.com/vitalif/vitastor/blob/master/docs/intro/architecture.en.md