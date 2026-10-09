# Trilium（TriliumNext）標籤/標記功能調查

## 摘要

Trilium（TriliumNext/trilium）是一款開源的層級式筆記軟體，具有完整的標記（tag/label）系統。本文調查其標籤功能的支援程度、運作方式與特色。

## 核心術語：Labels（標籤）而非 Tags

Trilium 官方術語使用 **「labels」（標籤）** 而非「tags」。Labels 是 Trilium 屬性系統（Attributes）的一部分，該系統包含兩種主要類型[^attr-docs]：

- **Labels（`#` 前綴）**：附著在筆記上的名稱-值配對（key-value pair），相當於傳統的標籤
- **Relations（`~` 前綴）**：連結兩則筆記之間的命名關係（類似關聯式資料庫的外部鍵）

## 標籤功能支援度

| 功能 | 支援狀態 |
|---|---|
| 為筆記附加標籤 | ✅ 完全支援 |
| 僅有名稱的標籤（如 `#archived`） | ✅ 支援 |
| 名稱+值的標籤（如 `#year=1999`） | ✅ 支援 |
| 多重值標籤（同一筆記可有多個同名標籤） | ✅ 支援 |
| 可搜尋標籤 | ✅ 完整索引化 |
| 圖形化標籤編輯器 | ✅ 支援 |
| 手動語法輸入 | ✅ 支援 |
| 階層式標籤（巢狀） | ❌ 標籤本身為扁平結構 |
| 樹狀繼承機制 | ✅ 三種繼承方式 |
| 系統預設標籤 | ✅ 約 50+ 個內建標籤 |
| 推廣屬性（Promoted Attributes）表單化 UI | ✅ 支援多種欄位類型 |
| 腳本化/自動化存取 | ✅ 支援 |

## 標籤的運作方式

### 基本語法

標籤在 Trilium UI 中以 `#` 前綴顯示，支援三種形式[^labels-docs]：

1. **純名稱（布林標籤）**：`#archived` — 僅表示該屬性存在
2. **名稱=值（單值標籤）**：`#genre=sci-fi`
3. **名稱=值（含空格值）**：`#genre="sci-fi novel"`

同一筆記可同時擁有多個相同名稱的標籤（multi-valued），例如 `#tag=python`、`#tag=web`、`#tag=database`。

### 系統預設標籤

Trilium 內建超過 50 個由系統解讀的預定義標籤[^attr-docs]，常見範例：

- `#archived` — 隱藏筆記於預設搜尋結果
- `#color` — 變更筆記在樹狀結構中的顏色
- `#iconClass` — 自訂筆記圖示
- `#template` — 標記筆記為範本
- `#readOnly` — 設為唯讀
- `#bookmarked` — 加入書籤
- `#workspace` — 工作區管理
- `#keyboardShortcut` — 快速鍵跳轉
- `#geolocation` — 地理定位
- `#viewType` — 設定子筆記的網格或清單檢視
- `#cssClass` — 進階主題自訂 CSS
- `#disableVersioning` — 關閉版本紀錄
- `#calendarRoot` — 設定日記根節點

### 推廣屬性（Promoted Attributes）

這是 Trilium 標籤系統的進階功能。透過定義「屬性定義」（attribute definition），可以將一般標籤轉換為表單化的編輯介面，支援以下欄位類型[^promoted-docs]：

- 文字（Text）
- 多行文字（Multi-line Text）
- 數字（Number，可設精確度）
- 布林值（Boolean，核取方塊）
- 下拉選單（Select）
- 日期（Date）
- 日期時間（Date & Time）
- 時間（Time）
- 網址（URL）
- 電子郵件（Email）
- 電話（Phone）
- 顏色（Color）
- 關聯（Relation，指向另一則筆記）

### 標籤定義（Label Definitions）

進階用法中，標籤可以用來描述其他標籤的行為[^attr-docs]。例如：

```
#label:myColor(inheritable)="promoted,alias=Color,multi,color"
```

此行定義了 `myColor` 標籤的類型別名、多重值支援、推廣行為與顏色選擇器行為。

## 階層性：扁平標籤 vs 樹狀繼承

### 標籤本身是扁平的

Trilium 沒有像 `parent:child:grandchild` 這類巢狀標籤語法。標籤名稱是單純的字串，不具備內建階層結構。

### 但擁有強大的樹狀繼承機制

Trilium 透過**三種屬性繼承機制**實現事實上的階層化標籤管理[^inheritance-docs]：

1. **標準繼承（Standard Inheritance）**：在標籤上設定 `isInheritable` 旗標（即 `#archived(inheritable)`），該標籤會自動傳遞給所有**子孫筆記**。例如，在資料夾筆記上標記 `#archived(inheritable)` 可隱藏整個子樹。

2. **複製繼承（Copying Inheritance）**：使用 `child:` 前綴（如 `#child:foo`），新建立的子筆記會**複製**該標籤（獨立副本，修改不影響父層）。可鏈式使用：`#child:child:child:foo` 會在第一層子代產生 `#child:child:foo`，第二層產生 `#child:foo`，第三層產生 `#foo`。

3. **範本繼承（Template Inheritance）**：透過 `#template` 標籤定義的範本筆記，其所有標籤會自動繼承給從該範本建立的所有筆記。

## 與傳統筆記軟體的比較

| 軟體 | 標籤機制 | 階層標籤 | 結構化值 | 繼承 |
|---|---|---|---|---|
| Trilium | Labels（#前綴） | ❌ 扁平 | ✅ 鍵值對 | ✅ 三種繼承 |
| Obsidian | Tags（#tag） | ✅ `/` 巢狀 | ❌ 純標籤 | ❌ 無 |
| Logseq | Tags/Page references | ✅ 命名空間 | ❌ 純標籤 | ❌ 無 |
| Joplin | Tags | ❌ 扁平 | ❌ 純標籤 | ❌ 無 |
| Notion | Database properties | ✅ 可關聯 | ✅ 多欄位類型 | ✅ Template |

## 結論

Trilium（TriliumNext）具備完整且功能強大的標籤/標記系統，雖然官方術語使用「labels」而非「tags」，但其本質就是通用的標記功能。相較於傳統筆記軟體的純標籤分類，Trilium 的優勢在於鍵值對結構化資料、可繼承性、推廣屬性的表單化 UI，以及超過 50 個系統內建標籤。然而，如果使用者需要的是巢狀/階層式標籤名稱（如 Obsidian 的 `#project/task/urgent`），Trilium 原生不支援此類語法，需透過樹狀筆記結構搭配繼承機制來達成類似效果。

---

[^attr-docs]: TrilimeNext. (n.d.). *Attributes*. Retrieved 2026-10-09, from https://triliumnext.github.io/Docs/Wiki/attributes.html

[^labels-docs]: TriliumNotes. (n.d.). *Labels*. Retrieved 2026-10-09, from https://docs.triliumnotes.org/user-guide/advanced-usage/attributes/labels

[^promoted-docs]: TrilimeNext. (n.d.). *Promoted Attributes*. Retrieved 2026-10-09, from https://triliumnext.github.io/Docs/Wiki/promoted-attributes.html

[^inheritance-docs]: TrilimeNext. (n.d.). *Attribute Inheritance*. Retrieved 2026-10-09, from https://triliumnext.github.io/Docs/Wiki/attribute-inheritance.html