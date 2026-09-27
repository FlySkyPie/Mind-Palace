# 開源權重機器學習模型：將影像或點雲轉換為 CityGML/CityJSON

本報告調查目前已公開之開源（open-weight）機器學習模型，可將衛星影像、街景影像或 LiDAR 點雲轉換為 CityGML 或 CityJSON 格式之三維城市模型。

## 既有工具概觀

目前最具規模的開源方案來自日本國土交通省（MLIT）的 **Project PLATEAU**，提供多套基於深度學習的 CityGML 生成工具。此外學術界亦有 HEAT、PolyDiffuse、RoomFormer 等模型可用於建築物平面圖重建，但需後處理才能產出 CityGML 格式。

---

## 1. Project PLATEAU — 3D-City-Model-Generator（最完整的 ML 方案）

- **程式碼**: [github.com/Project-PLATEAU/3D-City-Model-Generator](https://github.com/Project-PLATEAU/3D-City-Model-Generator)
- **輸入**: 衛星影像 + 可選街景影像 + 建築物腳印參數
- **輸出**: CityGML（PLATEAU v4 格式），LOD1–LOD3
- **技術**: PyTorch、transformers、OpenCLIP、CLIP、timm、vector-quantize-pytorch
- **權重狀態**: ✅ 程式碼完整開源，但模型權重需自行訓練或從 PLATEAU 專案獲取授權下載

此工具使用深度學習進行實例分割（instance segmentation）與語義分割（semantic segmentation）來提取建築物輪廓、屋頂形狀、道路與植栽，再產生含 LOD1–LOD3 建築物、道路、植栽與都市家具的 CityGML 模型[^plateau3dgen]。

[^plateau3dgen]: Project PLATEAU. (n.d.). 3D-City-Model-Generator. Retrieved 2026-09-27, from https://github.com/Project-PLATEAU/3D-City-Model-Generator

## 2. Project PLATEAU — Auto-Create-bldg-lod2-tool（點雲 → LOD2）

- **程式碼**: [github.com/Project-PLATEAU/Auto-Create-bldg-lod2-tool](https://github.com/Project-PLATEAU/Auto-Create-bldg-lod2-tool)
- **輸入**: DSM（數值表面模型）點雲 + 建築物腳印 + LOD1 CityGML
- **輸出**: LOD2 CityGML（含紋理）
- **技術**: PyTorch，使用 **HEAT（Holistic Edge Attention Transformer）** 進行結構化三維建築物重建，以及深度學習超解析度（super-resolution）強化屋頂與牆面紋理
- **權重狀態**: ✅ 程式碼開源，HEAT 權重另可下載（見下節）

這套工具將 HEAT 模型的平面圖輸出轉為 LOD2 幾何，並從航空照片進行紋理映射[^plateaulod2]。

[^plateaulod2]: Project PLATEAU. (n.d.). Auto-Create-bldg-lod2-tool. Retrieved 2026-09-27, from https://github.com/Project-PLATEAU/Auto-Create-bldg-lod2-tool

## 3. HEAT（Holistic Edge Attention Transformer）

- **論文**: CVPR 2022 [arxiv.org/abs/2111.15143](https://arxiv.org/abs/2111.15143)
- **程式碼**: [github.com/woodfrog/heat](https://github.com/woodfrog/heat)（137 stars，GPL-3.0）
- **輸入**: 二維光柵影像（衛星影像或點雲密度圖）
- **輸出**: 平面圖（planar graph），包含建築物輪廓與角點
- **權重**: ✅ 可下載 — [Dropbox 連結](https://www.dropbox.com/scl/fi/57kxrtdwma8h9m2osnjn5/heat_checkpoints.zip?rlkey=77cso90mi4aroj4wpbh0w2tiv&st=k1oi772j&dl=0)，提供 256px 與 512px 室外建築重建權重，以及 S3D 室內平面圖權重
- **限制**: 輸出為二維平面圖，需額外處理才能轉為 CityGML 三維幾何

HEAT 偵測影像中的角點並對邊緣候選進行分類，重建出建築物輪廓的平面圖[^heat]。

[^heat]: Chen, J., Qian, Y., & Furukawa, Y. (2022). HEAT: Holistic Edge Attention Transformer for Structured Reconstruction. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR)*. Retrieved 2026-09-27, from https://arxiv.org/abs/2111.15143

## 4. PolyDiffuse（多邊形擴散模型）

- **論文**: NeurIPS 2023 [arxiv.org/abs/2306.01461](https://arxiv.org/abs/2306.01461)
- **程式碼**: [github.com/woodfrog/poly-diffuse](https://github.com/woodfrog/poly-diffuse)（162 stars，GPL-3.0）
- **輸入**: 視覺感測器資料（影像或點雲）
- **輸出**: 多邊形形狀（建築物平面圖、HD 地圖的道路元素）
- **權重**: ✅ 可下載 — [Dropbox 連結](https://www.dropbox.com/scl/fi/f0z0x1xfy6qi0bbmxwgh4/s3d_pretrained_ckpts.zip?rlkey=w1wmjrrwvlwxf7jyhvj2vj8bj&st=0yz1p8en&dl=0)，包含 guidance network 與 denoising network
- **限制**: 與 HEAT 相同，輸出為二維多邊形，需後處理

PolyDiffuse 使用引導式集合擴散過程（guided set diffusion），透過 RoomFormer 的 proposal generator 加上去噪網路來重建多邊形形狀[^polydiffuse]。

[^polydiffuse]: Chen, J., Qian, Y., Huang, Y.-C., Chiu, L., Cheng, H.-Y., & Furukawa, Y. (2023). PolyDiffuse: Polygonal Shape Reconstruction via Guided Set Diffusion. In *Advances in Neural Information Processing Systems (NeurIPS)*. Retrieved 2026-09-27, from https://arxiv.org/abs/2306.01461

## 5. RoomFormer（兩層查詢平面圖重建）

- **論文**: CVPR 2023 [arxiv.org/abs/2211.15658](https://arxiv.org/abs/2211.15658)
- **程式碼**: [github.com/ywyue/RoomFormer](https://github.com/ywyue/RoomFormer)（339 stars，MIT 授權）
- **輸入**: 三維室內掃描資料
- **輸出**: 語義平面圖，含多個房間多邊形、門窗與房間類型
- **權重**: ✅ 可下載 — [Polybox 連結](https://polybox.ethz.ch/index.php/s/vlBo66X0NTrcsTC)
- **潛力**: 最接近 CityGML LOD4（室內結構）的開源模型，因其輸出包含房間語義（類型、門窗）

RoomFormer 使用兩層查詢機制（polygon-level 與 corner-level），在一個階段內平行預測所有房間多邊形[^roomformer]。

[^roomformer]: Yue, Y., Chen, J., Huang, Y.-C., Chiu, L., & Furukawa, Y. (2023). RoomFormer: Two-Level Queries for Floorplan Reconstruction. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR)*. Retrieved 2026-09-27, from https://arxiv.org/abs/2211.15658

## 6. CityDreamer（無邊界三維城市生成）

- **論文**: CVPR 2024 [arxiv.org/abs/2309.00610](https://arxiv.org/abs/2309.00610)
- **程式碼**: [github.com/hzxie/CityDreamer](https://github.com/hzxie/CityDreamer)（700+ stars，NTU S-Lab License 1.0）
- **HuggingFace**: [huggingface.co/hzxie/city-dreamer](https://huggingface.co/hzxie/city-dreamer) + [Demo Space](https://huggingface.co/spaces/hzxie/city-dreamer)
- **輸入**: 隨機噪聲（生成式，非從真實影像/點雲重建）
- **輸出**: 無邊界三維城市場景（BEV 體積渲染）
- **權重**: ✅ 可下載三組預訓練權重（LayoutGen、背景、建築物生成器）
- **限制**: 生成式模型，不產出 CityGML 語義結構；輸出為神經場，非顯式幾何

CityDreamer 使用組合式生成模型，以分離的神經場分別處理建築物實例與背景元素（道路、綠地）[^citydreamer]。

[^citydreamer]: Xie, H., Chen, Z., You, F., Hu, Z., Zhang, H., & Chen, W. (2024). CityDreamer: Compositional Generative Model for Unbounded 3D Cities. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR)*. Retrieved 2026-09-27, from https://arxiv.org/abs/2309.00610

## 7. 3D_building_reconstruction（街景影像 → LOD3 CityGML）

- **程式碼**: [github.com/chrise96/3D_building_reconstruction](https://github.com/chrise96/3D_building_reconstruction)（71 stars）
- **輸入**: 全景街景影像 + LOD2 CityGML
- **輸出**: LOD3 CityGML（含門窗開口）
- **技術**: Faster/Mask R-CNN 深度卷積神經網路，三階段流程：紋理提取 → 物件偵測（窗、門、天空） → 對齊至 LOD2 模型
- **權重**: ✅ 程式碼開源，訓練資料（980+ 張阿姆斯特丹立面影像，MS COCO 格式）附於專案中

此為碩士論文專案，透過深度學習從街景影像中偵測建築物開口，並將之合入 LOD2 CityGML 模型以升級為 LOD3[^lod3]。

[^lod3]: Eichenberger, C. (n.d.). 3D_building_reconstruction. Retrieved 2026-09-27, from https://github.com/chrise96/3D_building_reconstruction

## 8. 3dfier（點雲 → CityGML，非 ML，但最普及）

- **程式碼**: [github.com/tudelft3d/3dfier](https://github.com/tudelft3d/3dfier)（635 stars）
- **輸入**: 二維 GIS 資料集 + LAS/LAZ 分類點雲（ASPRS 類別）
- **輸出**: CityGML、CityJSON、OBJ、CSV、PostGIS、STL（LOD1）
- **技術**: 規則導向（非機器學習），將分類點雲的高程值擠出至二維多邊形上
- **支援**: 建築物（LOD1）、地形、道路、水域、森林、橋梁、分離面

3dfier 雖非 ML 模型，卻是目前最受歡迎的點雲轉 CityGML 開源工具，產出無相交三角形且水密的數值表面模型[^3dfier]。

[^3dfier]: Ledoux, H., Biljecki, F., Dukai, B., Kumar, K., Peters, R., Stoter, J., & Commandeur, T. (2021). 3dfier: automatic reconstruction of 3D city models. *Journal of Open Source Software*, 6(60), 2866. Retrieved 2026-09-27, from https://github.com/tudelft3d/3dfier

## 比較總表

| 模型/工具 | 輸入 | 輸出 | ML 方法 | 權重可下載 | CityGML 相容 |
|---|---|---|---|---|---|
| **3D-City-Model-Generator** | 衛星+街景影像 | CityGML LOD1–L3 | Transformers, CLIP, 分割 | 需授權(PLATEAU) | ✅ 原生輸出 |
| **Auto-Create-bldg-lod2** | DSM 點雲 + LOD1 | CityGML LOD2 | HEAT Transformer + 超解析 | ✅ HEAT 權重 | ✅ 原生輸出 |
| **HEAT** | 光柵影像/點雲密度圖 | 平面圖 | Edge Attention Transformer | ✅ Dropbox | ⚠️ 需後處理 |
| **PolyDiffuse** | 視覺資料 | 多邊形形狀 | 擴散模型 | ✅ Dropbox | ⚠️ 需後處理 |
| **RoomFormer** | 3D 室內掃描 | 語義平面圖 | Transformer | ✅ Polybox | ⚠️ 需後處理（室內） |
| **CityDreamer** | 噪聲（生成式） | 神經場 3D 城市 | 組合式神經場 | ✅ 直接連結 | ❌ 無語義結構 |
| **3D_building_reconstruction** | 街景影像+LOD2 | LOD3 CityGML | Faster/Mask R-CNN | ✅ 程式碼+資料 | ✅ 輸出 LOD3 |
| **3dfier** | 分類點雲+GIS | CityGML/CityJSON | 規則導向（非 ML） | N/A | ✅ 原生輸出 |
| **InfiniCity** | 噪聲（生成式） | 體素 3D 城市 | 三階段管線 | ❌ 無公開權重 | ❌ |

## 關鍵發現

1. **唯一完整端到端的 ML 方案**為 Project PLATEAU 的 3D-City-Model-Generator 與 Auto-Create-bldg-lod2-tool，前者從影像直接生成 CityGML LOD1–L3，後者從點雲生成 LOD2，但模型權重需透過 PLATEAU 專案取得授權。

2. **HEAT 與 PolyDiffuse** 提供可直接下載的開源權重，適合建築物輪廓/平面圖重建，但需自行開發 CityGML 後處理管線（將多邊形擠出為 LOD1/LOD2，再標註語義）。

3. **CityDreamer** 是唯一的 HuggingFace 上架開源權重模型，但其為生成式（非重建式），不產出 CityGML 相容的語義結構。

4. **RoomFormer** 對室內 LOD4 最具潛力，因其輸出含房間類型、門窗等語義資訊。

5. **若無 ML 需求**，3dfier 是將分類點雲轉 CityGML 最成熟且泛用的工具。

---

## 參考文獻

Chen, J., Qian, Y., & Furukawa, Y. (2022). HEAT: Holistic Edge Attention Transformer for Structured Reconstruction. Retrieved 2026-09-27, from https://arxiv.org/abs/2111.15143

Chen, J., Qian, Y., Huang, Y.-C., Chiu, L., Cheng, H.-Y., & Furukawa, Y. (2023). PolyDiffuse: Polygonal Shape Reconstruction via Guided Set Diffusion. Retrieved 2026-09-27, from https://arxiv.org/abs/2306.01461

Eichenberger, C. (n.d.). 3D_building_reconstruction. Retrieved 2026-09-27, from https://github.com/chrise96/3D_building_reconstruction

Ledoux, H., Biljecki, F., Dukai, B., Kumar, K., Peters, R., Stoter, J., & Commandeur, T. (2021). 3dfier: automatic reconstruction of 3D city models. *Journal of Open Source Software*, 6(60), 2866. Retrieved 2026-09-27, from https://github.com/tudelft3d/3dfier

Project PLATEAU. (n.d.). 3D-City-Model-Generator. Retrieved 2026-09-27, from https://github.com/Project-PLATEAU/3D-City-Model-Generator

Project PLATEAU. (n.d.). Auto-Create-bldg-lod2-tool. Retrieved 2026-09-27, from https://github.com/Project-PLATEAU/Auto-Create-bldg-lod2-tool

Xie, H., Chen, Z., You, F., Hu, Z., Zhang, H., & Chen, W. (2024). CityDreamer: Compositional Generative Model for Unbounded 3D Cities. Retrieved 2026-09-27, from https://arxiv.org/abs/2309.00610

Yue, Y., Chen, J., Huang, Y.-C., Chiu, L., & Furukawa, Y. (2023). RoomFormer: Two-Level Queries for Floorplan Reconstruction. Retrieved 2026-09-27, from https://arxiv.org/abs/2211.15658