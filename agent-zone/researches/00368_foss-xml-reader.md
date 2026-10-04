# FOSS XML 讀取器與編輯器綜合調查

## 概述

XML（Extensible Markup Language）是一種廣泛使用的標記式語言，用於結構化資料儲存與交換。本報告調查各程式語言與各平台上的自由開源（FOSS, Free and Open Source Software）XML 讀取器/剖析器/編輯器。

## XML 剖析模式

XML 剖析方式主要分為以下幾種：

| 模式 | 說明 | 記憶體使用 | 適用場景 |
|---|---|---|---|
| **DOM (Document Object Model)** | 將整份文件載入記憶體建立樹狀結構 | 高（文件大小 3-5 倍） | 隨機存取、修改、XPath 查詢 |
| **SAX (Simple API for XML)** | 事件驅動串流式推送剖析；剖析器呼叫註冊回呼 | 極低（與深度成正比） | 大型文件、順序處理 |
| **StAX (Streaming API for XML)** | 拉取式串流；應用程式按需要求下一個 Token | 低 | 大型文件、狀態依存處理 |
| **VTD-XML** | 非提取式剖析，使用 64 位元描述元；混合串流與隨機存取 | 低（文件大小 1.3-1.5 倍） | 高效能、大型文件、XPath |

## 各語言 XML 函式庫

### C/C++

| 函式庫 | 類型 | 授權 | 最新版本 | 維護狀態 |
|---|---|---|---|---|
| **Expat (libexpat)** | 串流/SAX 式 | MIT | 2.8.5 (2026-09-22) | ✅ 積極維護（慕尼黑市資助） |
| **libxml2** | DOM + SAX + 串流 | MIT | 2.15.4 (2026-09-04) | ⚠️ 維護者於 2025-09 退出；分支 libxml2-ee 採 AGPL |
| **Apache Xerces C++** | DOM + SAX | Apache 2.0 | 3.3.0 (2024-10-14) | ✅ 積極維護 |
| **VTD-XML** | 非提取式 VTD | GPL/Proprietary | 2.13_4 (2017) | ❌ 2017 年後停止更新 |

### Python

| 函式庫 | 類型 | 授權 | 說明 |
|---|---|---|---|
| **xml.etree.ElementTree** | DOM 式 (ET API) | PSFL (stdlib) | Python 標準函式庫；輕量，含 C 加速版 `cElementTree` |
| **lxml** | 完整功能 (ET + DOM + SAX) | BSD | 基於 libxml2/libxslt 的 Python 綁定；最快 Python 方案；支援 XPath, XSLT, Schema, RelaxNG |
| **xml.dom.minidom** | DOM | PSFL (stdlib) | 標準函式庫內的最小 DOM 實作 |
| **xml.sax** | SAX 事件驅動 | PSFL (stdlib) | Python 標準函式庫 SAX 實作，基於 Expat |
| **xml.parsers.expat** | SAX 式 (C 綁定) | PSFL (stdlib) | Expat C 函式庫的快速綁定 |

### Java

| 函式庫 | 類型 | 授權 | 最新版本 | 維護狀態 |
|---|---|---|---|---|
| **Apache Xerces2 Java** | DOM + SAX + StAX | Apache 2.0 | 2.12.2 (2022-01-24) | ✅ 維護中 |
| **StAX (JSR-173)** | 拉取式串流 | Java stdlib | 隨 JDK 捆綁 | ✅ Java 標準一部分 |
| **JAXB** | XML-to-Java 綁定 | 多種 | 持續更新 | ✅ XML 資料綁定 |
| **Apache XMLBeans** | XML-to-Java 綁定 | Apache 2.0 | 5.3.0 (2024-12-13) | ✅ 積極維護 |
| **JDOM** | DOM 式 (Java 導向) | Custom | 穩定 | ⚠️ 活動量低 |
| **Dom4j** | DOM 式 | BSD | 穩定 | ⚠️ 活動量低 |

### JavaScript/Node.js

| 函式庫 | 類型 | 授權 | 說明 |
|---|---|---|---|
| **node-xml (libxml2 binding)** | DOM 式 | MIT | Node.js 的 C 綁定 |
| **xml2js** | XML 轉 JSON | MIT | 將 XML 轉換為 JSON；最受歡迎的 npm XML 剖析器 |
| **SaxonJS** | XSLT/XPath | Custom | 來自 Saxonica |
| **fast-xml-parser** | 串流 | MIT | 基於 Rust/WASM，非常快速 |

### Rust

| 函式庫 | 類型 | 授權 | 說明 |
|---|---|---|---|
| **quick-xml** | 串流/事件式 | MIT | 純 Rust，非常快速，良好維護 |
| **serde-xml** | XML 序列化/反序列化 | MIT | 可與 serde 框架搭配使用 |
| **xmlparser** | 串流 | MIT | 最小拉取式剖析器 |

### Go

| 函式庫 | 類型 | 授權 | 說明 |
|---|---|---|---|
| **encoding/xml** | XML 剖析器 | Go stdlib | Go 標準函式庫一部分 |
| **libxml2 bindings (via CGO)** | DOM 式 | MIT | libxml2 的 Go 綁定 |

### PHP

| 函式庫 | 類型 | 授權 | 說明 |
|---|---|---|---|
| **SimpleXML** | DOM 式 | PHP License | PHP 自 PHP 5 起捆綁；物件導向 XML DOM |
| **PHP XML Expat-based** | 事件驅動 | PHP License | PHP 原生 SAX 剖析器 |
| **DOMDocument (PHP)** | DOM | PHP License | PHP 原生 DOM 實作 |

### Ruby

| 函式庫 | 類型 | 授權 | 說明 |
|---|---|---|---|
| **Nokogiri** | DOM + XPath | MIT | libxml2/libxslt 的 Ruby 綁定；最受歡迎 |
| **REXML** | 純 Ruby DOM | BSD | 輕量，純 Ruby |
| **LibXML-Ruby** | DOM 綁定 | MIT | libxml2 的 Ruby 綁定 |

### Perl

| 函式庫 | 類型 | 授權 | 說明 |
|---|---|---|---|
| **XML::LibXML** | DOM + SAX | MIT | libxml2 的 Perl 綁定 |
| **XML::Xerces** | DOM + SAX | Apache 2.0 | Apache Xerces C++ 的 Perl 綁定 |
| **XML::Parser** | 事件驅動 (Expat) | Perl License | 基於 Expat |

## CLI（命令列）XML 工具

| 工具 | 說明 | 基於 | 授權 | 維護狀態 |
|---|---|---|---|---|
| **xmllint** | 驗證、格式化、查詢 XML 檔案 | libxml2 | MIT | ⚠️ 跟隨 libxml2 狀態 |
| **xmlstarlet** | 完整 CLI XML 工具組：查詢、編輯、驗證、轉換、格式化 | libxml2 + libxslt | MIT | ❌ 最後版本 1.6.1 (2014-08-09) |
| **xpath** (來自 libxml2) | XPath 評估 CLI | libxml2 | MIT | 隨 libxml2 捆綁 |
| **saxon (CLI)** | XSLT/XQuery/XPath 處理 | Saxon engine | MPL 2.0 | ✅ 積極維護 |

### CLI 範例

**xmllint**（來自 libxml2）：
```
xmllint --valid --noout file.xml             # 驗證
xmllint --format file.xml                    # 格式化輸出
xmllint --xpath "//element" file.xml         # XPath 查詢
xmllint --schema schema.xsd file.xml         # XSD 驗證
```

**xmlstarlet**：
```
xmlstarlet sel -t -v "//element" file.xml          # 選取/查詢
xmlstarlet ed -u "//element" -v "new" file.xml      # 原地編輯
xmlstarlet val -e -s schema.xsd file.xml            # 驗證
xmlstarlet fo file.xml                               # 格式化
```

## GUI（圖形介面）XML 編輯器

### 1. XML Notepad (Microsoft)

- **說明：** 微軟發佈的開源 XML 編輯器，以 C# 撰寫，使用 .NET Framework，MIT 授權
- **特色：** XML Schema 感知 IntelliSense、XPath 查詢、XInclude 支援、XSLT 轉換、CSV/JSON/HTML 匯入轉換、XML diff、樹狀與文字雙重檢視、即時 XML Schema 驗證
- **最新版本：** 2.9.0.22 (2026-06-08)
- **平台：** 僅 Windows
- **GitHub：** https://github.com/microsoft/XmlNotepad

### 2. QXmlEdit

- **說明：** 基於 Qt 的多平台 XML 編輯器，少數圖形化 XSD 檢視器之一
- **特色：** 階層式 XML 元素檢視、快速導航、分割大型 XML 檔案、XPath 搜尋、XSD 檢視器、欄位檢視、地圖檢視、XML/XSD diff、程式碼片段、資料匿名化、SCXML 編輯器
- **最新版本：** 0.9.18 (2023-01)
- **授權：** GNU LGPL v2
- **平台：** Linux、macOS、Windows
- **GitHub：** https://github.com/lbellonda/qxmledit

### 3. Notepad++

- **說明：** 自由原始碼編輯器，支援 XML 語法突顯、摺疊，可透過 XML Tools 外掛擴充 XML 驗證與格式化
- **授權：** GPL
- **平台：** 僅 Windows
- **網站：** https://notepad-plus-plus.org/

### 4. Geany

- **說明：** 輕量 IDE，支援 XML/HTML 標籤自動完成、語法突顯、程式碼摺疊、符號清單
- **授權：** GPL v2
- **平台：** Linux、macOS、Windows
- **網站：** https://www.geany.org

### 5. Apache NetBeans

- **說明：** 完整 IDE，內建 XML 編輯能力，包含 XSD 支援、XSLT 編輯與除錯、DTD 支援、樹狀檢視
- **授權：** Apache 2.0
- **最新版本：** 30 (2026-05)
- **平台：** Linux、macOS、Windows、Solaris
- **GitHub：** https://github.com/apache/netbeans

## 維護狀態摘要

### ✅ 積極維護（截至 2026）

| 函式庫 | 最新版本證據 |
|---|---|
| **Expat** | 2.8.5 (2026-09-22)；慕尼黑市資助 |
| **lxml** | 7.0.0a3 (2026-06-16) |
| **Apache Xerces C++** | 3.3.0 (2024-10-14) |
| **Apache XMLBeans** | 5.3.0 (2024-12-13) |
| **Nokogiri** | GitHub 活躍維護 |
| **XML Notepad** | 2.9.0.22 (2026-06-08) |

### ⚠️ 維護疑慮／停滯

| 函式庫 | 狀態 |
|---|---|
| **libxml2** | 維護者 2025-09 退出；分支 libxml2-ee (AGPL)。原始儲存庫仍有貢獻但無活躍維護者 |
| **XMLStarlet** | 最後版本 1.6.1 (2014-08-09) — 已超過 12 年未更新 |
| **VTD-XML** | 最後版本 2017；實質上已廢棄 |
| **JDOM / Dom4j** | 活動量低 |

## 使用場景建議

| 使用場景 | 建議方案 |
|---|---|
| **嵌入式系統／超大檔案** | Expat (C 函式庫，最小記憶體足跡) |
| **一般 C/C++ 開發** | libxml2 (C 語言最完整功能集) |
| **Python XML（完整功能）** | lxml (最佳效能 + 功能) |
| **Python XML（簡單，標準函式庫）** | xml.etree.ElementTree (無外部依賴) |
| **Java XML（企業級）** | Apache Xerces (完整 Schema、標準合規) |
| **Java 串流處理** | StAX (隨 JDK 捆綁) |
| **Rust** | quick-xml (快速，純 Rust) |
| **Ruby** | Nokogiri (事實標準) |
| **PHP** | SimpleXML (內建 DOM) |
| **PHP 串流處理** | PHP XML Expat-based (內建 SAX) |
| **CLI 處理** | xmlstarlet 或 xmllint |
| **Windows GUI 專用 XML 編輯** | XML Notepad (微軟，MIT 授權，積極維護) |
| **跨平台 GUI XML 編輯** | QXmlEdit (LGPL，樹狀/圖形化檢視) |
| **輕量跨平台 GUI 編輯** | Geany |
| **完整 IDE 含 XML/XSLT 除錯** | Apache NetBeans |

## 參考資料

[^expat]: libexpat. (n.d.). Expat XML Parser. Retrieved 2026-10-03, from https://github.com/libexpat/libexpat
[^libxml2]: GNOME. (n.d.). libxml2. Retrieved 2026-10-03, from https://gitlab.gnome.org/GNOME/libxml2
[^xerces]: Apache Software Foundation. (n.d.). Apache Xerces. Retrieved 2026-10-03, from https://xerces.apache.org/
[^vtdxml]: VTD-XML. (n.d.). VTD-XML: The Next Generation XML Parser. Retrieved 2026-10-03, from https://vtd-xml.sourceforge.net/
[^lxml]: lxml. (n.d.). lxml - XML and HTML with Python. Retrieved 2026-10-03, from https://lxml.de/
[^xmlnotepad]: Microsoft. (n.d.). XML Notepad. Retrieved 2026-10-03, from https://github.com/microsoft/XmlNotepad
[^qxmledit]: Bellonda, L. (n.d.). QXmlEdit. Retrieved 2026-10-03, from https://github.com/lbellonda/qxmledit
[^notepadplusplus]: Notepad++ Contributors. (n.d.). Notepad++. Retrieved 2026-10-03, from https://notepad-plus-plus.org/
[^geany]: Geany Contributors. (n.d.). Geany. Retrieved 2026-10-03, from https://www.geany.org
[^netbeans]: Apache Software Foundation. (n.d.). Apache NetBeans. Retrieved 2026-10-03, from https://github.com/apache/netbeans
[^xmlstarlet]: XMLStarlet. (n.d.). XMLStarlet Command Line XML Toolkit. Retrieved 2026-10-03, from https://xmlstar.sourceforge.net/
[^sax]: Wikipedia. (2025-10-25). Simple API for XML. Retrieved 2026-10-03, from https://en.wikipedia.org/wiki/Simple_API_for_XML
[^dom]: Wikipedia. (2026-05-18). Document Object Model. Retrieved 2026-10-03, from https://en.wikipedia.org/wiki/Document_Object_Model
[^stax]: XML.com. (2003-09-17). StAX: Streaming API for XML. Retrieved 2026-10-03, from https://www.xml.com/pub/a/2003/09/17/stax.html
[^libxml2lwn]: LWN.net. (2025-09-17). libxml2 security policy. Retrieved 2026-10-03, from https://lwn.net/Articles/1025971/
[^libxml2unmaintained]: Linuxiac. (2025-09-24). libxml2 Becomes Officially Unmaintained. Retrieved 2026-10-03, from https://linuxiac.com/libxml2-becomes-officially-unmaintained/