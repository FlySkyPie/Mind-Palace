# getml/sqlgen 專案背景調查報告

## 概述

`sqlgen` 是一個以 C++20 撰寫、基於反射（reflection）機制的型別安全 ORM 與 SQL 查詢產生器，語法風格受 SQLAlchemy 和 Diesel 啟發，支援 PostgreSQL、SQLite、MySQL/MariaDB、DuckDB 等後端。該專案由德國公司 **Code17 GmbH**（品牌名 **getML**）開發與維護，以 MIT 授權釋出。[^sqlgen-repo]

## 母公司：Code17 GmbH (getML)

### 基本資料

- **公司全名**：Code17 GmbH（前身為 **The SQLNet Company GmbH**）
- **成立時間**：2017 年
- **總部地點**：德國萊比錫（Leipzig, Saxony），另於慕尼黑設有辦公室
- **公司規模**：11–50 名員工
- **產業類別**：IT 服務與資訊顧問、資料基礎設施與分析
- **商業登記**：Amtsgericht Leipzig HRB 34030
- **原始股本**：€25,000（2017 年設立時）
- **公司官網**：https://code17.io、https://getml.com
- **主要客戶**：PwC、AOK、Volkswagen 等[^northdata] [^linkedin-company]

### 商業模式

Freemium 模式：
- **Community Edition**：開源（ELv2 授權），可透過 `pip install getml` 安裝，提供 FastProp 演算法
- **Professional & Enterprise Edition**：付費版本，提供進階 ML 演算法（Relboost、Multirel、Fastboost、RelMT）、生產環境使用權、保證回應時間、教育訓練、地端/雲端部署等功能[^getml-enterprise]

### 開源產品生態系

| 產品 | 說明 | GitHub Stars | Forks |
|---|---|---|---|
| **getml-community** | 自動化特徵工程平台（關聯式資料與時間序列） | ~244 | ~20 |
| **reflect-cpp** | C++20 序列化/反序列化/驗證函式庫，支援 19+ 格式（JSON、Avro、CBOR、Parquet、XML、YAML 等） | ~1,937 | ~194 |
| **sqlgen** | 型別安全的 ORM 與 SQL 查詢產生器 | ~196 | ~22 |
| **deigma** | Python 型別安全樣板引擎 | ~1 | ~0 |

## 創辦人與核心團隊

### Dr. Patrick Urbanke（劉自成 / GitHub: liuzicheng1987）

- **學歷**：德國哥廷根大學（Georg-August-Universität Göttingen）博士，博士論文題目為《Essays on predictive analytics in e-commerce》（2016）
- **學術發表**：多篇 ICIS 論文，包括《Predicting product returns in e-commerce: the contribution of Mahalanobis feature extraction》（2015）與《A customized and interpretable deep neural network for high-dimensional business data》（2017）
- **經歷**：曾在亞洲生活 10 年，因此使用中文名劉自成；持有關聯式學習（relational learning）專利[^patent]
- **角色**：getML、reflect-cpp、sqlgen 的主要作者（main author）
- **所在地**：德國慕尼黑
- **語言**：英語、中文、德語
- **LinkedIn**：https://www.linkedin.com/in/patrick-urbanke/[^github-patrick] [^scholar-patrick]

### Manuel Bellersen

- **學歷**：
  - HTWK Leipzig（萊比錫應用科技大學）— 資訊科學（Informatik），2008–2011
  - HTW Berlin（柏林應用科技大學）— 媒體資訊科學（Medieninformatik），2011–2013，專攻遊戲技術與互動系統
- **經歷**：
  - Ypsilon.Net AG — Senior Lead Software Engineer（2014–2020，約 6 年）
  - EWERK — Software Engineer（2020–2023，約 3.5 年）
- **現職**：Code17 GmbH Data Scientist（2024 年 1 月至今）
- **技術棧**：C++、C、Python、Haskell、Unity、Godot、PostgreSQL、Docker 等
- **所在地**：德國路德城維滕貝格（Lutherstadt Wittenberg）
- **LinkedIn**：https://www.linkedin.com/in/manuel-bellersen/[^xing-manuel]

### Alexander Uhlig

- **角色**：Code17 GmbH 共同創辦人暨常務董事（Managing Director）
- **備註**：根據 North Data 2026 年 6 月的紀錄，Dr. Patrick Urbanke 已不再擔任常務董事[^northdata]

## 資金與創投

**目前沒有任何公開的募資紀錄。**

經過多方查證（Crunchbase、PitchBook、North Data、新聞搜尋），Code17 GmbH 並未揭露任何外部創投融資：

- Crunchbase 頁面回傳 403 無法存取
- North Data 顯示有「2 位已知活躍股東」（付費牆後），但極可能是創辦人本身而非外部投資人
- LinkedIn 公司頁面標註為「Privately Held」
- 無任何天使輪、種子輪、Series A 或任何機構投資的新聞報導

綜合判斷，**Code17 GmbH 極可能為自籌資金（bootstrapped）公司**，依靠企業客戶服務與 getML 產品收入有機成長。[^linkedin-company] [^northdata]

LinkedIn 公司標語描述：「我們建造兩件事：關鍵系統，以及營運這些系統的團隊。我們的程式碼驅動 DAX（德國股指）30% 的公司，並交易兆瓦級的能源。」這也暗示公司主要以服務收入營運。[^linkedin-company]

## 社群規模

- GitHub 組織（github.com/getml）：總計 14 個公開儲存庫
- 組織總星數：約 2,400+
- reflect-cpp 為組織內最受歡迎專案（~1,937 stars），sqlgen 其次（~196 stars）
- getml-community 主儲存庫有 2,244 次提交

## 開發脈絡

reflect-cpp 最初是為 getML 產品內部開發的序列化基礎設施，sqlgen 則建構在 reflect-cpp 之上。這兩個函式庫都是從 getML 商業產品中萃取出來並開源釋出，reflect-cpp 支援 C++20 與 C++26 的標準化反射機制，是目前少數已運用 C++26 反射的函式庫之一。[^reflect-cpp]

sqlgen 的文件與技術細節請見：https://getml.github.io/sqlgen/

[^sqlgen-repo]: getML. (n.d.). sqlgen: Type-safe ORM and SQL query generator for C++20. Retrieved 2026-09-25, from https://github.com/getml/sqlgen
[^getml-enterprise]: getML. (n.d.). getML Enterprise Edition. Retrieved 2026-09-25, from https://getml.com/latest/enterprise/
[^northdata]: North Data. (n.d.). Code17 GmbH, Leipzig. Retrieved 2026-09-25, from https://www.northdata.de/Code17+GmbH
[^linkedin-company]: LinkedIn. (n.d.). Code17 GmbH company profile. Retrieved 2026-09-25, from https://www.linkedin.com/company/code17-gmbh/
[^github-patrick]: GitHub. (n.d.). liuzicheng1987 profile. Retrieved 2026-09-25, from https://github.com/liuzicheng1987
[^scholar-patrick]: Google Scholar. (n.d.). Patrick Urbanke publications. Retrieved 2026-09-25, from https://scholar.google.com/scholar?q=%22Patrick+Urbanke%22
[^patent]: Google Patents. (2021). Relational feature learning / automated feature engineering. Retrieved 2026-09-25, from https://patents.google.com/patent/US20210056492A1/en
[^xing-manuel]: XING. (n.d.). Manuel Bellersen profile. Retrieved 2026-09-25, from https://www.xing.com/profile/Manuel_Bellersen
[^reflect-cpp]: getML. (n.d.). reflect-cpp: C++20 reflection/serialization library. Retrieved 2026-09-25, from https://github.com/getml/reflect-cpp