# 以 AST 為中介、可切換編解碼器的 Python 標記語言處理函式庫調查

## 摘要

本報告調查 Python 生態系中，以「抽象語法樹（AST, Abstract Syntax Tree）」作為中間資料表示（IR, Intermediate Representation），並具備模組化設計、允許替換解析器（decoder）與渲染器（encoder）的標記語言處理函式庫。理想目標是如同瑞士刀般，能同時支援多種輕量級標記語法（Markdown、MediaWiki wikitext、BBCode、Textile、reStructuredText、Creole 等）。

調查結論：**純 Python 生態系中，目前並不存在一個「以通用 AST 為中介、原生支援多種輸入語法並可輸出多種語法」的完整函式庫**。最接近此理想的是 Haskell 的 Pandoc（透過 pypandoc 從 Python 呼叫），其次是 Python 原生的 mistletoe、mistune、wenmode 等——它們具備良好的「解析 → AST → 渲染」分離架構，但輸入端僅支援 Markdown。

## 背景概念

### 什麼是 AST 中介架構

所謂「以 AST 為中介」，是指程式處理流程被拆解為三段：

1. **解析（Parse / Decode）**：將某種標記語法的原始文字轉換為結構化的樹狀資料（AST）。
2. **轉換（Transform）**：在 AST 上進行遍歷、修改、過濾等操作。
3. **渲染（Render / Encode）**：將 AST 序列化為目標格式（HTML、LaTeX、另一種標記語法）。

此架構的價值在於「解耦」：只要不同語法都能被解析成「同一套」AST 節點型別，就能達成 M 種輸入語法 × N 種輸出語法的組合，而不需要撰寫 M × N 個轉換器[^pandocast]。

### 語法（syntax）與語意（semantics）的區分

在評估函式庫時，需區分兩類能力：

- **語法自訂（syntax extensibility）**：能否在同一種語言（如 Markdown）之上新增外掛文法（如表格、註腳）。這幾乎所有現代函式庫都支援。
- **語意中立（semantic neutrality）**：AST 是否為「格式中立」，即不綁定特定輸出語言，且能由「多種」輸入語言產生。

使用者所描述的「可切換 decoder/encoder 支援不同語法」，真正需要的是後者。

## 主要函式庫分析

### Pandoc 與 pypandoc（跨語言的實際標準）

Pandoc 是以 Haskell 撰寫的通用文件轉換器，自稱「universal document converter（通用文件轉換器）」，能讀取與寫出數十種格式[^pandoc]。

- **輸入格式**：Markdown（含 CommonMark、GFM）、reStructuredText、AsciiDoc、Emacs Org-Mode、Textile、djot、MediaWiki、DokuWiki、Jira wiki、Creole、LaTeX、HTML、DocBook、EPUB、Jupyter notebook 等。
- **輸出格式**：上述之外，另包含 BBCode、XWiki、ZimWiki、Typst、PowerPoint、reveal.js、PDF 等。
- **中介表示**：Pandoc 內部使用一套原生的 Haskell AST（`[Block]`），且可序列化為 JSON 或 XML（Haskell AST ↔ JSON ↔ XML）[^pandoc]。這意味著「Pandoc JSON AST」本身可被視為一個跨語言的通用中介格式。

Pandoc 的架構「為每種輸入與輸出格式各自提供獨立模組」，因此新增格式只需新增一個模組[^pandoc]，這正是模組化解耦的典範。

- **pypandoc**：Python 的薄封裝，以子行程方式呼叫 pandoc 執行檔，提供 `convert_text()`、`convert_file()`，並可自動下載 pandoc 二進位檔。GitHub 星數約 1.2k[^pypandoc]。
- **panflute**：以 Python 撰寫的 Pandoc 過濾器框架，直接操作 Pandoc 的 JSON AST，提供 Pythonic 的 Element 模型（`Para`、`Header`、`Str` 等）。GitHub 星數約 553[^panflute]。

**評價**：這是目前唯一真正滿足「多語法輸入 → 通用 AST → 多語法輸出」需求的方案。缺點是核心為 Haskell，Python 端必須依賴外部執行檔（子行程），並非純 Python 函式庫。

### mistletoe（Python 原生 AST 架構的標竿）

mistletoe 是純 Python 的 CommonMark 解析器，自我定位為「快速、可擴充、符合規範」，並明確以 AST 為核心[^mistletoe]。

- **架構**：`Document`（AST 樹）為中心，渲染器負責將 AST 轉為目標格式。官方文件明言：「將 Markdown 解析為抽象語法樹，也讓我們能替換渲染器以輸出不同格式，而無須更動任何核心元件[^mistletoe]。」
- **內建輸出**：HTML、LaTeX、AST（除錯用）、Markdown（可重排文字）。
- **contrib 輸出**：HTML+MathJax、HTML+Pygments、HTML+TOC、GitHub wiki links、Jira Markdown、XWiki Syntax、Scheme。
- GitHub 星數約 1.1k[^mistletoe]。

**評價**：在 Python 原生函式庫中，這是「AST/渲染器分離」最徹底、內建輸出格式最多者。官方更指出「為 mistletoe 撰寫新渲染器是相對簡單的任務[^mistletoe]」。限制在於輸入端僅支援 Markdown；若要支援 wikitext 或 BBCode，需自行撰寫新的解析器，而該解析器必須產出 mistletoe 的節點型別（或改寫核心）。

### mistune（解析/渲染分離，含 AST 輸出）

mistune 是高效能的 Markdown 解析器，v3 為完全重寫版本[^mistune]。GitHub 星數約 3.1k。

- **架構**：`BlockParser` + `InlineParser` 產生 token 串（AST），再由可插拔的渲染器輸出。
- **內建渲染器**：`HTMLRenderer`、`MarkdownRenderer`、`RSTRenderer`（Markdown → reStructuredText）[^mistunerenderers]。
- **關鍵能力**：可透過 `renderer=None` 取得原始 AST token 而不渲染。

官方文件範例顯示，可直接以 `RSTRenderer` 將 Markdown 轉為 reStructuredText，或以 `MarkdownRenderer` 重新格式化 Markdown[^mistunerenderers]。這證明 mistune 已具備「同一輸入、多種輸出」的能力。

**評價**：效能優異、渲染器外掛成熟。與 mistletoe 相同，輸入端限於 Markdown。作者（lepture）已推出後繼專案 Wenmode，建議新專案採用。

### Wenmode（mdast 相容 AST 的新星）

Wenmode 為 mistune 作者的新作，定位為「可組合的 Markdown 工具組」，明確聚焦於 mdast 相容的 AST 輸出與可插拔渲染[^wenmode]。GitHub 星數約 49（新專案，星數低但為同作者既有專案的後繼）。

- **架構**：`Parser` → mdast 相容 AST → `Renderer`。可分別使用 `Parser.parse()` 取得 AST，再以 `HTMLRenderer().render(tree)` 渲染[^wenmode]。
- **mdast 相容**：AST 輸出遵循 unified/remark 生態系的 mdast（Markdown AST）規格，節點如 `root`、`paragraph`、`text`、`strong`、`link`，並可選擇附帶原始碼位置（`positions=True`）以支援編輯器整合[^wenmode]。
- **內建渲染器**：HTML、AsciiDoc（`AsciiDocRenderer`）；可自訂。

**評價**：mdast 相容性使其 AST 具備跨工具潛力（可與 JS 生態互通）。仍是 Markdown 輸入限定，但「解析器與渲染器可獨立選用」的設計最貼近使用者所想的模組化目標。星數偏低，需留意維護活躍度與成熟度。

### Marko（Frost Ming）

Marko 是 CommonMark 合規的純 Python 解析器，明確分離解析與渲染[^marko]。GitHub 星數約 470。

- **架構**：`marko.parse(text)` 回傳由 `Element` 節點組成的文件樹（AST），再呼叫 `renderer.render(ast)`。
- **內建渲染器**：`HTMLRenderer`、`ASTRenderer`（以 Python repr 輸出 AST 供除錯）、`XMLRenderer`。
- **擴充機制**：`MarkoExtension` 物件可同時定義 `parser_mixins`、`renderer_mixins`、`elements`，使解析與渲染擴充成對出現。

**評價**：AST 具備型別化的節點類別（`Paragraph`、`Heading`、`Strong` 等），並攜帶 `source_span`、`syntax_spans` 供來源對映。輸入限 Markdown。

### markdown-it-py（token 串架構）

markdown-it-py 為 JavaScript `markdown-it` 的 Python 移植，是 CommonMark 合規度最高的 Python 解析器之一。GitHub 星數約 1.4k[^mditpy]。

- **架構**：產生扁平化的 token 串（類似 markdown-it），每個 token 具 `type`、`tag`、`attrs`、`nesting`、`children`、`content`、`meta`、`block` 等欄位，可依 nesting 層級（開/合）視為樹來走訪。
- **外掛**：`mdit-py-plugins` 提供表格、註腳、front matter、容器等擴充。

**評價**：token 串並非正式的樹狀 AST，且輸出以 HTML 為主。其價值在於可完整自訂語法規則，但並非為跨格式轉換而設計。

### MyST-Parser（Markdown → docutils doctree 橋接）

MyST-Parser 將 Markdown（經由 markdown-it-py 解析）轉換為 docutils 的 doctree，使 Markdown 能用於 Sphinx/ docutils 生態。GitHub 星數約 899[^myst]。

**評價**：這是「將兩種生態的 AST 橋接」的實例：Markdown token 串 → docutils doctree。輸出沿用 docutils 的眾多 writer（HTML、LaTeX、man 等）。輸入仍限 Markdown，但展示了以既有 AST 作為轉換樞紐的模式。

### docutils（reStructuredText 的參考實作）

docutils 是 reStructuredText 的參考實作，解析器產生內部 doctree（節點樹），再由 writer 輸出 HTML、LaTeX、XML、man page、OpenDocument 等[^docutils]。

**評價**：doctree 是一套成熟的樹狀中介表示，輸出端模組化良好，但輸入端**僅支援 reStructuredText**，並非可自由替換輸入語法的設計。它的角色更適合作為「輸出目標 AST」，如 MyST-Parser 便將 Markdown 轉入其 doctree。

### mwparserfromhell（MediaWiki wikitext 專用）

mwparserfromhell 是解析 MediaWiki wikitext 的最佳 Python 函式庫。GitHub 星數約 907[^mwparser]。

- **架構**：解析器建立 `Wikicode` 的 AST，節點包含模板、標籤、wikilink 等，可檢查、修改，並序列化回 wikitext（支援往返保留 round-trip）。

**評價**：具備真正的 AST，但為 wikitext 專用，不輸出其他標記格式。若要支援 MediaWiki 輸入語法，此函式庫可作為「decoder」的基礎，但仍需自行將其 AST 映射到通用 AST。

### python-creole（透過 HTML 中介的多格式轉換）

python-creole 是純 Python 的多格式標記轉換器，提供 `creole2html`、`html2creole`、`html2rest`（需 docutils）、`html2textile` 等轉換器[^pythoncreole]。

- **架構**：**以 HTML 作為中介**，而非正式的 AST。例如 html → reStructuredText、html → textile 皆先經 HTML 的 DOM/字串處理。creole2html 部分源自 MoinMoin 專案的解析器。

**評價**：這是 Python 中最完整的「雙向多格式轉換」函式庫，但轉換是「兩兩配對經過 HTML」，並非透過共享 AST。最後版本為 1.4.10，發布於 2021 年，僅支援 Python 3.6–3.9，維護狀態保守。

### 其他單一格式函式庫

| 函式庫 | 輸入 | 輸出 | 星數 | 是否具 AST |
|---|---|---|---|---|
| Python-Markdown | Markdown | HTML | 約 4.3k[^pymarkdown] | 內部 element tree（未對外暴露為通用 AST） |
| python-textile | Textile | HTML | 約 74[^pytextile] | 無 |
| creole（PyPI） | WikiCreole | HTML | 無（2010 年後未維護）[^creole] | 無 |
| commonmark-py | Markdown | HTML | 約 236[^commonmarkpy] | 有，但已 Archived/Deprecated |

Python-Markdown 雖為最受歡迎的 Python Markdown 函式庫之一，但其內部 `ElementTree` 屬實作細節，並非為跨格式轉換而設計的 API，且官方承認非 CommonMark 合規[^pymarkdown]。

## 綜合比較

下表以「是否滿足使用者需求（以通用 AST 為中介、可切換 decoder/encoder、支援多語法）」為核心排序：

| 函式庫 | 通用 AST 中介 | 多語法輸入 | 多語法輸出 | 純 Python | 星數 |
|---|---|---|---|---|---|
| **pypandoc / Pandoc** | 是（Haskell AST） | 是（數十種） | 是（數十種） | 否（子行程） | 1.2k / 46.8k |
| **panflute** | 是（操作 Pandoc JSON AST） | 是 | 是 | 是（但依賴 pandoc 執行檔） | 553 |
| **mistletoe** | 是（Document 樹） | 否（僅 Markdown） | 是（10+ 格式） | 是 | 1.1k |
| **Wenmode** | 是（mdast 相容） | 否（僅 Markdown） | 是（HTML、AsciiDoc 等） | 是 | 49 |
| **mistune** | 是（token AST） | 否（僅 Markdown） | 是（HTML、RST、Markdown） | 是 | 3.1k |
| **marko** | 是（Element 樹） | 否（僅 Markdown） | 是（HTML、AST、XML） | 是 | 470 |
| **markdown-it-py** | token 串（非正式樹） | 否（僅 Markdown） | 否（主為 HTML） | 是 | 1.4k |
| **MyST-Parser** | 轉入 docutils doctree | 否（僅 Markdown） | 是（經 docutils） | 是 | 899 |
| **docutils** | 是（doctree） | 否（僅 RST） | 是（多種） | 是 | 70 |
| **mwparserfromhell** | 是（Wikicode 樹） | 否（僅 wikitext） | 否（僅 wikitext 往返） | 是 | 907 |
| **python-creole** | 否（以 HTML 為中介） | 是（Creole、HTML、RST、Textile） | 是 | 是 | 無（已停更） |
| **Python-Markdown** | 否 | 否 | 否（僅 HTML） | 是 | 4.3k |

## 關鍵結論

1. **無純 Python 的通用方案**：截至目前調查，純 Python 生態系中沒有函式庫同時實作「通用 AST + 多種輸入語法 + 多種輸出語法」。最接近的是透過 pypandoc 呼叫 Pandoc[^pandoc]。

2. **架構最接近的是 mistletoe 與 Wenmode**：兩者皆具備清楚的「解析 → AST → 可替換渲染器」分離[^mistletoe][^wenmode]。mistletoe 內建輸出格式最多（含 LaTeX、Jira、XWiki）；Wenmode 則以 mdast 相容 AST 取勝，利於跨工具互通。

3. **輸入端是共同瓶頸**：所有 Python 原生函式庫皆以 Markdown 為唯一輸入語言。若要支援 wikitext、BBCode 等，現有函式庫無法直接使用，需：
   - 為該語法撰寫新的 decoder，將輸出映射到既有 AST 節點型別；或
   - 自建一套通用 AST，並為每種語法各寫 decoder/encoder。

4. **可借鏡的橋接模式**：MyST-Parser 展示了「將 A 語言的解析結果轉入 B 生態的 AST」的可行做法[^myst]；mwparserfromhell 則提供了現成的 wikitext decoder 基礎[^mwparser]；python-creole 提供了雙向多格式轉換的參考（雖以 HTML 為中介）[^pythoncreole]。

5. **實務建議**：
   - 若允許外部依賴：直接用 Pandoc（pypandoc / panflute），可立即獲得最完整的多語法支援。
   - 若需純 Python 且輸入以 Markdown 為主：mistletoe 或 Wenmode。
   - 若目標是自建通用 AST 系統：可參考 Pandoc 的 AST 設計（或採用其 JSON AST 作為中介格式）、mistletoe 的渲染器擴充機制、以及 mwparserfromhell 的 wikitext 解析。

## 來源

[^pandoc]: Pandoc. (n.d.). *Pandoc - a universal document converter*. Retrieved 2026-11-01, from https://pandoc.org/

[^pandocast]: Pandoc 支援將內部 Haskell AST 雙向序列化為 JSON 與 XML，此 AST 即為跨格式轉換的中介表示。Pandoc. (n.d.). *Pandoc - a universal document converter*. Retrieved 2026-11-01, from https://pandoc.org/

[^pypandoc]: JessicaTegner. (n.d.). *pypandoc*. GitHub. Retrieved 2026-11-01, from https://github.com/JessicaTegner/pypandoc

[^panflute]: Correia, S. (n.d.). *panflute*. GitHub. Retrieved 2026-11-01, from https://github.com/sergiocorreia/panflute

[^mistletoe]: miyuchina. (n.d.). *mistletoe: A fast, extensible and spec-compliant Markdown parser in pure Python*. GitHub. Retrieved 2026-11-01, from https://github.com/miyuchina/mistletoe

[^mistune]: lepture. (n.d.). *mistune: A fast yet powerful Python Markdown parser with renderers and plugins*. GitHub. Retrieved 2026-11-01, from https://github.com/lepture/mistune

[^mistunerenderers]: lepture. (n.d.). *Renderers — Mistune documentation*. Retrieved 2026-11-01, from https://mistune.lepture.com/en/latest/renderers.html

[^wenmode]: lepture. (n.d.). *wenmode: A fast, composable Markdown parser and renderer toolkit*. GitHub. Retrieved 2026-11-01, from https://github.com/lepture/wenmode

[^marko]: frostming. (n.d.). *marko: A markdown parser with high extensibility*. GitHub. Retrieved 2026-11-01, from https://github.com/frostming/marko

[^mditpy]: executablebooks. (n.d.). *markdown-it-py: Markdown parser done right*. GitHub. Retrieved 2026-11-01, from https://github.com/executablebooks/markdown-it-py

[^myst]: executablebooks. (n.d.). *MyST-Parser: An extended commonmark compliant parser with bridges to docutils & sphinx*. GitHub. Retrieved 2026-11-01, from https://github.com/executablebooks/MyST-Parser

[^docutils]: docutils. (n.d.). *docutils: Documentation Utilities*. GitHub. Retrieved 2026-11-01, from https://github.com/docutils/docutils

[^mwparser]: earwig. (n.d.). *mwparserfromhell: A Python parser for MediaWiki wikicode*. GitHub. Retrieved 2026-11-01, from https://github.com/earwig/mwparserfromhell

[^pythoncreole]: jedie. (2021). *python-creole 1.4.10*. PyPI. Retrieved 2026-11-01, from https://pypi.org/project/python-creole/

[^pymarkdown]: waylan. (n.d.). *Python-Markdown*. GitHub. Retrieved 2026-11-01, from https://github.com/Python-Markdown/markdown

[^pytextile]: textile. (n.d.). *python-textile*. GitHub. Retrieved 2026-11-01, from https://github.com/textile/python-textile

[^creole]: Dopieralski, R. (2010). *creole*. PyPI. Retrieved 2026-11-01, from https://pypi.org/project/creole/

[^commonmarkpy]: readthedocs. (n.d.). *commonmark.py*. GitHub. Retrieved 2026-11-01, from https://github.com/readthedocs/commonmark.py
