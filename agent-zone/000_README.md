# Agent Zone

## 研究報告

- 輸出至 `researches/*.md`，不得建立或放置於其他子資料夾，移動報告至子資料夾是人類分類的權力。
- 檔案名稱必須遵守 `(\d){5}_([a-z-]+\.md)`，即五碼數字搭配 kebab-case 命名。
  - 五碼數字從 `00001` 算起，依序累加。
  - 檔案移動並不會改變累加規則，該編號是 `researches/*.md` 內唯一的。
- 研究必須建立在使用 Web Search/Fetch 工具之上，不得憑印象回答。
- 所有事實必須像維基百科一樣使用註腳 (Footnote) 語法標記引用與參考，如：
    ```markdown
    Markdown 是一種輕量級標記式語言[^lml]，目標是實現「易讀易寫」[^label2]。
    [^lml]: 輕量級標記式語言(英語：Lightweight Markup Language，簡稱LML)
    [^label2]: Markdown 語法受到一些既有text-to-HTML格式的影響
    ```
- 所有參考必須以改良板 APA (American Psychological Association) 格式引用：
    ```markdown
    Greater New Milford (n.d.). Who has time for a family meal? You do! Retrieved 2000-10-05, from http://www.familymealtime.org
    ```
    - APA 格式引用的各種種類細節請見 [001_apa-format.md](./001_apa-format.md)。
    - 改良的部份：原始標準使用美式英文，改良後使用 ISO 8601 格式。
- 每一份報告都必須是獨立文件，禁止引用其他文件或報告，禁止假設、提及或暗示某個對話前提，構成報告的所有上下文都必須收錄在報告內，使第一次閱讀報告或是只閱讀單一報告文件的人不會有背景不明的困惑。
- 除非另外提及，報告預設應以 zh_TW 撰寫。
- 如有示意圖需求，使用 mermaid，禁止 ASCII 圖表。
