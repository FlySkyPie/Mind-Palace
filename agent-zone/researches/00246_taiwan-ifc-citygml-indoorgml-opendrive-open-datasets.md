# 台灣 IFC / CityGML / IndoorGML / OpenDRIVE 開放資料集調查

## 摘要

本報告調查台灣可取得之 IFC（Industry Foundation Classes）、CityGML（3D 城市模型）、IndoorGML（室內空間資料）及 OpenDRIVE（道路網路／自駕車高精地圖）四種開放資料格式之現況。調查範圍涵蓋政府開放資料平台、學術機構及民間資源。

調查結果顯示：**CityGML 最為豐富**（全台約 650 萬棟建築物，可透過國土測繪中心取得 KMZ 格式原始資料，其資料標準以 CityGML 2.0 為基礎）；**OpenDRIVE 亦有完整供應管道**（內政部 HD Maps 圖資平台提供 xodr 格式、學術界 hetroD 資料集亦提供 OpenDRIVE）；**IFC 與 IndoorGML 目前尚無可直接下載之開放資料集**。

---

## 1. IFC（Industry Foundations Classes）— 建築資訊模型（BIM）

### 現況

- **政府開放資料平台**（data.gov.tw）上**無任何 IFC 格式的原始 BIM 模型資料集**可供直接下載。[^data-gov-tw]
- 內政部建築研究所於 2018 年完成「城市共同管道 3D-GIS 與 BIM-IFC 資訊交換與操作機制研擬」研究案，探討 IFC 與 CityGML 之間的格式轉換機制與語意對應，但其成果為 PDF 研究報告而非開放資料集。[^abri]
- 公共工程委員會設有 BIM 專區，主要為規範與政策推動，未提供原始 IFC 檔案。[^pcc]
- 台灣建築中心（TABC）提供 BIM 資訊服務平台與元件庫展示平台，但非開放原始資料集。[^tabc]
- 中華建築資訊模型標準協會（CBIMSA）推廣 openBIM 與 IFC 標準，亦未提供公開資料集。[^cbimsa]

### 結論

❌ **台灣目前無公開、可直接下載之 IFC 開放資料集。** 相關資源以政府研究報告與政策規範為主。

---

## 2. CityGML — 3D 城市模型

CityGML 是台灣目前最完整、最成熟的開放地理空間資料格式之一。

### 2.1 內政部國土測繪中心 — 多維度國家空間資訊服務平臺

這是全台灣最大規模的 3D 城市模型資料來源。[^nlsc-3d]

| 項目 | 內容 |
|------|------|
| 平台網址 | https://3dmaps.nlsc.gov.tw |
| 資料標準 | 以 **CityGML 2.0** 為基礎制定「三維建物模型資料標準」（2022 年發布）及「三維道路模型資料標準」（2023 年發布），並參考 **CityGML 3.0**[^nlsc-standard] |
| 模型範圍 | 全台灣約 **650 萬棟建築物**，LOD1 層級（另有少數 LOD2、LOD3）[^nlsc-2025] |
| 提供格式 | ✅ **KMZ** — 原始資料下載格式<br>✅ **3D Tiles** — 線上串流服務（https://3dtiles.nlsc.gov.tw）<br>✅ **OGC I3S** — 線上串流服務（https://i3s.nlsc.gov.tw）[^nlsc-3dservice] |
| 申請方式 | 自然人憑證或工商憑證線上申請，可框選範圍即時下載（最多約 13,000 個模型） |
| 備註 | 2025 年起調整為收費供應機制，仍提供測試用圖資下載[^moi-2025] |

**重要澄清**：儘管其資料標準以 CityGML 2.0 為概念基礎，但**目前實際對外開放下載的格式為 KMZ**，而非原始 CityGML（GML）。使用者若要取得真正的 CityGML 格式，需自行轉換。中央研究院 GIS 專題中心亦提供了 CityGML 轉 KMZ 的轉換工具教學。[^sinica-citygml]

### 2.2 臺北市政府相關平台

| 平台名稱 | 連結 | 說明 |
|----------|------|------|
| 智慧城市 3D 臺北 | https://3d.taipei | 3D 城市模型展示平台 |
| 臺北市多維度測繪管理系統 | https://3d.land.gov.taipei | 三維產權建物模型查詢 |
| Taipei GIS City 3D | https://bim.udd.gov.taipei/gis3d | 都市發展局 3D GIS 平台 |

⚠️ 以上平台以線上瀏覽為主，未提供原始 CityGML 檔案下載。

### 2.3 第三方資源

- **GitHub sheethub/tpe3d**[^github-tpe3d]：將臺北市開放 KMZ 格式的 3D 近似建物模型轉換為 **GeoJSON** 格式。
- **中研院 SinicaView**[^sinica-view]：3D 時空資訊整合平台，整合政府與學術空間資料。

### 結論

✅ **CityGML 資料極為豐富**。全台 650 萬棟 LOD1 建築可透過國土測繪中心平台申請下載（原始格式為 KMZ，資料標準與 CityGML 2.0 相容）。若需純粹的 CityGML 格式，需自行轉換或使用 OGC 標準網路串流服務。

---

## 3. IndoorGML — 室內空間資料

### 現況

- 台灣各級政府開放資料平台上**無任何 IndoorGML 格式之開放資料集**。
- 室內定位與室內導航在台灣以**商業解決方案**為主，例如 Mapxus（香港商）[^mapxus]、聯合通科技、恆準定位等，皆為封閉格式。
- 學術界（如國立陽明交通大學 HCIS Lab、國立成功大學等）有相關研究，但未公開 IndoorGML 格式資料集。[^nctu-hcis]

### 結論

❌ **台灣目前無公開之 IndoorGML 開放資料集。** 若要取得室內空間資料，可能需要從 BIM/IFC 模型自行轉換，或與特定場域（如桃園機場、台北車站）洽談合作。

---

## 4. OpenDRIVE — 道路網路 / 自駕車高精地圖

### 4.1 內政部 — HD Maps 圖資供應平臺

這是由國立成功大學高精地圖研究發展中心代管營運之官方高精地圖供應平台。[^hdmap]

| 項目 | 內容 |
|------|------|
| 平台網址 | https://hdmap.colife.org.tw |
| 管理單位 | 國立成功大學高精地圖研究發展中心 |
| 提供格式 | ✅ **ODRIVE（.xodr）** — 台灣高精地圖格式，以 OpenDRIVE 為基礎之台灣客製化衍生版本<br>✅ **Lanelet2（.osm）** — 自駕車終端格式<br>✅ LAS 點雲檔、SHP 向量檔、JPG 影像資料 |
| 涵蓋範圍 | 各縣市自駕車測試場域道路圖資 |
| 申請方式 | 註冊登入後線上申請，通過審核後免費下載（需產學研用途） |
| 上線時間 | 2023 年正式上線 |

**注意**：平台提供之 xodr 格式被標示為「臺灣高精地圖格式-ODRIVE」，為基於 OpenDRIVE 之台灣客製化版本，可能與標準 OpenDRIVE 有差異。[^hdmap]

### 4.2 hetroD 資料集 — 台灣混合車流自駕車軌跡資料集

由國立陽明交通大學 HCIS Lab 與德國 fka GmbH、UC Berkeley 合作發布。[^hetrod]

| 項目 | 內容 |
|------|------|
| 資料集名稱 | **hetroD**（Heterogeneous Traffic Dataset） |
| 研究單位 | 國立陽明交通大學 HCIS Lab + UC Berkeley + fka GmbH |
| 格式 | ✅ **OpenDRIVE**（含 3D 資訊 .fbx & .osgb）<br>✅ **Lanelet2** |
| 規模 | 17.5 小時、6 個台灣路口、65,000+ 軌跡、近 70% 為弱勢道路使用者 |
| 亮點 | 台灣特有之機車密集混合車流場景 |
| 取得方式 | 填寫申請表，學術研究免費使用 |
| 發表 | IEEE ICRA 2026 |

### 4.3 其他相關單位

- **工研院（ITRI）** — 新竹開放場域自駕運行計畫
- **車輛研究測試中心（ARTC）** — 亞洲前瞻智慧車電自駕車測試場域
- **臺北市北投士林科技園區** — 自駕車場域實證計畫

### 結論

✅ **OpenDRIVE 資料供應管道完整。** 官方管道（HD Maps 圖資平臺）提供 xodr 格式下載；學術界則有 hetroD 資料集（標準 OpenDRIVE 格式），特別適合研究台灣混合車流場景。

---

## 綜合比較

| 格式 | 台灣開放資料現況 | 可用性評級 | 主要取得管道 |
|------|----------------|-----------|-------------|
| **IFC** | ❌ 尚無公開開放資料集 | ⭐ | — |
| **CityGML** | ✅ 全台 650 萬棟 LOD1 建築，資料標準以 CityGML 2.0 為基礎 | ⭐⭐⭐⭐⭐ | 國土測繪中心 3dmaps.nlsc.gov.tw |
| **IndoorGML** | ❌ 無公開資料集 | ⭐ | — |
| **OpenDRIVE** | ✅ 官方供應平台 + 學術資料集 | ⭐⭐⭐⭐ | HD Maps 平台 / hetroD |

---

## 建議取得路徑

1. **CityGML（實質 KMZ 下載）** → https://3dmaps.nlsc.gov.tw 申請
2. **CityGML / 3D Tiles 串流** → https://3dtiles.nlsc.gov.tw 或 https://i3s.nlsc.gov.tw
3. **OpenDRIVE（xodr）** → https://hdmap.colife.org.tw 註冊申請
4. **OpenDRIVE（學術研究專用）** → https://levelxdata.com/hetrod-dataset/ 申請 hetroD
5. **IFC → CityGML 轉換參考** → 內政部建研所研究報告（PDF）

---

## 參考文獻

[^data-gov-tw]: 政府資料開放平臺. (n.d.). Retrieved 2026-09-26, from https://data.gov.tw/
[^abri]: 內政部建築研究所. (2018). 城市共同管道 3D-GIS 與 BIM-IFC 資訊交換與操作機制研擬. Retrieved 2026-09-26, from https://www.abri.gov.tw/News_Content_Table.aspx?n=807&s=39536
[^pcc]: 行政院公共工程委員會. (n.d.). BIM 專區. Retrieved 2026-09-26, from https://www.pcc.gov.tw/content/index?eid=1345&type=C&lang=1
[^tabc]: 台灣建築中心. (n.d.). BIM 資訊服務. Retrieved 2026-09-26, from https://www.tabc.org.tw/tw/modules/pages/skill01
[^cbimsa]: 中華建築資訊模型標準協會. (n.d.). Retrieved 2026-09-26, from https://www.cbimsa.org/
[^nlsc-3d]: 內政部國土測繪中心. (n.d.). 多維度國家空間資訊服務平臺. Retrieved 2026-09-26, from https://3dmaps.nlsc.gov.tw
[^nlsc-standard]: 內政部國土測繪中心. (n.d.). 三維國家底圖—三維建物與道路資料標準. Retrieved 2026-09-26, from https://www.nlsc.gov.tw/cp.aspx?n=16731
[^nlsc-2025]: 內政部國土測繪中心. (2025). 3D 建物模型公告. Retrieved 2026-09-26, from https://www.nlsc.gov.tw/NLSC_Content.aspx?n=1454&sms=9680&s=339861
[^nlsc-3dservice]: 內政部國土測繪中心. (n.d.). 三維國家底圖—3D Tiles 服務. Retrieved 2026-09-26, from https://www.nlsc.gov.tw/cl.aspx?n=15874
[^moi-2025]: 內政部. (2025). 3D 建物模型新聞稿. Retrieved 2026-09-26, from https://www.moi.gov.tw/News_Content.aspx?n=9&s=339860
[^sinica-citygml]: 中央研究院 GIS 專題中心. (n.d.). CityGML 轉 KMZ 工具教學. Retrieved 2026-09-26, from https://gis.rchss.sinica.edu.tw/citygml2kmz/
[^github-tpe3d]: sheethub. (n.d.). tpe3d — 臺北市 3D 近似建物模型 GeoJSON. Retrieved 2026-09-26, from https://github.com/sheethub/tpe3d
[^sinica-view]: 中央研究院. (n.d.). SinicaView 3D 時空資訊整合平台. Retrieved 2026-09-26, from https://earth.rchss.sinica.edu.tw
[^mapxus]: Mapxus. (n.d.). 室內地圖導航解決方案. Retrieved 2026-09-26, from https://www.mapxus.com/zh-tw
[^nctu-hcis]: 國立陽明交通大學 HCIS Lab. (n.d.). 人機互動與智慧系統實驗室. Retrieved 2026-09-26, from https://sites.google.com/site/yitingchen0524/hcis-lab
[^hdmap]: 內政部. (2023). HD Maps 圖資供應平臺. Retrieved 2026-09-26, from https://hdmap.colife.org.tw
[^hetrod]: Level X Data. (2026). hetroD Dataset — Heterogeneous Traffic Dataset for Autonomous Driving. Retrieved 2026-09-26, from https://levelxdata.com/hetrod-dataset/