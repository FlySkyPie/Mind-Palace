# 「圖像至點雲」領域中的家具移除技術調查

## 摘要

在 3D 重建（從多張圖像產生點雲）的情境下移除家具，屬於 3D 場景編輯/修補（inpainting）的子領域。本文調查截至 2026 年的現有技術，涵蓋三大策略路線：**重建前 2D 修補**、**重建中 3D 場景優化**，以及**重建後點雲/高斯編輯**。重點介紹近期以 NeRF 和 3D Gaussian Splatting（3DGS）為核心的物件移除方法，並提供實務管線建議。

---

## 1. 路線總覽

將家具從圖像中移除再產生點雲（或更廣義的 3D 場景），可歸為三類策略[^survey-nerf-editing]：

| 策略 | 時機 | 核心概念 |
|---|---|---|
| **A. 重建前（2D 修補 → 3D）** | 在將圖像送入重建管線之前 | 先用 2D 修補模型把家具塗掉，再對修補後的乾淨圖像做 3D 重建 |
| **B. 重建中（3D 感知修補）** | 在訓練 NeRF 或 3DGS 的過程中 | 聯合優化 3D 場景，讓遮罩區域被合理的內容填充 |
| **C. 重建後（3D 編輯）** | 在完整的 3D 場景重建之後 | 直接從點雲或高斯球中移除家具點，再修補空洞 |

[^survey-nerf-editing]: EricLee0224. (2026). Awesome Radiance Field-based 3D Editing [GitHub repository]. Retrieved 2026-09-25, from https://github.com/EricLee0224/awesome-nerf-editing

---

## 2. NeRF 基礎的物件移除與 3D 修補

Neural Radiance Fields（NeRF）是 2020 年以來主流的隱式 3D 表示法，以下方法支援在 NeRF 場景中移除物件：

| 方法 | 會議/期刊 | 年份 | 核心想法 |
|---|---|---|---|
| **NeRFiller**[^nerfiller] | CVPR | 2024 | 用生成式 3D 修補完成 NeRF 場景中的缺失區域，聯合優化多個視角 |
| **InNeRF360**[^inpaintnerf360] | CVPR | 2024 | 無邊界 360° NeRF 場景的文字引導物件修補 |
| **SIGNeRF**[^signerf] | CVPR | 2024 | 場景整合生成，可編輯／替換 NeRF 場景中的內容 |
| **MVIP-NeRF**[^mvipnerf] | CVPR | 2024 | 基於擴散先驗的多視角一致 3D NeRF 修補 |
| **MALD-NeRF**[^maldnerf] | ECCV | 2024 | 使用潛在擴散模型進行 NeRF 修補 |
| **NeRF-In**[^nerfin] | CG&A | 2024 | RGB-D 先驗引導的自由形式 NeRF 修補 |
| **SIn-NeRF2NeRF**[^sinnerf2nerf] | arXiv | 2024 | 分割+修補管線，用於 NeRF 場景編輯 |

[^nerfiller]: Weber, E., et al. (2024). NeRFiller: Completing Scenes via Generative 3D Inpainting. *CVPR 2024*. Retrieved 2026-09-25, from https://arxiv.org/abs/2312.04560
[^inpaintnerf360]: Wang, D., et al. (2024). InNeRF360: Text-Guided 3D-Consistent Object Inpainting on Unbounded NeRF. *CVPR 2024*. Retrieved 2026-09-25, from https://arxiv.org/abs/2305.15094
[^signerf]: Dihlmann, J., et al. (2024). SIGNeRF: Scene Integrated Generation for NeRF. *CVPR 2024*. Retrieved 2026-09-25, from https://arxiv.org/abs/2401.01647
[^mvipnerf]: Chen, Y., et al. (2024). MVIP-NeRF: Multi-view 3D Inpainting on NeRF via Diffusion Prior. *CVPR 2024*. Retrieved 2026-09-25, from https://arxiv.org/abs/2405.02859
[^maldnerf]: Huang, J., et al. (2024). MALD-NeRF: Taming Latent Diffusion for NeRF Inpainting. *ECCV 2024*. Retrieved 2026-09-25, from https://arxiv.org/abs/2404.09995
[^nerfin]: Liu, H., et al. (2022). NeRF-In: Free-Form NeRF Inpainting with RGB-D Priors. *Computer Graphics and Applications*. Retrieved 2026-09-25, from https://arxiv.org/abs/2206.04901
[^sinnerf2nerf]: Changmin, K., et al. (2024). SIn-NeRF2NeRF: Segmentation + Inpainting for NeRF Editing. Retrieved 2026-09-25, from https://arxiv.org/abs/2408.13285

這些方法的共同模式：先用遮罩標記要移除的區域，再透過擴散模型（diffusion model）引導 NeRF 填充該區域，並在多視角間保持一致性。

---

## 3. 3D Gaussian Splatting（3DGS）基礎的物件移除

3DGS 是 2023 年提出的顯式 3D 表示法，因其即時渲染能力和顯式的點結構，物件移除更直接——可以物理上刪除對應的高斯球，再修補空洞。2024-2026 年間該領域發展極為迅速：

| 方法 | 會議/期刊 | 年份 | 核心想法 |
|---|---|---|---|
| **GScream**[^gscream] | ECCV | 2024 | 專用 3DGS 物件移除管線：遮罩高斯球後用特徵一致學習修補空洞 |
| **Gaussian Grouping**[^gaussian-grouping] | ECCV | 2024 | 透過特徵分組分割/編輯 3DGS 場景中的任意物件 |
| **GaussianEditor**[^gaussian-editor] | CVPR | 2024 | 文字引導的 3DGS 編輯，含物件移除 |
| **GPGS**[^gpgs] | AAAI | 2026 | 幾何感知的 3DGS 物件移除，含投影圖像精煉 |
| **Inpaint360GS**[^inpaint360gs] | WACV | 2026 | 物件感知的 360° 3DGS 高效修補 |
| **Semantic-Guided 3DGS**[^semantic-gs] | arXiv | 2026 | 語義分割引導的暫態物件移除 |
| **InstaInpaint**[^instainpaint] | NeurIPS | 2025 | 大型重建模型支持的即時 3D 場景修補 |
| **DiGA3D**[^diga3d] | ICCV | 2025 | 粗到細的擴散式 3D 修補 |
| **Perspective-aware PA-Inpainter**[^painpainter] | ICCV | 2025 | 透視感知、多視角一致的 3DGS 修補 |

[^gscream]: Wu, T., et al. (2024). GScream: Learning 3D Geometry & Feature Consistent Gaussian Splatting for Object Removal. *ECCV 2024*. Retrieved 2026-09-25, from https://arxiv.org/abs/2404.13679
[^gaussian-grouping]: Ye, M., et al. (2024). Gaussian Grouping: Segment and Edit Anything in 3D Scenes. *ECCV 2024*. Retrieved 2026-09-25, from https://arxiv.org/abs/2312.00732
[^gaussian-editor]: Chen, Y., et al. (2024). GaussianEditor: Swift and Controllable 3D Editing with Gaussian Splatting. *CVPR 2024*. Retrieved 2026-09-25, from https://arxiv.org/abs/2311.14521
[^gpgs]: Yongjoon, J., et al. (2026). GPGS: Consistent 3D Object Removal via Geometry-Aware 3D Inpainting. *AAAI 2026*. Retrieved 2026-09-25, from https://ojs.aaai.org/index.php/AAAI/article/view/37515
[^inpaint360gs]: DFKI-AV. (2026). Inpaint360GS: Efficient Object-Aware 3D Inpainting via Gaussian Splatting for 360° Scenes. *WACV 2026*. Retrieved 2026-09-25, from https://arxiv.org/abs/2511.06457
[^semantic-gs]: (2026). Semantic-Guided 3D Gaussian Splatting for Transient Object Removal. Retrieved 2026-09-25, from https://arxiv.org/abs/2602.15516
[^instainpaint]: Dong, H., et al. (2025). InstaInpaint: Instant 3D-Scene Inpainting with Masked Large Reconstruction Model. *NeurIPS 2025*. Retrieved 2026-09-25, from https://arxiv.org/abs/2506.10980
[^diga3d]: (2025). DiGA3D: Diffusional Propagation for Versatile 3D Inpainting. *ICCV 2025*. Retrieved 2026-09-25, from https://arxiv.org/abs/2507.00429
[^painpainter]: (2025). Perspective-aware 3D Gaussian Inpainting with Multi-view Consistency. *ICCV 2025*. Retrieved 2026-09-25, from https://arxiv.org/abs/2510.10993

3DGS 方法的優勢在於：高斯球本身就是離散的 3D 點，刪除後可直接在 3D 空間修補，物理上更直觀。

---

## 4. 多視角 2D 修補管線（作為重建前處理）

如果選擇「先在 2D 移除家具，再重建 3D」的路線，以下方法值得關注：

| 方法 | 會議/期刊 | 年份 | 核心想法 |
|---|---|---|---|
| **MVInpainter**[^mvinpainter] | NeurIPS | 2024 | 學習多視角一致的 2D 修補，橋接 2D 與 3D 編輯 |
| **Instant3dit**[^instant3dit] | CVPR | 2025 | 快速多視角修補，用於 3D 物件編輯 |
| **Geometry-Aware MV Inpainting**[^geomvi] | BMVC | 2025 | 幾何感知的多視角場景修補擴散模型 |
| **ObjFiller-3D**[^objfiller3d] | arXiv | 2025 | 用影片擴散模型實現一致的多視角 3D 修補 |

[^mvinpainter]: Ew, R., et al. (2024). MVInpainter: Learning Multi-View Consistent Inpainting to Bridge 2D and 3D. *NeurIPS 2024*. Retrieved 2026-09-25, from https://arxiv.org/abs/2408.08000
[^instant3dit]: Barda, A., et al. (2025). Instant3dit: Multiview Inpainting for Fast Editing of 3D Objects. *CVPR 2025*. Retrieved 2026-09-25, from https://arxiv.org/abs/2412.00518
[^geomvi]: (2025). Geometry-Aware Diffusion Models for Multiview Scene Inpainting. *BMVC 2025*. Retrieved 2026-09-25, from https://arxiv.org/abs/2502.13335
[^objfiller3d]: (2025). ObjFiller-3D: Consistent Multi-view 3D Inpainting via Video Diffusion. Retrieved 2026-09-25, from https://arxiv.org/abs/2508.18271

---

## 5. 分割輔助的家具定位

在移除家具之前，必須先定位家具。以下方法可直接用於 3D 場景中的家具分割：

| 方法 | 會議 | 年份 | 說明 |
|---|---|---|---|
| **SAGA**[^saga] | AAAI | 2025 | 將 Segment Anything 適應到 3DGS——透過提示選取要移除的家具 |
| **Click-Gaussian**[^clickgaussian] | ECCV | 2024 | 點擊式互動分割 3DGS 中的物件 |
| **SANeRF-HQ**[^sanerfhq] | CVPR | 2024 | 高品質 NeRF 場景分割，用於目標編輯 |
| **SAM + SAM-3D** | — | 2023-24 | 先用 SAM 產生 2D 遮罩，再投影或聚類到 3D 空間 |

[^saga]: Jumpat, et al. (2025). SAGA: Segment Any 3D Gaussians. *AAAI 2025*. Retrieved 2026-09-25, from https://arxiv.org/abs/2312.00860
[^clickgaussian]: Choi, S., et al. (2024). Click-Gaussian: Interactive Segmentation to Any 3D Gaussians. *ECCV 2024*. Retrieved 2026-09-25, from https://arxiv.org/abs/2407.11793
[^sanerfhq]: Lyc, L., et al. (2024). SANeRF-HQ: Segment Anything for NeRF in High Quality. *CVPR 2024*. Retrieved 2026-09-25, from https://arxiv.org/abs/2312.01531

---

## 6. 實務管線建議

根據以上調查，針對「從多張室內圖像移除家具並產生點雲」的情境，建議以下三條可實作的管線：

### 管線 A：重建前 2D 修補（最簡單、最相容）

```
輸入圖像 → SAM 分割家具 → LaMa / SD Inpainting / FLUX Inpainting → COLMAP → 點雲
```

- **優勢**：與任何重建管線相容（COLMAP、NeRF、3DGS 都可用修補後的圖像）
- **劣勢**：多視角一致性需額外處理（MVInpainter 可解決此問題）
- **關鍵工具**：SAM[^sam]、LaMa[^lama]、Stable Diffusion Inpainting[^sd-inpaint]

### 管線 B：3DGS 後處理（適合需要即時反饋的場景）

```
輸入圖像 → COLMAP + 3DGS 重建含家具的場景 → SAGA 分割家具 → GScream / GPGS 移除並修補 → 匯出點雲
```

- **優勢**：移除效果直接在 3D 空間驗證，可迭代編輯
- **劣勢**：需要較強的 GPU
- **關鍵工具**：3DGS、SAGA、GScream[^gscream-code]、GPGS[^gpgs-code]

### 管線 C：NeRF 聯合優化（適合高品質需求）

```
輸入圖像 + 家具遮罩 → NeRF 訓練 + NeRFiller/InNeRF360 聯合修補 → 從 NeRF 萃取點雲
```

- **優勢**：多視角一致性最好，適合高品質輸出
- **劣勢**：訓練時間長（數小時），不適合快速迭代

[^sam]: Kirillov, A., et al. (2023). Segment Anything. *ICCV 2023*. Retrieved 2026-09-25, from https://github.com/facebookresearch/segment-anything
[^lama]: Suvorov, R., et al. (2022). Resolution-robust Large Mask Inpainting with Fourier Convolutions. *WACV 2022*. Retrieved 2026-09-25, from https://github.com/saic-mdal/lama
[^sd-inpaint]: RunwayML. (2023). Stable Diffusion Inpainting. Retrieved 2026-09-25, from https://huggingface.co/runwayml/stable-diffusion-inpainting
[^gscream-code]: Wu, T. (2024). GScream [GitHub repository]. Retrieved 2026-09-25, from https://github.com/w-ted/gscream
[^gpgs-code]: Yongjoon, J. (2026). GPGS [GitHub repository]. Retrieved 2026-09-25, from https://github.com/yongjoon99/GPGS

---

## 7. 關鍵開放問題

1. **大型家具與背景混淆**：沙發、床等大型家具遮擋大量背景，修補品質取決於生成模型對室內場景的理解。
2. **多視角一致性**：單純逐幀 2D 修補容易產生視角間不一致的閃爍，需 MVInpainter 或 3D 感知方法處理。
3. **幾何正確性**：修補區域的幾何結構（如牆面垂直、地板水平）需額外的深度/法向量約束。
4. **保留原始點雲精度**：修補區域的點雲密度可能低於原始掃描區域，需注意密度一致性。

---

## 8. 相關綜述資源

| 資源 | 類型 | 說明 |
|---|---|---|
| Awesome Nerf Editing[^survey-nerf-editing] | GitHub 列表 | 500+ 篇 NeRF/3DGS 編輯論文的持續更新清單 |
| 3DGS Applications Survey[^survey-gs-apps] | TPAMI 2026 | 涵蓋分割、編輯、生成的 3DGS 應用綜述 |
| Editing Radiance Fields Survey[^survey-editing] | MVA 2025 | 隱式與顯式輻射場編輯的全面回顧 |

[^survey-gs-apps]: (2026). A Survey on 3D Gaussian Splatting Applications: Segmentation, Editing, Generation. *IEEE TPAMI*. Retrieved 2026-09-25, from https://arxiv.org/abs/2508.09977
[^survey-editing]: (2025). Editing Implicit and Explicit Representations of Radiance Fields: A Survey. *Machine Vision and Applications*. Retrieved 2026-09-25, from https://arxiv.org/abs/2412.17628