# 讀取 .rdf（Resource Description Framework）檔案的 FOSS 方案調查

## 摘要

`.rdf` 是 RDF（Resource Description Framework，資源描述框架）常見的副檔名，通常代表以 **RDF/XML** 語法序列化的資料檔[^rdfwiki]。要讀取這類檔案，開源社群累積了多個成熟且活躍的 FOSS（Free and Open Source Software）工具鏈，橫跨 Java、Python、JavaScript、C、Rust 等語言，從命令列轉檔工具、程式庫到圖形化檢視器皆有涵蓋。

本報告先說明 RDF 的資料模型與常見序列化格式，再依語言分類整理主流 FOSS 讀取／解析方案，最後提供選型建議。

## 什麼是 RDF

RDF 是一套用來描述與交換圖形資料（graph data）的方法，最初由 W3C 作為 metadata 的資料模型而設計，提供多種語法與格式，其中以 Turtle（Terse RDF Triple Language）最被廣泛使用[^rdfwiki]。

RDF 的核心是**三元組（triple）**，形式為 **主詞—述詞—受詞（subject–predicate–object）**：

- **主詞（Subject）**：被描述的資源，以 IRI（Internationalized Resource Identifier）或 blank node 表示。
- **述詞（Predicate）**：資源的性狀或關係，以 IRI 表示。
- **受詞（Object）**：關係的值，可以是 IRI、blank node，或 literal（字面值，如字串、數字、日期）[^rdfwiki]。

自 RDF 1.1 起，資源以 IRI 識別；IRI 是 URI 的泛化，允許國際化字元。主詞為 IRI 或 blank node，blank node 所指示的資源稱為匿名資源，無法直接從 RDF 語句辨識[^rdfwiki]。

RDF 是抽象的資料模型，具有多種具體序列化格式[^rdfwiki]：

| 格式 | 說明 | 常見副檔名 |
|---|---|---|
| RDF/XML | 最早的 W3C 標準序列化，XML 語法，有時被直接誤稱為「RDF」 | `.rdf`, `.owl` |
| Turtle | 緊湊、人類易讀的格式，現今最常見 | `.ttl` |
| N-Triples | 極簡、逐行、易解析的格式 | `.nt` |
| N-Quads | N-Triples 的擴充，可序列化多個圖 | `.nq` |
| TriG | Turtle 的擴充，支援資料集（多圖） | `.trig` |
| JSON-LD | 以 JSON 為基礎的序列化，W3C Recommendation | `.jsonld` |
| Notation3 (N3) | 類似 Turtle 的非標準格式，含推論規則 | `.n3` |
| RDF/JSON、TriX | 其他表示法 | `.rj`, `.trix` |

W3C 於 1999 年將 RDF 採納為 Recommendation；2004 年修訂為 RDF 1.0 系列，2014 年更新為現行的 **RDF 1.1** 系列文件，至今仍為現行 W3C Recommendation[^rdfwiki]。

## FOSS 讀取與解析方案總覽

下表的 GitHub 星數以 2026-10-24 的 GitHub API 查詢結果為準。

| 工具 | 語言 | 授權 | 用途 | 星數 |
|---|---|---|---|---|
| Apache Jena | Java | Apache 2.0 | RDF/SPARQL 完整框架 + 三元組儲存 + 伺服器 | 1,482[^gh-jena] |
| Eclipse RDF4J | Java | BSD 3-Clause (EDL 1.0) | RDF/SPARQL 完整框架 + 儲存 + 伺服器 + Workbench | 422[^gh-rdf4j] |
| RDFLib | Python | BSD 3-Clause | 純 Python RDF/SPARQL 程式庫 | 2,539[^gh-rdflib] |
| rdflib.js | JavaScript | MIT | JS 連結資料程式庫 | 598[^gh-rdflibjs] |
| N3.js | JavaScript | MIT | 快速串流 RDF 解析/輸出/儲存 | 797[^gh-n3] |
| rdf-parse.js | TypeScript | MIT | 通用自動偵測 RDF 串流解析器 | 68[^gh-rdfparse] |
| rdf-serialize.js | TypeScript | MIT | 通用 RDF 序列化輸出 | 9[^gh-rdfserialize] |
| Serd | C | ISC | 輕量零依賴 Turtle/N-Triples 程式庫 | 100[^gh-serd] |
| Raptor（rapper） | C | LGPL/GPL/Apache 2.0 | RDF 解析/序列化程式庫 + CLI | 175[^gh-raptor] |
| Redland librdf | C（多語言綁定） | LGPL/GPL/Apache 2.0 | C RDF 程式庫 + 三元組儲存 | 105[^gh-librdf] |
| Rasqal（roqet） | C | LGPL/GPL/Apache 2.0 | RDF 查詢程式庫（SPARQL/RDQL） | 73[^gh-rasqal] |
| Oxigraph | Rust | Apache 2.0 | SPARQL 圖形資料庫 | 2,003[^gh-oxigraph] |
| Protégé Desktop | Java | 開源（BSD 風格） | 本體編輯器，可開／存 RDF/XML、Turtle 等 | 1,599[^gh-protege] |
| WebVOWL | JavaScript | MIT | 以圖形視覺化 OWL/RDF 本體 | 1,016[^gh-webvowl] |
| Trifid | TypeScript | Apache 2.0 | 連結資料伺服器，內建圖形瀏覽 UI | 102[^gh-trifid] |
| OWL-RL | Python | W3C 軟體授權 | 在 RDFLib 上實作 OWL 2 RL 推論 | 180[^gh-owlrl] |

## Java 生態系

### Apache Jena（含 riot 命令列工具）

Apache Jena 是一個自由且開源的 Java 框架，用於建構語意網與連結資料應用程式[^gh-jena]。其 IO 子系統（RIOT）提供多種格式的解析與寫入[^jena-io]：

- 支援解析／輸出：Turtle、N-Triples、N-Quads、TriG、RDF/XML、JSON-LD、RDF Thrift、RDF Protobuf、RDF/JSON、TriX，並可自動偵測 gzip/bzip2 壓縮[^jena-io]。
- **riot** 是 Jena 的命令列解析器，預設依副檔名猜測輸入語法（例如 `.rdf` 對應 RDF/XML）；讀取 stdin 時假設為 N-Quads/N-Triples。可用於驗證（`--validate`）、檢查 IRI/literal（`--check`）、計時（`--time`）與推論（`riotcmd.infer --rdfs`）[^jena-io]。
- 常見用法：`riot --output turtle file.rdf`。

此外 Jena 另提供 **Fuseki**（SPARQL 端點伺服器與 Web UI）與 **TDB**（原生三元組儲存），適合需要查詢或持久化的情境[^gh-jena]。

### Eclipse RDF4J

Eclipse RDF4J 是 Java 的可擴展 RDF 框架（原 Sesame），支援 RDF/XML、Turtle、N-Triples、N-Quads、TriG、JSON-LD、TriX、Binary RDF 等格式的解析與序列化[^gh-rdf4j]。它附帶 **RDF4J Workbench**（瀏覽／查詢 RDF 的 Web UI）、RDF4J Server、SPARQL 端點與 SHACL 驗證，並支援多種儲存後端（native、memory、LMDB）[^gh-rdf4j]。

### Protégé Desktop

Protégé 是免費、開源的本體編輯器，支援 OWL 2.0，可開啓、編輯與儲存 RDF/XML、Turtle、N-Triples、OWL/XML 等格式，並具備外掛架構、推論支援與圖形化本體檢視[^gh-protege]。它主要定位為本體編輯器，但同時可作為 RDF 檢視／編輯工具。

## Python 生態系

### RDFLib

RDFLib 是 Python 處理 RDF 最完整的程式庫，提供 RDF/XML、N3、N-Triples、N-Quads、Turtle、TriX、TriG、JSON-LD、HexTuples、RDFa、Microdata 的解析與序列化，並內建 SPARQL 1.1 實作[^gh-rdflib]。

其命令列工具 **rdfpipe** 可在各種 RDF 序列化之間轉檔[^rdflib-rdfpipe]：

- 可從檔案路徑、URL 或 stdin（`-`）讀取，輸出至 stdout。
- 輸入格式可由副檔名自動猜測（`--no-guess` 可關閉），未指定時預設為 `xml`（RDF/XML）；輸出預設為 `n3`。
- 支援格式包含 `xml`、`n3`、`nt`、`turtle`、`trig`、`trix`、`json-ld`、`nquads`、`hext`、`pretty-xml`、`longturtle` 等，並可透過 `FORMAT:(+)KW,-KW,KW=VALUE` 傳入參數（如 `turtle:spacious`）[^rdflib-rdfpipe]。
- 常見用法：`rdfpipe -i xml -o turtle input.rdf > output.ttl`。

另一工具 **rdf2dot** 可將 RDF 圖輸出為 GraphViz DOT 以視覺化[^rdflib-tools]。

### OWL-RL

OWL-RL 是建構在 RDFLib 之上的 OWL 2 RL 推論實作，可擴充圖形中的三元組，適合需要在讀取後進行本體推論的情境[^gh-owlrl]。

## JavaScript／TypeScript 生態系

- **N3.js**：閃電般快速、符合規範的串流 RDF JavaScript 程式庫，可解析 Turtle、TriG、N-Triples、N-Quads、Notation3，並輸出 Turtle、TriG、N-Triples、N-Quads；內建 in-memory `N3.Store`，支援比記憶體更大的檔案串流處理[^gh-n3]。
- **rdf-parse.js**：依 content type 或副檔名自動選擇序列化格式，輸出 RDF/JS 相容的 quads 串流，支援 TriG、N-Quads、Turtle、N-Triples、JSON-LD、RDF/XML、RDFa、Microdata 等；內部整合 N3.js 等多個 parser[^gh-rdfparse]。
- **rdf-serialize.js**：rdf-parse.js 的對向工具，負責序列化輸出[^gh-rdfserialize]。
- **rdflib.js**：瀏覽器與 Node.js 用的 RDF 程式庫，可讀寫 RDF/XML、Turtle、N3，讀取 RDFa 與 JSON-LD，並具備連結資料客戶端與 SPARQL 圖形比對能力[^gh-rdflibjs]。
- **Trifid**：輕量連結資料伺服器與代理，內建 **Graph Explorer** 圖形瀏覽 UI 與 **YASGUI** SPARQL 查詢介面，可用於互動探索 RDF 資料[^gh-trifid]。

## C 生態系

- **Raptor（rapper）**：Redland RDF 語法程式庫，提供 RDF/XML、N-Quads、N-Triples 1.0/1.1、Turtle 2008/2013、TriG、RDFa 1.0/1.1、RSS/Atom 等 parser，以及 RDF/XML、Turtle、N-Triples、N-Quads、GraphViz DOT 等 serializer；附帶 **rapper** 命令列工具[^gh-raptor]。
- **Redland librdf**：自 2000 年發展的成熟 C 程式庫，提供 RDF 高階介面、儲存、查詢與操作，附帶 **rdfproc** 命令列工具與多種語言綁定[^gh-librdf]。
- **Serd**：輕量、零外部依賴的 C 程式庫，讀寫 Turtle、TriG、N-Triples、N-Quads，以串流、常數記憶體使用與速度見長，附帶 **serdi** CLI[^gh-serd]。
- **Rasqal（roqet）**：處理 SPARQL 1.0 與多數 SPARQL 1.1 的查詢程式庫，附帶 **roqet** CLI[^gh-rasqal]。

## Rust 生態系

**Oxigraph** 是以 Rust 實作的 SPARQL 圖形資料庫，可作為 RDF 資料的儲存與查詢後端，擁有超過 2,000 顆星，是活躍的現代替代方案[^gh-oxigraph]。

## 視覺化與瀏覽工具

- **WebVOWL**：以 JavaScript 將 OWL／RDF 本體視覺化為圖形，MIT 授權，超過 1,000 顆星[^gh-webvowl]。
- **RDF4J Workbench**、**Jena Fuseki**：以 Web UI 瀏覽與查詢 RDF 資料[^gh-rdf4j][^jena-io]。
- **Protégé**：圖形化本體檢視與編輯[^gh-protege]。
- **Trifid Graph Explorer**：互動式圖形瀏覽[^gh-trifid]。

## 命令列轉檔工具對照

| 工具 | 所屬 | 用途 |
|---|---|---|
| riot | Apache Jena | Java，RDF 解析與格式轉換[^jena-io] |
| rdfpipe | RDFLib | Python，RDF 轉檔[^rdflib-rdfpipe] |
| rapper | Raptor | C，RDF 解析與轉換[^gh-raptor] |
| rdfproc | Redland librdf | C，RDF 處理[^gh-librdf] |
| serdi | Serd | C，RDF 解析與轉換[^gh-serd] |
| roqet | Rasqal | C，SPARQL 查詢[^gh-rasqal] |
| rdf2dot | RDFLib | Python，輸出 GraphViz DOT 視覺化[^rdflib-tools] |

## 選型建議

- **只要命令列快速看內容／轉檔**：Python 的 `rdfpipe`（pip 安裝 rdflib）最輕便；或 C 的 `serdi`、`rapper`。
- **Python 應用整合**：RDFLib 最完整，需推論時搭配 OWL-RL。
- **Java 應用整合**：Apache Jena 或 Eclipse RDF4J，並依需求選用 Fuseki／RDF4J Server 做 SPARQL 端點。
- **Web 前端／Node.js**：N3.js（效能）或 rdf-parse.js（自動偵測格式）。
- **高效能、串流、大檔**：Serd（C）或 N3.js（JS）。
- **互動視覺化**：WebVOWL、Protégé、Trifid Workbench。

## 對來源的批判性反思

- 本報告多數授權與功能資訊直接來自各專案官方 GitHub repository 及其 GitHub API metadata（星數、授權、語言），屬第一手且權威來源[^gh-jena][^gh-rdflib][^gh-rdf4j] 等。星數會隨時間變動，本表以 2026-10-24 查詢為基準。
- RDF 模型與格式的說明引用 Wikipedia 條目[^rdfwiki]，該條目對 W3C 標準的描述與官方規格一致，且 RDF 是成熟且穩定的標準，資訊可信度高；惟報告撰寫時 W3C 官方頁面以 403 阻擋自動擷取，故未直接引用 W3C 文件。
- 少數專案（如 N3.js、raptor、rdflib.js、OWL-RL、Protégé）的 GitHub API 授權欄位顯示為「Other／NOASSERTION」，這是 GitHub 對非標準 LICENSE 檔案的判定，並非表示其非開源；實際授權需以各專案 LICENSE 檔案為準。
- rdf-serialize.js、rdf-parse.js 的星數偏低（9 與 68），但在 RDF/JS 生態系中是常用且被 Comunica 等框架依賴的關鍵元件，故仍收錄並附上背景說明。
- 多數工具已相當成熟且持續維護（查詢日期可見近期 commit 活動），選型風險低。

## 參考文獻

[^rdfwiki]: Wikipedia. (n.d.). Resource Description Framework. Retrieved 2026-10-24, from https://en.wikipedia.org/wiki/Resource_Description_Framework

[^gh-jena]: Apache Software Foundation. (n.d.). Apache Jena, A free and open source Java framework for building Semantic Web and Linked Data applications [Computer software]. Retrieved 2026-10-24, from https://github.com/apache/jena

[^jena-io]: Apache Software Foundation. (n.d.). Apache Jena - Reading and Writing RDF in Apache Jena. Retrieved 2026-10-24, from https://jena.apache.org/documentation/io/

[^gh-rdf4j]: Eclipse Foundation. (n.d.). Eclipse RDF4J: scalable RDF for Java [Computer software]. Retrieved 2026-10-24, from https://github.com/eclipse-rdf4j/rdf4j

[^gh-rdflib]: RDFLib. (n.d.). RDFLib is a Python library for working with RDF, a simple yet powerful language for representing information [Computer software]. Retrieved 2026-10-24, from https://github.com/RDFLib/rdflib

[^rdflib-rdfpipe]: RDFLib. (n.d.). rdfpipe — RDFLib documentation. Retrieved 2026-10-24, from https://rdflib.readthedocs.io/en/stable/apidocs/rdflib.tools.rdfpipe/

[^rdflib-tools]: RDFLib. (n.d.). Tools — RDFLib documentation. Retrieved 2026-10-24, from https://rdflib.readthedocs.io/en/stable/apidocs/rdflib.tools/

[^gh-rdflibjs]: Linked Data. (n.d.). Linked Data API for JavaScript (rdflib.js) [Computer software]. Retrieved 2026-10-24, from https://github.com/linkeddata/rdflib.js

[^gh-n3]: RDF/JS. (n.d.). N3.js: Lightning fast, spec-compatible, streaming RDF for JavaScript [Computer software]. Retrieved 2026-10-24, from https://github.com/rdfjs/N3.js

[^gh-rdfparse]: Rubensworks. (n.d.). rdf-parse.js: Parses RDF from any serialization [Computer software]. Retrieved 2026-10-24, from https://github.com/rubensworks/rdf-parse.js

[^gh-rdfserialize]: Rubensworks. (n.d.). rdf-serialize.js: Serializes RDF to any serialization [Computer software]. Retrieved 2026-10-24, from https://github.com/rubensworks/rdf-serialize.js

[^gh-serd]: Drobilla. (n.d.). Serd: A lightweight C library for RDF syntax [Computer software]. Retrieved 2026-10-24, from https://github.com/drobilla/serd

[^gh-raptor]: Dajobe. (n.d.). Raptor: Redland Raptor RDF syntax library [Computer software]. Retrieved 2026-10-24, from https://github.com/dajobe/raptor

[^gh-librdf]: Dajobe. (n.d.). librdf: Redland librdf RDF API and triple stores [Computer software]. Retrieved 2026-10-24, from https://github.com/dajobe/librdf

[^gh-rasqal]: Dajobe. (n.d.). Rasqal: Redland Rasqal RDF Query Library [Computer software]. Retrieved 2026-10-24, from https://github.com/dajobe/rasqal

[^gh-oxigraph]: Oxigraph. (n.d.). Oxigraph: SPARQL graph database [Computer software]. Retrieved 2026-10-24, from https://github.com/oxigraph/oxigraph

[^gh-protege]: Protégé Project. (n.d.). Protege Desktop [Computer software]. Retrieved 2026-10-24, from https://github.com/protegeproject/protege

[^gh-webvowl]: VisualDataWeb. (n.d.). WebVOWL: Visualizing ontologies on the Web [Computer software]. Retrieved 2026-10-24, from https://github.com/VisualDataWeb/WebVOWL

[^gh-trifid]: Zazuko. (n.d.). Trifid: Lightweight Linked Data Server and Proxy [Computer software]. Retrieved 2026-10-24, from https://github.com/zazuko/trifid

[^gh-owlrl]: RDFLib. (n.d.). OWL-RL: A simple implementation of the OWL2 RL Profile on top of RDFLib [Computer software]. Retrieved 2026-10-24, from https://github.com/RDFLib/OWL-RL
