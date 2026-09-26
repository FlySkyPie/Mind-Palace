# C++ 影像處理函式庫調查：縮放、裁切等基礎操作

## 概述

本文調查適用於 C++ 專案的影像處理函式庫，重點關注縮放 (scaling/resizing)、裁切 (cropping)、旋轉 (rotation)、格式轉換等基礎影像操作。根據專案需求規模與整合難易度，將函式庫分為輕量級與重量級兩大類。

## 輕量級函式庫（低依賴、易整合）

### 1. stb（Sean Barrett 單檔函式庫）

- **GitHub**: [nothings/stb](https://github.com/nothings/stb)[^stb]
- **授權**: Public domain / MIT
- **GitHub Stars**: ~34.7k
- **組成**:
  - `stb_image.h` (~8,000 LoC) — 讀取 JPG、PNG、TGA、BMP、PSD、GIF、HDR、PIC
  - `stb_image_write.h` (~1,700 LoC) — 寫入 PNG、TGA、BMP
  - `stb_image_resize2.h` (~10,700 LoC) — 高品質縮放
- **整合方式**: 直接複製 `.h` 檔到專案，在一個 translation unit 中 `#define STB_IMAGE_IMPLEMENTATION`
- **相依性**: 零
- **優點**: 整合極簡單、無需建置系統、佔用空間極小、廣泛用於遊戲引擎與 texture loading
- **缺點**: C API（非 idiomatic C++）、無內建裁切/旋轉（需手動操作 pixel）、無新格式支援計畫
- **適用場景**: 小專案、遊戲開發、快速原型、嵌入式系統、texture 載入

### 2. CImg（Cool Image Library）

- **官方網站**: [cimg.eu](https://cimg.eu/)[^cimg]
- **GitHub**: [GreycLab/CImg](https://github.com/GreycLab/CImg) (~1.7k stars)
- **授權**: CeCILL-C（LGPL-like）或 CeCILL（GPL 相容）
- **組成**: 單一 header 檔 `CImg.h`，約 60K LoC
- **整合方式**: 單一 header、無需外部函式庫
- **相依性**: 核心零相依；可選用 libjpeg、libpng、libtiff、FFMPEG、OpenCV 等擴充格式支援
- **功能**: 載入/儲存、顯示、縮放、裁切、旋轉、濾波、形態學運算、繪圖、統計、3D 物件、使用者互動、支援 4D 影像
- **優點**: 真正自包含、高度可攜、thread-safe、plugin 機制完善
- **缺點**: 單一 header 龐大 (~1.5 MB)、編譯慢、API 綁在單一類別上、授權較嚴格
- **適用場景**: 教學、快速原型、小型影像處理工具

### 3. LodePNG

- **GitHub**: [lvandeve/lodepng](https://github.com/lvandeve/lodepng)[^lodepng]
- **授權**: zlib
- **GitHub Stars**: ~2.4k
- **組成**: `lodepng.cpp` + `lodepng.h`，無相依性
- **功能**: PNG 編碼/解碼，支援 ANSI C (C89) 與 C++
- **適用場景**: 僅需處理 PNG 格式的專案

### 4. Boost.GIL（Generic Image Library）

- **GitHub**: [boostorg/gil](https://github.com/boostorg/gil)[^gil]
- **授權**: Boost Software License 1.0
- **GitHub Stars**: ~199（屬 Boost 專案）
- **組成**: Header-only C++14、需 Boost headers
- **相依性**: 需 Boost；可選用 libjpeg/libpng/libtiff/libraw 做 I/O
- **功能**: Pixel 類型抽象、image view、色彩轉換 (RGB↔CMYK)、直方圖均衡化、卷積；可撰寫泛型演算法
- **優點**: 型別安全、header-only、效能佳（template 最佳化）
- **缺點**: 學習曲線陡峭（template metaprogramming 重）、I/O 支援有限、文件老舊、需 Boost
- **適用場景**: 已使用 Boost 的專案、需要撰寫泛型影像演算法的學術/研究程式碼

## 重量級函式庫（功能完整但依賴較重）

### 5. libvips

- **GitHub**: [libvips/libvips](https://github.com/libvips/libvips)[^vips]
- **授權**: LGPL-2.1+
- **GitHub Stars**: ~11.7k
- **相依性**: 中等。需 glib-2.0；大量可選格式後端 (libjpeg, libpng, libtiff, libwebp, libheif 等)
- **格式支援**: 極佳 — JPEG, JPEG 2000, JPEG XL, TIFF, PNG, WebP, HEIC, AVIF, PDF, SVG 等
- **功能**: ~300 種操作，包含縮放、裁切、旋轉、仿射變換、卷積、形態學、色彩管理、直方圖等
- **優點**: 極快且記憶體效率極高（tile-based pipeline）、不需將整張圖載入記憶體、適合巨型影像
- **缺點**: C API 基於 glib、pipeline 模型有學習曲線、啟用所有格式時相依樹較重
- **適用場景**: 伺服端影像處理、Web service、批次處理大型影像

### 6. OpenCV

- **官方網站**: [opencv.org](https://opencv.org/)[^opencv]
- **授權**: Apache 2.0
- **GitHub Stars**: ~91k
- **二進位大小**: ~200 MB
- **相依性**: 重 — 需 CMake、可選 Eigen、TBB、IPU、CUDA 等
- **格式支援**: 良好 — JPEG、PNG、TIFF、WebP、BMP、HDR、OpenEXR 等
- **功能**: 完整電腦視覺套件 — 物件偵測、面部辨識、機器學習、DNN、相機校正、GPU 加速
- **優點**: 生態系最大、GPU/CUDA 加速、文件成熟、社群龐大
- **缺點**: 若僅基礎操作則過重、編譯慢、二進位大、學習曲線陡
- **適用場景**: 需要電腦視覺 + 影像處理的大型專案（機器人、ML 管線、研究）

### 7. ImageMagick / Magick++

- **GitHub**: [ImageMagick/ImageMagick](https://github.com/ImageMagick/ImageMagick)[^magick]
- **授權**: ImageMagick License（寬鬆）
- **GitHub Stars**: ~17.5k
- **格式支援**: **200+ 種格式** — 所有函式庫中最廣
- **功能**: 縮放、裁切、旋轉、翻轉、格式轉換、模糊、銳化、形態學、色彩量化、liquid rescaling (seam carving)、繪圖、合成、動畫
- **優點**: 格式覆蓋無可匹敵、CLI 工具完善 (`magick convert`)、OpenCL GPU 加速
- **缺點**: **重大安全歷史** (ImageTragick 漏洞)、依賴極重、對大型影像處理速度不如 libvips
- **適用場景**: 需要讀寫冷門格式、CLI 批次處理

### 8. Leptonica

- **GitHub**: [DanBlooBerg/leptonica](https://github.com/DanBlooBerg/leptonica)[^leptonica]
- **授權**: BSD-2-Clause
- **GitHub Stars**: ~2.1k
- **語言**: ANSI C
- **相依性**: 輕 — 需 libjpeg、libpng、libtiff、libwebp、libgif
- **功能**: Rasterops、仿射變換、二值/灰階形態學、rank filter、卷積、connected components、色彩量化、skew 偵測、二值化、頁面分割、dewarping、條碼偵測
- **優點**: **文件影像分析** 表現出色、Tesseract OCR 的底層函式庫、140+ 回歸測試
- **適用場景**: OCR 前置處理管線（搭配 Tesseract）、文件掃描/分析

## 比較總表

| 函式庫 | Header-Only | 相依性 | 格式支援 | 效能 | 最佳用途 |
|---|---|---|---|---|---|
| **stb** | ✅ 是 | 零 | 中等（無 TIFF）| 尚可 | 快速整合載入/縮放 |
| **CImg** | ✅ 是 | 零（核心）| 中等（需外部 I/O）| 尚可 | 快速原型/教學 |
| **Boost.GIL** | ✅ 是 | 需 Boost | 有限（需外部 I/O）| 佳 | 泛型演算法 |
| **libvips** | 否 | 中等 | 極佳 | **最快** | 伺服端 Web 處理 |
| **OpenCV** | 否 | 重 | 佳 | 快（GPU） | 電腦視覺 |
| **ImageMagick** | 否 | 極重 | **200+ 格式** | 中等 | 冷門格式瑞士刀 |
| **Leptonica** | 否 | 輕 | 佳 | 佳 | 文件 OCR 處理 |

## 使用情境建議

- **只需載入、縮放、儲存常見格式（JPG/PNG/BMP）且零設定**：使用 **stb** (`stb_image.h` + `stb_image_resize2.h` + `stb_image_write.h`)
- **建立影像處理 Web service**：使用 **libvips**（快速、低記憶體、pipeline 架構）
- **完整電腦視覺管線（偵測、追蹤、ML）**：使用 **OpenCV**
- **文件掃描 / OCR 前置處理**：使用 **Leptonica**（常搭配 Tesseract OCR）
- **需要轉換冷門格式**：使用 **ImageMagick**
- **已在用 Booost 的專案中撰寫泛型演算法**：使用 **Booost.GIL**
- **單一 header 即可的小型遊戲引擎**：使用 **CImg** 或 **stb**

## 參考資料

[^stb]: Barrett, S. (n.d.). *stb — single-file public domain libraries for C/C++. GitHub. Retrieved 2026-09-25, from https://github.com/nothings/stb

[^cimg]: Tschumperlé, D. (n.d.). *CImg — The C++ Template Image Processing Library. Retrieved 2026-09-25, from https://cimg.eu/

[^lodepng]: van de Ven, L. (n.d.). *LodePNG. GitHub. Retrieved 2026-09-25, from https://github.com/lvandeve/lodepng

[^gil]: Beman Dawes, et al. (n.d.). *Boost.GIL — Generic Image Library. GitHub. Retrieved 2026-09-25, from https://github.com/boostorg/gil

[^vips]: Martinez, K., Cupitt, J., et al. (n.d.). *libvips — a fast image processing library. GitHub. Retrieved 2026-09-25, from https://github.com/libvips/libvips

[^opencv]: OpenCV team. (n.d.). *OpenCV — Open Source Computer Vision Library. Retrieved 2026-09-25, from https://opencv.org/

[^magick]: ImageMagick Studio LLC. (n.d.). *ImageMagick — Convert, Edit, or Compose Digital Images. GitHub. Retrieved 2026-09-25, from https://github.com/ImageMagick/ImageMagick

[^leptonica]: Bloomberg, D. (n.d.). *Leptonica — Image Processing Library. GitHub. Retrieved 2026-09-25, from https://github.com/DanBloomberg/leptonica