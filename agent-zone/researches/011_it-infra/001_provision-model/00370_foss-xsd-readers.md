# FOSS XSD 閱讀工具調查（面向人類閱讀，非剖析器）

## 概述

XSD（XML Schema Definition）是一種描述 XML 文件結構、元素與型別的綱要語言。本報告調查開放原始碼社群中，專注於**讓人類閱讀、瀏覽、理解 XSD 檔案**的工具，而非那些作為程式庫供其他軟體呼叫的 XSD 剖析器。這些工具涵蓋 HTML 文件產生、SVG 圖形化顯示、桌面 GUI 互動式瀏覽等面向。

---

## 一、HTML 文件產生器

### xs3p

xs3p 是歷史最悠久的 XSD 文件產生器之一，以 XSLT 樣式表方式運作，將 XSD 檔轉換為 XHTML 格式的文件，包含交叉引用與樣例 XML 檢視。

- **分支一：bitfehler fork**[^xs3p-bitfehler]
  - 現代化改造：Bootstrap 樣式、HTML5、UTF-8、支援 `<documentation>` 元素的 Markdown
  - 語言：XSLT（需 XSLT 處理器如 Saxon、xsltproc）
  - 狀態：2019 年封存，唯讀

- **分支二：metanorma fork**[^xs3p-metanorma]
  - 由 Metanorma 專案維護，活躍開發（截至 2025 年仍有更新）
  - 用於 ISO/TC 211 與 UnitsML 綱要
  - 語言：XSLT
  - 狀態：活躍

- **原始版（FiForms）**[^xs3p-fiforms]
  - 原始 xs3p 1.1.5 版，採用 BSD 授權
  - 位於 SourceForge

### Xseed（metanorma）[^xseed]

Ruby gem，單一指令即可產生 XSD 文件的完整 HTML 頁面，並自動嵌入每個元素的 SVG 圖示。

- 指令：`xseed html schema.xsd -o docs/index.html`
- 輸出：HTML + 各元素的 SVG 圖示（以 `<object>` 標籤嵌入）
- 授權：MIT
- 語言：Ruby
- GitHub：https://github.com/metanorma/xseed

### xsddoc（xframe 原始版）[^xsddoc-xframe]

Java 為基礎的文件產生器，支援命令列與 Apache Ant 整合，可自訂頁首頁尾。

- 授權：LGPL 2.1
- 語言：Java
- 維修分支（evgeniy-polyakov）：https://github.com/evgeniy-polyakov/xsddoc

### 其他 HTML 產生器

- **5im-0n/xsddoc**[^xsddoc-5im0n]：簡潔的 XSLT 樣式表，將 XSD 轉為美觀的 HTML 表格
- **PHP XSD to Documentation Generator**[^php-xsd-doc]：PHP 工具，產生具響應式設計、可搜尋、多語言的 Bootstrap 文件
- **GPHemsley/xsd-doc**[^gphemsley]：Unlicense（公有領域）授權，功能簡約
- **terrajobst/xsddoc**[^terrajobst]：C#/.NET 的 Sandcastle 外掛，輸出 CHM 說明檔（2024 封存）

---

## 二、圖形化顯示工具（SVG / 圖示）

### XSD Diagram（dgis/xsddiagram）[^xsddiagram]

最成熟且受歡迎的 FOSS XSD 圖示工具（GitHub 247 星）。Windows 桌面 GUI 應用（C# .NET 2.0），以互動式樹狀圖呈現 XSD 元素、群組與屬性。可匯出 SVG、PNG、JPG、TXT、CSV、EMF，並支援命令列模式。

- 授權：GPL-2.0
- 語言：C#（可於 Mono/Linux 執行）
- GitHub：https://github.com/dgis/xsddiagram

### XsdVi 家族

源自 Václav Slavětínský 的 Java 應用，將 W3C XML Schema 轉為互動式 SVG 樹狀圖。

- **原始版（SourceForge）**[^xsdvi-sourceforge]
  - 授權：GPL
  - 語言：Java

- **metanorma fork**[^xsdvi-metanorma]
  - 維護中，支援 Linux/macOS/Windows
  - 語言：Java（Maven）

- **XsdVi2（siriusxiv fork）**[^xsdvi2]
  - 命令列 Java 工具
  - 授權：GPL-3.0
  - GitHub：https://github.com/siriusxiv/XsdVi2

- **xsdvi-ruby**[^xsdvi-ruby]
  - 純 Ruby 移植版，產生可折疊/展開的 SVG 圖示
  - 授權：BSD-3-Clause
  - 語言：Ruby
  - GitHub：https://github.com/metanorma/xsdvi-ruby

### jxsd（vitalyzotov）[^jxsd]

Java 21 命令列工具，遵循 Altova XMLSpy 視覺慣例將 XSD 元件渲染為 SVG/PNG 圖示。支援遞迴展開、文件顯示，可發佈為獨立二進位檔（不需 Java 環境）。

- 授權：Apache 2.0
- 語言：Java 21
- GitHub：https://github.com/vitalyzotov/jxsd

### XSD Atlas（iltano/xsd-viewer）[^xsd-atlas]

輕量級、免安裝、以瀏覽器為基礎的 XSD 圖示檢視器。僅需一支 HTML 檔案，可離線執行或託管於 GitHub Pages。純 JavaScript，無相依套件，無建置程序，可匯出 SVG。

- 閱讀方式：直接在瀏覽器中開啟
- 語言：JavaScript
- GitHub：https://github.com/iltano/xsd-viewer
- 即時示範：https://iltano.github.io/xsd-viewer/

### xsd2erd[^xsd2erd]

Python 腳本，將 XSD 轉換為 Mermaid.js 的 ER 圖（實體關係圖）。

- 語言：Python
- GitHub：https://github.com/amkuipers/xsd2erd

---

## 三、桌面 GUI 應用

### XSD Viewer（AnyKey1）[^xsd-viewer-anykey1]

Electron + React + TypeScript 建構的桌面應用，提供：
- 綱要樹狀導覽
- 相依圖圖示（React Flow + Dagre）
- 型別詳細檢視面板
- 語法高亮的原始碼檢視（附可點選型別連結）
- XML 驗證（透過 xmllint）

- 授權：MIT
- 語言：TypeScript（Electron + React）
- GitHub：https://github.com/AnyKey1/xsd-viewer

### DiagramEditorXSD[^diagram-editor]

Windows 11 圖形化工具，可建立、檢視與修改 XSD 文件。

- 語言：C#（.NET）
- GitHub：https://github.com/ArtemSvirid/DiagramEditorXSD

---

## 四、比較表

| 工具 | 類別 | 輸出格式 | 授權 | 語言 | 活躍度 |
|------|------|----------|------|------|--------|
| xs3p（bitfehler） | HTML 文件 | HTML5+Bootstrap | — | XSLT | 封存 |
| xs3p（metanorma） | HTML 文件 | HTML | — | XSLT | 活躍 |
| Xseed | HTML 文件 | HTML+SVG 圖示 | MIT | Ruby | 活躍 |
| xsddoc（xframe） | HTML 文件 | HTML | LGPL 2.1 | Java | 有維護分支 |
| XSD Diagram | 圖示 GUI | GUI + SVG/PNG | GPL-2.0 | C# | 活躍 |
| XsdVi（metanorma） | 圖示產生 | 互動式 SVG | — | Java | 活躍 |
| xsdvi-ruby | 圖示產生 | 互動式 SVG | BSD-3-Clause | Ruby | 活躍 |
| jxsd | 圖示產生 | SVG/PNG | Apache 2.0 | Java 21 | 新 |
| XSD Atlas | 網頁圖示 | 瀏覽器 + SVG | — | JS | 活躍 |
| xsd2erd | ER 圖 | Mermaid.js | — | Python | 實驗性 |
| XSD Viewer（AnyKey1） | 桌面 GUI | GUI | MIT | TS/Electron | 活躍 |
| DiagramEditorXSD | 桌面 GUI | GUI | — | C# | 新 |

---

## 五、推薦指引

- **只需產生文件** → Xseed（最自動化）或 xs3p-metanorma（最穩定）
- **需要圖形化瀏覽** → XSD Atlas（免安裝）或 XSD Diagram（功能最完整）
- **偏好桌面應用** → XSD Viewer（AnyKey1）（MIT 授權，Electron 跨平台）
- **只需要 SVG 圖示** → xsdvi-ruby 或 jxsd
- **整合進 CI/CD** → Xseed（Ruby gem）或 XsdVi（Java CLI）

---

[^xs3p-bitfehler]: bitfehler. (n.d.). xs3p — XSD to HTML documentation generator (fork). Retrieved 2026-10-01, from https://github.com/bitfehler/xs3p
[^xs3p-metanorma]: Metanorma. (n.d.). xs3p — Active fork of xs3p XSD-to-HTML stylesheet. Retrieved 2026-10-01, from https://github.com/metanorma/xs3p
[^xs3p-fiforms]: FiForms. (n.d.). xs3p — XML Schema to XHTML documentation. Retrieved 2026-10-01, from https://xml.fiforms.org/xs3p/
[^xseed]: Metanorma. (n.d.). Xseed — XSD documentation generator with SVG diagrams. Retrieved 2026-10-01, from https://github.com/metanorma/xseed
[^xsddoc-xframe]: Evgeniy Polyakov. (n.d.). xsddoc — XSD documentation generator (Java). Retrieved 2026-10-01, from https://github.com/evgeniy-polyakov/xsddoc
[^xsddoc-5im0n]: 5im-0n. (n.d.). xsddoc — Simple XSLT for XSD-to-HTML tables. Retrieved 2026-10-01, from https://github.com/5im-0n/xsddoc
[^php-xsd-doc]: patrickjaja. (n.d.). PHP XSD to Documentation Generator. Retrieved 2026-10-01, from https://github.com/patrickjaja/php-xsd-to-documentation-generator
[^gphemsley]: GPHemsley. (n.d.). xsd-doc. Retrieved 2026-10-01, from https://github.com/GPHemsley/xsd-doc
[^terrajobst]: terrajobst. (n.d.). xsddoc — Sandcastle plug-in for CHM help. Retrieved 2026-10-01, from https://github.com/terrajobst/xsddoc
[^xsddiagram]: dgis. (n.d.). XSD Diagram — Windows GUI XSD viewer and diagram generator. Retrieved 2026-10-01, from https://github.com/dgis/xsddiagram
[^xsdvi-sourceforge]: Václav Slavětínský. (n.d.). XsdVi — Java XSD-to-SVG diagram generator. Retrieved 2026-10-01, from https://sourceforge.net/projects/xsdvi/
[^xsdvi-metanorma]: Metanorma. (n.d.). xsdvi — Maintained fork of XsdVi. Retrieved 2026-10-01, from https://github.com/metanorma/xsdvi
[^xsdvi2]: siriusxiv. (n.d.). XsdVi2 — Java CLI fork of XsdVi. Retrieved 2026-10-01, from https://github.com/siriusxiv/XsdVi2
[^xsdvi-ruby]: Metanorma. (n.d.). xsdvi-ruby — Pure Ruby XSD-to-SVG diagram generator. Retrieved 2026-10-01, from https://github.com/metanorma/xsdvi-ruby
[^jxsd]: vitalyzotov. (2025). jxsd — Java 21 XSD-to-SVG/PNG CLI. Retrieved 2026-10-01, from https://github.com/vitalyzotov/jxsd
[^xsd-atlas]: iltano. (n.d.). XSD Viewer / XSD Atlas — Browser-based XSD diagram viewer. Retrieved 2026-10-01, from https://github.com/iltano/xsd-viewer
[^xsd2erd]: amkuipers. (n.d.). xsd2erd — XSD to Mermaid.js ERD converter. Retrieved 2026-10-01, from https://github.com/amkuipers/xsd2erd
[^xsd-viewer-anykey1]: AnyKey1. (n.d.). XSD Viewer — Electron desktop app for XSD browsing. Retrieved 2026-10-01, from https://github.com/AnyKey1/xsd-viewer
[^diagram-editor]: ArtemSvirid. (n.d.). DiagramEditorXSD — Windows GUI XSD editor/viewer. Retrieved 2026-10-01, from https://github.com/ArtemSvirid/DiagramEditorXSD