# 室內場景「影像轉點雲」開放原始碼解決方案調查

## 概述

本報告調查適用於室內場景（如房間、走廊、建築內部）的 FOSS（Free and Open Source Software）影像轉點雲解決方案。涵蓋傳統攝影測量（Photogrammetry）、神經輻射場（NeRF）與 3D Gaussian Splatting 三條主要技術路線，並比較各工具在室內場景的適用性。

---

## 1. 傳統攝影測量（SfM + MVS）管線

傳統管線分為兩階段：**Structure from Motion (SfM)** 從影像中推估相機姿態並產出稀疏點雲，**Multi-View Stereo (MVS)** 將稀疏點雲稠密化。此路線幾何精度最佳，適合室內計量級應用。

### 1.1 COLMAP

COLMAP 是目前最廣泛使用的 SfM + MVS 管線，以 New BSD 授權釋出。[^colmap] 接受無序影像集合，依序執行特徵匹配、SfM（產出稀疏點雲）、MVS（深度圖融合產出稠密點雲）。內建稠密重建管線可產出含法向量資訊的點雲，後續可進行 Poisson / Delaunay 網格化。提供 GUI、CLI、Python 綁定（pycolmap），附有室內場景樣本資料集（如 Gerrard Hall）。

室內場景優勢：支援大量影像、文件完善、精度經學術界多年驗證。

### 1.2 Meshroom（AliceVision）

Meshroom 以 MPL-2.0 授權釋出，是以節點圖（node graph）為視覺介面的攝影測量軟體。[^meshroom] 底層為 AliceVision 框架，涵蓋特徵匹配、SfM、MVS 稠密重建、網格化、紋理貼圖完整管線。近期加入 **MrGSplat 外掛** 支援 3D Gaussian Splatting、**DepthEstimation 外掛** 支援 AI 深度估計。

室內場景優勢：圖形化介面降低入門門檻，節點圖可自訂室內管線流程。

### 1.3 OpenMVG + OpenMVS

OpenMVG（「open Multiple View Geometry」）以 MPL-2.0 授權釋出，是專注於 SfM 的 C++ 函式庫。[^openmvg] 擅長全域式 SfM（對大型影像集合穩健）。OpenMVS 以 AGPL-3.0 授權釋出，承接 SfM 的相機姿態與稀疏點雲，使用 PatchMatch 與 Semi-Global Matching (SGM) 演算法產出稠密點雲、網格與紋理。[^openmvs] 兩者常搭配使用。

室內場景優勢：OpenMVG 的全域式 SfM 對室內影像組（大量重疊、相似紋理）有較強韌性。

### 1.4 OpenSfM

OpenSfM 以 BSD-2-Clause 授權釋出，原由 Mapillary 開發，現由社群維護。[^opensfm] 以 Python 為主搭配 C++ 效能關鍵元件。從 SfM 一路做到稠密點雲、網格、正射影像（DSM/orthophoto），支援 GPU 加速與控制點地理定位。內建品質報告。

室內場景優勢：支援子模型拆分（submodel splitting），對大型室內場景（如整層樓）可分散處理。

### 1.5 MicMac

MicMac 以 CECILL-B 授權釋出，由法國國家地理與森林資訊研究所（IGN）自 2007 年起開發。[^micmac] 功能完整的攝影測量套件，學習曲線陡峭但計量精度極高。可透過 MeshroomMicMac 外掛與 Meshroom 整合。

室內場景優勢：高計量精度，適合需要絕對座標的室內測繪應用。

---

## 2. NeRF（神經輻射場）路線

NeRF 以類神經網路隱式建模場景，視覺品質極高，幾何精度次於傳統管線。

### 2.1 Nerfstudio

Nerfstudio 以 Apache-2.0 授權釋出，是模組化的 NeRF 框架。[^nerfstudio] 輸入影像後，內部調用 COLMAP 求取相機姿態，再訓練 NeRF 模型。可透過 `ns-export pointcloud` 匯出點雲。支援多種模型如 `nerfacto`（真實場景預設）、`splatfacto`（3D Gaussian Splatting）。附有網頁版互動檢視器。

室內場景優勢：`nerfacto` 對室內真實場景效果佳，匯出點雲流程簡單。

---

## 3. 3D Gaussian Splatting 路線

2023 年 SIGGRAPH 發表的新技術，以 3D 高斯原語表達場景，兼具高視覺品質與即時渲染效能。

### 3.1 原始 3D Gaussian Splatting（Inria）

以自訂研究授權釋出。[^3dgs] 輸入影像與 COLMAP 稀疏點雲，訓練後產出 `.ply` 格式的 3D 高斯點雲（含位置、顏色、不透明度、旋轉、縮放等屬性）。提供即時等級的新視角合成。

### 3.2 gsplat

gsplat 以 Apache-2.0 授權釋出，是高度最佳化的 CUDA Gaussian Splatting 渲染函式庫。[^gsplat] 較原始實作節省 4 倍 GPU 記憶體、快 15%。支援多種相機模型（針孔、魚眼、全景、LiDAR），被 Nerfstudio 內部採用。

### 3.3 OpenSplat

OpenSplat 以 AGPL-3.0 授權釋出，是 WebODM 生態系的 C++ Gaussian Splatting 實作。[^opensplat] 支援 CPU、NVIDIA CUDA、AMD ROCm、Apple Metal 四種後端，無 GPU 仍可在 CPU 上執行（約慢 100 倍）。接受 COLMAP、OpenSfM、OpenMVG、Nerfstudio 等格式輸入，輸出 `.ply`、`.splat`、`.spz`、`.rad`。

### 3.4 3DGS Indoor Reconstruction Pipeline

以 MIT 授權釋出的室內專門管線。[^3dgs_indoor] 流程：ROS bag → 影像提取 → ORB-SLAM3 相機追蹤 → COLMAP 格式 → OpenSplat 訓練 → 3DGS PLY 點雲。專為室內環境（房間、走廊）設計。

---

## 4. 輔助工具：Open3D

Open3D 以 MIT 授權釋出，雖然不是重建管線本身，但為點雲處理不可或缺的工具。[^open3d] 支援場景重建、點雲配準、Poisson 曲面重建、視覺化等，可用於清理與分析產出的室內點雲。

---

## 5. 室內場景推薦工作流程

### 傳統攝影測量（最佳幾何精度）

```
影像 → COLMAP (SfM → 稀疏點雲) → OpenMVS (稠密點雲) → Open3D (後處理)
```

圖形化替代：`影像 → Meshroom`

### NeRF（最佳視覺品質）

```
影像 → Nerfstudio (ns-process-data) → 訓練 nerfacto → ns-export pointcloud
```

### Gaussian Splatting（即時渲染、高視覺品質）

```
影像 → COLMAP (稀疏 SfM) → OpenSplat 或原始 3DGS → .ply 點雲
```

---

## 6. 比較總表

| 需求 | 推薦工具 | 授權 |
|---|---|---|
| 最易入門 | Meshroom（GUI）、Nerfstudio（CLI） | MPL-2.0 / Apache-2.0 |
| 最佳幾何精度 | COLMAP + OpenMVS | BSD / AGPL-3.0 |
| 最佳視覺品質 | Nerfstudio、OpenSplat / 3DGS | Apache-2.0 / AGPL-3.0 |
| 僅有 CPU | COLMAP、OpenMVG、OpenSfM、MicMac | 多種 |
| 室內專門 | 3DGS Indoor Pipeline、Nerfstudio | MIT / Apache-2.0 |
| 大型室內場景 | OpenSfM（子模型拆分）、COLMAP | BSD-2-Clause / BSD |
| 輸出格式 | 全部支援 `.ply` 點雲 | — |

---

## 參考資料

[^colmap]: COLMAP. (n.d.). Structure-from-Motion and Multi-View Stereo. Retrieved 2026-09-25, from https://github.com/colmap/colmap

[^meshroom]: AliceVision. (n.d.). Meshroom — 3D Reconstruction Software. Retrieved 2026-09-25, from https://github.com/alicevision/Meshroom

[^openmvg]: openMVG. (n.d.). open Multiple View Geometry library. Retrieved 2026-09-25, from https://github.com/openMVG/openMVG

[^openmvs]: Ceccarelli, S. (n.d.). OpenMVS — Multi-View Stereo Reconstruction Library. Retrieved 2026-09-25, from https://github.com/cdcseacave/openMVS

[^opensfm]: OpenSfM Community. (n.d.). OpenSfM — Structure from Motion. Retrieved 2026-09-25, from https://github.com/OpenSfM/OpenSfM

[^micmac]: IGN. (n.d.). MicMac — Photogrammetry Suite. Retrieved 2026-09-25, from https://github.com/micmacIGN/micmac

[^nerfstudio]: Nerfstudio Project. (n.d.). Nerfstudio — Modular NeRF Framework. Retrieved 2026-09-25, from https://github.com/nerfstudio-project/nerfstudio

[^3dgs]: Kerbl, B., Kopanas, G., Leimkühler, T., & Drettakis, G. (2023). 3D Gaussian Splatting for Real-Time Radiance Field Rendering. ACM Transactions on Graphics (SIGGRAPH), 42(4). Retrieved 2026-09-25, from https://github.com/graphdeco-inria/gaussian-splatting

[^gsplat]: Nerfstudio Project. (n.d.). gsplat — CUDA-Accelerated Gaussian Splatting. Retrieved 2026-09-25, from https://github.com/nerfstudio-project/gsplat

[^opensplat]: WebODM. (n.d.). OpenSplat — Free and Open Source 3D Gaussian Splatting. Retrieved 2026-09-25, from https://github.com/WebODM/OpenSplat

[^3dgs_indoor]: Jin, Z. (n.d.). 3DGS Indoor Reconstruction Pipeline. Retrieved 2026-09-25, from https://github.com/JinZhengzhen/3dgs-indoor-reconstruction

[^open3d]: Intel Labs. (n.d.). Open3D — A Modern Library for 3D Data Processing. Retrieved 2026-09-25, from https://github.com/isl-org/Open3D