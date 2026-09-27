# IndoorGML 專案背景調查報告

## 概要

**IndoorGML 不是一家公司或新創企業**，而是一個由 **Open Geospatial Consortium（OGC）** 制定的開放標準，專門用於室內空間資訊的資料模型與交換格式。本報告調查其組織結構、團隊、資金來源、社群規模與採用狀況。

---

## 1. 主導組織：Open Geospatial Consortium（OGC）

IndoorGML 是 OGC 旗下的正式標準之一。OGC 是一個成立於 1994 年的 **國際非營利自願共識標準組織**，在美國註冊為 501(c)(6) 非營利組織，在美國、比利時、英國設有辦公室，擁有約 **470 多個會員組織**，涵蓋政府、產業與學術機構。[^ogc-overview]

indoorgml.net 網站是 IndoorGML 標準的公開資訊入口，由 OGC 與韓國釜山大學（Pusan National University）的 STEMLab 共同維護，網站上同時標示 OGC 與 IndoorGML 標誌。[^indoorgml-site]

---

## 2. 核心團隊與主要貢獻者

IndoorGML 由 **OGC IndoorGML 標準工作組（SWG）** 開發，該工作組於 **2012 年 1 月** 成立。[^ogc-swg-2012]

### IndoorGML 1.1（2020 年 11 月發布）主要編輯群

| 姓名 | 所屬機構 | 角色 |
|---|---|---|
| **Ki-Joune Li（李基俊）** | 釜山大學（PNU） | 主編／標準創始人 |
| Jiyeong Lee | 首爾市立大學 | 共同作者 |
| Thomas H. Kolbe | 慕尼黑工業大學（TUM） | 共同作者 |
| Sisi Zlatanova | 新南威爾斯大學（UNSW） | 共同作者 |
| Jeremy Morley | 英國地形測量局（Ordnance Survey） | 共同作者 |
| Claus Nagel | Virtual City Systems | 共同作者 |
| Thomas Becker | 柏林工業大學 | 共同作者 |

[^indoorgml-11]

### IndoorGML 2.0 編輯群（現行開發中）

| 姓名 | 所屬機構 |
|---|---|
| **Sisi Zlatanova**（主編） | 新南威爾斯大學（UNSW） |
| Abdoulaye Diakite | CityGeometrix／UNSW |
| Taehoon Kim | 釜山大學（PNU） |
| Ki-Joune Li | 釜山大學（PNU） |

### IndoorGML 2.0 Part 1（概念模型，已核准）

| 姓名 | 所屬機構 |
|---|---|
| Sisi Zlatanova | 新南威爾斯大學 |
| Ki-Joune Li | 釜山大學 |
| Abdoulaye Diakite | CityGeometrix |
| Jeremy Morley | 英國地形測量局 |
| Taehoon Kim | 日本產業技術綜合研究所（AIST） |

[^indoorgml-20]

### 關鍵人物：Ki-Joune Li（李基俊）

**Dr. Ki-Joune Li** 是 IndoorGML 公認的創始核心人物，任職於韓國釜山大學，其所領導的 **STEMLab** 開發了 IndoorGML 主要的開源工具鏈，其電子郵件（lik@pnu.edu）亦為 indoorgml.net 上的主要聯絡窗口。

---

## 3. 提交機構（Supporting Organizations）

### IndoorGML 1.1
- 釜山大學（Pusan National University）
- 首爾市立大學（University of Seoul）
- 慕尼黑工業大學（Technical University of Munich）
- 柏林工業大學（Technical University of Berlin）
- 新南威爾斯大學（University of New South Wales）
- All4Land
- 英國地形測量局（Ordnance Survey）

### IndoorGML 2.0 Part 1
- 新南威爾斯大學
- 釜山大學
- 英國地形測量局
- 首爾市立大學
- CityGeometrix
- 日本產業技術綜合研究所（AIST）

[^indoorgml-11][^indoorgml-20]

---

## 4. 資金來源

**IndoorGML 沒有創投（VC）投資或商業實體資助**。其資金結構如下：

1. **OGC 會員費** — OGC 的主要營運經費來自 470 多個會員組織的年費，會員包括政府機構（USGS、NASA、NGA、Ordnance Survey 等）、商業公司（Esri、Google、Oracle、Hexagon、Bentley Systems 等）以及學術機構。[^ogc-overview]

2. **學術研究補助** — 釜山大學 STEMLab 的開發工作由韓國國家級研究計畫支持（具體補助計畫名稱未在網站上公開）。

3. **OGC 創新計畫（COSI）** — OGC 的合作解決方案與創新計畫（Collaborative Solutions and Innovation Program）透過政府機構贊助測試平台（Testbeds）與先導專案（Pilots），IndoorGML 曾在此類計畫中獲得採用。

---

## 5. 社群規模

### GitHub 開源工具（STEMLab 組織）

STEMLab（https://github.com/STEMLab）共有 **51 個倉庫**，IndoorGML 相關主要工具：

| 倉庫 | Stars | Forks | 說明 |
|---|---|---|---|
| **InEditor** | 33 | 18 | Web-based IndoorGML 編輯器 |
| **InViewer** | 26 | 16 | Three.js 3D 檢視器 |
| **InViewer-Desktop** | 16 | 7 | Unity3D 桌面檢視器 |
| **InFactory** | 18 | 12 | RESTful 伺服器，用於建立 IndoorGML 文件 |
| **indoorgml-dev** | 4 | 2 | 開發用倉庫（範例資料、簡報、工具） |

所有倉庫皆以 **MIT 授權** 開源。[^stemlab-github]

星數屬於中低規模，反映這是一個高度專業的學術／標準社群，而非消費性開源專案。

---

## 6. 採用狀況

IndoorGML 被應用於以下領域：

- **學術研究** — 最廣泛的採用場景，特別是在室內導航、LBS（Location-Based Services）與空間分析領域。
- **政府先導專案** — 英國地形測量局（Ordnance Survey）作為提交機構，已採用並貢獻此標準。
- **OGC 互通性測試** — IndoorGML 曾參與 OGC 室內 3D 工作坊與 MWC 展示等創新計畫。
- **商業地理資訊產品** — Esri、Hexagon 等 OGC 會員公司在其產品中支援 IndoorGML。
- **互補標準體系** — IndoorGML 設計上與 CityGML（城市模型）、IFC（建築資訊模型）及 IMDF（Apple 室內地圖格式）互補，而非競爭。

---

## 7. 近期發展

| 時間 | 里程碑 |
|---|---|
| 2012 年 1 月 | OGC 成立 IndoorGML 室內位置標準工作組 |
| 2014 年 12 月 | IndoorGML 1.0 發布 |
| 2016 年 9 月 | IndoorGML 1.0.2 發布 |
| 2018 年 3 月 | IndoorGML 1.0.3 發布 |
| 2020 年 11 月 | IndoorGML 1.1 發布（現行穩定版本） |
| 2024–2026 年 | IndoorGML 2.0 Part 1 核准，Part 2 開發中 |

### IndoorGML 2.0 重大變革

IndoorGML 2.0 是該標準的重大現代化版本，主要改變包括：
- 分為多部分結構（Part 1：UML 概念模型，Part 2：GML / SQL / JSON 實作）
- 資料模型簡化
- 新增 SQL 與 JSON 編碼格式（原僅支援 GML）

[^indoorgml-20][^indoorgml-news]

---

## 8. 結論

IndoorGML 是一個由 **OGC 非營利標準組織** 主導、以 **韓國釜山大學 STEMLab** 為核心開發力量的開放標準專案。它不是公司或新創，沒有創投資金，而是透過 OGC 會員費、學術研究補助和政府合作計畫運作。主要推動者為 **Ki-Joune Li 教授**（釜山大學）與 **Sisi Zlatanova 教授**（UNSW），社群規模小而專注（GitHub 星數各 4–33），但其標準被 Esri、Google、Hexagon 等大型組織採用。目前正積極開發 IndoorGML 2.0，朝向多格式支援與現代化邁進。

---

## 參考資料

[^ogc-overview]: Open Geospatial Consortium. (n.d.). *OGC Overview*. Retrieved 2026-09-25, from https://www.ogc.org/about/overview/

[^indoorgml-site]: IndoorGML. (n.d.). *IndoorGML — Open Data Model and XML Schema for Indoor Spatial Information*. Retrieved 2026-09-25, from https://www.indoorgml.net/

[^ogc-swg-2012]: Open Geospatial Consortium. (2012, January 24). *The OGC Forms IndoorGML Indoor Location Standards Working Group*. Retrieved 2026-09-25, from https://www.ogc.org/news/the-ogc-forms-indoorgml-indoor-location-standards-working-group/

[^indoorgml-11]: Open Geospatial Consortium. (2020). *OGC IndoorGML 1.1 Standard* (Doc No. 19-011r4). Retrieved 2026-09-25, from https://docs.ogc.org/is/19-011r4/19-011r4.html

[^indoorgml-20]: Open Geospatial Consortium. (2024). *OGC IndoorGML 2.0 Part 1: Conceptual Model* (Doc No. 22-045r5). Retrieved 2026-09-25, from https://docs.ogc.org/is/22-045r5/22-045r5.html

[^stemlab-github]: STEMLab. (n.d.). *STEMLab GitHub Organization*. Retrieved 2026-09-25, from https://github.com/STEMLab

[^indoorgml-news]: IndoorGML. (n.d.). *IndoorGML News*. Retrieved 2026-09-25, from https://www.indoorgml.net/news/