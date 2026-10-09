# TriliumNext 標籤（Labels）系統完全指南

## 概述

TriliumNext Notes（GitHub 星數：38,259[^stars]）是一個開源、自託管、可腳本化的筆記知識庫。其**標籤（Labels）系統**——官方統稱為**屬性（Attributes）**——是 Metadata 層的核心，支援從簡單標記到結構化表單、從樹狀繼承到進階搜尋的完整功能。

屬性分為兩大類：**Labels**（`#name=value`，文字鍵值對）與 **Relations**（`~name=@note`，指向其他筆記的連結）[^attr-doc]。

---

## 1. 系統屬性 vs 使用者自定義屬性

- **系統屬性**：Trilium 內建解讀的屬性，如 `#color`、`#iconClass`、`#readOnly`、`#archived`，影響筆記的行為與樣式。
- **使用者自定義屬性**：由使用者自由建立的標籤與關係，用於分類、搜尋、腳本控制。Trilium 對兩者無嚴格區分，僅依名稱是否匹配內建關鍵字判斷[^attr-doc]。

---

## 2. 標籤語法

建立標籤的語法如下：

| 語法 | 意義 |
|---|---|
| `#myLabel` | 布林標籤（無值），如同旗標 |
| `#myLabel=value` | 含值的文字標籤 |
| `#myLabel="Hello world"` | 雙引號包裹（值含空格時必須） |
| `#myLabel='Hello "world"'` | 單引號包裹（值含雙引號時使用） |
| `#myLabel(inheritable)` | 可繼承的布林標籤 |
| `#myLabel(inheritable)=value` | 可繼承且含值的標籤 |

UI 建立方式：
- 右側欄 Attributes 面板 → + 按鈕 → "Add new label"
- 底部狀態列 → 屬性按鈕 → 輸入 `#name` 或 `#name=value`
- 快捷鍵：`Alt + L`

Labels 在清單中以 `#` 前綴顯示，有別於 Relations 的 `~` 前綴[^labels-doc]。

---

## 3. 標籤類型（Promoted Attributes）

透過**推廣屬性定義（Promoted Attribute Definition）**可以指定值的編輯類型，改變 UI 中的輸入方式：

| 類型 | 說明 |
|---|---|
| Text（預設） | 自由文字輸入 |
| Multi-line text | 多行文字區域 |
| Number | 數字輸入，可設小數精度 |
| Checkbox (boolean) | 核取方塊，true/false |
| Select | 下拉選單（輕量，無顏色/圖示） |
| Date | 日期選擇器 |
| Date & Time | 日期+時間選擇器 |
| Time | 時間選擇器 |
| URL | 顯示按鈕開啟連結 |
| Email | 開啟系統預設郵件程式 |
| Phone | 觸發系統通話 |
| Color | 顏色選取器 |
| Relation | 指向另一則筆記 |

> **注意**：這些類型僅影響 UI 編輯方式，資料層不做強制驗證，更改類型不會改變已存在的值[^promoted-doc]。

---

## 4. 推廣屬性（Promoted Attributes）

**推廣屬性**是指將屬性顯示在筆記標題下方的可收合表單區域，將隱藏的 metadata 變成顯眼的可編輯欄位。

建立步驟：
1. 右側欄 Attributes 面板 → + 按鈕 → "Add new label definition" 或 "Add new relation definition"
2. 設定 **name**（將作為標籤/關係名稱）
3. 勾選 **Promoted** 以顯示在筆記頂端
4. 可選設定 **Alias**（顯示名稱，可含空格）
5. 設定 **Inheritable** 以應用至子筆記
6. 選擇 **type**（text, number, date 等）
7. 儲存

內部運作方式：定義會建立一個以 `label:` 或 `relation:` 前綴的特殊屬性。例如：

```
#label:myColor(inheritable)="promoted,alias=Color,multi,color"
```

該定義充當模板。當使用者填入值時，筆記上會建立一個**獨立的非定義標籤**（如 `#myColor=red`）。定義可設為可繼承以影響整個子樹，而每個筆記保有各自的值[^promoted-doc]。

---

## 5. 可繼承標籤

標籤（與關係）可以標記為 **Inheritable**（可繼承），方法是在 UI 勾選或手動語法加入 `(inheritable)`：

```
#myLabel(inheritable)=value
```

有三種繼承機制：

| 類型 | 說明 |
|---|---|
| **標準繼承（Standard Inheritance）** | 屬性標記 `isInheritable=true` 後，應用至該筆記及其所有子孫。例如在頂層設定 `#archived(inheritable)` 可隱藏整個子樹 |
| **複製繼承（Copying Inheritance）** | 使用 `child:` 前綴（如 `#child:attributeName`），新建子筆記時自動**複製**一份獨立的屬性。可鏈式使用（`#child:child:attributeName`）。即使移動筆記，複製依然保留 |
| **模板繼承（Template Inheritance）** | 透過模板建立筆記時繼承模板的屬性。另外 `~inherit` 關係可繞過父子關係直接繼承屬性 |

---

## 6. 多重值標籤

Trilium 的屬性可**多重值（multi-value）**——同一筆記上可以存在多個同名標籤。例如一本書可以同時有：

```
#genre="sci-fi"
#genre="fantasy"
```

搭配 Promoted Attributes（定義中啟用 `multi` 模式）時特別強大，使用者可以輕鬆加入多個值。

---

## 7. 標籤定義

**標籤定義**是一種特殊的屬性，用於定義其他屬性的行為與顯示。語法為：

```
#label:attributeName(options)="config"
```

例如：

```
#label:myColor(inheritable)="promoted,alias=Color,multi,color"
```

設定字串中的選項包含：
- `promoted` — 顯示為推廣屬性
- `alias=Name` — 顯示名稱
- `multi` — 允许多重值
- `single` — 僅允許單一值
- 類型名稱（如 `color`, `date`, `number`, `boolean`, `url`, `email`, `phone`）

這些定義本身也可設為可繼承，使整個子樹共享相同表單欄位。

---

## 8. 關係（Relations）

**Relations** 類似 Labels，但值是指向**另一則筆記**的連結。

語法：

```
~myRelation=@
```

輸入 `~` + 名稱 + `=@` 後會出現自動完成方塊供選擇目標筆記。也可以從樹狀結構複製筆記後貼上。

可繼承關係語法：

```
~myRelation(inheritable)=@
```

內建關係一覽：

| 關係 | 用途 |
|---|---|
| `~template` | 透過模板繼承屬性 |
| `~renderNote` | 指向 Render Note 渲染的筆記 |
| `~widget` | 自訂 Widget 參考 |
| `~runOn*` | 腳本觸發（如 `runOnNoteCreation`） |
| `~inherit` | 無需父子關係的屬性繼承 |
| `~shareCss`, `~shareJs`, `~shareHtml`, `~shareTemplate`, `~shareFavicon` | 分享自訂化 |

**反向關係（Inverse Relations）：** 可以定義配對關係（如 `~isParentOf` 和 `~isChildOf`），當加入一個關係時，目標筆記會自動加入/移除對應的反向關係[^relations-doc]。

---

## 9. 搜尋與組織

標籤深度整合於 Trilium 的搜尋系統，支援豐富的屬性查詢語言：

### 基本搜尋

| 查詢 | 結果 |
|---|---|
| `#book` | 含有 `book` 標籤的筆記 |
| `#!book` | **不含** `book` 標籤的筆記 |
| `#book or #author` | 含有任一標籤的筆記 |
| `#book and #publicationYear = 1954` | 同時符合兩者的筆記 |
| `towers #book` | 全文搜尋 + 屬性組合 |

### 值運算子

| 運算子 | 意義 | 範例 |
|---|---|---|
| `= value` | 等於 | `#publicationYear = 1954` |
| `!= value` | 不等於 | `#genre != "fantasy"` |
| `*=* value` | 包含子字串 | `#genre *=* fan` |
| `=* value` | 以...開頭 | `#title =* "The"` |
| `*=` value | 以...結尾 | `#title *= "Rings"` |
| `>=`, `<=`, `>`, `<` | 數值比較 | `#publicationYear >= 1950` |
| `%= regex` | 正規表達式 | `#publicationYear %= '19[0-9]{2}'` |

### 智慧日期

```
#dateNote >= TODAY-30       # 最近 30 天
#dateNote >= NOW-3600       # 最近 1 小時
#dateNote >= MONTH-3        # 最近 3 個月
#dateNote >= YEAR-1         # 最近 1 年
```

### 關係搜尋

```
~author.title *=* Tolkien    # 尋找作者標題含 "Tolkien" 的筆記
~author.relations.son.title = 'Christopher Tolkien'   # 鏈式關係
```

### 筆記屬性搜尋

```
note.type = code AND note.mime = 'application/json'
note.parents.title = 'Books'
note.ancestors.title = 'Books'
note.children.title = 'sub-note'
note.dateCreated =* '2019-05'
```

### 排序與限制

```
#author=Tolkien orderBy #publicationDate desc, note.title limit 10
```

### 底層語法

快捷寫法 `#publicationYear = 1954` 是以下寫法的語法糖：

```
note.labels.publicationYear = 1954
```

而 `~author.title *=* Tolkien` 是：

```
note.relations.author.title *=* Tolkien
```

---

## 10. 常用系統標籤參考

以下是一些常用的內建系統標籤：

| 標籤 | 用途 |
|---|---|
| `#disableVersioning` | 跳過此筆記的版本記錄 |
| `#archived` | 隱藏於搜尋/對話框 |
| `#readOnly` | 設為唯讀 |
| `#color` | 樹狀結構與連結顏色（任何 CSS 色值） |
| `#iconClass` | 透過 CSS class 自訂圖示 |
| `#template` | 使筆記可作為模板使用 |
| `#sorted` | 自動按字母排序子筆記 |
| `#top` / `#bottom` | 在排序清單中固定置頂/置底 |
| `#keyboardShortcut` | 設快捷鍵跳至該筆記（如 `ctrl+alt+e`） |
| `#run` / `#runAtHour` | 腳本執行觸發 |
| `#workspace` | 工作區設定 |
| `#viewType` | 子筆記顯示模式（grid 或 list） |
| `#geolocation` | 經緯度（用於地理地圖） |
| `#fullContentWidth` | 編輯器全寬顯示 |
| `#shareAlias` | 分享用的乾淨網址別名 |

---

## 總結

Trilium 的標籤/屬性系統是一個**統一且靈活的 metadata 層**，貫穿從簡單標記、結構化資料輸入、筆記設定、腳本控制、主題自訂到進階搜尋的所有面向。關鍵概念總結如下：

1. **Labels**（`#name=value`）— 文字 metadata；**Relations**（`~name=@note`）— 筆記連結
2. **類型系統**（text, number, boolean, date, URL, email, phone, color, select, relation）— 透過推廣屬性定義提供
3. **推廣屬性** — 將標籤展示為表單欄位
4. **繼承機制**（標準、`child:` 複製、模板）— 透過筆記樹傳播屬性
5. **多重值** — 同名標籤可共存
6. **標籤定義**（`#label:name`）— 定義類型、多重性、別名與推廣設定
7. **反向關係** — 自動維護雙向關聯
8. **搜尋整合** — 豐富的查詢語言，支援運算子、鏈式關係、數值/日期比較、正則表達式、排序與全文組合

---

## 參考資料

[^stars]: TriliumNext. (n.d.). Trilium Notes. Retrieved 2026-10-09, from https://github.com/TriliumNext/Trilium

[^attr-doc]: TriliumNext. (n.d.). Attributes. Retrieved 2026-10-09, from https://github.com/TriliumNext/Trilium/blob/main/docs/User%20Guide/User%20Guide/Advanced%20Usage/Attributes.md

[^labels-doc]: TriliumNext. (n.d.). Labels. Retrieved 2026-10-09, from https://github.com/TriliumNext/Trilium/blob/main/docs/User%20Guide/User%20Guide/Advanced%20Usage/Attributes/Labels.md

[^relations-doc]: TriliumNext. (n.d.). Relations. Retrieved 2026-10-09, from https://github.com/TriliumNext/Trilium/blob/main/docs/User%20Guide/User%20Guide/Advanced%20Usage/Attributes/Relations.md

[^promoted-doc]: TriliumNext. (n.d.). Promoted Attributes. Retrieved 2026-10-09, from https://github.com/TriliumNext/Trilium/blob/main/docs/User%20Guide/User%20Guide/Advanced%20Usage/Attributes/Promoted%20Attributes.md

[^inherit-doc]: TriliumNext. (n.d.). Attribute Inheritance. Retrieved 2026-10-09, from https://triliumnext.github.io/Docs/Wiki/attribute-inheritance.html

[^search-doc]: TriliumNext. (n.d.). Search. Retrieved 2026-10-09, from https://triliumnext.github.io/Docs/Wiki/search.html