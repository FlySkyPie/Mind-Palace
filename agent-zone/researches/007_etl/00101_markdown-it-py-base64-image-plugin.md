# 使用 markdown-it-py 建立 Base64 圖片萃取 Plugin 的實作方案

## 問題描述

Markdown 文件中可能含有內嵌的 Base64 圖片（格式為 `![alt](data:image/png;base64,...`)），需要將其萃取成獨立圖片檔案，並將 Markdown 中的內嵌 URL 取代為本地檔案路徑。

## 解決方案架構

使用 `markdown-it-py`（Python 實作的 markdown-it 解析器）撰寫一個 **Core Rule 插件**，在解析完成後對 Token 流進行後處理（post-processing），掃描所有 `image` 型態的 token，提取 Base64 資料、寫入磁碟、並修改 token 的 `src` 屬性[^deepwiki-plugin]。

[^deepwiki-plugin]: DeepWiki for executablebooks/markdown-it-py. (n.d.). *5.2 Writing Plugins*. Retrieved 2026-09-20, from https://deepwiki.com/executablebooks/markdown-it-py/5.2-writing-plugins

## 為什麼選擇 Core Rule 模式

markdown-it-py 提供三種 Ruler Chain：Core、Block、Inline[^arch]。本方案採用 **md.core.ruler** 的原因如下：

| Chain | 用途 | 適合性 |
|---|---|---|
| Core | 文件層級後處理，Token 流已完整建立 | ✅ 最佳 — 可在解析完成後遍歷並修改所有 token |
| Block | 區塊層級解析（如程式碼區塊、引用） | ❌ 不適合 — image token 是 inline 層級 |
| Inline | 行內標記解析（如粗體、連結、圖片） | ❌ 不適合 — 解析過程中進行 I/O 寫入違反分工原則 |

[^arch]: executablebooks. (n.d.). *markdown-it-py Architecture*. Retrieved 2026-09-20, from https://markdown-it-py.readthedocs.io/en/latest/architecture.html

## Token 結構關鍵知識

### Image Token 的產生方式

當 markdown-it-py 解析 `![alt text](image.png "title")` 時，`markdown_it/rules_inline/image.py` 會產生一個 **self-closing token**[^image-src]：

```python
token = state.push("image", "img", 0)     # type="image", tag="img", nesting=0
token.attrs = {"src": href, "alt": ""}     # src 為圖片 URL，alt 初始為空字串
token.children = tokens or None            # alt 文字解析後的 inline token 列表
token.content = content                    # 原始 alt 文字字串
if title:
    token.attrSet("title", title)          # 若 markdown 有 title 屬性
```

[^image-src]: executablebooks. (n.d.). *markdown-it-py/rules_inline/image.py*. Retrieved 2026-09-20, from https://github.com/executablebooks/markdown-it-py/blob/master/markdown_it/rules_inline/image.py

### Token 在串流中的位置

Image token 會作為 `"inline"` 型態 token 的 **children** 存在[^token-src]：

```
paragraph_open (tag="p", nesting=1)
  inline (tag="", nesting=0, children=[
    Token(type="image", tag="img", nesting=0,
          attrs={"src": "data:image/png;base64,...", "alt": ""},
          children=[Token(type="text", content="alt")],
          content="alt")
  ])
paragraph_close (tag="p", nesting=-1)
```

### attrs 屬性的資料型態

markdown-it-py 的 `Token.attrs` 是 **Python dict**（`dict[str, str | int | float]`），與上游 JavaScript 版的 list-of-lists 不同[^token-attrs]。這使得修改非常直覺：

```python
# 直接 dict 存取
token.attrs["src"] = "images/abc123.png"

# 或使用輔助方法
token.attrSet("src", "images/abc123.png")
src = token.attrGet("src")
```

[^token-attrs]: executablebooks. (n.d.). *markdown-it-py/markdown_it/token.py*. Retrieved 2026-09-20, from https://github.com/executablebooks/markdown-it-py/blob/master/markdown_it/token.py

## 完整 Plugin 實作

```python
import base64
import os
import re
import uuid
from markdown_it import MarkdownIt
from markdown_it.rules_core import StateCore


def base64_image_plugin(
    md: MarkdownIt,
    *,
    output_dir: str = "images",
    url_prefix: str = "images",
) -> None:
    """
    markdown-it-py plugin to extract base64-embedded images from markdown,
    save them as local files, and replace the inline data URL with a local path.
    """

    def _extract_images(state: StateCore) -> None:
        for token in state.tokens:
            if token.type == "inline" and token.children:
                for child in token.children:
                    if child.type == "image":
                        src = child.attrs.get("src", "")
                        match = re.match(
                            r"data:(?P<mime>[^;]+);base64,(?P<data>.+)",
                            src,
                        )
                        if not match:
                            continue

                        mime = match.group("mime")
                        b64_data = match.group("data")

                        # Determine file extension from MIME type
                        ext = mime.split("/")[-1] if "/" in mime else "bin"
                        filename = f"{uuid.uuid4().hex}.{ext}"
                        filepath = os.path.join(output_dir, filename)

                        # Ensure output directory exists
                        os.makedirs(output_dir, exist_ok=True)

                        # Decode and write
                        image_bytes = base64.b64decode(b64_data)
                        with open(filepath, "wb") as f:
                            f.write(image_bytes)

                        # Replace src with local path
                        child.attrs["src"] = f"{url_prefix}/{filename}"

    # Run after the "inline" core rule so all inline tokens are populated
    md.core.ruler.after("inline", "base64_images", _extract_images)
```

## 使用方式

```python
md = MarkdownIt()
md.use(base64_image_plugin, output_dir="assets/images", url_prefix="/images")

html = md.render("![demo](data:image/png;base64,iVBORw0KGgo...)")
```

## 設計決策說明

### 1. `md.core.ruler.after("inline", ...)` 的時機點

使用 `after("inline", ...)` 而非 `push()` 確保在所有 inline 層級的解析完成之後才執行。`"inline"` 核心 rule 負責將 `"inline"` token 的 `content` 解析為子 token 並填入 `children` 陣列[^inline-rule]；若在此之前執行，`token.children` 會是空的，無法遍歷 image token。

[^inline-rule]: executablebooks. (n.d.). *markdown-it-py/rules_core/inline.py*. Retrieved 2026-09-20, from https://github.com/executablebooks/markdown-it-py/blob/master/markdown_it/rules_core/inline.py

### 2. 使用 uuid 而非語意檔名

因 Base64 圖片通常沒有原始檔名資訊，使用隨機 UUID 避免檔名衝突。若需要保留關聯性，可在 `token.meta` 中儲存對應資訊供後續處理。

### 3. 支援的 MIME 類型

透過正則表達式 `data:(?P<mime>[^;]+);base64,(?P<data>.+)` 自動擷取 MIME type，並以此決定副檔名，適用於 `image/png`、`image/jpeg`、`image/gif`、`image/webp`、`image/svg+xml` 等常見格式。注意 `image/svg+xml` 的副檔名會是 `svg+xml`，若需要可額外處理。

## 替代方案比較

| 方案 | 優點 | 缺點 |
|---|---|---|
| **Core Rule（本方案）** | 不受輸出格式限制；可在 render 前做任意修改 | 需要理解 token 結構 |
| Render Rule 覆寫 | 簡單，只需覆寫 image 的 render 函式[^render] | 僅影響 HTML 輸出；若需要保留修改後的 token 供其他用途則不適用 |
| Inline Ruler 取代 image rule | 可在解析階段攔截 | 實作複雜，需完整理解 state machine；I/O 操作不該放在解析階段 |

[^render]: executablebooks. (n.d.). *markdown-it-py Using*. Retrieved 2026-09-20, from https://markdown-it-py.readthedocs.io/en/latest/using.html

## 相關參考實作

`executablebooks/mdit-py-plugins` 套件中有多個使用相同模式的 plugin 可供參考[^mdit-plugins]：

- **tasklists** — `md.core.ruler.after("inline", "github-tasklists", fcn)`，遍歷 token 並修改 attrs / 插入新 token[^tasklists]
- **wordcount** — `md.core.ruler.push("wordcount", ...)`，遍歷 token 讀取內容[^wordcount]
- **anchors** — `md.core.ruler.push("anchor", ...)`，修改 heading token 的 attrs 並插入錨點 token[^anchors]
- **attrs** — 使用 inline ruler + block ruler + core ruler 三層協作，直接針對 image token 修改 attrs[^attrs]

[^mdit-plugins]: executablebooks. (n.d.). *mdit-py-plugins*. Retrieved 2026-09-20, from https://github.com/executablebooks/mdit-py-plugins
[^tasklists]: executablebooks. (n.d.). *mdit-py-plugins/tasklists/__init__.py*. Retrieved 2026-09-20, from https://github.com/executablebooks/mdit-py-plugins/blob/master/mdit_py_plugins/tasklists/__init__.py
[^wordcount]: executablebooks. (n.d.). *mdit-py-plugins/wordcount/__init__.py*. Retrieved 2026-09-20, from https://github.com/executablebooks/mdit-py-plugins/blob/master/mdit_py_plugins/wordcount/__init__.py
[^anchors]: executablebooks. (n.d.). *mdit-py-plugins/anchors/index.py*. Retrieved 2026-09-20, from https://github.com/executablebooks/mdit-py-plugins/blob/master/mdit_py_plugins/anchors/index.py
[^attrs]: executablebooks. (n.d.). *mdit-py-plugins/attrs/index.py*. Retrieved 2026-09-20, from https://github.com/executablebooks/mdit-py-plugins/blob/master/mdit_py_plugins/attrs/index.py

## 結論

使用 `md.core.ruler.after("inline", ...)` 註冊一條後處理規則，遍歷 token 流中的 `"inline"` token 的 `children`，找到 `type == "image"` 的子 token，解析其 `attrs["src"]` 中的 Base64 data URI，寫入本地檔案後將 `src` 替換為本地路徑，即可乾淨地解決問題。此模式簡單、模組化且不影響輸出格式。