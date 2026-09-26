# 調查 OGC IndoorGML 1.1 (19-011r4) 規格原始碼

## 摘要

本報告調查 OGC 文件 19-011r4（IndoorGML 1.1 標準）在 GitHub 上是否存在原始碼儲存庫。結果顯示，**該標準本身無專屬 GitHub 原始碼儲存庫**，但其相關的標準工作組儲存庫、XSD 綱要與可執行測試套件分別託管於 GitHub 及 OGC 官方綱要伺服器上。

## 調查結果

### 1. IndoorGML 1.1 (19-011r4) XSD 綱要位置

IndoorGML 1.1 的正式 XML 綱要（XSD）檔案**不是以 GitHub 儲存庫方式維護**，而是直接發布於 OGC 官方綱要伺服器[^ogcschemas]：

**https://schemas.opengis.net/indoorgml/1.1/**

該目錄包含：
- `indoorgmlcore.xsd`（約 19KB）— 核心模組
- `indoorgmlnavi.xsd`（約 13KB）— 導航模組

另有一個 ZIP 封存檔可供下載：`indoorgml-1_1_0.zip`。

伺服器上的 `ReadMe.txt` 記載了來源說明[^readme]：

> **2023-04-11 Ki-Joune Li** — *v1.1: Added indoorGML 1.1.0 as indoorGML/1.1 from OGC 19-011r4*

### 2. GitHub 上的相關儲存庫

以下為 OGC 組織旗下與 IndoorGML 相關的 GitHub 儲存庫：

| 儲存庫 | 狀態 | 用途 |
|--------|------|------|
| [opengeospatial/IndoorGML-SWG](https://github.com/opengeospatial/IndoorGML-SWG)[^swg] | 活躍 | IndoorGML 標準工作組 — 目前專注開發 **IndoorGML 2.0**（Part I 已發布為 OGC 22-045r5，Part II 編碼綱要仍在起草中） |
| [opengeospatial/ets-indoorgml10](https://github.com/opengeospatial/ets-indoorgml10)[^ets] | 歷史 | IndoorGML 1.0 可執行測試套件（用於符合性測試） |
| [opengeospatial/www.indoorgml.net](https://github.com/opengeospatial/www.indoorgml.net)[^www] | 靜態 | www.indoorgml.net 網站原始碼 |

重點說明：
- **IndoorGML-SWG** 儲存庫**不包含** 19-011r4（v1.1）的原始碼或綱要，其主要內容為工作組會議記錄、簡報，以及 IndoorGML 2.0 的開發文件[^swg]。
- **ets-indoorgml10** 僅涵蓋 IndoorGML 1.0 版本的符合性測試，並非 1.1 版。

### 3. 結論

OGC IndoorGML 1.1 規格（19-011r4）**不存在**一個專門的 GitHub 儲存庫來追蹤其綱要原始碼或文件原始碼。其 XSD 綱要直接以 OGC 官方綱要伺服器作為發布管道。開發者若需取得 IndoorGML 1.1 綱要，應直接自上述 OGC 綱要伺服器擷取；若需追蹤 IndoorGML 標準的未來發展（2.0 版），則應關注 IndoorGML-SWG 儲存庫。

## 參考文獻

[^swg]: Open Geospatial Consortium. (n.d.). IndoorGML-SWG. Retrieved 2026-09-25, from https://github.com/opengeospatial/IndoorGML-SWG
[^ets]: Open Geospatial Consortium. (n.d.). ets-indoorgml10. Retrieved 2026-09-25, from https://github.com/opengeospatial/ets-indoorgml10
[^www]: Open Geospatial Consortium. (n.d.). www.indoorgml.net. Retrieved 2026-09-25, from https://github.com/opengeospatial/www.indoorgml.net
[^ogcschemas]: Open Geospatial Consortium. (n.d.). OGC Schema Server — indoorGML/1.1/. Retrieved 2026-09-25, from https://schemas.opengis.net/indoorgml/1.1/
[^readme]: Li, K.-J. (2023-04-11). ReadMe.txt at schemas.opengis.net/indoorgml/. Retrieved 2026-09-25, from https://schemas.opengis.net/indoorgml/ReadMe.txt