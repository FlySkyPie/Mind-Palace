# COLMAP 逐張影像遮罩（Per-Image Mask）功能研究

## 概述

COLMAP 支援**逐張影像提供獨立遮罩**，讓使用者可以針對每張照片排除特定區域（如路人、家具、招牌等），使這些區域不會參與特徵提取與後續的三維重建。

## 兩種遮罩機制

### 1. 逐張影像遮罩 `--ImageReader.mask_path`

這是最常用的遮罩方式，可為每張照片指定獨立的遮罩圖檔。

**運作方式：**
- 提供一個遮罩資料夾路徑，內含與輸入影像對應的 PNG 遮罩。
- 遮罩檔案的相對路徑（相對於遮罩根目錄）必須與影像檔案的相對路徑（相對於影像根目錄）一致。
- 遮罩檔案名稱 = 原始影像檔名 + `.png` 後綴附加於檔名之後。

**命名範例：**

```
影像： /path/to/images/abc/012.jpg
遮罩： /path/to/masks/abc/012.jpg.png
```

**遮罩格式要求：**
- 灰階 PNG 圖檔（單通道或三通道均可，COLMAP 會自行處理）。
- **像素值 0（黑色）** = 遮罩區域，不提取特徵。
- **非零像素值** = 有效區域，正常提取特徵。

**命令列使用方式：**

```bash
colmap feature_extractor \
    --database_path $PROJECT/database.db \
    --image_path $PROJECT/images \
    --ImageReader.mask_path $PROJECT/masks
```

**注意**：正確的 CLI 參數是 `--ImageReader.mask_path`，而非 `--mask_path`，後者會導致 `unrecognised option` 錯誤。[^issue1545]

### 2. 全域相機遮罩 `--ImageReader.camera_mask_path`

若所有照片都需要套用**相同的遮罩**（例如統一遮除浮水印或畫面邊緣的固定元素），可使用此選項：

```bash
colmap feature_extractor \
    --database_path $PROJECT/database.db \
    --image_path $PROJECT/images \
    --ImageReader.camera_mask_path /path/to/global_mask.png
```

此遮罩會套用到所有影像上。[^camera_mask]

## 遮罩的內部運作機制

COLMAP 的特徵遮罩處理流程如下：

1. **先對整張影像提取特徵**（包括遮罩區域）。
2. **過濾位於遮罩區（黑色像素）的特徵點**，將其及其描述子一併移除。
3. **僅保留有效區域的特徵點**，存入資料庫進行後續匹配。[^deepwiki]

這也解釋了為何在 GUI 的「資料庫管理」中仍可能看到遮罩區域內顯示出特徵點 — 因為遮罩是提取「後」才過濾，而視覺化顯示的是過濾前的結果。[^issue1460]

## 稠密重建階段的遮罩

遮罩也可以在稠密立體融合（Stereo Fusion）階段使用，參數為 `--StereoFusion.mask_path`，格式與 `ImageReader.mask_path` 相同。[^faq]

## 實用注意事項與已知問題

| 問題 | 說明 |
|---|---|
| **遮罩未生效** | 確保遮罩黑色區域的像素值**精確為 0**，極暗但非零的像素（如 1 或 2）會被視為有效區域。[^issue1460] |
| **錯誤的參數名稱** | 使用 `--ImageReader.mask_path`，而非 `--mask_path`。[^issue1545] |
| **檔案命名規則** | 遮罩檔名必須為 `原始檔名.副檔名.png`（如 `photo.jpg.png`）。 |
| **camera_mask_path 問題** | 部分版本回報 `camera_mask_path` 無法正常運作。[^issue1830] |
| **不可用於 automatic_reconstructor** | 遮罩功能僅在 `feature_extractor` 階段可用，無法透過 `automatic_reconstructor` GUI 管線使用。[^issue208] |

## Python 整合範例

```python
import subprocess, os

basedir = "/path/to/project"
mask_dir = os.path.join(basedir, "masks")

feature_extractor_args = [
    'colmap', 'feature_extractor',
    '--database_path', os.path.join(basedir, 'database.db'),
    '--image_path', os.path.join(basedir, 'images'),
    '--ImageReader.mask_path', mask_dir,
    '--ImageReader.single_camera', '1',
]
subprocess.check_output(feature_extractor_args, universal_newlines=True)
```

## 結論

COLMAP **完全支援逐張影像遮罩**。建立一組灰階 PNG 遮罩（每張照片一張），黑色（像素值 0）為排除區域，非零為保留區域，並在執行 `colmap feature_extractor` 時透過 `--ImageReader.mask_path` 參數傳入遮罩資料夾路徑，即可讓 COLMAP 忽略路人、家具等不需要的區域，只對有效區域提取特徵進行三維重建。

## 參考資料

[^faq]: COLMAP. (n.d.). *FAQ — Mask image regions*. Retrieved 2026-09-26, from https://colmap.readthedocs.io/en/latest/faq.html#mask-image-regions
[^cli]: COLMAP. (n.d.). *Command Line Interface*. Retrieved 2026-09-26, from https://colmap.github.io/cli.html
[^deepwiki]: DeepWiki. (n.d.). *COLMAP — Feature Extraction and Matching (Feature Masking section)*. Retrieved 2026-09-26, from https://deepwiki.com/colmap/colmap/7.1-feature-extraction-and-matching
[^issue1545]: GitHub. (2022). *Issue #1545 — How to use --mask_path*. Retrieved 2026-09-26, from https://github.com/colmap/colmap/issues/1545
[^issue1460]: GitHub. (2020). *Issue #1460 — Image masks not working?*. Retrieved 2026-09-26, from https://github.com/colmap/colmap/issues/1460
[^issue1016]: GitHub. (2019). *Issue #1016 — How to use "mask path"*. Retrieved 2026-09-26, from https://github.com/colmap/colmap/issues/1016
[^issue1830]: GitHub. (2023). *Issue #1830 — camera_mask_path does not work*. Retrieved 2026-09-26, from https://github.com/colmap/colmap/issues/1830
[^issue208]: GitHub. (2019). *Issue #208 — Implement masking of images in feature extraction*. Retrieved 2026-09-26, from https://github.com/colmap/colmap/issues/208