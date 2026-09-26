# FOSS Altova XMLSpy 替代方案調查報告

## 背景

Altova XMLSpy 是一套商業 XML 整合開發環境（IDE），提供 XML 編輯、XSLT 轉換與除錯、XPath 評估、XQuery 開發、XML Schema（XSD）視覺化編輯、DTD/Relax NG/Schematron 驗證、XSL-FO 轉換、JSON 編輯、程式碼產生等完整功能[^xmlspy]。本報告旨在調查其自由開源（FOSS）替代方案。

## 重點替代方案

### 1. EditiX XML Editor

EditiX 是基於 Java 的 XML IDE，功能最接近 XMLSpy 的 FOSS 選擇。

- **功能涵蓋**：XML 編輯（即時驗證與輔助輸入）、XSLT 1.0/2.0/3.0 編輯器與除錯器、XPath 支援、XQuery 支援、W3C XML Schema 視覺化編輯器、DTD/Relax NG/Schematron 驗證、XSL-FO 轉換、XML 比對、JSON/JSON Schema 編輯、多檔案搜尋、專案管理、AI 輔助整合。
- **授權**：GPL v3（另提供商業授權版本）
- **平台**：Windows、macOS、Linux（Java 8+）
- **原始碼**：https://github.com/AlexandreBrillant/Editix-xml-editor
- **官方網站**：https://www.editix.com/

### 2. FreeXmlToolkit

FreeXmlToolkit 是最新且最完整的 XMLSpy 替代品，特別提供 XMLSpy 風格的格狀（Grid）與圖形檢視。

- **功能涵蓋**：XML 編輯（語法高亮、自動完成、程式碼折疊）、**Grid/Graphic 檢視**（類 XMLSpy 的表格化結構編輯）、XSD 驗證與視覺化檢視器、XSLT 3.0 轉換（即時預覽）、XPath 與 XQuery 評估、XSD 文件產生、從 Schema 產生範例 XML、Schematron 驗證（含視覺化規則編輯器）、XProc 3.0 管線支援、PDF 產生（XSL-FO）、數位簽章、JSON 編輯與 JSON Schema 驗證、Schema Library 與 XML Catalog 管理、批次處理。
- **授權**：Apache 2.0
- **平台**：Windows 10/11、macOS（Apple Silicon & Intel）、Linux（.deb/.rpm/portable）
- **原始碼**：https://github.com/karlkauc/FreeXmlToolkit
- **官方網站**：https://karlkauc.github.io/FreeXmlToolkit/

### 3. XML Copy Editor

輕量級、快速的有效性驗證 XML 編輯器。

- **功能涵蓋**：XML 文字編輯、DTD/XML Schema/RELAX NG 驗證、XSLT 轉換、XPath 評估、排版美化、語法高亮、程式碼折疊、標籤完成與鎖定、拼字與樣式檢查、內建 XHTML/XSL/DocBook/TEI 支援、Word 文件匯入匯出（僅限 Windows）。
- **授權**：GPL
- **平台**：Windows、macOS、Linux
- **原始碼**：https://sourceforge.net/projects/xml-copy-editor/
- **官方網站**：https://xml-copy-editor.sourceforge.io/

### 4. Eclipse WTP XML Editors and Tools

Eclipse IDE 的 XML 編輯工具組，同樣提供 XSLT 除錯功能。

- **功能涵蓋**：XML 編輯（原始碼與設計檢視）、XML Schema 與 DTD 編輯器、XSL 開發工具（內容輔助、驗證、語法上色、除錯）、XPath 支援、HTML/CSS/JSON/JSP 編輯。
- **授權**：EPL 2.0
- **平台**：跨平台（Eclipse IDE — Windows、macOS、Linux）
- **原始碼**：https://github.com/eclipse-sourceediting/sourceediting
- **官方網站**：https://marketplace.eclipse.org/content/eclipse-xml-editors-and-tools

### 5. VS Code XML Extension（Red Hat）

將 VS Code 打造成具 Schema 感知的 XML 編輯器，適合輕量級日常 XML 工作。

- **功能涵蓋**：Schema 感知的程式碼自動完成、DTD 與 XSD 驗證、XSL 支援、XInclude 支援、XML Catalog、文件格式化、符號高亮與大綱、重新命名、程式碼動作、Schema 快取、XML 最小化、從文法產生 XML、從 XML 產生 Schema、RelaxNG 支援（實驗性）、XML 顏色與參考功能。
- **授權**：EPL 2.0
- **平台**：Windows、macOS、Linux（VS Code 擴充，v0.15+ 不需 Java）
- **原始碼**：https://github.com/redhat-developer/vscode-xml
- **官方網站**：https://marketplace.visualstudio.com/items?itemName=redhat.vscode-xml

### 6. BaseX

高效能 XML 資料庫引擎與 XQuery/XPath 開發環境。

- **功能涵蓋**：XQuery 4.0 處理器（完整支援）、XPath 3.1 支援、XSLT 處理、互動式 GUI（桌面與 Web 版）、客戶端-伺服器架構、支援 XML/HTML/JSON/CSV/文字/二進位資料儲存與查詢、全文檢索、RESTful API。
- **授權**：BSD 3-Clause
- **平台**：Windows、macOS、Linux（Java，JDK 21+）
- **原始碼**：https://github.com/BaseXdb/basex
- **官方網站**：https://basex.org/

### 7. eXist-db

原生 XML 資料庫與 XQuery 應用程式平台。

- **功能涵蓋**：XQuery 3.1 處理器、XSLT 支援、RESTful Web 服務、瀏覽器版 IDE（eXide，含語法上色、程式碼完成、錯誤檢查）、XForms 框架、應用程式平台、全文檢索、套件管理系統。
- **授權**：LGPL
- **平台**：跨平台（Java），提供 Docker 映像檔
- **原始碼**：https://github.com/eXist-db/exist
- **官方網站**：https://exist-db.org/

### 8. Saxon-HE（Home Edition）

XSLT/XQuery 處理的事實標準開源引擎，多數上述工具內部使用 Saxon。

- **功能涵蓋**：XSLT 3.0 處理器、XPath 3.1 處理器、XQuery 3.1 處理器、XML Schema 1.0 驗證、命令列處理、Java/.NET/C/Python/JavaScript API。
- **授權**：Mozilla Public License
- **平台**：Java、.NET、C/C++、Python、PHP、JavaScript（瀏覽器/Node.js）
- **官方網站**：https://www.saxonica.com/

### 9. 其他 CLI 工具

- **xmllint（libxml2）**：XML 解析與驗證（DTD、XSD、Relax NG）、XPath 評估、排版美化。授權 MIT。https://gitlab.gnome.org/GNOME/libxml2
- **XMLStarlet**：命令列 XML 工具集，支援 XPath 查詢、XSLT 轉換、XML 編輯（選取/編輯/刪除/新增節點）。授權 MIT。https://xmlstar.sourceforge.net/

## 功能對照表

| 工具 | XML 編輯 | XSD Schema | XSLT 除錯 | XPath | XQuery | 授權 | 主要優勢 |
|------|----------|------------|------------|-------|--------|------|----------|
| EditiX | ✅ | ✅ 視覺化 | ✅ 除錯器 | ✅ | ✅ | GPLv3 | 最完整的 XMLSpy IDE |
| FreeXmlToolkit | ✅ Grid+Tree | ✅ 視覺化+文件產生 | ✅ 3.0 即時 | ✅ | ✅ | Apache 2.0 | XMLSpy 格狀檢視，現代跨平台 |
| XML Copy Editor | ✅ | ✅ 驗證 | ✅ 基本 | ✅ | ❌ | GPL | 輕量快速 |
| Eclipse WTP | ✅ | ✅ 編輯器 | ✅ 除錯器 | ✅ | ❌ | EPL 2.0 | 成熟的 XSLT 除錯 |
| VS Code XML | ✅ Schema感知 | ✅ 驗證 | ❌ | ❌ | ❌ | EPL 2.0 | 最佳輕量編輯 |
| BaseX | GUI瀏覽 | ❌ | ❌ | ✅ | ✅ 4.0 | BSD 3 | XQuery/資料庫強項 |
| eXist-db | Web IDE | ❌ | ❌ | ✅ | ✅ 3.1 | LGPL | XML 資料庫 + XQuery |
| Saxon-HE | ❌ | ✅ 驗證 | ❌ | ✅ 3.1 | ✅ 3.1 | MPL | XSLT/XQuery 引擎 |

## 建議組合

- **最接近 XMLSpy 的完整替代**：以 **EditiX** 或 **FreeXmlToolkit** 為主 IDE，搭配 **BaseX** 或 **Saxon-HE** 處理 XQuery/XSLT。
- **輕量方案**：**VS Code + Red Hat XML Extension** + **Saxon-HE**（命令列），需要格狀檢視時再開啟 **FreeXmlToolkit**。
- **伺服端 XML/XQuery**：以 **BaseX** 或 **eXist-db** 為資料庫引擎，以 **EditiX** 或 **FreeXmlToolkit** 為開發前端。

## 來源反思

本報告主要基於各專案官方網站與說明文件，以及 osalt.com 與 alternativeto.net 等第三方比對平台的資料。部分較小專案（如 Treebeard）可能存在維護停滯的風險，建議在實際採用前確認專案活躍度。功能對照表僅反映各工具在預設配置下的能力，部分工具可透過外掛或搭配使用擴充功能。

[^xmlspy]: Altova. (n.d.). XMLSpy XML Editor. Retrieved 2026-09-25, from https://www.altova.com/xmlspy-xml-editor
[^editix]: EditiX XML Editor. (n.d.). XML Editor. Retrieved 2026-09-25, from https://www.editix.com/xml-editor.html
[^freetk]: Karl Kauc. (n.d.). FreeXmlToolkit. Retrieved 2026-09-25, from https://karlkauc.github.io/FreeXmlToolkit/
[^xmlcopy]: SourceForge. (n.d.). XML Copy Editor. Retrieved 2026-09-25, from https://xml-copy-editor.sourceforge.io/
[^eclipse]: Eclipse Foundation. (n.d.). Eclipse XML Editors and Tools. Retrieved 2026-09-25, from https://marketplace.eclipse.org/content/eclipse-xml-editors-and-tools
[^vscode]: Red Hat. (n.d.). XML Extension for VS Code. Retrieved 2026-09-25, from https://marketplace.visualstudio.com/items?itemName=redhat.vscode-xml
[^basex]: BaseX GmbH. (n.d.). BaseX XML Database. Retrieved 2026-09-25, from https://basex.org/
[^exist]: eXist-db. (n.d.). eXist-db Native XML Database. Retrieved 2026-09-25, from https://exist-db.org/
[^saxon]: Saxonica. (n.d.). Saxon HE. Retrieved 2026-09-25, from https://www.saxonica.com/
[^osalt]: osalt.com. (n.d.). Altova XMLSpy Alternatives. Retrieved 2026-09-25, from https://www.osalt.com/xmlspy
[^altnet]: AlternativeTo. (n.d.). Altova XMLSpy Alternatives. Retrieved 2026-09-25, from https://alternativeto.net/software/altova-xmlspy/