# FOSS XML 閱讀工具調查（以人為導向，非剖析器）

## 概述

本報告調查自由開源（FOSS, Free and Open Source Software）中**專為人類閱讀、瀏覽、理解 XML 文件**而設計的工具。不同於程式開發者使用的 XML 剖析器/程式庫，此處收錄的工具皆具備 GUI 樹狀檢視、CLI 格式化輸出、或瀏覽器互動式檢視等能力，以降低人類閱讀 XML 的認知負擔。

XML（Extensible Markup Language）是一種廣泛使用的標記式語言，用於結構化資料儲存與交換。人類直接閱讀原始 XML 原始碼往往困難，尤其當文件缺乏縮排、命名空間繁雜、或結構深層嵌套時。因此，專為人類設計的 XML 閱讀工具便扮演重要角色——它們能將 XML 轉化為易於瀏覽的樹狀結構、格式化的文字輸出、或表格化檢視。

---

## 一、桌面 GUI XML 瀏覽器（Tree View / 編輯器）

### 1. FreeXmlToolkit（強力推薦）

最完整的 FOSS XML 工作站，功能接近 XMLSpy。

| 屬性 | 內容 |
|---|---|
| **說明** | 整合式桌面 XML 編輯器/檢視器，具備圖形化樹狀檢視、表格檢視（Grid View）、文字編輯器並列。支援 XPath 測試、XSLT 轉換、PDF 輸出、XSD 驗證、Schema 文件生成 |
| **平台** | Windows（.exe/.msi/portable）、macOS（.dmg/.pkg，Apple Silicon & Intel）、Linux（.deb/.rpm/portable） |
| **授權** | Apache 2.0 |
| **GitHub** | https://github.com/karlkauc/FreeXmlToolkit |
| **語言** | Java / JavaFX |
| **狀態** | 積極維護（2026 年 9 月仍有提交，2,554 commits，7 stars） |
| **特色** | IntelliSense、程式摺疊、樹狀/表格雙檢視、XSD 視覺化、XPath/XQuery 測試、XSLT 轉換、PDF 輸出、數位簽章、Schema 收藏庫 |

[^freetoolkit]: kauc, k. (n.d.). FreeXmlToolkit. Retrieved 2026-10-03, from https://github.com/karlkauc/FreeXmlToolkit

### 2. EditiX XML Editor

專業級開源 XML 編輯器，附樹狀導覽。

| 屬性 | 內容 |
|---|---|
| **說明** | 全功能 XML 編輯器，含樹狀導覽、XSLT 編輯器/除錯器、視覺化 W3C Schema 編輯器、專案管理 |
| **平台** | 跨平台（Java）— Windows、Linux、macOS |
| **授權** | GPL-3.0（另有商用授權） |
| **GitHub** | https://github.com/AlexandreBrillant/Editix-xml-editor |
| **語言** | Java |
| **狀態** | 積極維護（2026 年 9 月仍有提交，152 commits，76 stars，29 forks） |
| **特色** | 樹狀檢視、XSLT 除錯器、視覺化 Schema 編輯器、XPath/XQuery、AI 外掛（付費擴充） |

[^editix]: Brillant, A. (n.d.). EditiX XML Editor. Retrieved 2026-10-03, from https://github.com/AlexandreBrillant/Editix-xml-editor

### 3. QXmlEdit（Linux 首選）

基於 Qt 的多平台 XML 編輯器，少數具備圖形化 XSD 檢視器的 FOSS 工具。

| 屬性 | 內容 |
|---|---|
| **說明** | 階層式 XML 元素檢視、快速導航、分割大型 XML 檔案、XPath 搜尋、XSD 檢視器、欄位檢視、地圖檢視、XML/XSD diff、程式碼片段、資料匿名化、SCXML 編輯模式 |
| **平台** | Linux（首選）、Windows、macOS（macOS 支援較弱） |
| **授權** | GNU LGPL v2 |
| **GitHub** | https://github.com/lbellonda/qxmledit |
| **最新版本** | 0.9.18（2023-01） |
| **狀態** | 維護中但版本較舊（183 stars，826 commits） |

[^qxmledit]: Bellonda, L. (n.d.). QXmlEdit. Retrieved 2026-10-03, from https://github.com/lbellonda/qxmledit

### 4. Microsoft XML Notepad（Windows 最佳選擇）

微軟發佈的開源 XML 編輯器，直覺的樹狀 UI 最適合 Windows 使用者。

| 屬性 | 內容 |
|---|---|
| **說明** | 樹狀檢視、XML Schema 感知 IntelliSense、XPath 查詢、XInclude 支援、XSLT 轉換、CSV/JSON/HTML 匯入轉換、XML diff、樹狀與文字雙重檢視、即時 XML Schema 驗證 |
| **平台** | 僅 Windows |
| **授權** | MIT |
| **GitHub** | https://github.com/microsoft/XmlNotepad |
| **最新版本** | 2.9.0.22（2026-06-08） |
| **狀態** | 積極維護（~1,200 stars，561 commits） |

[^xmlnotepad]: Microsoft. (n.d.). XML Notepad. Retrieved 2026-10-03, from https://github.com/microsoft/XmlNotepad

### 5. XML Tool (xmltool)

輕量級 Java/JavaFX 工具，專為以可摺疊樹狀瀏覽 XML 而設計。

| 屬性 | 內容 |
|---|---|
| **說明** | 開啟 XML 檔案後以可摺疊樹狀呈現 XML 資料，簡單直覺。作者自述「因為買不起 XMLSpy」而開發 |
| **平台** | 跨平台（Java/JavaFX）— Windows、macOS、Linux |
| **授權** | GPL-3.0 |
| **GitHub** | https://github.com/cmiles74/xmltool |
| **語言** | Clojure / JavaFX |
| **狀態** | 低活動量（最後更新 2021 年 5 月，8 stars，2 forks） |
| **特色** | 樹狀瀏覽、無效字元清除、分頁控制台輸出 |

[^xmltool]: Miles, C. (n.d.). xmltool. Retrieved 2026-10-03, from https://github.com/cmiles74/xmltool

### 6. 其他桌面工具

| 工具 | 平台 | 狀態 | 授權 | 簡述 |
|---|---|---|---|---|
| **XSemmel** | 僅 Windows（.NET/C#） | 已停產（最後更新 2017-11） | BSD-2-Clause | 前功能豐富的 XML 編輯器，含樹狀檢視、XSD 程式補全、欄位檢視、XPath 命名空間支援、XML 比較、XSLT、XQuery、批次驗證。33 stars[^xsemmel] |
| **StructuredXmlEditor** | 僅 Windows（C#） | 低活動（最後更新 2022-12） | Apache 2.0 | 定義檔驅動的圖形化 XML 編輯器，也支援 JSON/YAML。94 stars[^structxml] |
| **xmltab-wlx** | 僅 Windows（Total Commander 外掛） | 維護中（最後更新 2024-12） | GPL-2.0 | Total Commander 內以樹狀/表格混合檢視 XML，支援欄位過濾、排序、語法突顯、美化。C 語言撰寫，20 stars[^xmltab] |
| **openXJV** | 跨平台（Python/pip/AppImage） | 積極維護（2026 年 6 月更新） | GPL-3.0 | 德國司法電子送達 XML（XJustiz 標準）專用檢視器，支援排序/過濾、全文搜尋、OCR、PDF 生成。15 stars，174 commits[^openxjv] |

[^xsemmel]: Carver, F. (n.d.). XSemmel. Retrieved 2026-10-03, from https://github.com/fcarver/xsemmel
[^structxml]: Lyeeedar. (n.d.). StructuredXmlEditor. Retrieved 2026-10-03, from https://github.com/Lyeeedar/StructuredXmlEditor
[^xmltab]: little-brother. (n.d.). xmltab-wlx. Retrieved 2026-10-03, from https://github.com/little-brother/xmltab-wlx
[^openxjv]: digidigital. (n.d.). openXJV. Retrieved 2026-10-03, from https://github.com/digidigital/openXJV

---

## 二、瀏覽器式 Web XML 檢視器

### 1. Pretty JSON & XML

| 屬性 | 內容 |
|---|---|
| **說明** | 將 XML 以可排序、可搜尋的表格或可摺疊樹狀呈現。也支援格式化/最小化、內嵌 base64 圖片預覽。100% 用戶端執行（JavaScript），資料不外流。單一 HTML 檔案，無建置步驟 |
| **平台** | 任何現代瀏覽器（跨平台） |
| **授權** | MIT |
| **GitHub** | https://github.com/aleksandre-mtchedlishvili/prettyjsonxml.com |
| **語言** | HTML / Vanilla JavaScript（單一檔案，無相依） |
| **狀態** | 積極維護（2026 年 9 月仍有提交，100 commits，3 stars） |
| **特色** | 表格檢視（自動偵測欄位並可排序）、可摺疊樹狀檢視、即時全文搜尋、format/minify、內嵌 base64 圖片預覽、虛擬滾動處理 9MB+ 檔案、淺色/深色主題 |

[^prettyjson]: Mtchedlishvili, A. (n.d.). Pretty JSON & XML. Retrieved 2026-10-03, from https://github.com/aleksandre-mtchedlishvili/prettyjsonxml.com

### 2. xml-viewer（wangruofeng）

| 屬性 | 內容 |
|---|---|
| **說明** | 純前端單一 HTML 檔案 XML 檢視器。可透過四種方式載入 XML：檔案選取、拖放、貼上、URL。顯示為可摺疊樹狀，附語法著色 |
| **平台** | 任何瀏覽器（跨平台） |
| **授權** | MIT |
| **GitHub** | https://github.com/wangruofeng/xml-viewer |
| **語言** | HTML / Vanilla JavaScript（單一檔案） |
| **狀態** | 維護中（2026 年 7 月更新） |
| **特色** | 可摺疊樹狀檢視、按層級展開/收合、複製 XPath/節點/屬性、語法著色、淺色/深色主題、統計（檔案大小、節點數、最大深度）、行動裝置自適應 |

[^xmlviewer]: wangruofeng. (n.d.). xml-viewer. Retrieved 2026-10-03, from https://github.com/wangruofeng/xml-viewer

### 3. TreeDoc Viewer（Web App + VS Code 擴充）

| 屬性 | 內容 |
|---|---|
| **說明** | 多格式（XML/JSON/YAML/CSV）樹狀檢視器。提供 Web App 即時示範，亦有 VS Code 擴充版本 |
| **平台** | 任何瀏覽器 + VS Code |
| **授權** | MIT |
| **即時示範** | https://treedoc.github.io |
| **GitHub** | https://github.com/treedoc/TreedocViewer |
| **語言** | Vue 3 + PrimeVue |
| **狀態** | 積極維護（v2，44 stars，453 commits） |
| **特色** | 樹狀檢視、表格檢視（陣列資料）、原始碼檢視、模式比對與運算欄位 |

[^treedoc]: Treedoc. (n.d.). TreeDoc Viewer. Retrieved 2026-10-03, from https://github.com/treedoc/TreedocViewer

---

## 三、CLI / 終端機 XML 閱讀工具

### 1. treeq（終端機首選）

以鍵盤為主的互動式 TUI 樹狀 XML/JSON 檢視器。

| 屬性 | 內容 |
|---|---|
| **說明** | 快速終端機樹狀視覺化，支援 XML/JSON/YAML/NDJSON。互動式 TUI 具備 Vim 按鍵綁定（h/j/k/l、gg/G），增量模糊搜尋，節點 8 色標記，路徑複製（OSC 52 協定），檢視模式與挑選模式。可視為 `jq` 的伴侶工具——先用 treeq 探索文件結構與路徑，再用 jq 查詢 |
| **平台** | 跨平台（Homebrew 支援 macOS/Linux，`cargo install treeq` 通用） |
| **授權** | MIT |
| **GitHub** | https://github.com/ognjen-vuceljic/treeq |
| **語言** | Rust |
| **狀態** | 非常活躍（v0.4.0，136 commits，2026 年 10 月更新，Homebrew 可用） |
| **特色** | Vim 按鍵導航、fzf 風格快遞搜尋、8 色節點標記、路徑複製（OSC 52 剪貼簿）、`--pick` 互動挑選模式、`--agent` 旗標（供 AI 編碼工具使用）、`--stats` 文件統計、`--path` 指定查詢路徑、shell 補全（zsh/bash/fish）、tmux 整合 |
| **安裝** | `brew install treeq` 或 `cargo install treeq` |

[^treeq]: Vuceljic, O. (n.d.). treeq. Retrieved 2026-10-03, from https://github.com/ognjen-vuceljic/treeq

### 2. XMLStarlet

經典 CLI XML 工具組，格式化輸出對人類閱讀極有幫助。

| 屬性 | 內容 |
|---|---|
| **說明** | 命令列 XML 工具組：格式化（`fo`）、XPath 查詢（`sel`）、編輯（`ed`）、驗證（`val`）、XSLT 轉換（`tr`）。`xmlstarlet fo file.xml` 可將任意 XML 格式化為縮排整齊的輸出 |
| **平台** | 跨平台 CLI（Linux、macOS、Windows 經 WSL/Cygwin） |
| **授權** | MIT |
| **GitHub** | https://github.com/XMLStarlet/xmlstarlet |
| **語言** | C |
| **最新版本** | 1.6.1（2014-08-09） |
| **狀態** | 穩定但版本較舊；GitHub 上仍有人維護（819 commits）。原始 SourceForge 儲存庫已遷移至 GitHub |
| **實用範例** | `xmlstarlet fo file.xml`（格式化）、`xmlstarlet sel -t -v "//element" file.xml`（查詢元素） |

[^xmlstarlet]: XMLStarlet. (n.d.). XMLStarlet. Retrieved 2026-10-03, from https://github.com/XMLStarlet/xmlstarlet

### 3. xmllint

幾乎所有 Linux 發行版預裝的 CLI XML 格式化工具。

| 屬性 | 內容 |
|---|---|
| **說明** | GNOME libxml2 專案的一部份。`xmllint --format file.xml` 可將 XML 格式化為縮排整齊的輸出。亦支援 DTD 驗證（`--valid`）、XSD 驗證（`--schema`）、XPath 評估（`--xpath`）、互動 shell 模式（`--shell`） |
| **平台** | 跨平台 CLI（多數 Linux/macOS 預裝；Windows 可經由包裝管理器安裝） |
| **授權** | MIT（libxml2 的一部份） |
| **GitHub** | https://github.com/GNOME/libxml2 |
| **語言** | C |
| **狀態** | 積極維護（GNOME 專案，7,824 commits，763 stars，441 forks）。注意：2025 年 9 月原始維護者退出，但 GNOME 組織持續維護 |

[^xmllint]: GNOME. (n.d.). libxml2. Retrieved 2026-10-03, from https://github.com/GNOME/libxml2

### 4. yq / xq（kislyuk/yq）

以 `jq` 風格查詢 XML 的 CLI 工具。

| 屬性 | 內容 |
|---|---|
| **說明** | `yq` 是命令列 YAML/XML/TOML 處理器，包裝 `jq`。其中 `xq` 命令專門處理 XML：將 XML 轉碼為 JSON（透過 xmltodict），經由 `jq` 過濾查詢，亦可加 `-x` 旗標轉回 XML 格式輸出。適合在管線中查詢 XML |
| **平台** | 跨平台 CLI（`pip install yq`，Homebrew 可用 `python-yq`） |
| **授權** | Apache 2.0 |
| **GitHub** | https://github.com/kislyuk/yq |
| **語言** | Python |
| **狀態** | 非常活躍（3,000+ stars，85 forks） |
| **特色** | `xq` 處理 XML、`yq` 處理 YAML、`tomlq` 處理 TOML、雙向轉碼（`-x` 將查詢結果轉回 XML）、XML 串流支援（`--xml-item-depth` 用於大型文件） |

[^yq]: Kislyuk, A. (n.d.). yq. Retrieved 2026-10-03, from https://github.com/kislyuk/yq

---

## 四、文字編輯器外掛（輔助 XML 閱讀）

以下並非專門的 XML 閱讀工具，但提供 XML 結構摺疊、語法突顯等功能，足以應付日常閱讀：

| 編輯器 | XML 支援說明 |
|---|---|
| **VS Code** | 內建 XML 語法突顯、摺疊。擴充「XML Tools」可增加格式化與樹狀檢視 |
| **Vim / Neovim** | 內建 `:syntax on` 搭配 `foldmethod` 可摺疊 XML 標籤 |
| **Emacs** | `nxml-mode` 提供即時驗證、縮排、結構摺疊 |
| **Kate（KDE）** | 內建 XML 語法突顯與摺疊 |
| **Geany** | 輕量 IDE，支援 XML/HTML 標籤自動完成、語法突顯、程式碼摺疊、符號清單。GPL v2，跨平台[^geany] |

[^geany]: Geany. (n.d.). Geany. Retrieved 2026-10-03, from https://www.geany.org

---

## 五、用途推薦

| 你的需求 | 建議工具 |
|---|---|
| 完整取代 XMLSpy | **FreeXmlToolkit** — 樹狀 + 表格 + 文字 + XSD + XSLT + PDF 一應俱全 |
| 快速看一下 XML 結構 | **xml-viewer**（瀏覽器，單一 HTML 檔案）或 **TreeDoc Viewer**（Web） |
| 在終端機瀏覽 XML | **treeq** — 互動 TUI，Vim 按鍵，模糊搜尋，節點標記 |
| 純格式化 XML 輸出 | **xmllint --format**（已預裝）或 **XMLStarlet fo** |
| 從命令列查詢 XML | **xq**（kislyuk/yq）或 **XMLStarlet sel** |
| 以表格檢視 XML | **Pretty JSON & XML**（瀏覽器式，可排序表格） |
| AI 輔助 / 編碼工具探索 XML | **treeq** 搭配 `--agent`、`--stats`、`--path` 旗標 |
| Windows 桌面 GUI | **XML Notepad**（微軟開源，樹狀 UI 最直覺） |
| Linux 桌面 GUI | **QXmlEdit**（階層式樹狀檢視 + XSD 檢視器） |

---

## 六、跨平台支援總覽

| 工具 | 跨平台？ | 安裝方式 |
|---|---|---|
| **FreeXmlToolkit** | ✅ Windows、macOS、Linux | 三者皆有原生安裝檔 |
| **EditiX XML Editor** | ✅ Windows、macOS、Linux | Java 跨平台，下載 JAR |
| **QXmlEdit** | ✅ Windows、macOS、Linux | 套裝管理器或 GitHub Releases |
| **XML Tool (xmltool)** | ✅ Windows、macOS、Linux | Java/JavaFX |
| **treeq** | ✅ Linux、macOS、Windows | `brew install treeq` / `cargo install treeq` |
| **Pretty JSON & XML** | ✅ 任何瀏覽器 | 下載單一 HTML 檔案，雙擊開啟 |
| **xml-viewer** | ✅ 任何瀏覽器 | 下載單一 HTML 檔案，雙擊開啟 |
| **XMLStarlet** | ✅ Linux、macOS、Windows（WSL） | `apt install xmlstarlet` / `brew install xmlstarlet` |
| **xmllint** | ✅ Linux、macOS、Windows | 多數系統預裝 |
| **yq/xq** | ✅ 所有平台 | `pip install yq` |
| **XML Notepad** | ❌ 僅 Windows | `winget install XmlNotepad` |
| **XSemmel** | ❌ 僅 Windows | 已停產 |
| **xmltab-wlx** | ❌ 僅 Windows | Total Commander 外掛 |
| **StructuredXmlEditor** | ❌ 僅 Windows | C# 原生 |
| **openXJV** | ✅ Windows、Linux、macOS | `pip install openxjv` / AppImage / Windows installer |

---

## 七、維護狀態摘要

### ✅ 積極維護（截至 2026）

| 工具 | 最新版本證據 |
|---|---|
| **FreeXmlToolkit** | 2026 年 9 月仍有提交 |
| **EditiX XML Editor** | 2026 年 9 月仍有提交，分支 `2027` |
| **XML Notepad** | 2.9.0.22（2026-06-08） |
| **Pretty JSON & XML** | 2026 年 9 月更新 |
| **xml-viewer** | 2026 年 7 月更新 |
| **TreeDoc Viewer** | 2026 年持續提交 |
| **treeq** | v0.4.0（2026 年 10 月） |
| **xmllint** | GNOME 專案持續維護 |
| **yq/xq** | 3,000+ stars，活躍 |

### ⚠️ 維護疑慮／停滯

| 工具 | 狀態 |
|---|---|
| **XMLStarlet** | 最後版本 1.6.1（2014-08-09）— 超過 12 年未發行新版 |
| **QXmlEdit** | 最後版本 0.9.18（2023-01）— 2 年多未更新 |
| **XML Tool (xmltool)** | 最後更新 2021 年 5 月 |
| **XSemmel** | 最後更新 2017 年 11 月 — 已停產 |
| **StructuredXmlEditor** | 最後更新 2022 年 12 月 |

---

## 參考

[^freetoolkit]: kauc, k. (n.d.). FreeXmlToolkit. Retrieved 2026-10-03, from https://github.com/karlkauc/FreeXmlToolkit
[^editix]: Brillant, A. (n.d.). EditiX XML Editor. Retrieved 2026-10-03, from https://github.com/AlexandreBrillant/Editix-xml-editor
[^qxmledit]: Bellonda, L. (n.d.). QXmlEdit. Retrieved 2026-10-03, from https://github.com/lbellonda/qxmledit
[^xmlnotepad]: Microsoft. (n.d.). XML Notepad. Retrieved 2026-10-03, from https://github.com/microsoft/XmlNotepad
[^xmltool]: Miles, C. (n.d.). xmltool. Retrieved 2026-10-03, from https://github.com/cmiles74/xmltool
[^xsemmel]: Carver, F. (n.d.). XSemmel. Retrieved 2026-10-03, from https://github.com/fcarver/xsemmel
[^structxml]: Lyeeedar. (n.d.). StructuredXmlEditor. Retrieved 2026-10-03, from https://github.com/Lyeeedar/StructuredXmlEditor
[^xmltab]: little-brother. (n.d.). xmltab-wlx. Retrieved 2026-10-03, from https://github.com/little-brother/xmltab-wlx
[^openxjv]: digidigital. (n.d.). openXJV. Retrieved 2026-10-03, from https://github.com/digidigital/openXJV
[^prettyjson]: Mtchedlishvili, A. (n.d.). Pretty JSON & XML. Retrieved 2026-10-03, from https://github.com/aleksandre-mtchedlishvili/prettyjsonxml.com
[^xmlviewer]: wangruofeng. (n.d.). xml-viewer. Retrieved 2026-10-03, from https://github.com/wangruofeng/xml-viewer
[^treedoc]: Treedoc. (n.d.). TreeDoc Viewer. Retrieved 2026-10-03, from https://github.com/treedoc/TreedocViewer
[^treeq]: Vuceljic, O. (n.d.). treeq. Retrieved 2026-10-03, from https://github.com/ognjen-vuceljic/treeq
[^xmlstarlet]: XMLStarlet. (n.d.). XMLStarlet. Retrieved 2026-10-03, from https://github.com/XMLStarlet/xmlstarlet
[^xmllint]: GNOME. (n.d.). libxml2. Retrieved 2026-10-03, from https://github.com/GNOME/libxml2
[^yq]: Kislyuk, A. (n.d.). yq. Retrieved 2026-10-03, from https://github.com/kislyuk/yq
[^geany]: Geany. (n.d.). Geany. Retrieved 2026-10-03, from https://www.geany.org