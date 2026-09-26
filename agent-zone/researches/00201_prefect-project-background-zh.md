# Prefect（PrefectHQ）專案背景調查報告

## 概述

Prefect 是一個開源的工作流程編排（workflow orchestration）框架，主要用 Python 撰寫，讓資料工程師能夠以純 Python 程式碼建立、排程、監控具備生產等級可靠度的資料管線。該專案由 Prefect Technologies, Inc. 維護，總部位於美國華盛頓特區，採遠端優先（remote-first）的工作模式。[^prefect-gh] [^prefect-company]

## 創辦人與創立故事

**Jeremiah Lowin** 是 Prefect 的創辦人暨 CEO。在創立 Prefect 之前，他是 **Apache Airflow 的 PMC（Project Management Committee）成員**，深刻了解既有工作流程編排工具的不足。Prefect 於 **2018 年** 成立，核心理念是讓工作流程編排變得 **Pythonic**——使用簡單的 Python decorator，而非 DSL 或複雜的設定檔。[^why-prefect]

產品的演進歷程：

- **2018–2021（Prefect 1.0）**：引入「功能性編排」（functional orchestration），支援 task mapping 與 Pythonic decorator
- **2022（Prefect 2.0）**：取消工作流程必須寫成明確 DAG 的限制，擁抱原生 Python 控制流程（if/else、while 迴圈）
- **2024（Prefect 3.0）**：開源事件與自動化引擎，導入交易型編排、可攜式執行、相較 v2 減少 90% 執行時開銷
- **2025–2026**：推出 **FastMCP**（開源 MCP server 框架）與 **Prefect Horizon**（企業級 MCP 閘道）；**2026 年 7 月收購 Dagster Labs**[^acquire-dagster] [^why-prefect]

## 募資歷程

| 輪次 | 金額 | 領投方 | 日期 | 說明 |
|------|------|--------|------|------|
| Series A | $1,250 萬 | Index Ventures | 2020 年 | 脫離隱身模式 |
| Series B | $3,500 萬 | Tiger Global Management | 2022 年 5 月 | 擴展平台 |
| Series C | $4,200 萬 | Accel | 2024 年 5 月 | 加速資料/AI/代理工作流程 |

**已知總募資金額：約 $8,950 萬美元**[^series-c]

## 估值

Prefect 為私有公司，未公開揭露估值。考量其由 Tiger Global、Index Ventures、Accel 等頂級創投領投、總計約 $8,950 萬的募資，以及顯著的企業客戶採用率，業界普遍認為其屬於資金充裕的成長階段公司。

## 團隊規模

根據 Built In 網站資料，Prefect 共有 **63 名全職員工**。公司採遠端優先模式，官方辦公室位於華盛頓特區（總部）與紐約市。[^builtin]

## 商業模式：開放核心 + 雲端服務

Prefect 採用典型的**開放核心（Open Source Core）+ 託管雲端（Managed Cloud）** 模式：

- **開源（Apache 2.0）**：Prefect 框架本身為免費、Apache 2.0 授權的 Python 函式庫。同時也開發 **FastMCP**（開源 MCP server 框架）與 **ControlFlow**（代理式 AI 工作流程）
- **Prefect Cloud（付費層級）**[^pricing]：
  - **Hobby** — 免費，2 位使用者，5 個部署，500 分鐘 Serverless 運算
  - **Starter** — $100/月，3 位使用者，20 個部署，75 小時 Serverless
  - **Team** — $100/使用者/月，4-8 位使用者，100 個部署，服務帳號，24 小時稽核日誌
  - **Enterprise** — 客製定價，SSO、RBAC、PrivateLink、99.99% SLA、HIPAA/SOC 2 Type II
- **Prefect Horizon** — 企業級 MCP 閘道產品線

## 社群指標

| 指標 | 數值 |
|------|------|
| GitHub Stars | ~23.9k |
| GitHub Forks | ~2.5k |
| 提交數（Commits） | ~22,111 |
| 社群成員 | 25,000+（Slack/docs 社群）|
| 每分鐘編排的工作流程數 | 100K+ |
| 每月自動化資料任務數 | 2 億+ |
| PyPI 月下載量 | ~670 萬+ |
| 授權條款 | Apache 2.0 |

[^prefect-gh]

## 知名客戶與使用者

**企業客戶：**
- Progressive Insurance（財富 50 強）
- Humana
- Blackstone
- JPMorgan Chase
- Adobe
- Cisco
- CalPERS

**科技與消費品牌：**
- Cash App（Block/Square）
- Meta
- Square
- NASA
- WHOOP（生產事故減少 75%）
- Ramp
- Snorkel AI
- Barstool Sports
- Flatiron Health（癌症研究資料管線）
- Washington Nationals（MLB 球隊）
- Equinox
- Foursquare
- Ashby
- Eight Sleep
- CoinList

[^pricing]

## 重大近期事件

### 收購 Dagster Labs（2026 年 7 月）

2026 年 7 月 13 日，Prefect 宣布收購 **Dagster Labs**，包含 Dagster 產品、程式碼庫、客戶關係以及多位 Dagster 團隊成員。Dagster 將保留其名稱與開源授權，Dagster+ 商業產品繼續獲得支援。Dagster 創辦人 Nick Schrock 與 CEO Pete Hunt 加入 Prefect。收購後，Prefect 表示過去一年已在盈利且快速成長的基礎上運營。[^acquire-dagster]

合併後 Prefect 擁有三大開源產品家族：
1. **Dagster** — 成果層（定義與驗證資料/資產）
2. **Prefect** — 執行層（可靠執行複雜工作流程）
3. **FastMCP** — 存取層（管理 AI 代理的 MCP 存取）

### 其他產品發布

- **2026 年 1 月**：推出 **Prefect Horizon** — 企業 MCP 閘道平台
- **2026 年 5 月**：擴展 Snowflake 整合，登上 Snowflake Marketplace
- **2026 年 8 月**：**FastMCP 4** 正式發布，支援 MCP 2026-07-28 協定

## 總結

Prefect 是由 Apache Airflow PMC 成員 Jeremiah Lowin 於 2018 年創立的工作流程編排公司，至今累計募資約 $8,950 萬美元，投資人包括 Index Ventures、Tiger Global Management、Accel 等頂級創投。公司擁有 63 名員工、25,000+ 社群成員、23.9k GitHub Stars，客戶涵蓋 NASA、Meta、JPMorgan Chase 等知名組織。2026 年收購 Dagster Labs 後，產品組合擴展為涵蓋成果定義（Dagster）、工作流程執行（Prefect）、AI 代理存取控管（FastMCP）三大面向的完整平台。

---

## 參考資料

[^prefect-gh]: PrefectHQ. (n.d.). Prefect — GitHub Repository. Retrieved 2026-09-25, from https://github.com/PrefectHQ/prefect

[^prefect-company]: Prefect Technologies, Inc. (n.d.). About Prefect — Company & Mission. Retrieved 2026-09-25, from https://www.prefect.io/company

[^why-prefect]: Prefect Technologies, Inc. (n.d.). Why Prefect — Documentation. Retrieved 2026-09-25, from https://docs.prefect.io/v3/get-started/why-prefect

[^pricing]: Prefect Technologies, Inc. (n.d.). Prefect Cloud Pricing. Retrieved 2026-09-25, from https://www.prefect.io/pricing

[^acquire-dagster]: Lowin, J. (2026-07-13). Prefect Acquires Dagster Labs. Retrieved 2026-09-25, from https://www.prefect.io/prefect-acquires-dagster

[^series-c]: Prefect Technologies, Inc. (2024-05-23). Prefect Raises $42M Series C Led by Accel. Retrieved 2026-09-25, from https://www.prefect.io/blog

[^builtin]: Built In. (n.d.). Prefect Company Profile. Retrieved 2026-09-25, from https://builtin.com/company/prefect