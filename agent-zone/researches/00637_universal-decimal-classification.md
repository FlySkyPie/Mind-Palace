# 通用十進位分類法（Universal Decimal Classification, UDC）介紹

## 摘要

通用十進位分類法（Universal Decimal Classification, UDC，法文原名 Classification Décimale Universelle, CDU）是一套國際通用的文獻與圖書分類系統，用來系統性組織人類知識的所有分支。它由比利時文獻學家 Paul Otlet 與 Henri La Fontaine 於十九世紀末、以 Melvil Dewey 的《杜威十進位分類法》（Dewey Decimal Classification, DDC）為基礎發展而來，並於 1905 年首次以法文出版[^udcwiki1]。UDC 最大的特色在於它是全世界第一套「分析－合成式」（analytico-synthetic，亦稱「分面式／faceted」）分類法，允許使用者將複雜主題拆解成若干成分，再以符號重新組合成精確的分類號[^udccstructure]。

本報告依序介紹 UDC 的歷史沿革、與 DDC 的關係、維護機構、標記法結構、主類表、輔助表、連結符號、特性、使用案例、授權與現行版本，並比較 UDC 與 DDC、LCC 之差異。所有內容皆基於網路檢索所得之公開來源，並以標準 Markdown 註腳搭配修改版 APA 格式（使用 ISO 8601 日期）標註來源。

---

## 一、歷史沿革

### 1.1 起源（1895 年）

UDC 由比利時文獻學家兼律師 **Paul Otlet**（1868–1944）與 **Henri La Fontaine**（1854–1943）共同發展。1895 年，兩人建立了「世界書目索引」（Universal Bibliographic Repertory，法文 Répertoire Bibliographique Universel, RBU），其目標是為所有已出版資訊編製一份完整的分类索引。為了排列索引條目，他們需要一套分類系統[^udchistory]。

Otlet 得知 Melvil Dewey 於 1876 年首次出版的 DDC，便於 1895 年寫信給 Dewey，請求允許將 DDC 翻譯成法文。Dewey 同意授權，但該計畫很快便超出單純翻譯的範疇：Otlet 與 La Fontaine 做出了革命性的創新，將 Dewey 原本純粹「列舉式」（enumerative）的分類，改造成「分析－合成式」（faceted）系統，成為世界第一套此類分類法[^udcwiki1][^udchistory]。

### 1.2 第一版（1902–1907 年）

第一版名為《Manuel du Répertoire Bibliographique Universel》（世界書目索引手冊），由比利時皇家圖書館（Koninklijke Bibliotheek, KB）於 **1902 至 1907 年**間以法文出版，收錄約 **33,000 個細分項**，並引入了具革命性的特徵：通用輔助表（common auxiliary tables）、特殊輔助表（special auxiliaries）以及表達關係的符號（合成標記法）[^udchistory]。

### 1.3 擴充與更名

- **1927–1933 年**：第二版法文版問世，由荷蘭專利局的 Frits Donker Duyvis 共同擔任編輯，細分項超過 **70,000 個**。此版本獨立發行，並採用《Classification Décimale Universelle》（CDU）之名，此名稱至今仍見於各種語言的翻譯版本[^udchistory]。
- **1934–1951 年**：第三版德文版，由 Carl Walther 主編，細分項達約 **140,000 個**[^udchistory]。
- 自 1933 至 1993 年，第二版法文版（連同《Extensions and Corrections to the UDC》中核准的修訂）一直是所有 UDC 版本的官方來源[^udchistory]。

### 1.4 機構沿革

- **1895 年**：Otlet 與 La Fontaine 創立「國際書目學會」（Institut International de Bibliographie, IIB）。
- **1931 年**：IIB 更名為「國際文獻學會」（Institut International de Documentation, IID），總部遷至海牙。
- **1937 年**：改組為「國際文獻聯盟」（Fédération Internationale de Documentation, FID）。
- **1988 年**：更名為「國際資訊與文獻聯盟」（International Federation for Information and Documentation, FID），並於 2000 年解散[^udchistory]。
- **1992 年 1 月 1 日**：**UDC 聯盟（UDC Consortium, UDCC）** 正式接手 UDC 的所有權，由 FID、BSI 及其他四家出版商為創始成員[^udchistory]。

### 1.5 聯合國教科文組織「世界記憶」

由 UDC 編排的世界書目索引目錄現存於比利時蒙斯（Mons）的 **Mundaneum** 博物館，並於 2013 年被收錄進聯合國教科文組織（UNESCO）「世界記憶」（Memory of the World）國際名錄[^udchistory]。

---

## 二、與杜威十進位分類法（DDC）的關係

UDC 直接衍生自 DDC（名稱中的「十進位」即源於此），兩者關係密切：

- DDC 的分類號是 UDC 分類號的基礎，尤其是在 0–9 各大類的原始架構上[^udcwiki1]。
- UDC 原本僅計畫作為 DDC 的法文翻譯，最終卻成為一套被徹底重新設計的系統[^udchistory]。
- UDC 有別於 DDC 的幾項重大創新包括[^udcwiki1][^udccstructure]：
  1. **分析－合成式（分面式）結構**：DDC 主要為列舉式；UDC 則允許概念無限組合。
  2. **關係符號**（冒號、加號等）：DDC 不使用這類符號。
  3. **通用輔助表**（語言、地區、時間、形式等），可跨所有主題重複使用。
  4. **各主類中專屬的特殊輔助表**。
  5. **具階層與語法表達力的標記法**，便於機器判讀與解析。
  6. **自始即以國際為範疇**：DDC 最初是為美國設計的系統。

維基百科的 DDC 條目亦指出：「杜威十進位分類法的分類號構成了通用十進位分類法的基礎，後者將基本的杜威分類號與選定的標點符號（逗號、冒號、括號等）相結合。」[^ddcwiki]

---

## 三、維護機構：UDC 聯盟（UDCC）

- **名稱**：UDC Consortium（UDCC）
- **性質**：非營利國際出版商協會
- **總部**：荷蘭海牙，荷蘭皇家圖書館（Koninklijke Bibliotheek）[^udcudcc]
- **成立**：1992 年 1 月 1 日[^udchistory]
- **版權所有者**：UDC Consortium
- **官方語言**：英語
- **現任主席**：Irena Sešek（斯洛維尼亞）
- **現任副主席**：Ana Stevanović（塞爾維亞）
- **主編**：Aida Slavic（英國）[^udcudcc]

UDCC 目前有九個成員機構[^udcudcc]：

1. AENOR International S.A.U.（西班牙）
2. 全俄科學技術資訊研究所 VINITI（俄羅斯）
3. 捷克共和國國家圖書館
4. 斯洛伐克國家圖書館
5. 盧布爾雅那國家與大學圖書館（斯洛維尼亞）
6. 薩格勒布國家與大學圖書館（克羅埃西亞）
7. 波蘭國家圖書館
8. 塞切尼國家圖書館（匈牙利）
9. 塞爾維亞國家圖書館

聯盟負責任命 **UDC 編輯團隊**與 **UDC 諮詢委員會**（由國際分類學專家組成的機構），以監督內容與修訂工作[^udcudcc]。

---

## 四、結構與標記法

### 4.1 分類類型

UDC 屬於 **分析－合成式（分面式）** 分類法。不同於純列舉式系統（如傳統 DDC 或 LCC），UDC 允許主題的屬性與主題之間的關係進行無限組合[^udccstructure]。

### 4.2 標記法（Notation）

- **標記類型**：阿拉伯數字以十進位方式排列，並搭配標點符號與各種符號[^udcnotation]。
- **語言中立**：標記法使用普遍可辨識的符號（阿拉伯數字與常見標點），不受語言限制[^udcnotation]。
- **具階層表達力**（hierarchically expressive）：數字越長代表類別越專精；刪去末幾位數字即可還原為較廣的類別[^udcnotation]。
- **具語法表達力**（syntactically expressive）：標點符號會中斷數字序列，表示該表達式是由多個類別組合而成，而非單一類別[^udcnotation]。
- **排架順序**：每個數字都被視為一個省略了小數點的十進位小數，藉此決定排序[^udcwiki1]。
- **可讀性標點**：為方便閱讀，每三位數字後插入一個點（例如 `539.120.811`）[^udcwiki1]。

### 4.3 主類表（0–9）

UDC 將人類知識分為 **10 個主類**（其中第 4 類為空號）[^udcwiki1]：

| 類號 | 描述 |
|------|------|
| **0** | 科學與知識。組織。電腦科學。資訊科學。文獻學。圖書館學。機構。出版物 |
| **1** | 哲學。心理學 |
| **2** | 宗教。神學 |
| **3** | 社會科學 |
| **4** | *（空號）* —— 1960 年代將語言學移至第 8 類後空出 |
| **5** | 數學。自然科學 |
| **6** | 應用科學。醫學。技術 |
| **7** | 藝術。娛樂。體育 |
| **8** | 語言學。文學 |
| **9** | 地理。傳記。歷史 |

> **關於第 4 類**：此類於 1960 年代被空出，方法是將原本置於第 4 類的語言學移入第 8 類，藉此為成長迅速的領域保留未來的擴充空間[^udccstructure]。

每個主類會再階層式細分。例如 `539.120.811` 由下列階層衍生而來：5（自然科學）→ 53（物理學）→ 539（物質的物理本質）→ 539.12（基本粒子）→ 539.120（理論問題）→ 539.120.8（強交互作用）→ 539.120.81（量子色動力學）→ 539.120.811（格點 QCD）[^udcwiki1]。

### 4.4 兩種表（Tables）

**A. 主表（Main Tables / Schedules）**

包含各知識學科，以 0–9 十個類別階層式劃分，收錄 **超過 60,000 個細分項**。每個主類開頭可能設有 **特殊輔助表**（special auxiliary tables），用以表達該主題範圍內反覆出現的面向；這些特殊輔助標記以三種指示符之一開頭：`.0`（點零）、`-`（連字號）、`'`（撇號）[^udccstructure]。

**B. 通用輔助表（Common Auxiliary Tables）**

包含可套用於整個分類體系的通用概念，共有 **超過 15,000 個通用輔助項**[^udcwiki2]：

| 指示符 | 表 | 概念 |
|--------|-----|------|
| `=...` | 1c | 語言 |
| `(0...)` | 1d | 文獻形式 |
| `(1/9)` | 1e | 地區 |
| `(=...)` | 1f | 人種、族群、國籍 |
| `"..."` | 1g | 時間 |
| `-02` | 1k | 性質 |
| `-03` | 1k | 材料 |
| `-04` | 1k | 關係、過程、操作 |
| `-05` | 1k | 人物與個人特徵 |

### 4.5 通用輔助項範例

- **語言（`=...`）**：`=111` 英語；`=112.2` 德語；`=161.1` 俄語[^udcwiki1]。
- **形式（`(0...)`）**：`(075)` 教科書；`(03)` 參考書；`(084.3)` 地圖；`(0.034.2)` 數位檔案[^udcwiki1]。
- **地區（`(1/9)`）**：`(410)` 英國；`(540)` 印度；`(73)` 美國；`(4)` 歐洲[^udcwiki1]。
- **族群／國籍（`(=...)`）**：`(=111)` 英語族群；`(=111:71)` 加拿大英語人口[^udcwiki1]。
- **時間（`"..."`）**：`"19"` 二十世紀；`"1993-1996"` 1993 至 1996 年；`"327.502"` 二月[^udcwiki1]。
- **一般特徵（`-0...`）**：`-02` 性質（`-021` 存在性、`-022` 量值、`-023` 形狀）；`-03` 材料（`-034` 金屬、`-035` 有機材料、`-036` 塑膠）；`-04` 關係與過程（`-042` 相態關係、`-043` 一般過程）；`-05` 人物（`-051` 作為行為者的人、`-053` 依年齡區分的人、`-055` 依性別區分的人）[^udcwiki1]。

### 4.6 連結符號（Connecting Signs）

| 符號 | 名稱 | 意義 | 範例 |
|------|------|------|------|
| `+` | 加號 | 並列（AND，兩主題） | `59+636` —— 動物學與動物飼養 |
| `/` | 斜線 | 連續延伸（範圍） | `592/599` —— 系統動物學 |
| `:` | 冒號 | 簡單關係 | `17:7` —— 倫理學與藝術的關係 |
| `::` | 雙冒號 | 固定順序的關係 | `311:[622+669](485)` |
| `[ ]` | 方括號 | 分組 | `311:[622+669](485)` —— 瑞典的礦業與冶金統計 |
| `*` | 星號 | 引入非 UDC 標記 | `523.4*433` —— 行星學，小行星 Eros |
| `A/Z` | 字母延伸 | 直接以字母標示 | `821.133.1MOL` —— 法國文學，莫里哀作品 |

### 4.7 複雜 UDC 表達式範例

`338.48(734.211.4)"197"(084.3)(0.034.2PDF)`

解析[^udcwiki1]：

- `338.48` —— 觀光（主類號）
- `(734.211.4)` —— 地區：美國緬因州格拉夫頓郡
- `"197"` —— 時間：1970 年代
- `(084.3)` —— 形式：地圖
- `(0.034.2)` —— 形式：數位檔案（PDF）

整串意義為：**美國緬因州格拉夫頓郡 1970 年代的觀光地圖（PDF 檔）**。

---

## 五、特性

UDC 的主要特性如下[^udcfactsheet]：

1. **國際性範疇** —— 自創始即設計為國際系統，而非先有國家系統再加以改造。
2. **分析－合成式** —— 結合「前組合」（pre-coordinated）與「後組合」（post-coordinated）索引。
3. **分面式** —— 允許主題屬性與關係無限組合。
4. **語言中立** —— 標記以阿拉伯數字與通用標點構成。
5. **可無限擴充** —— 十進位標記允許在不更動既有分類的情況下新增細分。
6. **多語言** —— 已以 40 種以上語言出版；UDC 摘要版提供 57 種以上語言。
7. **普遍適用** —— 涵蓋人類知識的所有分支。
8. **機器可讀** —— 具嚴謹語法規則，便於自動化解析。
9. **以學科為基礎** —— 概念置於研究該概念所屬的領域之下。

---

## 六、使用案例

- **圖書館**：各類型圖書館（公共、學校、學術、專門）的排架與內容索引[^udcfactsheet]。
- **國家書目**：約有 30 個國家採用[^udcfactsheet]。
- **書目資料庫**：對文章、書籍等資源進行詳細主題索引。
- **文獻服務、資訊入口網站、線上資料庫**。
- **出版**：常見於科學期刊文章索引。

### 使用統計

- 全球 **135 個國家**使用[^udcfactsheet]。
- 約 **150,000 至 250,000** 個文獻典藏。
- 全球約 **150,000 所圖書館**採用[^udcfactsheet]。

### 採用 UDC 的大型資料庫

| 資料庫 | 記錄數 |
|--------|--------|
| VINITI RAS（俄羅斯） | 2,800 萬 |
| COBIB.SI（斯洛維尼亞） | 350 萬 |
| 匈牙利全國聯合目錄（MOKKA） | 290 萬 |
| NEBIS（瑞士） | 260 萬 |
| PORBASE（葡萄牙） | 150 萬 |

---

## 七、授權

UDC 內容的版權屬於 **UDC Consortium**。官方政策為[^udclicences]：

**無需授權的情況：**

- 依已出版之紙本或線上服務套用 UDC，供自身索引或組織之用。
- 以人工方式應用於排架、目錄與資訊檢索。
- 散布《多語言 UDC 摘要》（Multilingual UDC Summary）中的 **2,600 個類別**（該摘要以 **創用 CC 姓名標示－相同方式分享 3.0（CC BY-SA 3.0）**授權釋出）。

**需要授權的情況：**

- 以任何語言或媒介出版 UDC 系統（任何部分）。
- 使用 UDC 主參考檔（Master Reference File, MRF）資料。

**兩種標準授權類型：**

1. **UDC 出版授權**（UDC Publishing Licence）—— 供出版與散布 UDC 之用。
2. **UDC MRF 使用授權**（UDC MRF Use Licence）—— 供內部參考、出版前作業與研究專案之用。

---

## 八、現行版本

- **現行權威版本**：**UDC MRF 12**（Master Reference File 第 12 版）[^udcmrf]。
- **發布日期**：2018 年 12 月[^udcmrf]。
- **下一版本**：**UDC MRF 13**（預計 2026 年發布）[^udcmrf]。
- **規模**：現行 MRF 約 **72,000 個細分項**；歷史上完整紙本版曾達約 220,000 個細分項[^udcmrf]。
- **更新頻率**：每年於《Extensions and Corrections to the UDC》（ISSN 0014-5424）發布更新[^udcmrf]。
- **資料庫格式**：以 MySQL 維護，並以 tagged text 與 XML 匯出格式散布[^udcmrf]。
- **線上取用**：透過 **UDC Hub**（付費訂閱），提供英語、法語、西班牙語、荷蘭語、克羅埃西亞語、捷克語、波蘭語、塞爾維亞語、斯洛伐克語、斯洛維尼亞語等[^udcmrf]。
- **免費取用**：《多語言 UDC 摘要》，含 57 種以上語言的 2,600 個類別（CC BY-SA 3.0）[^udcmrf]。
- **鏈結資料（Linked Data）**：udcdata.info[^udcmrf]。

---

## 九、與 DDC、LCC 之差異

### 9.1 UDC 與 DDC

| 面向 | UDC | DDC |
|------|-----|-----|
| **類型** | 分析－合成式（分面式） | 主要為列舉式 |
| **標記法** | 十進位數字 + 標點符號表達關係 | 純十進位數字 |
| **主題關係** | `+`、`:`、`[ ]`、`*`、`A/Z` 等，可表達複雜關係 | 關係表達能力極為有限 |
| **輔助表** | 完整的通用輔助表（地區、時間、語言、形式、人物、材料、性質） | 輔助表有限（主要為地區與文學形式） |
| **起源** | 國際（比利時） | 美國 |
| **主要用途** | 書目／詳細索引與排架 | 主要用於排架 |
| **階層表達** | 完全具表達力 | 部分具表達力 |
| **語言** | 40 種以上語言；標記語言中立 | 主要為英語 |
| **維護機構** | UDC Consortium（非營利） | OCLC（營利） |
| **第 4 類** | 空號（保留擴充用） | 用於語言（400s） |

### 9.2 UDC 與 LCC

| 面向 | UDC | LCC |
|------|-----|-----|
| **類型** | 分析－合成式（分面式） | 列舉式 |
| **標記法** | 阿拉伯數字（十進位）+ 標點 | 字母 + 數字（英數混合） |
| **理論基礎** | 具強烈的理論／知識論基礎 | 務實、以館藏為導向的設計 |
| **合成能力** | 概念可無限組合 | 合成能力極為有限 |
| **大類數** | 10 個主類（0–9） | 21 個主類（A–Z，省略 I、O、W、X、Y） |
| **語言** | 已以 40 種以上語言出版 | 主要為英語 |
| **範疇** | 自始即為國際性 | 為美國國會圖書館館藏設計，以美國為中心 |
| **使用地區** | 130 個以上國家 | 主要為美國大型學術／研究圖書館 |
| **標記表達力** | 具完整階層與語法表達力 | 不具表達力，標記號具任意性 |
| **機器可讀性** | 內建語法規則，可自動解析 | 有限，無語法結構 |
| **維護機構** | UDCC（非營利） | 美國國會圖書館（美國政府） |

### 9.3 關鍵差異總結

1. **UDC 是唯一自創始即具國際性的系統** —— DDC 與 LCC 皆為美國系統，之後才被改造以適用於國際。
2. **UDC 開創分面／分析－合成式分類** —— 它是世界第一套此類系統，並啟發了後來的分面分類法（如 Bliss、冒號分類法）。
3. **UDC 的語法標記法** 使其可進行機器解析與電腦化檢索，能力超越 DDC 與 LCC。
4. **UDC 的專精度更高** —— 其輔助表允許遠比 DDC 更高的專指性；LCC 則刻意限制於美國國會圖書館館藏範圍。
5. **UDC 使用純數字 + 標點標記**，無字母標記，因而真正做到語言中立；LCC 使用字母（偏向英語），DDC 使用數字但不含語法符號。

---

## 十、來源評析

本報告主要依據 **UDC Consortium（UDCC）官方網站**，該機構為 UDC 的版權所有者與維護者，屬第一手權威來源，可信度最高[^udcfactsheet][^udchistory][^udcstructure][^udcnotation][^udcudcc][^udcmrf][^udclicences]。補以 **維基百科**條目（UDC、DDC、LCC）作為交叉驗證與補充範例[^udcwiki1][^ddcwiki]，維基百科屬共筆來源，惟其內容與 UDCC 官方資料一致，可相互佐證。

需注意的限制：

- 部分統計數字（如使用國家數、圖書館數量、資料庫記錄數）來自 UDCC 自身的行銷性資料，可能存在正面偏誤，應視為官方宣稱值而非獨立查核數據。
- 第十三版 MRF 的發布時程（2026 年）係官方規劃值，實際發布日期可能變動。
- 本報告未引用各地圖書館的實務使用經驗，若需評估特定情境下的適用性，宜另行查證。

---

## 參考文獻

[^udcwiki1]: Wikipedia. (n.d.). *Universal Decimal Classification*. Retrieved 2026-10-24, from https://en.wikipedia.org/wiki/Universal_Decimal_Classification

[^udcwiki2]: LIS Academy. (n.d.). *Universal Decimal Classification: Library and Information Science*. Retrieved 2026-10-24, from https://lis.academy/information-processing-retrieval/universal-decimal-classification-library-information/

[^ddcwiki]: Wikipedia. (n.d.). *Dewey Decimal Classification*. Retrieved 2026-10-24, from https://en.wikipedia.org/wiki/Dewey_Decimal_Classification

[^udccstructure]: UDC Consortium. (n.d.). *UDC structure and tables*. Retrieved 2026-10-24, from https://udcc.org/index.php/site/page?view=about_structure

[^udcnotation]: UDC Consortium. (n.d.). *UDC notation*. Retrieved 2026-10-24, from https://udcc.org/index.php/site/page?view=about_notation

[^udchistory]: UDC Consortium. (n.d.). *UDC history*. Retrieved 2026-10-24, from https://udcc.org/index.php/site/page?view=about_history

[^udcudcc]: UDC Consortium. (n.d.). *About UDC Consortium*. Retrieved 2026-10-24, from https://udcc.org/index.php/site/page?view=about_udcc

[^udcfactsheet]: UDC Consortium. (n.d.). *UDC fact sheet*. Retrieved 2026-10-24, from https://udcc.org/index.php/site/page?view=factsheet

[^udcmrf]: UDC Consortium. (n.d.). *UDC Master Reference File*. Retrieved 2026-10-24, from https://udcc.org/index.php/site/page?view=mrf

[^udclicences]: UDC Consortium. (n.d.). *UDC licences*. Retrieved 2026-10-24, from https://udcc.org/index.php/site/page?view=licences

[^udcsummary]: UDC Consortium. (n.d.). *Multilingual UDC Summary*. Retrieved 2026-10-24, from http://www.udcsummary.info/php/index.php

[^udcdata]: UDC Consortium. (n.d.). *UDC linked data*. Retrieved 2026-10-24, from http://udcdata.info/
