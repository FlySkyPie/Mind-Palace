# TU Delft 兩位數 LOD（如 LOD1.2、LOD2.2）是否為 CityJSON 標準的一部份？

## 摘要

由 TU Delft 研究員 Filip Biljecki、Hugo Ledoux 與 Jantien Stoter 於 2016 年提出的改良式 LOD（Level of Detail）細分規範（兩位數 LOD，例如 LOD1.2、LOD2.1、LOD2.2），**已正式納入 CityJSON v2.0.2 標準**，不僅在規格書中明文允許，亦列於 JSON Schema 的列舉值中，並於規格書範例中實際使用。CityJSON 本身已成為 OGC（Open Geospatial Consortium）官方標準（文件編號 20-072r5），故兩位數 LOD 實質上已是國際標準的一部分。

## 1. 什麼是兩位數 LOD？

傳統 CityGML 將建築物模型的細節等級分為 LOD0 至 LOD4 共五個粗粒度等級。兩位數 LOD 改良系統將每個等級細分為小數點後的子層級，共定義了 16 種 LOD 值（涵蓋 0.0 至 3.3 的網格），例如：

- **LOD1**：無屋頂結構的方塊模型
- **LOD1.2**：具備一定屋頂結構細節的方塊模型
- **LOD2**：具屋頂結構與紋理的建築模型
- **LOD2.2**：LOD2 範圍內更細緻的變體

如此可更精確地描述與比較 3D 資料集的幾何細節程度。[^biljecki2016]

## 2. 是否為 CityJSON 標準的一部分？

**是，完全正式支援。** 具體體現在三處：

### 2.1 規格書明文允許

CityJSON v2.0.2 規格第 3 節（Geometry Objects）中明確指出：

> A Geometry object **must** have one member with the name `"lod"`. The value must be a string with the LoD identifying the level-of-detail (LoD) of the geometry. This can be either a single digit (following the CityGML standards), or "X.Y"-formatted if the improved LoDs by TU Delft are used.[^spec-lod]

即：`lod` 成員為必填，其值為字串，可以是單一數字（遵循 CityGML）或 `X.Y` 格式（若使用 TU Delft 改良版 LOD）。

### 2.2 JSON Schema 明確定義

在 CityJSON 的幾何體原始 JSON Schema 中，`"Lods"` 的列舉值完整包含以下所有合法值：[^schema]

```
"0", "1", "2", "3",
"0.0", "0.1", "0.2", "0.3",
"1.0", "1.1", "1.2", "1.3",
"2.0", "2.1", "2.2", "2.3",
"3.0", "3.1", "3.2", "3.3"
```

### 2.3 規格範例實際使用

CityJSON 規格文件中的範例即使用了兩位數 LOD，例如：

- `"lod": "2.2"` 出現在 `CompositeSolid` 範例中
- `"lod": "2.1"` 與 `"lod": "1.3"` 出現在幾何模板（Geometry Templates）範例中[^spec-examples]

## 3. 提出者與背景關係

### 提出者

改良式 LOD 系統由荷蘭 **TU Delft**（Delft University of Technology）的 Filip Biljecki、Hugo Ledoux 與 Jantien Stoter 於 2016 年發表：[^biljecki2016]

> Biljecki, F., Ledoux, H., & Stoter, J. (2016). An improved LOD specification for 3D building models. *Computers, Environment and Urban Systems*, 59, 25–37. DOI: 10.1016/j.compenvurbsys.2016.04.005

值得注意的是，**Hugo Ledoux 同時也是 CityJSON 規格的核心編輯者與開發者之一**，這也解釋了為何兩位數 LOD 能順暢地被 CityJSON 採用。

### CityJSON 與 CityGML 在 LOD 上的關係

- **CityGML** 傳統上定義 LOD0–LOD4 五個等級，使用單一數字。
- **CityJSON** 是 CityGML 資料模型的 JSON 編碼替代格式，在 LOD 上同時支援傳統單一數字（向後相容）與 TU Delft 兩位數 LOD（提供更高精確度）。
- CityJSON v2.0.2 已是 **OGC 官方標準**（文件編號 20-072r5）[^ogc]，因此兩位數 LOD 也隨之成為國際標準的一環。
- 注意：CityJSON 並**不支援 LOD4**，因為 CityJSON 實作的是 CityGML 3.0.0 的子集，而 LOD4 已在 CityGML 3.0 中被移除。

## 結論

TU Delft 提出的兩位數 LOD 系統**確實是 CityJSON 標準的正式組成部分**——從規格文字、JSON Schema 到範例均有完整支援，且隨 CityJSON 通過 OGC 標準化程序而獲得國際標準地位。

## 參考文獻

[^biljecki2016]: Biljecki, F., Ledoux, H., & Stoter, J. (2016). An improved LOD specification for 3D building models. *Computers, Environment and Urban Systems*, 59, 25–37. Retrieved 2026-09-27, from https://doi.org/10.1016/j.compenvurbsys.2016.04.005

[^spec-lod]: CityJSON Specification v2.0.2, Section 3 – Geometry Objects: "lod" member description. Retrieved 2026-09-27, from https://www.cityjson.org/specs/2.0.2/#geometry-objects

[^schema]: CityJSON GitHub – geomprimitives.schema.json, "Lods" enum definition. Retrieved 2026-09-27, from https://raw.githubusercontent.com/cityjson/specs/main/schemas/geomprimitives.schema.json

[^spec-examples]: CityJSON Specification v2.0.2 – Examples section: CompositeSolid and Geometry Templates using two-digit LODs. Retrieved 2026-09-27, from https://www.cityjson.org/specs/2.0.2/#geometry-objects

[^ogc]: OGC CityJSON Standard 20-072r5. Retrieved 2026-09-27, from https://docs.ogc.org/cs/20-072r5/20-072r5.html