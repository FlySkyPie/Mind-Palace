# Token 串流 vs AST：以 Markdown 處理為核心的設計取捨

## 為何討論這個主題

在資料分析與格式轉換工具中，核心問題是：**如何在中間層表示結構化資料？** 兩種常見路線是：

- **Token 串流（Token stream）**：以扁平序列為主體、部分嵌套為輔的資料表示
- **AST（Abstract Syntax Tree，抽象語法樹）**：完全以樹狀結構為核心的資料表示

這兩種路線並非抽象優劣之爭，而是具體的工程設計取捨。本文以 Markdown 處理器 **markdown-it-py** 的 Token 串流設計為錨點，對照傳統 AST 的作法，說明各適用的場景與背後的權衡。[^markdownit]

## Token 串流的設計（以 markdown-it-py 為例）

markdown-it-py 的文檔開宗明義：

> "Instead of traditional AST we use more low-level data representation — tokens. The difference is simple: Tokens are a simple sequence (Array). Opening and closing tags are separate." [^markdownit]

這句話包含三個關鍵設計決策：

### 決策一：資料結構是陣列，不是樹

Token 串流頂層是一個**陣列（Array）**，而非樹狀結構。這意味著你可以用線性索引存取、用 `for` 迴圈走訪，不需要遞迴或樹走訪演算法。

### 決策二：開標籤與關標籤是分離的 Token

在 AST 中，一個節點（如 `BlockQuote`）本身就代表了整個結構，開與關是隱含的。在 Token 串流中，開與關是**兩個不同的 Token 物件**，分別攜帶各自的屬性。例如：

```
[
  { type: 'bullet_list_open', level: 0 },
  { type: 'list_item_open', level: 1 },
  { type: 'paragraph_open', level: 2 },
  { type: 'inline', level: 2, children: [...] },
  { type: 'paragraph_close', level: 2 },
  { type: 'list_item_close', level: 1 },
  { type: 'bullet_list_close', level: 0 },
]
```

這種設計讓渲染器（Renderer）可以直接將開/關 Token 映射到 HTML 的開/關標籤（`<ul>` / `</ul>`），不需額外轉換邏輯。[^markdownit]

### 決策三：局部嵌套透過 .children 處理

純陣列無法表達行內格式的嵌套（如 `粗體中的**斜體**`）。為了解決這個問題，markdown-it-py 引入了一種特殊的 Token 類型 — 「行內容器」（inline container），其 `.children` 屬性包含一個嵌套的 Token 串流：

```
頂層 Token 串流（陣列）
├── blockquote_open
├── paragraph_open
├── inline  ← .children 屬性內含另一個 Token 串流
│   ├── text("這是 ")
│   ├── strong_open
│   ├── text("粗體")
│   ├── strong_close
│   └── text(" 文字")
├── paragraph_close
└── blockquote_close
```

這是**有節制的嵌套**：只在需要的地方（行內格式）開放嵌套，不讓整個資料結構都變成樹。[^markdownit]

## Token 串流 vs AST：對照比較

| 面向 | Token 串流 | AST |
|---|---|---|
| **結構** | 扁平陣列 + 局部 .children 嵌套 | 完全遞迴的樹狀結構 |
| **開/閉標籤** | 顯式分離為兩個 Token | 隱含在單一節點中 |
| **走訪方式** | 線性掃描（索引迴圈） | 遞迴走訪（樹遍歷） |
| **抽象層次** | 低：保留大多數原始結構細節 | 高：省略語法細節（括號、分隔符等） |
| **對 Renderer 友善** | ✅ 直接對應 HTML 線性標籤流 | ❌ 需額外將樹轉為線性標籤流 |
| **對 Transformation 友善** | ❌ 結構資訊分散在序列中 | ✅ 樹狀結構容易進行結構變換 |
| **嵌套深度** | 固定兩層（block → inline） | 任意深度 |
| **擴充規則** | 在陣列中插入/移除 Token 即可 | 需操作樹節點，語意更重 |
| **記憶體** | 較輕量（無節點間指標） | 較重（節點物件 + 父子參考） |

## 為什麼 markdown-it-py 選擇 Token 串流而非 AST？

根據其文檔與設計原則，原因如下：

### 1. KISS 原則 — 不引入不必要的複雜度

專案作者的觀點是：傳統 AST 對其任務是不必要的（"not needed for our tasks"）。Markdown 的文法相對簡單，Token 串流已經足夠。[^markdownit]

### 2. Renderer 的自然映射

HTML 本身就是線性標籤流（`<ul><li><p>...</p></li></ul>`）。Token 串流的「開/關分離」可以直接一對一映射到 HTML 的開/關標籤。Renderer 做的事情本質上就是：

```python
for token in tokens:
    if token.type.endswith('_open'):
        output += f'<{tag}>'
    elif token.type.endswith('_close'):
        output += f'</{tag}>'
```

AST 要做同樣的事需要先將樹「展平」（flatten）為線性順序。

### 3. 規則與插件的可組合性

markdown-it-py 使用獨立的規則鏈（core → block → inline）處理資料，每個規則都是獨立函式，可以安全地啟用/停用。Token 串流是「唯寫」（write-only）的 — 規則只會新增 Token 到串流末端，不會回頭修改。這大幅降低了規則間的耦合。[^markdownit]

### 4. 可轉換性：Token 串流也可以產生 AST

文檔特別說明：如果你需要 AST，你可以只跑 Parser 不跑 Renderer，然後自行將 Token 串流轉換為 AST。Token 串流保留了足夠的資訊（開/關配對、層級、類型）來重建樹狀結構。[^markdownit]

## 在資料分析與轉換場景中的適用原則

從 markdown-it-py 的設計可以歸納出一個通用原則：

### 適合用 Token 串流的場景

| 特徵 | 說明 |
|---|---|
| **目標格式為線性** | 如 HTML、XML、純文字 — 輸出本身就是序列 |
| **文法相對簡單** | 嵌套深度有限，不需任意深度的遞迴結構 |
| **重視插件擴展性** | 規則可獨立插入，不需修改核心節點定義 |
| **需要低延遲** | 線性掃描比樹走訪快，且可流水線化 |
| **以渲染為主** | 主要工作是「輸入 → 輸出轉換」，而非「結構變換」 |

具體案例：
- Markdown → HTML 轉換器（markdown-it、marked）
- 輕量級模板引擎（將模板解析為 Token 串流後直接輸出）
- JSON 序列化管線（將結構化資料展平為線性 Token 再輸出）

### 適合用 AST 的場景

說回來，AST 仍然是許多場景的合理選擇：

| 特徵 | 說明 |
|---|---|
| **深度結構變換** | 如程式碼重構（移動函式、提取方法）、編譯器最佳化 |
| **跨階段分析** | 需要型別檢查、符號解析、資料流分析 |
| **多種輸出格式** | 同一棵樹可產生 HTML、LaTeX、純文字等 |
| **需要結構查詢** | 如「找出所有條件式中未被覆蓋的 else 分支」 |

具體案例：
- 編譯器與轉譯器（GCC, Babel, TypeScript）
- 靜態分析工具（ESLint, SonarQube）
- 程式碼生成工具

## 「Token 串流」與「Lexer Token」的混淆風險

一個重要的名詞澄清：

| 概念 | 來自 | 特性 |
|---|---|---|
| **編譯器 Token（Lexer Token）** | Lexical Analysis | 扁平、無嵌套、無開/關配對概念、純粹「詞彙單元」 |
| **Markdown Token 串流** | markdown-it-py | 有開/關配對、有局部 .children 嵌套、帶層級資訊 |

這兩個概念都被稱為「Token」，但層次完全不同。編譯器 Token 是 Parsing 前的原始素材，而 Markdown Token 串流則是 Parsing 後的結構化產物 — 它其實是 AST 的**刻意扁平化版本**，而非 Lexer 的輸出。[^lexical]

如果要用編譯器類比，Markdown Token 串流比較接近 **Parse Tree（語法分析樹）的扁平化變體**，而非 Lexer Token。

## 結論

Token 串流與 AST 之間的選擇不是「誰比較先進」的技術問題，而是**工程設計取捨**：

- Token 串流的優勢在於**簡單、線性、Renderer 友善**，適合目標格式為序列、文法結構單純、以渲染輸出為主的系統。
- AST 的優勢在於**結構完整、變換靈活**，適合需要深度分析、多階段轉換、或任意深度變換的系統。

markdown-it-py 以 Token 串流取代 AST 的作法證明了：**當問題域足夠單純時，更底層的表示法反而是更好的設計** — 它降低了認知負擔、減少了耦合、讓插件開發更容易。

如果一個系統的主要工作是「將格式 A 轉換為格式 B」，且兩者都是線性序列，那麼 Token 串流應該是你的起點。只有在需要「對結構進行深度操作」時，才值得引入 AST 的複雜度。

---

## 參考文獻

[^markdownit]: executable book project. (n.d.). Design principles — markdown-it-py. Retrieved 2026-09-19, from https://markdown-it-py.readthedocs.io/en/latest/architecture.html

[^lexical]: Wikimedia Foundation. (n.d.). Lexical analysis. *Wikipedia*. Retrieved 2026-09-19, from https://en.wikipedia.org/wiki/Lexical_analysis