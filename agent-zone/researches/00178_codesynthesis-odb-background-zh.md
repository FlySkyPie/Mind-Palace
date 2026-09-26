# Code Synthesis (codesynthesis.com) 與 ODB 專案背景調查

## 摘要

本報告調查 Code Synthesis 公司及其旗艦產品 ODB（C++ ORM 框架）的背景，涵蓋公司沿革、創辦人與團隊、資金來源、商業模式、社群規模、知名用戶以及專案現狀。

## 1. 公司概覽

**Code Synthesis** 是一家總部位於南非開普敦的系統軟體開發公司，成立於 2005 年[^cs-website]。法律實體名稱為 **Code Synthesis Tools CC**（Close Corporation，南非封閉型公司），在歐盟另設有分支部門 **Code Synthesis OU**，地址為 Sepapaja 6, Tallinn 15551, Estonia[^cs-contact]。公司規模極小，截至 2026 年 7 月僅有 **3 名員工**[^tracxn]。

公司專注領域包括：建構系統、物件持久化、領域特定語言（DSL）及其映射、編譯器設計、程式碼生成，以及 C++ 程式語言的源碼到源碼轉換[^cs-website]。所有產品均為開源，但透過雙重授權模式獲利。

## 2. 創辦人與團隊

### Boris Kolpackov — 創辦人兼核心開發者

Boris Kolpackov 是 Code Synthesis 的創辦人，自稱為「Founder & CHO（Chief Hacking Officer）」[^cppcon2014]。他是 **ODB、XSD、XSD/e、build2** 等所有公司核心產品的設計者與主要實作者。其個人部落格「A Sense of Design」從 2006 年活躍至 2018 年[^boris-blog]。

背景亮點：
- 位於南非開普敦
- GitHub 帳號：[@boris-kolpackov](https://github.com/boris-kolpackov)（57 位追隨者）[^boris-github]
- 同時是 **build2**（現代 C++ 建構工具鏈）與 **cppget.org**（開源 C++ 套件倉庫，595+ 套件）的創建者
- 曾在 CppCon 2014 接受 Microsoft Channel 9 專訪[^cppcon2014]

### 其他團隊成員

根據 GitHub 組織頁面與 Tracxn 資料，除 Boris 外，另一位公開可識別的成員是 **Karen Arutyunov**（GitHub: [@karen-arutyunov](https://github.com/karen-arutyunov)）[^codesynthesis-gh-org]。公司官網曾招聘開普敦地區的 C++ 軟體工程師，顯示有擴張需求但團隊仍維持極小規模[^cs-jobs]。

## 3. 資金來源

**完全自力更生（Bootstrapped），零外部融資。[^tracxn][^crunchbase]**

- Tracxn 將 Code Synthesis 標記為 **「Unfunded」**（未獲融資）狀態
- Crunchbase 同樣顯示無任何融資紀錄
- 公司自 2005 年起持續營運 **21 年以上**，完全依靠商業授權收入維持

## 4. 商業模式

Code Synthesis 採用 **雙重授權（Dual-Licensing）模式**，提供多個層級[^cs-odb-license]：

### 授權層級

| 授權類型 | 費用 | 適用對象 |
|----------|------|----------|
| **GNU GPL v2** | 免費 | 開源專案，要求衍生程式碼以 GPL 釋出 |
| **NCUEL**（非商業用途與評估授權） | 免費 | Oracle/SQL Server 商業版的非商業/評估用途 |
| **FPL**（免費專有授權） | 免費 | 小型專案（產生程式碼 ≤10,000 行，約 10–20 個持久化物件） |
| **CPL**（商業專有授權） | 付費（<$1,500/€1,500） | 閉源商業使用，包含法律保證與賠償、商業級技術支援 |

### 定價

- 最便宜的 CPL 授權低於 **$1,500/€1,500**[^cs-odb-license]
- 定價基於產生的程式碼行數（SLOC），不公開透明定價（要求客戶來信詢價，以獲得初步溝通機會）
- 公司承諾 **同一價格**，不進行價格歧視[^cs-odb-license]

### 其他收入來源

- **優先技術支援合約** — 保證回應時間，包含錯誤修復或提供替代方案
- **客製化開發服務** — XML 詞彙設計、編譯器前後端開發、自訂語言映射[^cs-support]

### 產品組合

| 產品 | 說明 | 最新版本 |
|------|------|----------|
| **ODB** | C++ ORM 系統（旗艦產品） | 2.6.0（2026-07-28） |
| **CodeSynthesis XSD** | XML Schema 到 C++ 資料綁定編譯器 | 4.2.0（2023-10-05） |
| **CodeSynthesis XSD/e** | 嵌入式/行動裝置輕量版 XSD | 3.4.0（2025-02-11） |
| **build2** | 現代 C/C++ 建構工具鏈與套件管理器（MIT 授權） | 0.16.0（2023-07-04） |
| **CLI** | 命令列介面編譯器（for C++） | — |

## 5. 社群規模

### GitHub 指標（截至 2026 年 9 月）[^codesynthesis-gh-org]

| 倉庫 | Stars | Forks | 提交數 |
|------|-------|-------|--------|
| **odb** | 63 | 6 | 5,139 |
| **xsd** | 10 | 3 | — |
| **xsde** | 4 | 0 | — |
| **build2**（獨立組織） | 681 | — | — |

> 注意：ODB 的 GitHub 星數偏低（63），但官方主要倉庫自託管於 `git.codesynthesis.com`，GitHub 僅為鏡像，因此 GitHub 星數不能完全反映實際用戶規模。

### 下載量與社群支持

- 官方網站稱 ODB **「每週下載量達數百次」**[^cs-odb]
- 提供多個郵件列表（odb-users、xsd-users 等），均附可搜尋存檔
- Wiki 位於 `wiki.codesynthesis.com`

## 6. 知名用戶與案例

### ODB 企業客戶[^cs-odb-customers]

| 公司 | 產業 | 資料庫 |
|------|------|--------|
| **Symantec Corporation** | 網路安全 | 多種資料庫 |
| **Intel Corporation**（以色列分部） | 半導體 | 多種資料庫 |
| **Lockheed Martin** | 航太/國防 | SQLite |
| **BAE Systems Detica** | 國防/情報 | PostgreSQL |
| **Fraunhofer-Gesellschaft** | 研究 | SQLite |
| **ABB AG** | 工業自動化 | SQLite（VxWorks 嵌入式） |
| **Check Point Software** | 網路安全 | SQLite |
| **Sandia National Laboratories** | 國家安全 | ODB 用戶（見官網 quote） |
| **Liquid Capital Markets** | 金融/做市 | PostgreSQL（取代 Versant VOD） |
| **Quortus Ltd** | 電信 | MySQL |
| **DATADVANCE**（EADS 合資企業） | 航太 | SQLite |
| **KDE Project**（Plasma Media Center） | 開源 | ODB + Qt/SQLite |

### XSD 企業客戶（部分）[^cs-xsd-customers]

**Boeing、Lockheed Martin、Siemens AG、Barclays Bank、Deutsche Börse Group、Ericsson、Philips、Sony（多個部門）、Qualcomm、Thales Group、SAAB、Bombardier、Fermilab、Novell、Computer Associates、Sony Online Entertainment、Konami Gaming** 等。

### 著名案例[^cs-xsd-customers]

- **Galileo 衛星導航系統**（歐洲太空總署）— 透過 VEGA 和 Terma 使用 XSD
- **Fermilab NOvA 微中子實驗** — 約 450 節點的資料擷取系統使用 XSD
- **Complete Genomics** — DNA 定序資料格式
- **Lockheed Martin** — Multiple Kill Vehicle 模擬、電子戰系統

## 7. 近期版本與專案狀態

**專案極度活躍，絕非棄維護狀態。[^cs-website]**

| 日期 | 版本 | 亮點 |
|------|------|------|
| **2026-07-28** | ODB 2.6.0 | C++11 程式碼庫現代化、`std::optional` 映射、直接物件關係載入、SQLite WAL 模式 |
| **2025-02-11** | XSD/e 3.4.0 | 維護版本 |
| **2025-01-14** | ODB 2.5.0 | 批次操作、PostgreSQL 預存程序、SQLite 多資料庫附加 |
| **2023-10-05** | XSD 4.2.0 | 最後支援 C++98 的版本；預設改為 C++11 |
| **2023-07-04** | build2 0.16.0 | 系統套件管理器整合 |

## 8. 競品與替代方案

| 替代方案 | 類型 | 資料庫支援 | 說明 |
|----------|------|------------|------|
| **sqlpp11** | 類型安全 SQL 嵌入式 DSL | MySQL、PostgreSQL、SQLite | 編譯時 SQL 驗證；大量模板樣板碼 |
| **sqlite-orm** | 現代標頭檔式 ORM | 僅 SQLite | 流暢 API；2,670 Stars |
| **hiberlite** | Hibernate 風格 ORM | 僅 SQLite | 輕量（<3K 行） |
| **SOCI** | 資料庫存取層 | 多種資料庫 | 非完整 ORM |
| **ormpp** | 現代 C++17 ORM | MySQL、PostgreSQL、SQLite | 較新、較小眾 |

**ODB 的關鍵差異化優勢**：
- **唯一真正跨資料庫的 C++ ORM**（SQLite、PostgreSQL、MySQL、Oracle、SQL Server 全部支援）
- **程式碼生成方式**（不同於模板元程式設計）— 使用真正的 C++ 解析器（GCC 外掛），而非標頭檔掃描
- **成熟程式碼庫**（15+ 年開發、5,139 次提交）
- **商業支援與法律保障** — 對國防、航太等企業客戶至關重要
- **資料庫 Schema 演化支援**
- **Boost 與 Qt 設定檔（Profiles）**

## 9. 總結

Code Synthesis 是一家**小型、自力更生、持續獲利**的公司，已可持續營運超過 **21 年**，完全無須外部資金。創辦人 Boris Kolpackov 是唯一的公開核心人物，帶領一個 3 人團隊同時維護多個 C++ 開源專案。其雙重授權模式讓個人開發者可免費使用 GPL 版本，而企業客戶則支付商業授權費用獲得法律保護與技術支援。ODB 2.6.0 於 2026 年 7 月發布，顯示專案開發仍然非常活躍。

## 參考資料

[^cs-website]: Code Synthesis. (n.d.). Code Synthesis — System Software Development Company. Retrieved 2026-09-25, from https://www.codesynthesis.com/

[^cs-contact]: Code Synthesis. (n.d.). Contact. Retrieved 2026-09-25, from https://www.codesynthesis.com/contact/

[^cs-odb]: Code Synthesis. (n.d.). ODB — C++ ORM. Retrieved 2026-09-25, from https://www.codesynthesis.com/products/odb/

[^cs-odb-license]: Code Synthesis. (n.d.). ODB License. Retrieved 2026-09-25, from https://www.codesynthesis.com/products/odb/license.xhtml

[^cs-odb-customers]: Code Synthesis. (n.d.). ODB Customers. Retrieved 2026-09-25, from https://www.codesynthesis.com/products/odb/customers.xhtml

[^cs-xsd-customers]: Code Synthesis. (n.d.). XSD Customers. Retrieved 2026-09-25, from https://www.codesynthesis.com/products/xsd/customers.xhtml

[^cs-support]: Code Synthesis. (n.d.). Support. Retrieved 2026-09-25, from https://www.codesynthesis.com/support/

[^cs-jobs]: Code Synthesis. (n.d.). Jobs. Retrieved 2026-09-25, from https://www.codesynthesis.com/company/jobs.xhtml

[^boris-blog]: Kolpackov, B. (n.d.). About. A Sense of Design. Retrieved 2026-09-25, from https://www.codesynthesis.com/~boris/blog/about/

[^boris-github]: boris-kolpackov. (n.d.). GitHub Profile. Retrieved 2026-09-25, from https://github.com/boris-kolpackov

[^codesynthesis-gh-org]: codesynthesis-com. (n.d.). GitHub Organization. Retrieved 2026-09-25, from https://github.com/codesynthesis-com

[^cppcon2014]: Microsoft Channel 9. (2014). CppCon 2014: Boris Kolpackov Interview. Retrieved 2026-09-25, from https://learn.microsoft.com/en-us/shows/goingnative-cppcon-2014-interviews/cppcon-2014-boris-kolpackov

[^tracxn]: Tracxn. (2026). Code Synthesis Company Profile. Retrieved 2026-09-25, from https://tracxn.com/d/companies/code-synthesis/

[^crunchbase]: Crunchbase. (n.d.). Code Synthesis. Retrieved 2026-09-25, from https://www.crunchbase.com/organization/code-synthesis