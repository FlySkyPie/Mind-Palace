# 與 BIM 相容但簡化的公民友善資料格式調查

## 摘要

建築資訊模型（BIM）在公共工程與都市規劃中日益普及，然而其核心交換格式 IFC（Industry Foundation Classes）因結構複雜，對非專業公民構成極高的理解門檻。本研究調查是否存在同時與 BIM 相容、又足夠簡化以利公民參與的資料格式。結果顯示，**目前尚無專為公民設計的「BIM Lite」或「公民 BIM」格式標準**，但存在一系列可作為替代方案的格式與技術路徑。最值得關注的方案包括：**CityJSON**（OGC 標準，以 JSON 編碼 3D 城市模型，檔案量僅為 CityGML 的 1/7）、**glTF/XKT 轉換管線**（透過 IfcOpenShell 與 xeokit 將 IFC 轉為網頁友善格式）、**GeoJSON**（簡易建築足跡格式）、**COBie**（試算表形式設施管理資料）與 **BCF**（問題協作格式）。其中，瑞士蘇黎世市於 2024 年已實際運用 xeokit SDK 打造公民參與平台，為相關領域提供了重要參考案例。

---

## 1. 研究背景

BIM（Building Information Modeling）已是現代建築與土木工程的標準作業流程，其核心交換標準 IFC（ISO 16739-1:2024）定義了數百種實體類型及其關係，涵蓋幾何、材料、成本、流程等面向[^ifc-wiki]。然而，這種全面性也帶來極高的複雜度——IFC-SPF（.ifc）格式基於 ISO 10303-21（STEP），對非專業人士而言幾乎無法直接閱讀或理解[^ifc-formats]。

隨著公民參與（public participation）在都市規劃中的重要性提升，如何讓不具備 BIM 專業知識的一般市民能夠理解、瀏覽甚至反饋建築規劃資訊，已成為一個實際的研究與實務課題。

---

## 2. 現有標準與格式盤點

### 2.1 IFC 系列編碼格式

buildingSMART 定義了多種 IFC 編碼，各自在可讀性、檔案大小與相容性之間取得不同權衡[^ifc-formats]：

| 格式 | 副檔名 | 相較 IFC-SPF 大小 | 公民友善度 |
|------|--------|-------------------|-----------|
| IFC-SPF | .ifc | 100%（基準） | ★☆☆☆☆ |
| ifcXML | .ifcXML | ~113% | ★★☆☆☆ |
| ifcJSON | .json | ~148% | ★★★☆☆ |
| ifcZIP | .ifcZIP | ~17%（壓縮） | ★★☆☆☆ |
| ifcOWL / Turtle / RDF | — | 816-1372% | ★★☆☆☆ |

其中 **ifcJSON** 值得關注——它將 IFC 資料模型編碼為 JSON，可直接被瀏覽器與 JavaScript 工具解析。然而根據 buildingSMART 社群描述，ifcJSON 的優先目標是「向後相容性與往返能力」，**非專業人士可讀性並非首要考量**[^ifcjson]。

### 2.2 CityJSON —— 3D 城市模型的簡化替代方案

**CityJSON** 是 OGC 國際標準（文件 20-072r5），旨在以更簡潔的方式取代基於 GML/XML 的 CityGML[^cityjson]。

關鍵特性：
- **檔案量僅為 CityGML 的 1/7**
- 以 **JSON** 編碼，原生支援網頁生態系
- **扁平化層級結構**，簡化儲存與處理
- 與 CityGML **雙向轉換**可行
- 擁有網頁檢視器（CityJSON Ninja）、驗證器與 CLI 工具（cjio）
- 支援擴充機制以自訂物件/屬性

**FlatCityBuf** 是其下一代二進位格式（2025-2026），基於 FlatBuffers 技術：
- 反序列化速度提升 **9-250 倍**
- 記憶體使用量減少 **2-6 倍**
- 支援**部分資料讀取**（僅透過 HTTP Range Request 取得所需位元組）
- 已有 Rust、C++、Python、TypeScript 四種原生實作
- 網頁端可直接開啟 68GB 的 3DBAG 資料集（1060 萬棟建築）[^flatcitybuf]

### 2.3 COBie —— 試算表形式的設施管理資料

**COBie**（Construction Operations Building Information Exchange）是專注於設施管理交接的 BIM 資料子集，以**試算表（.xlsx）**為主要交付格式[^cobie-wiki]。

優點：
- 一般市民對 Excel 試算表的熟悉度遠高於 IFC
- COBieLite 是更精簡的 XML 格式版本（美國國家建築科學院發展）
- 被美國聯邦政府（GSA P-100）與英國標準（BS 1192-4）強制採用

### 2.4 gbXML —— 建築性能分析的簡化格式

**gbXML**（Green Building XML）是開放模式，專注於將 BIM 資料傳遞至工程分析工具（特別是能源分析），其關注範圍比完整 IFC 更窄[^gbxml-wiki]，主要涵蓋：
- 建築幾何形狀
- 空間/區域
- 熱區與 HVAC 系統

### 2.5 BCF —— 問題協作格式

**BCF**（BIM Collaboration Format）並非完整模型的格式，而是基於 BIM 模型的輕量級**問題追蹤格式**[^bcf-wiki]。它允許不同利害關係人就模型特定位置提出議題：
- 以 .bcfzip（XML + PNG 截圖）傳遞
- 透過 IFC GUID 參照模型元素
- 支援檔案交換與 REST API 兩種模式

### 2.6 GeoJSON —— 簡易地理資料格式

**GeoJSON**（RFC 7946）以 JSON 表示地理特徵（點、線、多邊形），廣泛用於 Leaflet、Mapbox 等網路地圖工具[^geojson-wiki]。雖非專為建築設計，但**可以簡潔表示建築足跡**，是目前最廣泛使用的公民友善地理格式之一。

### 2.7 glTF / 3D Tiles —— 網頁 3D 視覺化格式

**glTF**（GL Transmission Format，ISO/IEC 12113:2022）被稱為「3D 界的 JPEG」，以 JSON + 二進位緩衝區組成，專為網頁與行動端傳遞最佳化[^gltf-wiki]。**3D Tiles** 基於 glTF 建構，加入空間索引以支援大規模串流。

---

## 3. 關鍵技術管線：將 IFC 轉為網頁友善格式

雖然缺乏專為公民設計的 BIM 格式，但目前已有成熟的開放原始碼工具鏈，可將專業 BIM 資料轉換為公民可瀏覽的網頁格式：

```
IFC 模型
    ↓
IfcConvert (IfcOpenShell)  或   cxConverter (Creoox)
    ↓
GLB (二進位 glTF)
    ↓
xeokit-convert (convert2xkt)
    ↓
XKT (極度壓縮格式)
    ↓
xeokit Viewer 在瀏覽器中呈現
```

**IfcOpenShell** 是關鍵的開源轉換工具，其 `IfcConvert` 命令列工具可直接將 IFC 轉為 GLB[^ifcopenshell]：

```bash
$ IfcConvert model.ifc model.glb
```

**xeokit SDK**（AGPLv3）則提供高效能的瀏覽端渲染引擎，其專屬 **XKT 格式**可將 49MB 的 IFC 壓縮至 1.5MB，載入時間僅約 2-3 秒[^xeokit]。

---

## 4. 實例：蘇黎世市的公民參與平台

2024 年，**瑞士蘇黎世市土木工程局**（Tiefbauamt）首次在官方公眾諮詢程序中，將互動式 3D BIM 模型與傳統規劃文件一同公開[^zurich-case]。

面臨的挑戰：
- 市民無法理解 2D 技術圖面
- BIM 資料需要專業工具才能檢視

採用方案（基於 xeokit SDK）：
- 視圖篩選器：切換「現有環境」與「規劃方案」
- 主題分類顯示（道路、建築物）
- 直覺化 UI：工具提示、圖示、影片教學
- 距離量測、座標查詢、地形坡度顯示
- 自訂剖面切割
- **內嵌聯絡表單**供市民回饋意見

結論：該市將「抽象工程圖面轉化為直覺的瀏覽器 3D 體驗」，使規劃資訊對所有人開放。

---

## 5. 認知負荷理論的啟示

雖然目前尚無直接連結認知負荷理論與 BIM 公民參與的研究，但既有理論具有明確的指導意義[^cog-load]：

- **內在負荷**（intrinsic load）：處理 IFC 的複雜實體關係本身就構成高內在負荷
- **外部負荷**（extraneous load）：專業 BIM 工具的操作複雜度會增加無關的外部負荷
- **專業反轉效應**（expertise reversal effect）：為專業人士設計的工具對初學者而言效果可能適得其反

這意味著：**為公民設計的 BIM 介面不應等同於專業工具的簡化版，而需要從公民的認知起點重新設計**。

---

## 6. 結論與建議

### 6.1 目前沒有專為公民設計的 BIM 格式

本研究的核心發現：**標準化組織與業界尚未提出一個名為「BIM Lite」或「公民 BIM」的專用資料格式**。現有方案皆為專業工具鏈的副產物。

### 6.2 最實用的替代方案

按公民友善程度排序：

| 方案 | 型態 | 公民友善度 | 最佳用途 |
|------|------|-----------|---------|
| **CityJSON** | 3D 城市模型（JSON） | ★★★★☆ | 都市規劃公開諮詢 |
| **glTF → XKT 管線** | 3D 視覺化 | ★★★★☆ | 網頁端 3D 模型瀏覽 |
| **GeoJSON** | 2D 足跡（JSON） | ★★★★★ | 簡易地圖呈現建築位置 |
| **COBie / COBieLite** | 試算表 | ★★★★☆ | 設施資料公開 |
| **BCF** | 問題回報 | ★★★☆☆ | 公民回饋收集 |
| **gbXML** | 性能資料（XML） | ★★★☆☆ | 能源與環境資訊公開 |
| **ifcJSON** | 完整 BIM（JSON） | ★★☆☆☆ | 開發者整合 |

### 6.3 建議方向

1. **參考 CityJSON** 的設計理念，建立一個專為公民設計的 BIM 子集格式，僅保留視覺化所需的幾何、類型與基本屬性
2. **採用 xeokit 類似的工具鏈**，建立從 IFC 到公民網頁檢視器的自動化管線
3. **借鑒蘇黎世案例**，在公民參與平台中整合視覺化瀏覽與回饋機制
4. **結合認知負荷理論**，從使用者研究出發設計公民介面，而非直接沿用專業 BIM 工具的操作邏輯

---

## 參考文獻

[^ifc-wiki]: Industry Foundation Classes. (n.d.). In *Wikipedia*. Retrieved 2026-09-25, from https://en.wikipedia.org/wiki/Industry_Foundation_Classes

[^ifc-formats]: buildingSMART. (n.d.). *IFC Formats*. Retrieved 2026-09-25, from https://technical.buildingsmart.org/standards/ifc/ifc-formats/

[^ifcjson]: buildingSMART Community. (n.d.). *ifcJSON*. Retrieved 2026-09-25, from https://github.com/buildingsmart-community/ifcJSON

[^cityjson]: CityJSON. (n.d.). *CityJSON — A Compact JSON-based Encoding for 3D City Models*. Retrieved 2026-09-25, from https://www.cityjson.org/

[^flatcitybuf]: CityJSON. (2026-08-10). *FlatCityBuf Updates*. Retrieved 2026-09-25, from https://www.cityjson.org/news/2026/08/10/flatcitybuf-updates/

[^cobie-wiki]: COBie. (n.d.). In *Wikipedia*. Retrieved 2026-09-25, from https://en.wikipedia.org/wiki/COBie

[^gbxml-wiki]: Green Building XML. (n.d.). In *Wikipedia*. Retrieved 2026-09-25, from https://en.wikipedia.org/wiki/Green_Building_XML

[^bcf-wiki]: BIM Collaboration Format. (n.d.). In *Wikipedia*. Retrieved 2026-09-25, from https://en.wikipedia.org/wiki/BIM_Collaboration_Format

[^geojson-wiki]: GeoJSON. (n.d.). In *Wikipedia*. Retrieved 2026-09-25, from https://en.wikipedia.org/wiki/GeoJSON

[^gltf-wiki]: glTF. (n.d.). In *Wikipedia*. Retrieved 2026-09-25, from https://en.wikipedia.org/wiki/GlTF

[^ifcopenshell]: IfcOpenShell. (n.d.). *IfcOpenShell — The Open Source IFC Toolkit*. Retrieved 2026-09-25, from https://ifcopenshell.org/

[^xeokit]: xeokit SDK. (n.d.). *xeokit SDK — 3D BIM/CAD Visualization Toolkit*. Retrieved 2026-09-25, from https://xeokit.io/

[^zurich-case]: xeokit. (2024). *How Zurich Enables Digital Civic Participation through a 3D BIM Viewer*. Retrieved 2026-09-25, from https://xeokit.io/success-stories/how-zurich-enables-digital-civic-participation-through-a-3d-bim-viewer/

[^cog-load]: TeachTogether.tech. (n.d.). *Cognitive Load Theory*. Retrieved 2026-09-25, from https://teachtogether.tech/en/index.html#ch:cognitive-load