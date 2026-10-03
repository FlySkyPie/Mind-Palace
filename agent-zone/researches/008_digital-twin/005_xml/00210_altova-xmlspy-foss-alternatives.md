# Altova XMLSpy 自由開源替代方案調查

> 調查日期：2026-09-26

## 摘要

Altova XMLSpy 是一套功能完整的商用 XML 編輯器，支援 XSD (XML Schema Definition) 編輯與視覺化、XSLT 轉換、XPath 查詢、XML 驗證等功能。本報告調查其自由開源替代方案，特別著重 Web 端解決方案。

## 目錄

1. [Web 端方案](#1-web-端方案)
2. [桌面端開源方案](#2-桌面端開源方案)
3. [XSD 圖形化視覺工具](#3-xsd-圖形化視覺工具)
4. [功能比較表](#4-功能比較表)
5. [結論與建議](#5-結論與建議)

---

## 1. Web 端方案

### 1.1 VS Code for Web + Red Hat XML Extension（推薦）

VS Code 的瀏覽器版本 vscode.dev 搭配 Red Hat XML 擴充套件，是目前最強大的免費 XML 編輯 Web 方案。Red Hat XML 擴充套件（`redhat.vscode-xml`）基於 Eclipse LemMinX 語言伺服器，採用 EPL 2.0 授權，截至 2026 年 9 月已有超過 1,070 萬次安裝[^redhat-vscode-xml]。

Web 端支援的功能包括：

- **XSD 驗證與 Schema-Aware 程式碼補全**：根據關聯的 XSD 提供上下文感知的元素、屬性建議
- **XML 編輯**：即時語法錯誤回報、自動閉合標籤、格式化摺疊
- **XSLT 支援**：語法高亮與基礎編輯支援
- **XInclude 支援**
- **XML Catalogs**：支援 Schema 路徑對應

⚠️ **限制**：缺乏視覺化的 XSD 圖形編輯器。

### 1.2 XSD Editor Pro

由 gouthams11 開發的單頁 HTML 工具，完全在瀏覽器中執行。貼上 XSD 原始碼後可以樹狀結構編輯、增刪元素、自動儲存，支援深色模式[^xsd-editor-pro]。

### 1.3 xsd-editor-app

MIT 授權的 React + Node.js Web 應用，提供互動式樹狀檢視、元素/型別/屬性管理、跨 Schema 搜尋、簡單型別限制編輯（pattern、enumeration、length、數值限制），支援即時驗證與拖曳排序[^xsd-editor-app-gh]。

---

## 2. 桌面端開源方案

### 2.1 FreeXmlToolkit（Apache 2.0）

最活躍且功能完整的開源 XML 工具套件，由 Karl Kauc 開發，採用 JavaFX 架構，跨平台支援 Windows/macOS/Linux。截至 2026 年已有 2,500+ commits，最新版本 2.1.0[^freexmltoolkit]。

完整功能列表：

- **XSD 工具**：圖形化檢視、文字檢視、樹狀檢視、型別庫、型別編輯器、Schema 品質分析（命名慣例檢查、統計資料）、文件產生器（HTML/PDF/Word）、樣本 XML 產生器、Schema 扁平化
- **XML 編輯器**：XSD 驅動的 IntelliSense、Grid 檢視、樹狀檢視
- **XSLT/XPath/XQuery**：即時預覽、Saxon 支援、XProc 3.0 管線
- 數位簽章、PDF 產生（Apache FOP）、Schematron 支援

### 2.2 Microsoft XML Notepad（MIT 授權）

Microsoft 維護的 XML 編輯器，使用 C# / .NET Framework，僅支援 Windows。特色為樹狀 UI、即時 Schema 驗證、XSLT 輸出內嵌檢視器、XInclude 支援、XML Diff 工具。最新釋出為 2026 年 6 月[^xml-notepad]。

### 2.3 Xsd Explorer

JavaFX 桌面應用，提供 XMLSpy 風格的 Schema 圖形化檢視、搜尋定義、偵測根元素、產生樣本 XML、扁平化 Schema、驗證 XML 檔案。支援 Windows x64 與 Linux（需 JDK 17+ JavaFX）[^xsdexplorer]。

---

## 3. XSD 圖形化視覺工具

這些工具專門用於視覺化 XSD 結構，全部在瀏覽器中執行：

| 工具名稱 | 網址 | 特點 |
|---------|------|------|
| XSD Atlas | iltano.github.io/xsd-viewer/ | 開源，互動式圖表，匯出 SVG |
| ToolXML XSD Visualizer | toolxml.com/xsd-visualizer/ | 100% 客戶端，樹狀圖，匯出 PNG/SVG |
| XSD Schema Viewer | digitaltoolpad.com/tools/xsd-schema-viewer | 即時編輯原始碼，型別瀏覽 |
| Online XSD Viewer | xsd-viewer.online/ | 三種檢視模式（樹狀/圖形/原始碼），XSD 驗證 |

所有工具均為客戶端執行，無需上傳資料至伺服器[^toolxml][^xsdatlas]。

---

## 4. 功能比較表

| 功能 | XMLSpy | Red Hat XML (vscode.dev) | FreeXmlToolkit | XML Notepad |
|------|--------|-------------------------|---------------|-------------|
| XSD 圖形編輯 | ✅ | ❌ | ✅ 圖形檢視 | ❌ |
| XSD 驗證 | ✅ | ✅ | ✅ | ✅ |
| XML 編輯 | ✅ | ✅ | ✅ | ✅ |
| XSLT 轉換 | ✅ | ⚠️ 任務模式 | ✅ | ✅ |
| XPath 支援 | ✅ | ✅ | ✅ | ✅ |
| Web 端可用 | ❌ | ✅ | ❌ | ❌ |
| 完整開源 | ❌ | ✅ (EPL 2.0) | ✅ (Apache 2.0) | ✅ (MIT) |
| Schema 視覺化 | ✅ 圖形化 | ❌ | ✅ 圖形樹狀 | ❌ |

---

## 5. 結論與建議

### 若偏好 Web 方案

**最佳組合**：使用 [vscode.dev](https://vscode.dev) 安裝 Red Hat XML 擴充套件處理日常 XML/XSD 編輯與驗證，搭配 [XSD Atlas](https://iltano.github.io/xsd-viewer/) 或 [ToolXML XSD Visualizer](https://toolxml.com/xsd-visualizer/) 進行 XSD 結構視覺化瀏覽。

### 若可接受桌面端

**FreeXmlToolkit** 是最接近 XMLSpy 完整功能的開源方案，涵蓋 XSD 編輯、圖形化檢視、品質分析、文件產生、XSLT/XPath/XQuery 等，且跨平台、活躍維護中[^freexmltoolkit]。

### 已知缺口

目前沒有任何 Web 端的開源方案能完整取代 XMLSpy 的**所見即所得圖形化 XSD Schema 編輯器**。若需要拖曳式 XSD 圖形編輯，仍需依賴桌面端軟體（FreeXmlToolkit 或 Xsd Explorer）或商用方案（Oxygen XML Editor）。

---

[^redhat-vscode-xml]: Red Hat. (2026). XML Language Support by Red Hat - Visual Studio Marketplace. Retrieved 2026-09-26, from https://marketplace.visualstudio.com/items?itemName=redhat.vscode-xml

[^xsd-editor-pro]: Goutham, S. (n.d.). XSD Editor Pro. Retrieved 2026-09-26, from https://github.com/gouthams11/xsd-editor-pro

[^xsd-editor-app-gh]: Kokash, S. (n.d.). xsd-editor-app. Retrieved 2026-09-26, from https://github.com/SaeedKokash/xsd-editor-app

[^freexmltoolkit]: Kauc, K. (n.d.). FreeXmlToolkit. Retrieved 2026-09-26, from https://github.com/karlkauc/FreeXmlToolkit

[^xml-notepad]: Microsoft. (n.d.). XML Notepad. Retrieved 2026-09-26, from https://github.com/microsoft/XmlNotepad

[^xsdexplorer]: Tadamovsky, S. (n.d.). Xsd Explorer. Retrieved 2026-09-26, from https://xsdexplorer.com/

[^toolxml]: ToolXML. (n.d.). XSD Visualizer. Retrieved 2026-09-26, from https://toolxml.com/xsd-visualizer/

[^xsdatlas]: Iltaño. (n.d.). XSD Atlas. Retrieved 2026-09-26, from https://iltano.github.io/xsd-viewer/