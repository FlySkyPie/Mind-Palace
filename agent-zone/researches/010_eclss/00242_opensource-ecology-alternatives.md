# Open Source Ecology 替代方案調查報告

## 概述

Open Source Ecology（OSE）是一個由 Marcin Jakubowski 於 2003 年創立的開放原始碼硬體組織，以 Global Village Construction Set（GVCS）為核心，目標是開發 50 種能夠建立「小型文明」的工業機器，涵蓋農業、營造、能源與製造等領域[^ose-wiki]。本報告調查與 OSE 理念相近的開放原始碼硬體、適當科技（Appropriate Technology）及自給自足導向的替代組織與專案，並比較其差異。

## 主要替代方案

### 農業與農機領域

#### 1. L'Atelier Paysan（農民工坊）

- **網址**：https://www.latelierpaysan.org/
- **成立年份**：約 2009-2010 年
- **所在地**：法國
- **描述**：法國合作社組織，支持農民自行設計與建造農機設備。提供 **130 種以上** 的自由開源設計圖（CC BY-NC-SA 授權），包含中耕機、播種機、收穫機、溫室、烘窯等。其核心理念是「奪回農場的技術自主權」，反對大型農業機械對小農的依賴結構。[^atelier-paysan]
- **與 OSE 的差異**：只聚焦農業機械（不含營建、能源、製造），且有強烈的政治教育色彩，近年經歷組織重整（2026 年 4 月 SCIC 進入清算，由 Communs Paysans 與 Soudons, fermes! 兩個協會接續營運）。

#### 2. FarmBot

- **網址**：https://farmbot.io/
- **成立年份**：2011 年
- **所在地**：美國（創辦人 Rory Aronson）
- **描述**：開放原始碼的精準農業 CNC 機器人，採用龍門架系統（Cartesian coordinate robot），可自動播種、澆水、除草與土壤感測。硬體設計與軟體皆為開放原始碼（MIT 授權），已在全球 500 所以上教育機構使用。[^farmbot-wiki][^farmbot-site]
- **與 OSE 的差異**：單一產品（精密自動化菜園機器人），規模較小，且以商業銷售套件為主要營運模式，而非 OSE 的 50 種工業機器組合。

#### 3. WikiMaraîcher

- **網址**：https://wikimaraicher.ca/
- **所在地**：加拿大魁北克
- **描述**：魁北克的法語維基平台，彙整自建市場園藝設備的設計圖，部分與 L'Atelier Paysan 交叉收錄。
- **與 OSE 的差異**：單一地區、單一領域的小型知識庫，缺乏組織性開發。

### 建材營造與住房領域

#### 4. Open Building Institute（OBI）

- **網址**：https://openbuildinginstitute.org/
- **成立年份**：2016 年（與 OSE 合併）
- **描述**：OSE 在住房領域的衍生組織，開發開放原始碼的生態住宅模組。原型專案 Seed Eco-Home 是一座 1,400 平方英尺的房屋，50 人可在 5 天內完成建造，材料成本約 30,000-50,000 美元。[^obi]
- **與 OSE 的差異**：只聚焦住房與建築，本質上是 OSE 的建築分支。

#### 5. WikiHouse

- **網址**：https://wikihouse.cc/
- **成立年份**：2011 年
- **所在地**：英國（由 00 設計事務所與 Open Systems Lab 維護）
- **描述**：開放原始碼建築系統，用 CNC 切割膠合板製成類似拼圖的建築模塊，無需專業技能即可在一天內組裝房屋骨架。設計檔案以 Creative Commons 授權發布。[^wikihouse-wiki]
- **與 OSE 的差異**：使用單一建材（膠合板）與單一製造方法（CNC 切割），而非 OSE 的多樣化建材與機器組合。較偏向建築設計社群而非製造工程。

#### 6. OpenStructures

- **網址**：https://openstructures.net/
- **描述**：基於共享幾何網格（40mm 方格的「OS grid」）的開放原始碼模組化建構系統。任何零件、元件或結構只要符合此網格標準即可互通。[^openstructures]
- **與 OSE 的差異**：偏概念性與設計哲學，以通用標準取代製造機器；沒有實際的機器藍圖。

### 回收與材料加工領域

#### 7. Precious Plastic

- **網址**：https://www.preciousplastic.com/
- **成立年份**：2013 年
- **所在地**：荷蘭恩荷芬（創辦人 Dave Hakkens）
- **描述**：開放原始碼塑膠回收機器專案，涵蓋碎紙機、擠出機、射出成型機與壓縮機。全球 1,100 多個工作空間，56 個國家參與，每年回收約 140 萬公斤塑膠。採 CC-BY-SA 4.0 授權。[^precious-plastic-wiki]
- **與 OSE 的差異**：只處理塑膠回收單一材料流，但全球社群規模遠超 OSE，商業模式也較成熟（含市集與驗證工作空間制度）。

### 製造與工具機領域

#### 8. RepRap Project

- **網址**：https://reprap.org/
- **成立年份**：2005 年
- **描述**：自我複製的開放原始碼 3D 列印機專案，是桌上型 3D 列印革命的源頭。核心概念是讓機器能夠列印自身大部分零件。[^reprap]
- **與 OSE 的差異**：單一產品（3D 列印機），但其「自我複製」概念與 OSE 的自給自足哲學高度共鳴。

#### 9. Maslow CNC

- **網址**：https://www.maslowcnc.com/
- **描述**：低成本的開放原始碼 CNC 銑床（垂直壁掛設計，約 500 美元）。可用於家具與板材加工。
- **與 OSE 的差異**：單一工具機專案，範圍較小。

### 知識平台與協作基礎設施

#### 10. Appropedia

- **網址**：https://www.appropedia.org/
- **成立年份**：2006 年
- **描述**：最大的適當科技維基平台，收錄 4,500 多個專案（太陽能烹飪、雨水收集、堆肥廁所、自然建築等）。與密西根理工大學的 Open Sustainability Technology Lab 密切合作。[^appropedia]
- **與 OSE 的差異**：知識庫而非硬體開發組織；不生產自己的機器設計，而是彙整與記錄現有專案。涵蓋範圍更廣但深度較淺。

#### 11. Wikifactory

- **網址**：https://wikifactory.com/
- **描述**：開放原始碼硬體協作平台，超過 20,000 個專案，支援 CAD 檔案協作與版本管理。涵蓋機器人、無人機、農業科技、家具等領域。
- **與 OSE 的差異**：平台公司而非單一任務組織，沒有如 GVCS 的策展式文明套件。

### 發展與適當科技領域

#### 12. Practical Action

- **網址**：https://practicalaction.org/
- **成立年份**：1966 年（原名 Intermediate Technology，由 E.F. Schumacher 創立）
- **描述**：國際發展組織，在開發中國家推廣低成本、適當科技方案（農業、能源、水資源、氣候韌性）。受 Schumacher 的《小即是美》（Small Is Beautiful）影響深遠。[^practical-action]
- **與 OSE 的差異**：傳統 NGO，非開放原始碼硬體專注，但在哲學上共享「給予社區自建工具的能力」的理念。透過既有援助體系運作，而非試圖「重新啟動文明」。

#### 13. Fab Lab Network / Fab Foundation

- **網址**：https://fabfoundation.org/
- **描述**：由 MIT Neil Gershenfeld 教授發起的全球數位製造實驗室網絡，2,000 多個實驗室遍布全球。每個 Fab Lab 配備雷射切割機、CNC、3D 列印機、電子工作檯等標準設備。
- **與 OSE 的差異**：物理工作空間網絡而非機器設計專案。使用商業／工業工具而非開放原始碼工具，定位為教育與研究機構。

#### 14. Low-Tech Magazine

- **網址**：https://solar.lowtechmagazine.com/
- **描述**：探討低科技、低能耗、適當科技解決方案的部落格與出版物。網站本身由太陽能供電（沒有陽光時會離線）。
- **與 OSE 的差異**：純出版品，不生產機器藍圖。

## 比較摘要

| 組織 | 焦點領域 | 規模 | 成熟度 | OSE 可比項目 |
|------|---------|------|--------|-------------|
| **L'Atelier Paysan** | 小型農業機械 | 歐洲（法國為主） | 成熟（130+ 設計圖） | OSE 的農業機器群 |
| **FarmBot** | 精準農業機器人 | 全球 500+ 教育機構 | 商品化產品 | OSE 的曳引機／播種機 |
| **Open Building Institute** | 生態住房 | 美國 | 早期階段 | OSE 的住房／營造 |
| **WikiHouse** | 膠合板建築 | 全球原型 | 已建立 | OSE 的鋸木廠與 CEB 壓磚機 |
| **Precious Plastic** | 塑膠回收 | 全球 1,100+ 工作空間 | 非常成熟 | OSE 的工業機器＋社群模式 |
| **RepRap** | 3D 列印 | 全球大型社群 | 非常成熟 | OSE 的 3D 列印機＋自我複製概念 |
| **Appropedia** | 適當科技知識庫 | 全球 | 非常成熟 | OSE 的維基與文件 |
| **Practical Action** | 適當科技發展 | 全球南方 | 非常成熟 | OSE 的適當科技哲學 |
| **OpenStructures** | 模組化設計標準 | 概念／設計圈 | 早期階段 | OSE 的 Construction Set Design |

## 關鍵觀察

1. **最接近的替代方案**：L'Atelier Paysan（農業機械）、Precious Plastic（開放機器＋社群模式）、Open Building Institute（OSE 直接衍生）。
2. **缺乏直接競爭者**：目前沒有任何其他組織像 OSE 一樣試圖建立橫跨農業、營造、能源與製造的 50 種「文明套件」。大多數替代方案都專注於單一領域。
3. **歐洲生態系較豐富**：法國（L'Atelier Paysan）、英國（WikiHouse）、荷蘭（Precious Plastic）皆有成熟的開放原始碼硬體組織，美國反而以單一產品專案為主。
4. **知識平台與硬體開發的分工**：Appropedia 與 Wikifactory 提供協作基礎設施但不生產機器，類似 OSE 生態系中的輔助角色。

## 參考資料

[^ose-wiki]: Wikipedia. (n.d.). Open Source Ecology. Retrieved 2026-09-25, from https://en.wikipedia.org/wiki/Open_Source_Ecology
[^atelier-paysan]: L'Atelier Paysan. (n.d.). Qui sommes-nous. Retrieved 2026-09-25, from https://www.latelierpaysan.org/Qui-sommes-nous/
[^farmbot-wiki]: Wikipedia. (n.d.). FarmBot. Retrieved 2026-09-25, from https://en.wikipedia.org/wiki/FarmBot
[^farmbot-site]: FarmBot. (n.d.). Open-Source CNC Farming. Retrieved 2026-09-25, from https://farmbot.io/
[^obi]: Open Building Institute. (n.d.). Retrieved 2026-09-25, from https://openbuildinginstitute.org/
[^wikihouse-wiki]: Wikipedia. (n.d.). WikiHouse. Retrieved 2026-09-25, from https://en.wikipedia.org/wiki/WikiHouse
[^openstructures]: OpenStructures. (n.d.). Retrieved 2026-09-25, from https://openstructures.net/
[^precious-plastic-wiki]: Wikipedia. (n.d.). Precious Plastic. Retrieved 2026-09-25, from https://en.wikipedia.org/wiki/Precious_Plastic
[^reprap]: RepRap Project. (n.d.). Retrieved 2026-09-25, from https://reprap.org/wiki/RepRap
[^appropedia]: Appropedia. (n.d.). Retrieved 2026-09-25, from https://www.appropedia.org/
[^practical-action]: Practical Action. (n.d.). Retrieved 2026-09-25, from https://practicalaction.org/