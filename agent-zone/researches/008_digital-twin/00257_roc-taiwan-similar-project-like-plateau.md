# 中華民國（臺灣）類似日本 Project PLATEAU 之國家級計畫調查

## 背景說明

日本 **Project PLATEAU** 是由國土交通省（MLIT）主導的國家級數位雙生（Digital Twin）計畫，自 2020 年起以 **CityGML**（OGC 國際標準）為核心，建置全國 3D 城市模型，並將資料以開放資料（Open Data）形式釋出，累計覆蓋超過 250 個城市，目標於 2027 年前達約 500 個城市[^plateau-en][^plateau-about]。

本報告旨在調查中華民國（臺灣）是否存在性質、規模或技術取徑上與 PLATEAU 相似的計畫。

## 主要發現：臺灣確實存在多項高度相似的計畫

臺灣最接近 PLATEAU 的核心計畫由 **內政部國土測繪中心（National Land Surveying and Mapping Center, NLSC）** 主導，受 **國家發展委員會** 政策推動，並經 **行政院** 核定[^nlsc-3d-base][^ndc]。

### 1. 邁向 3D 智慧國土－國家底圖空間資料基礎建設計畫（110-114 年）

- **主辦單位**：內政部（國土測繪中心執行）
- **核定機關**：行政院（109 年 5 月 26 日院臺建字第 1090012087 號函核定）
- **時間範圍**：2021 年～2025 年
- **內容**：將 2D 國家底圖升級為 3D 國家底圖，以空載光達（LiDAR）持續更新全臺數值地形模型（DTM）；產製全國三維建物模型及三維道路模型；訂定三維資料標準；建置「多維度國家空間資訊服務平臺」[^nlsc-3d-plan]。

### 2. 邁向 3D 智慧國土－國家底圖空間資料基礎建設延續計畫（115-119 年）

- **時間範圍**：2026 年～2030 年
- **內容**：延續前期，以 LiDAR 持續更新 DTM[^nlsc-dtm]。

### 3. 多維度空間資訊基礎圖資測製及更新計畫（112-116 年）

- **核定機關**：行政院（111 年 11 月 11 日院臺建字第 1110019092 號函核定）
- **時間範圍**：2023 年～2027 年
- **這項計畫與 PLATEAU 的 3D 城市建築模型理念最為一致。** 具體涵蓋：
  - **1/1000 地形圖** 及 **三維網格模型（Mesh 模型）** 各 **16.32 萬公頃**
  - **LOD2 建物模型**（含屋頂結構） **1.632 萬公頃**
  - **LOD3 精細建物模型** **800 棟**
  - 策略：「第 1 年航拍取像，第 2 年製圖建模」
  - 截至 2024 年已於 14 個縣市完成 28,970 公頃圖資[^nlsc-multi-dim]

### 4. 三維國家底圖建置（核心業務項目，2019 年起）

- 產製全臺三維建物模型（支援 LOD1、LOD2）
- 產製三維道路模型
- 發布符合 **OGC I3S** 及 **3D Tiles** 標準的三維網路服務
- 2026 年 7 月已釋出 2025 年版三維建物模型（含臺北市、新北市、臺中市等 8 縣市）；2026 年 9 月更新三維道路模型（含國道及快速公路）[^nlsc-news-2025][^nlsc-news-road]

### 5. 多維度國家空間資訊服務平臺（Taiwan 3D Map Service）

- **網址**：https://3dmaps.nlsc.gov.tw
- **功能**：導入全國三維建物模型、三維道路模型、三維影像模型；提供 OGC I3S 及 3D Tiles 網路服務；超過 **185 個系統** 介接使用（如國家災害防救中心災害情資網、水利署 GIS 平台等）；每月以 20～30 萬人次持續成長[^nlsc-3d-service]。

### 6. 三維資料標準制定（2020-2023 年）

- **三維建物模型資料標準**（2022 年 8 月公布）：以 **CityGML 2.0** 為基礎
- **三維道路模型資料標準**（2023 年 12 月公布）：參考 **CityGML 2.0 及 3.0**
- 確保資料格式一致性，產製符合標準的 GML 開放格式[^nlsc-standard]

## 與 Project PLATEAU 之對照比較

| 比較項目 | 日本 PLATEAU | 臺灣（內政部國土測繪中心） |
|---|---|---|
| 主導機關 | 國土交通省（MLIT） | 內政部（國土測繪中心執行） |
| 政策統籌 | — | 國家發展委員會 |
| 啟動年度 | 2020 年 | 2019 年（三維建物產製）／2021 年（核定計畫） |
| 核心計畫 | 3D 都市モデル整備 | 邁向 3D 智慧國土／多維度空間資訊基礎圖資測製 |
| 建物模型等級 | LOD1、LOD2（全國）、LOD3、LOD4 | LOD1、LOD2、LOD3（800 棟） |
| 資料標準 | CityGML（PLATEAU Standard v5.1） | CityGML 2.0 為基礎 |
| 網路服務格式 | 3D Tiles、MVT | OGC I3S、3D Tiles、KMZ |
| 瀏覽平台 | PLATEAU VIEW | 多維度國家空間資訊服務平臺 |
| 覆蓋範圍 | 250+ 城市，目標 500 城市 | 逐步擴展中，LOD2 覆蓋 1.632 萬公頃 |
| 開放授權 | 開放資料（含商業使用） | 政府開放資料（部分需申請／付費） |
| 開源生態 | 重度開源（GitHub: Project-PLATEAU） | 無對應開源專案 |

## 關鍵差異

1. **開源生態**：PLATEAU 在 GitHub 上擁有大量開源工具（PLATEAU SDK、GIS Converter、CesiumJS/QGIS/Unreal 外掛等）[^plateau-github]；臺灣目前無對應的開源程式碼專案，資料以 KMZ 格式提供或透過 OGC 標準網路服務存取，部分資料需經「臺灣地圖商店」申請購買[^nlsc-map-store]。

2. **格式策略**：PLATEAU 以 CityGML 為原生格式再轉換多種輸出；臺灣則以 KMZ 及 OGC I3S/3D Tiles 網路服務為主，三維建物模型資料標準雖以 CityGML 2.0 為基礎，但終端發布格式並非 CityGML。

3. **數位發展部角色**：**數位發展部（moda）** 主要聚焦於數位政府、資通安全、開放資料標準等，並非 3D 城市模型或數位雙生的主導機關[^moda]。

## 結論

**中華民國（臺灣）確實存在多項與日本 Project PLATEAU 高度相似的國家級計畫。** 其中最核心的是行政院核定、內政部國土測繪中心執行的「邁向 3D 智慧國土」系列計畫及「多維度空間資訊基礎圖資測製及更新計畫」。這些計畫在技術標準（CityGML、3D Tiles）、模型細緻度（LOD1/2/3）及公開服務平台等面向與 PLATEAU 高度一致。主要差異在於開源生態的成熟度及資料開放授權方式。

## 參考來源

[^plateau-en]: Ministry of Land, Infrastructure, Transport and Tourism. (n.d.). Project PLATEAU (English). Retrieved 2026-09-27, from https://www.mlit.go.jp/plateau/en/

[^plateau-about]: Ministry of Land, Infrastructure, Transport and Tourism. (n.d.). PLATEAU About（プロジェクトについて）. Retrieved 2026-09-27, from https://www.mlit.go.jp/plateau/about/

[^plateau-github]: Project PLATEAU. (n.d.). GitHub Organization. Retrieved 2026-09-27, from https://github.com/Project-PLATEAU

[^nlsc-3d-base]: 內政部國土測繪中心. (n.d.). 三維國家底圖建置. Retrieved 2026-09-27, from https://www.nlsc.gov.tw/cl.aspx?n=15874

[^nlsc-3d-plan]: 內政部國土測繪中心. (n.d.). 國家底圖空間資料基礎建設計畫. Retrieved 2026-09-27, from https://www.nlsc.gov.tw/cp.aspx?n=16733

[^nlsc-multi-dim]: 內政部國土測繪中心. (n.d.). 多維度空間資訊基礎圖資測製及更新計畫. Retrieved 2026-09-27, from https://www.nlsc.gov.tw/cp.aspx?n=17087

[^nlsc-dtm]: 內政部國土測繪中心. (n.d.). 數值地形模型. Retrieved 2026-09-27, from https://www.nlsc.gov.tw/cp.aspx?n=1853

[^nlsc-3d-service]: 內政部國土測繪中心. (n.d.). 多維度國家空間資訊服務平臺. Retrieved 2026-09-27, from https://www.nlsc.gov.tw/cp.aspx?n=16731

[^nlsc-standard]: 內政部國土測繪中心. (n.d.). 三維資料標準. Retrieved 2026-09-27, from https://www.nlsc.gov.tw/cp.aspx?n=16731

[^nlsc-news-2025]: 內政部國土測繪中心. (2026). 2025 年三維建物模型成果開放下載. Retrieved 2026-09-27, from https://www.nlsc.gov.tw/en/News_Content.aspx?n=2109&sms=10297&s=339862

[^nlsc-news-road]: 內政部國土測繪中心. (2026). 三維道路模型更新成果開放下載. Retrieved 2026-09-27, from https://www.nlsc.gov.tw/en/News_Content.aspx?n=2109&sms=10297&s=340864

[^nlsc-map-store]: 內政部國土測繪中心. (n.d.). 臺灣地圖商店. Retrieved 2026-09-27, from https://whgis-nlsc.moi.gov.tw

[^ndc]: 國家發展委員會. (n.d.). 國發會官方網站. Retrieved 2026-09-27, from https://www.ndc.gov.tw/

[^moda]: 數位發展部. (n.d.). 官方網站. Retrieved 2026-09-27, from https://www.moda.gov.tw/