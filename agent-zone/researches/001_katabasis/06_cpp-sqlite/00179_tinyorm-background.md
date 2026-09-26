# TinyORM 專案背景調查

## 專案概要

TinyORM 是一個現代的 C++ ORM 函式庫，由 Silver Zachara（GitHub 帳號：silverqx）以個人名義開發維護。專案創建於 2021 年 1 月 14 日[^repo]，採用 MIT 授權條款[^license]，支援 MySQL、PostgreSQL 與 SQLite 資料庫，並提供 Query Builder、Migration、Seeder 等功能[^topics]。

## 公司與組織

**TinyORM 背後沒有任何公司或組織支援。** 這是一個純粹的個人開源專案。作者 Silver Zachara 的 GitHub 個人資料中 `company` 欄位為空，所在地為斯洛伐克（Slovakia），個人簡介為「EOL ~ EOF < TSR」[^silverqx]。

## 團隊成員

TinyORM 本質上是 **單人專案**：

| 貢獻者 | GitHub | 提交數 | 角色 |
|--------|--------|--------|------|
| Silver Zachara | [silverqx](https://github.com/silverqx) | ~5,733 commits | 主要維護者 |
| Alonso Schaich | [SchaichAlonso](https://github.com/SchaichAlonso) | 6 commits | 次要貢獻者 |

AUTHORS 檔案中除了上述兩人外，僅列出「感謝 bug 回報」，無其他具名貢獻者[^authors]。

## 資金來源

**沒有任何機構性或商業資金支援。** 專案僅開放以下捐贈管道[^donations]：

- **Bitcoin**：`1NiF2cTvYxUj8FTZJnGn1ycN4yisWfo1vJ`
- **PayPal**：`paypal.me/silverzachara`
- **GitHub Sponsors**：頁面存在但未設定任何贊助方案，實質上未啟用[^sponsors]

作者在捐贈文件中表示：「我希望能繼續開發並改進這個專案，只要還能做到我就會繼續。但未來並不明朗。」[^donations]

## 社群規模

| 指標 | 數值 |
|------|------|
| GitHub Stars | 353 |
| Forks | 37 |
| Watchers | 353 |
| Subscribers | 8 |
| 開放 Issues | 15 |
| 總提交數 | 5,739 |
| 貢獻者 | 2（1 主要 + 1 次要） |
| 單元測試數 | 3,378 |
| 最新版本 | v0.38.1（2026-08-22） |
| 最後推送 | 2025-04-02 |

對於 C++ ORM 函式庫這個小眾類別而言，這屬於中等規模的社群。專案討論僅限於 GitHub Issues 與 Discussions，無其他社群平台[^repo]。

## 外部媒體與關注度

TinyORM 的外部媒體覆蓋率 **極低**：

**存在的外部提及：**
- **Reddit r/cpp**：作者本人發布過 3 篇自我宣傳貼文（2024 年 2 月、2024 年 12 月、2025 年 4 月）[^reddit1][^reddit2][^reddit3]
- **Microsoft vcpkg**：TinyORM 有官方 vcpkg port，由作者本人維護，收錄於 Microsoft vcpkg registry[^vcpkg]

**搜尋後無結果的項目：**
- 無任何第三方部落格文章、技術媒體報導
- 無任何大會演講或技術座談
- 無 Hacker News 討論
- 無 YouTube 影片介紹
- 無 Stack Overflow 提及
- 無 dev.to、Lobste.rs 等平台內容
- 未收錄於 Wikipedia ORM 列表或 Awesome C++ 等精選清單

## 相關專案

作者維護的相關專案包括[^silverqx_repos]：

| 專案 | 用途 |
|------|------|
| [TinyORM-HelloWorld](https://github.com/silverqx/TinyORM-HelloWorld) | 入門範例（1 star） |
| [TinyOrmPlayground](https://github.com/silverqx/TinyOrmPlayground) | 個人測試場（~1,600 次資料庫查詢） |
| TinyDrivers | QtSql 模組的替代方案（目前僅支援 MySQL） |
| TinyMySql | TinyDrivers 的 MySQL 驅動 |
| tom | CLI 遷移工具（支援 bash/zsh/pwsh tab completion） |

官方文件網站為 [www.tinyorm.org](https://www.tinyorm.org)，以 Docusaurus 建置[^docs]。

## 總結

TinyORM 是一個高品質的單人 C++ 開源專案，由斯洛伐克開發者 Silver Zachara 獨立維護。無公司、無創投、無機構資金支援，僅依賴極少量的個人捐贈。社群規模中等（353 stars），外部媒體覆蓋率極低。專案雖已獲得 Microsoft vcpkg 官方收錄，但其長期永續性因缺乏資金與團隊支援而存在不確定性。

---

[^repo]: GitHub. (n.d.). silverqx/TinyORM. Retrieved 2026-09-25, from https://github.com/silverqx/TinyORM
[^license]: GitHub API. (n.d.). Repository license: MIT. Retrieved 2026-09-25, from https://api.github.com/repos/silverqx/TinyORM
[^topics]: GitHub. (n.d.). TinyORM topics: cplusplus20, database, migrations, mysql, orm, postgresql, query-builder, seeder, sqlite. Retrieved 2026-09-25, from https://github.com/silverqx/TinyORM
[^silverqx]: GitHub. (n.d.). silverqx profile. Retrieved 2026-09-25, from https://api.github.com/users/silverqx
[^authors]: GitHub. (n.d.). TinyORM AUTHORS file. Retrieved 2026-09-25, from https://github.com/silverqx/TinyORM/blob/main/AUTHORS
[^donations]: GitHub. (n.d.). TinyORM donations.mdx. Retrieved 2026-09-25, from https://raw.githubusercontent.com/silverqx/TinyORM/main/docs/donations.mdx
[^sponsors]: GitHub. (n.d.). Sponsor silverqx. Retrieved 2026-09-25, from https://github.com/sponsors/silverqx
[^reddit1]: Reddit r/cpp. (2024-02). TinyORM - a modern C++ ORM library. Retrieved 2026-09-25, from https://www.reddit.com/r/cpp/comments/1ata9ro/tinyorm_a_modern_c_orm_library/
[^reddit2]: Reddit r/cpp. (2024-12). TinyORM v0.30.0 released with TinyDrivers MySQL driver. Retrieved 2026-09-25, from https://www.reddit.com/r/cpp/comments/1hu3sjs/tinyorm_v0300_released_with_tinydrivers_mysql/
[^reddit3]: Reddit r/cpp. (2025-04). TinyORM v0.37.0 released. Retrieved 2026-09-25, from https://www.reddit.com/r/cpp/comments/1jm4qbz/tinyorm_v0370_released/
[^vcpkg]: Microsoft. (n.d.). vcpkg port: tinyorm. Retrieved 2026-09-25, from https://github.com/microsoft/vcpkg/tree/master/ports/tinyorm
[^silverqx_repos]: GitHub. (n.d.). silverqx repositories. Retrieved 2026-09-25, from https://github.com/silverqx?tab=repositories
[^docs]: TinyORM. (n.d.). Official documentation. Retrieved 2026-09-25, from https://www.tinyorm.org