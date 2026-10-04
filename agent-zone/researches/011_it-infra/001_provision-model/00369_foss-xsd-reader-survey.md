# FOSS XSD（XML Schema Definition）讀取工具調查

## 概觀

本報告調查自由開源（FOSS）的 XSD（XML Schema Definition）讀取工具、程式庫與解析器，涵蓋多種程式語言與用途（純解析、驗證、程式碼生成、視覺化）。

---

## 調查結果

### Python

#### 1. **xmlschema**（⭐ 476）

GitHub 上最完整的 Python XSD 1.0/1.1 實作。可將 XSD 檔案解析為 Schema 物件、驗證 XML 文件、將 XML 解碼為 Python dict/JSON，並支援 XPath 導覽與離線快取。[^xmlschema]

#### 2. **xsData**（⭐ 452）

專注於 XSD 到 Python dataclass 的資料綁定與程式碼生成。支援 XSD 1.0/1.1、WSDL 1.1、DTD，具有完整型別提示。CLI 工具 `xsdata generate` 可將 XSD 生成為 Python 程式碼。[^xsdata]

#### 3. **lxml**（⭐ 3.1k）

高效能 XML 處理程式庫（C binding 至 libxml2），提供 `etree.XMLSchema()` 進行 XSD 驗證。非專門的 XSD 解析器，但為 Python 生態中最廣泛使用的 XML/XSD 工具。[^lxml]

### Java

#### 4. **XsdParser**（xmlet）（⭐ 87）

專為解析 XSD 而生的 Java 程式庫，支援 42/42 種 XSD 1.0 元素，具備完整參考解析（`xs:ref`、`xs:group ref`）、Schema 層級約束驗證與 Visitor 模式。提供 Maven 套件 `com.github.xmlet:xsdParser`。[^xmlet]

### Rust

#### 5. **xsd-parser**（Bergmann89）（⭐ 88）

五階段管線架構（Parsing → Interpreting → Optimizing → Generating → Rendering），支援 `serde` 與 `quick_xml`，可從 XSD 生成 Rust 程式碼。效能表現優異。[^bergmann]

#### 6. **xsd-parser-rs**（lumeohq）（⭐ 116）

XSD/WSDL 轉 Rust 程式碼生成器，最初為 ONVIF 規格而設計，支援 SOAP/WSDL。[^lumeo]

### TypeScript / JavaScript

#### 7. **cxsd**（⭐ 113）

串流式 XSD 解析器兼 XML 解析器生成器。可自動下載所有匯入的 `.xsd` 檔案，輸出 `.js` 與 `.d.ts`（TypeScript 型別定義）。[^cxsd]

### Go

#### 8. **xgen**（⭐ 426）

XSD 解析器與多語言程式碼生成器，支援 **Go、C、Java、Rust、TypeScript** 五種語言。純 Go 實作，提供 CLI 與 Hook 介面。[^xgen]

#### 9. **xsd2go**（⭐ 110）

XSD 轉 Go struct 與 XML 解析器生成器，支援命名空間覆蓋。與 NIST SCAP 安全合規相關。[^xsd2go]

### Erlang

#### 10. **Erlsom**（⭐ 269）

提供 SAX 剖析器、DOM 剖析器與資料綁定器（XSD → Erlang records），三種模式因應不同場景：串流、泛用 DOM、XSD 驅動型別檢查。[^erlsom]

### 轉換工具

#### 11. **Jing/Trang**（⭐ 258）

Java 實作，可在 RELAX NG、DTD、XSD、Schematron 之間進行 Schema 格式轉換。[^trang]

### 視覺化 / GUI

純 XSD 視覺化的專門 FOSS 工具較少，常見做法有：

- **Eclipse IDE** — 內建 XML Schema 探索器/檢視器
- **VS Code** + Red Hat XML 擴充 — 提供 Schema 導覽
- **xmlschema** + Graphviz — 以 Python 生成 Schema 圖形
- **Spider**（ladybug-tools）— 瀏覽器端的 gbXML XSD 結構渲染工具[^spider]

---

## 用途推薦

| 需求 | 推薦工具 |
|---|---|
| 純粹解析 XSD 成可遍歷結構 | `xmlschema`（Python）、`XsdParser`（Java）、`Erlsom`（Erlang） |
| 從 XSD 生成程式碼 | `xsData`（Python）、`xgen`（多語言）、`xsd-parser`（Rust） |
| 驗證 XML 是否符合 XSD | `lxml`（Python）、`xmlschema`（Python） |
| XSD 轉 JSON / dict | `xmlschema`（內建 `.to_dict()`） |
| Schema 視覺化 | `xmlschema` + Graphviz、Eclipse、VS Code 擴充 |
| 跨語言程式碼生成 | `xgen`（Go → Go/C/Java/Rust/TS） |
| TypeScript / JS 生態 | `cxsd` |

---

## 參考

[^xmlschema]: sissaschool. (n.d.). xmlschema — XML Schema validation and conversion for Python. Retrieved 2026-10-01, from https://github.com/sissaschool/xmlschema

[^xsdata]: tefra. (n.d.). xsData — XML data binding and code generation for Python. Retrieved 2026-10-01, from https://github.com/tefra/xsdata

[^lxml]: lxml. (n.d.). lxml — XML and XSLT processing in Python (libxml2 bindings). Retrieved 2026-10-01, from https://github.com/lxml/lxml

[^xmlet]: xmlet. (n.d.). XsdParser — XSD parser for Java with full element support. Retrieved 2026-10-01, from https://github.com/xmlet/XsdParser

[^bergmann]: Bergmann89. (n.d.). xsd-parser — Rust XSD schema parser and code generator. Retrieved 2026-10-01, from https://github.com/Bergmann89/xsd-parser

[^lumeo]: lumeohq. (n.d.). xsd-parser-rs — Rust XSD/WSDL parser and code generator. Retrieved 2026-10-01, from https://github.com/lumeohq/xsd-parser-rs

[^cxsd]: charto. (n.d.). cxsd — Streaming XSD parser and JavaScript/TypeScript parser generator. Retrieved 2026-10-01, from https://github.com/charto/cxsd

[^xgen]: xuri. (n.d.). xgen — Multi-language code generator from XSD schema. Retrieved 2026-10-01, from https://github.com/xuri/xgen

[^xsd2go]: GoComply. (n.d.). xsd2go — XSD to Go struct and XML parser generator. Retrieved 2026-10-01, from https://github.com/GoComply/xsd2go

[^erlsom]: willemdj. (n.d.). Erlsom — XSD-driven XML data binding for Erlang. Retrieved 2026-10-01, from https://github.com/willemdj/erlsom

[^trang]: relaxng. (n.d.). jing-trang — RELAX NG, DTD, XSD, Schematron schema conversion. Retrieved 2026-10-01, from https://github.com/relaxng/jing-trang

[^spider]: ladybug-tools. (n.d.). spider — gbXML schema file viewer. Retrieved 2026-10-01, from https://github.com/ladybug-tools/spider