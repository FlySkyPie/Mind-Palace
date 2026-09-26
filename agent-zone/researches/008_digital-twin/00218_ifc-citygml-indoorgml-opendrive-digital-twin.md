# IFC、CityGML、IndoorGML、OpenDRIVE 是否皆屬於數位孿生（Digital Twin）？

## 問題

「IFC、CityGML、IndoorGML、OpenDRIVE 這四者是否皆屬於數位孿生（Digital Twin），但以不同面向的角度切入？」本文旨在回答此問題。

## 結論（簡答）

**是，但並非「類型」而是「資料標準」**。這四者各自是不同尺度、不同領域的**數位孿生底層資料模型**，各自服務於數位孿生的某個面向。它們本身並不足以構成完整的數位孿生，而是作為靜態語意／幾何核心，須搭配即時感測器資料、動態模擬與雙向回饋才能成為成熟的數位孿生。

---

## 1. 各標準概述

### 1.1 IFC（Industry Foundation Classes）

- **制定組織**：buildingSMART International / ISO 16739-1:2024[^bsi-ifc]
- **領域**：建築、工程、營造（AEC）
- **尺度**：單一建築物／資產（微觀）
- **建模內容**：以專案為單位，描述建築物內的每一個構件——牆、板、樑、管線、閥門、門窗——以及它們的材料、類型、關係與所圍塑的空間。幾何通常以實體模型（擠出、布林運算）表達，使用局部工程坐標系（公釐）。[^bsi-ifc]
- **數位孿生角色**：**建築數位孿生的語意資料核心**。學術文獻大量使用「IFC-based digital twin」一詞（Frontiers in Energy Research, 2024；IOP Science, 2025；IEEE, 2025）[^frontiers-ifc-dt][^iop-ifc-dt][^ieee-ifc-dt]。NIBS（National Institute of Building Sciences）的白皮書亦建議將 BIM（以 IFC 為開放標準）與數位孿生技術整合。[^nibs-dt]

### 1.2 CityGML（City Geography Markup Language）

- **制定組織**：Open Geospatial Consortium（OGC）— 3.0 版現行[^ogc-citygml]
- **領域**：地理資訊系統（GIS）、都市規劃
- **尺度**：城市／區域（中觀）
- **建模內容**：以地理參考特徵描述城市——建築物、道路、植栽、水域、橋樑、隧道、街道家具等，每個物件都有語意類型、識別碼與邊界曲面（屋頂、牆面、地面），幾何為邊界表示法（B-Rep），定義多個細緻度層級（LOD0–LOD4）。[^ogc-citygml]
- **數位孿生角色**：**都市數位孿生（Urban Digital Twin）的底層語意模型**。OGC 官方標準頁面直接將 CityGML 定位為「智慧城市與都市數位孿生應用」的基礎標準。[^ogc-citygml] OGC 討論文件《Urban Digital Twins: Integrating Infrastructure, natural environment and people》（24-025）將其列為都市數位孿生的核心標準。[^ogc-udt]

### 1.3 IndoorGML

- **制定組織**：Open Geospatial Consortium（OGC）— 2.0 版現行[^ogc-indoorgml]
- **領域**：室內空間資訊與導航
- **尺度**：建築物內部／子建築（奈觀）
- **建模內容**：專注於室內空間的**導航拓樸**——空間間的連通性、路徑網路，而非詳細幾何。OGC 明確指出：「雖然 CityGML、KML、IFC 等標準以幾何、製圖與語意觀點處理建築內部空間，IndoorGML 則刻意聚焦於以導航為目的的室內空間建模。」[^ogc-indoorgml-spec]
- **數位孿生角色**：**室內導航層的資料標準**，可與 IFC（建築細節）和 CityGML（城市情境）互補。Krishna Lodha 的《OGC Standards for Digital Twins》簡報將其列為數位孿生領域的核心 OGC 標準之一。[^ogc-dt-slides]

### 1.4 OpenDRIVE

- **制定組織**：ASAM（Association for Standardization of Automation and Measuring Systems）— v1.9.0（2026 年 5 月）[^asam-opendrive]
- **領域**：自動駕駛輔助系統（ADAS）與自動駕駛（AD）模擬
- **尺度**：路段／道路網路（線性）
- **建模內容**：以 XML 格式描述**靜態道路網路**——道路幾何、車道、交叉路口、號誌、標線等。使用 s/t 坐標系統（沿參考線的縱向與橫向偏移），專為駕駛模擬設計。[^asam-opendrive]
- **數位孿生角色**：**道路網路數位孿生的資料標準**。GitHub 上的「OpenDRIVE Digital Twin Generator」專案[^github-odr-dt]及對應論文《Automated Digital Twin Construction for Highway Scenarios Using LiDAR Point Clouds and OpenStreetMap》（arXiv, 2026）[^arxiv-odr-dt] 均明確使用「OpenDRIVE digital twin」一詞。MathWorks 文件亦將 OpenDRIVE 視為建立環境數位孿生的核心格式。[^mathworks-odr]

---

## 2. 比較：四者作為數位孿生的不同面向

若將真實世界視為一個多尺度的連續體，這四種標準各自對應到不同的「切片」：

| 標準 | 尺度 | 領域 | 核心問題 | 數位孿生面向 |
|------|------|------|----------|--------------|
| **IFC** | 建築物（微觀） | AEC / BIM | 這棟建築由哪些構件組成？如何建造與維護？ | 建築設施孿生（Building DT） |
| **CityGML** | 城市（中觀） | GIS / 都市規劃 | 這個城市區域有哪些地物？它們在真實世界中的位置與關係？ | 都市孿生（Urban DT） |
| **IndoorGML** | 室內（奈觀） | 室內導航 | 人在建築物內部如何移動？空間之間如何連通？ | 室內導航孿生層 |
| **OpenDRIVE** | 道路（線性） | 自駕模擬 | 車輛在這條路上會看到什麼車道、號誌與路形？ | 道路基礎設施孿生 |

關鍵在於：**它們不是彼此競爭的替代方案，而是描述不同主體（Subject）的互補標準**。3D Geospatial 的分析指出：「IFC 將建築描述為一個營造專案……CityGML 將城市描述為一組地理參考特徵……兩者在實務上的區別不是細節多寡，而是**主體不同**……成熟的數位孿生通常兩者並存而非二選一。」[^3dg-ifc-citygml]

ASAM 與 OGC 聯合發表的《OpenDRIVE with CityGML》概念文件（2026）亦採用同樣觀點：「ASAM OpenDRIVE 與 OGC CityGML 採用根本不同的幾何建模典範，各自適合其領域。因此本概念文件將它們視為**互補的表述**，各有不同的責任範圍。」[^asam-ogc-odr-citygml]

---

## 3. 重要釐清：為何不能說「它們是數位孿生的類型」？

OGC 都市數位孿生討論文件（24-025）提出了數位孿生的成熟度層級[^ogc-udt]：

1. **數位模型（Digital Model）**—— 靜態資料，無自動資料流
2. **數位影子（Digital Shadow）**—— 單向資料流（實體 → 數位）
3. **數位孿生（Digital Twin）**—— 雙向資料流（實體 ↔ 數位），含自動回饋

IFC、CityGML、IndoorGML、OpenDRIVE 目前主要服務於第 1 層（數位模型），需進一步連結 IoT 感測器（如 OGC SensorThings API）與動態模擬引擎，才能達到真正的雙向數位孿生。將它們稱為「數位孿生的類型」可能引發誤解——更精確的說法是：**它們各自是不同領域數位孿生的語意／幾何基礎架構**。

---

## 4. 整合趨勢

學術界與標準組織正積極推動跨標準整合以建立更完整的數位孿生：

- **IFC + CityGML**：最常見的整合配對。MDPI Sensors（2024）論文探討將 IFC 資料轉換為 CityGML 格式以強化都市數位孿生的互通性。[^mdpi-ifc-citygml]
- **CityGML + IndoorGML**：IndoorGML 2.0 規格說明它與 CityGML 在室內空間建模上各有分工，可互補使用。[^ogc-indoorgml-spec]
- **OpenDRIVE + CityGML**：ASAM-OGC 聯合概念文件（2026）正式探討兩者的整合模式。[^asam-ogc-odr-citygml]
- **IFC + CityGML + IndoorGML + OpenStreetMap**：W78-2024 會議論文以共同中介模型整合這四種綱要，實現建築資料到城市地圖與導航模型的轉換。[^w78-integration]

---

## 5. 總結

IFC、CityGML、IndoorGML、OpenDRIVE 四者**確實都可歸屬於數位孿生的大傘之下**，但它們不是數位孿生的**類型**，而是各自服務於數位孿生在不同尺度與領域的**資料模型層**。它們從不同面向切入：IFC 從建築構件、CityGML 從城市地物、IndoorGML 從室內導航拓樸、OpenDRIVE 從道路網路——各有其主體、坐標系統與細緻度層級。成熟的跨尺度數位孿生需要同時整合這些標準。

---

## 參考文獻

[^bsi-ifc]: buildingSMART International. (n.d.). Industry Foundation Classes (IFC). Retrieved 2026-09-25, from https://www.buildingsmart.org/standards/bsi-standards/industry-foundation-classes/
[^ogc-citygml]: Open Geospatial Consortium. (n.d.). CityGML. Retrieved 2026-09-25, from https://www.ogc.org/standards/citygml/
[^ogc-indoorgml]: Open Geospatial Consortium. (n.d.). IndoorGML. Retrieved 2026-09-25, from https://www.ogc.org/standards/indoorgml/
[^ogc-indoorgml-spec]: Open Geospatial Consortium. (2024). OGC IndoorGML 2.0 Part 1: Conceptual Model. Retrieved 2026-09-25, from https://docs.ogc.org/is/22-045r5/22-045r5.html
[^asam-opendrive]: ASAM e.V. (n.d.). ASAM OpenDRIVE. Retrieved 2026-09-25, from https://www.asam.net/standards/detail/opendrive/
[^ogc-udt]: Open Geospatial Consortium. (2024). Urban Digital Twins: Integrating Infrastructure, natural environment and people (OGC Discussion Paper 24-025). Retrieved 2026-09-25, from https://docs.ogc.org/dp/24-025.html
[^3dg-ifc-citygml]: 3D Geospatial. (n.d.). IFC vs CityGML for Building Twins. Retrieved 2026-09-25, from https://www.3d-geospatial.com/3d-geospatial-fundamentals-for-digital-twins/3d-format-standards-comparison/ifc-vs-citygml-for-building-twins/
[^frontiers-ifc-dt]: Frontiers in Energy Research. (2024). Digital twin modeling method based on IFC standards for building construction processes. Retrieved 2026-09-25, from https://www.frontiersin.org/journals/energy-research/articles/10.3389/fenrg.2024.1334192/full
[^iop-ifc-dt]: IOP Science. (2025). IFC-based digital twin for real-time building data. Journal of Physics: Conference Series, 3140(4), 042023. Retrieved 2026-09-25, from https://iopscience.iop.org/article/10.1088/1742-6596/3140/4/042023
[^ieee-ifc-dt]: IEEE. (2025). Innovative IFC Classification in BIM Models Ready to be Converted Into Digital Twin. Retrieved 2026-09-25, from https://ieeexplore.ieee.org/document/11471271
[^mdpi-ifc-citygml]: MDPI Sensors. (2024). Digital Twin Smart City: Integrating IFC and CityGML with Semantic Graph, 24(12), 3761. Retrieved 2026-09-25, from https://www.mdpi.com/1424-8220/24/12/3761
[^ogc-dt-slides]: Lodha, K. (n.d.). OGC Standards for Digital Twins. Retrieved 2026-09-25, from https://krishnaglodha.quarto.pub/ogc-standards-for-digital-twins/
[^github-odr-dt]: ftgTUGraz. (2026). OpenDRIVE Digital Twin Generator. Retrieved 2026-09-25, from https://github.com/ftgTUGraz/opendrive-digital-twin-generator
[^arxiv-odr-dt]: arXiv. (2026). Automated Digital Twin Construction for Highway Scenarios Using LiDAR Point Clouds and OpenStreetMap, 2606.16570. Retrieved 2026-09-25, from https://arxiv.org/abs/2606.16570
[^asam-ogc-odr-citygml]: ASAM & OGC. (2026). OpenDRIVE with CityGML Concept Paper. Retrieved 2026-09-25, from https://publications.pages.asam.net/standards/ASAM_OpenDRIVE_with_CityGML/index.html
[^nibs-dt]: National Institute of Building Sciences. (2025). Digital Twins for the Built Environment. Retrieved 2026-09-25, from https://www.nibs.org/wp-content/uploads/2025/04/DigitalTwinsBuiltEnvironment.pdf
[^w78-integration]: ITC W78. (2024). Multiple schema integration through a common intermediate model: a study on IFC, CityGML, IndoorGML, and OpenStreetMap. Retrieved 2026-09-25, from https://itc.scix.net/pdfs/w78-2024-paper_129.pdf
[^mathworks-odr]: MathWorks. (n.d.). OpenDRIVE and Digital Twins. Retrieved 2026-09-25, from https://www.mathworks.com/