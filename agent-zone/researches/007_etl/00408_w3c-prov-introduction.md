# W3C PROV：網路來源追蹤標準

## 概述

W3C PROV 是 W3C 於 2013 年發布的一套標準系列，用於在網路上**表述與交換來源資訊（provenance information）**。W3C 將 provenance 定義為：

> 「一份記錄，描述在產生、影響或傳遞某項資料或事物過程中所涉及的人員、機構、實體與活動。」[^prov-dm]

來源資訊可用於評估資料的品質、可靠性與可信度。PROV 的設計目標是在**異質系統之間達成可互通性（interoperability）**。

## PROV 標準家族

PROV 共包含 12 份文件，依受眾分為三個層級[^prov-overview]：

| 文件 | 名稱 | 說明 | 類型 |
|------|------|------|------|
| 1 | PROV-PRIMER | 入門導覽與教學 | Note |
| 2 | **PROV-O** | OWL2 本體對映至 RDF | **Recommendation** |
| 3 | PROV-XML | 資料模型的 XML 綱要 | Note |
| 4 | **PROV-DM** | 概念資料模型（核心規格） | **Recommendation** |
| 5 | **PROV-N** | 人類可讀的文字記法 | **Recommendation** |
| 6 | **PROV-CONSTRAINTS** | 有效性約束與推理規則 | **Recommendation** |
| 7 | PROV-AQ | HTTP 存取與查詢機制 | Note |
| 8 | PROV-DC | PROV-O 與 Dublin Core 的對映 | Note |
| 9 | PROV-DICTIONARY | 鍵-實體對集合 | Note |
| 10 | PROV-SEM | 一階邏輯形式語意 | Note |
| 11 | PROV-LINKS | 跨 Bundle 的連結 | Note |
| 12 | PROV-OVERVIEW | 本文件本身 | Note |

所有 PROV 詞彙的命名空間為 `http://www.w3.org/ns/prov#`（前綴：`prov`）。

## 核心資料模型（PROV-DM）

PROV-DM 圍繞三種基本節點類型與它們之間的關係，分為 **6 個元件**[^prov-dm][^prov-readthedocs]。

### 三大核心概念

| 概念 | 定義（來自 PROV-DM） | 例 |
|------|---------------------|-----|
| **Entity**（實體） | 具有某種固定面向的物理、數位、概念或其他事物 | 檔案、文件、資料集、物理物件 |
| **Activity**（活動） | 在時間跨度內發生、對實體產生作用的事物 | 執行程式、編輯文件、發布 |
| **Agent**（代理） | 對活動的發生、實體的存在或其他代理的活動承擔某種責任的事物 | 人物、組織、軟體 |

這三者對應三種**視角**：
- **資料流視角** — 實體之間的衍生關係（`wasDerivedFrom`）
- **流程視角** — 活動使用與產生實體（`used`、`wasGeneratedBy`）
- **責任視角** — 代理對活動/實體的責任歸屬（`wasAttributedTo`、`wasAssociatedWith`）

---

### 元件 1：實體與活動（時間骨幹）

| PROV 概念 | 方向 | 說明 |
|-----------|------|------|
| **Generation**（`wasGeneratedBy`） | Entity ← Activity | 實體由某活動產生 |
| **Usage**（`used`） | Activity → Entity | 活動使用了某實體 |
| **Communication**（`wasInformedBy`） | Activity ← Activity | 一活動受另一活動通知 |
| **Start**（`wasStartedBy`） | Activity ← Entity | 活動由某實體觸發開始 |
| **End**（`wasEndedBy`） | Activity ← Entity | 活動由某實體觸發結束 |
| **Invalidation**（`wasInvalidatedBy`） | Entity ← Activity | 實體被某活動消滅 |

### 元件 2：衍生（Derivation）

| PROV 概念 | 說明 |
|-----------|------|
| **Derivation**（`wasDerivedFrom`） | 一實體從另一實體衍生而來 |
| **Revision**（`wasRevisionOf`） | 特化—某一版本為另一版本的修訂 |
| **Quotation**（`wasQuotedFrom`） | 特化—內容引用自另一實體 |
| **Primary Source**（`hadPrimarySource`） | 特化—基於某原始來源 |

### 元件 3：代理、責任與影響

| PROV 概念 | 方向 | 說明 |
|-----------|------|------|
| **Attribution**（`wasAttributedTo`） | Entity → Agent | 實體的存在歸因於某代理 |
| **Association**（`wasAssociatedWith`） | Activity → Agent | 活動與某代理相關聯 |
| **Delegation**（`actedOnBehalfOf`） | Agent → Agent | 代理代表另一代理行事 |
| **Influence**（`wasInfluencedBy`） | 通用 | 某事物影響了另一事物（最通用的關係） |
| 代理子類型 | — | `Person`、`Organization`、`SoftwareAgent`（透過 `prov:type` 表達） |

### 元件 4：Bundle（綑綁）

**Bundle** 是「一組具名的來源描述，且本身即為實體，從而允許表述**來源的來源（provenance of provenance）**」。這使得 Accountability 與信賴成為可能——可以記錄誰做了哪些 PROV 斷言。

### 元件 5：替代實體

| PROV 概念 | 說明 |
|-----------|------|
| **Specialization**（`specializationOf`） | 實體的更特定版本 |
| **Alternate**（`alternateOf`） | 表述同一事物不同面向的實體 |
| **Mention**（`mentionOf`） | 同時指名描述該特定實體的 Bundle 的特化 |

### 元件 6：集合（Collection）

| PROV 概念 | 說明 |
|-----------|------|
| **Collection**（類型 `prov:Collection`） | 包含其他實體的集合實體 |
| **Membership**（`hadMember`） | 連接集合與其成員的關係 |
| **EmptyCollection**（類型 `prov:EmptyCollection`） | 空集合 |

## PROV 序列化格式

PROV 陳述可用多種可互換的格式表達[^prov-overview]：

| 格式 | 說明 |
|------|------|
| **PROV-N** | 人類可讀文字記法（如 `entity(ex:e1)`） |
| **PROV-O (RDF/Turtle)** | OWL2 本體映射至 RDF / 語意網 |
| **PROV-XML** | 基於 XML 綱要的序列化 |
| **PROV-JSON** | JSON 序列化 |
| **PROV-JSONLD** | JSON-LD 序列化 |

## 典型範例：新聞文章來源追蹤

以下是 W3C PROV Primer 中的經典範例——一篇名為 "Crime rises in cities" 的新聞文章，以 **PROV-O (Turtle)** 表述[^prov-tutorial]：

```turtle
@prefix prov: <http://www.w3.org/ns/prov#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix foaf: <http://xmlns.com/foaf/0.1/> .
@prefix ex: <http://www.example.org#> .

ex:article      a prov:Entity ;
               dcterms:title "Crime rises in cities" .
ex:dataset1     a prov:Entity .
ex:regionList   a prov:Entity .
ex:composition1 a prov:Entity .
ex:chart1       a prov:Entity .

ex:compose1    a prov:Activity .
ex:illustrate1 a prov:Activity .

ex:compose1     prov:used           ex:dataset1 , ex:regionList .
ex:composition1 prov:wasGeneratedBy ex:compose1 .
ex:illustrate1  prov:used           ex:composition1 .
ex:chart1       prov:wasGeneratedBy ex:illustrate1 .

ex:derek a prov:Agent , prov:Person ;
         foaf:givenName "Derek" .
ex:compose1    prov:wasAssociatedWith ex:derek .
ex:chart1      prov:wasAttributedTo  ex:derek .
ex:derek       prov:actedOnBehalfOf  ex:chartgen .
ex:chartgen    a prov:Agent , prov:Organization ;
               foaf:name "Chart Generators Inc" .
```

**語意**：Derek（人物）受僱於 Chart Generators Inc.（組織）。他執行 composing 活動，使用 dataset1 與 regionList，產生 composition1；接著執行 illustrating 活動，產生 chart1。圖表 chart1 歸屬於 Derek。

## 應用案例

| 領域 | 案例 | 使用的 PROV 特性 |
|------|------|-------------------|
| **學術出版** | W3C Primer 新聞文章創作流程 | Entity、Activity、Agent、`wasDerivedFrom`、`wasRevisionOf` |
| **科學工作流程** | Taverna / DataONE 的 D-PROV 擴展[^prov-tutorial] | 工作流程來源追溯、資料衍生鏈 |
| **材料科學** | MatPROV 資料集（NeurIPS 2025）——從文獻中提取合成流程[^matprov] | PROV-DM 規模化應用、LLM 提取 |
| **跨資料來源溯源** | DBpedia ↔ Wikipedia 來源整合[^prov-tutorial] | `wasDerivedFrom` 跨來源連結 |
| **版本控制** | git2prov——Git 倉庫 → PROV[^prov-tutorial] | Activity = commit、Agent = author |
| **AI/ML 管線** | MLflow + PROV-O 整合[^ml-prov] | 完整訓練歷程溯源、SPARQL 稽核查詢 |
| **能源電網** | Apache Atlas + PROV-O（CAISO、PJM、ERCOT）[^energy-prov] | FERC/NERC 法規遵循、即時來源驗證 |
| **政府紀錄** | 英國 Gazette 官方公報來源追蹤[^gazette] | 責任歸屬、發布稽核軌跡 |

## 核心設計原則

1. **斷言的來源（Asserted provenance）** — PROV 記錄的是某人對發生了什麼的描述，而非絕對真理。衝突的斷言可以並存。
2. **互通性（Interoperability）** — 同一模型適用於 RDF、XML、JSON、文字格式。
3. **可擴展性（Extensibility）** — 子類型與特化允許領域特定的擴展。
4. **來源的來源（Provenance of provenance）** — Bundle 機制允許表述誰說了什麼關於來源的話。
5. **命名空間識別碼** — 所有識別碼均為 IRI，實現全球互通。

## PROV 的定位：不追蹤實體內部狀態

PROV 專注於**起源與因果關係**（誰做了什麼、用了什麼、產生了什麼），而非實體內部狀態的逐步變化。它不記錄「Entity 在時間 t 的值為何」（那是日誌或事件溯源的工作），而是回答「這個 Entity 是經由什麼過程產生的」。因此 PROV 最適合用於描述**資料轉換鏈**、**工作流程**與**責任歸屬**。

## 參考文獻

[^prov-overview]: W3C. (2013). PROV-OVERVIEW: An Overview of the PROV Family of Documents. Retrieved 2026-10-01, from https://www.w3.org/TR/prov-overview/
[^prov-dm]: W3C. (2013). PROV-DM: The PROV Data Model. Retrieved 2026-10-01, from https://www.w3.org/TR/prov-dm/
[^prov-readthedocs]: PROV Python Library. (n.d.). PROV Data Model Explanation. Retrieved 2026-10-01, from https://prov.readthedocs.io/en/latest/explanation/prov-dm.html
[^prov-tutorial]: Groth, P. (n.d.). PROV Tutorial. Retrieved 2026-10-01, from https://github.com/pgroth/PROVTutorial
[^ml-prov]: Ranjan Kumar. (n.d.). Provenance in AI: Auto Capturing Provenance with MLflow and W3C PROV-O. Retrieved 2026-10-01, from https://ranjankumar.in/provenance-in-ai-auto-capturing-provenance-with-mlflow-and-w3c-prov-o-in-pytorch-pipelines-part-4
[^matprov]: MatPROV Project. (2025). MatPROV: PROV-DM for Material Synthesis from Scientific Literature. Retrieved 2026-10-01, from https://github.com/MatPROV-project/matprov-experiments
[^energy-prov]: ToolFusion. (n.d.). Data Provenance Tracking: W3C PROV-O & Apache Atlas. Retrieved 2026-10-01, from https://energy.toolfusion.net/data-provenance-tracking-w3c-prov-o-apache-atlas.html
[^gazette]: Moreau, L. (n.d.). Provenance in the Wild: Provenance at the Gazette. Retrieved 2026-10-01, from https://www.w3.org/2011/prov/wiki/Provenance_in_the_Wild