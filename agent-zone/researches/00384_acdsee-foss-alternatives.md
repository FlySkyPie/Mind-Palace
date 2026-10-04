# ACDSee 開源替代方案研究報告

## 概述

ACDSee 是一套商業付費的數位照片管理與編輯軟體，提供目錄管理、標籤/評分、RAW 檔案處理、非破壞性編輯及批次處理等功能。本報告針對其功能定位，尋找自由開源（FOSS）替代方案。

## 完整替代方案（管理＋編輯兼具）

### 1. digiKam — 最佳全方位替代方案

digiKam 是功能最完整的數位資產管理（DAM）軟體，支援 Windows、macOS、Linux 三大平台，採用 GPLv2+ 授權[^digikam-features]。

**目錄與組織功能：**
- 支援多來源收藏（本機、卸除式媒體、網路磁碟）
- 分層專屬相簿、標籤（關鍵字、虛擬相簿）、評分（星級、顏色標籤、旗標）
- AI 驅動自動標籤、人臉偵測與辨識（OpenCV 深度神經網路，GPU 加速）
- 美學偵測：AI 自動分類照片品質並指派旗標
- 地理定位：世界地圖檢索
- 進階搜尋：標籤、日期、時光軸、人臉、地理位置、相似度（重複偵測）、自然語言 LLM 搜尋
- 支援 SQLite 或 MariaDB 資料庫後端
- 重複影像指紋偵測、批次更名、群組管理
- 可管理十萬張以上照片資料庫

**編輯功能：**
- 16-bit 色深、ICC 色彩管理
- RAW 匯入（原生 LibRaw 或外部 Darktable/RawTherapee）
- 完整編輯工具組：曲線、色階、HSL、白平衡、色彩平衡、銳利化、降噪、鏡頭校正（Lensfun）、透視校正、液態縮放、紅眼修正等
- G'Mic-Qt 濾波器整合
- 影像版本管理（非破壞性）
- 批次佇列管理器（多核心平行處理）
- 外掛：印刷、月曆、全景拼接、影片 slideshow、HTML 畫廊、Email 寄送

**分享：** Flickr、Piwigo、Dropbox、OneDrive、Pinterest、Box、MediaWiki、DLNA 串流。

### 2. Darktable — 專業 RAW 處理與組織

Darktable 是虛擬燈箱（lighttable）與暗房（darkroom）思維設計的 RAW 處理軟體，4×32-bit 浮點管線、GPU 加速（OpenCL）、非破壞性編輯，支援 Linux、macOS、Windows、BSD，GPLv3 授權[^darktable-features]。

**組織功能：** 資料庫驅動管理、星級/顏色標籤、關鍵字與 EXIF/IPTC/XMP 搜尋、智慧相簿（動態收藏）、底片捲檢視、序列與重複處理、彈性過濾排序。

**注意：** Darktable 的組織功能圍繞 RAW 工作流程設計，適合「編輯為主、組織為輔」的使用情境。

### 3. RawTherapee — 進階 RAW 轉換器

支援數百種相機機型的 RAW 轉換引擎，提供完整色彩校正、降噪、銳利化、鏡頭校正工具[^rawtherapee]。GPLv3 授權，支援 Windows/macOS/Linux。組織功能較弱，建議搭配 digiKam 使用。

### 4. GIMP — 頂級影像編輯器（非管理員）

GIMP 為 Photoshop 等級的影像操縱/修圖軟體，具備圖層、遮罩、色版、數百種濾鏡與外掛。GIMP 3.2+ 引入非破壞性編輯。GPLv3 授權[^gimp]。**無資料庫目錄功能**，需搭配 digiKam 或 Shotwell 等管理軟體使用。

## 輕量管理／瀏覽方案（基本編輯）

### 5. Shotwell

GNOME 桌面環境的簡易照片管理員，支援從相機或磁碟匯入，以日期、事件、標籤、評分組織。提供基本裁剪、紅眼修正、色彩調整。LGPLv2.1+ 授權，主力 Linux 平台[^shotwell]。

### 6. Gwenview

KDE 桌面環境的快速看圖軟體，支援目錄瀏覽、評分、刪除、縮放、裁剪、旋轉、紅眼修正。提供影像標註功能（箭頭、形狀、文字框、印章）。GPLv2+ 授權[^gwenview]。

### 7. nomacs

輕量 Qt 跨平台看圖軟體，支援 RAW（LibRaw）、PSD、TIFF、AVIF/HEIC/JXL（KImageFormats 外掛）。提供多實例同步、批次處理、slideshow。GPLv3 授權[^nomacs]。

### 8. PhotoQt

現代化 Qt/QML 看圖軟體，支援 140+ 影像格式、EXIF/IPTC/XMP 元資料顯示（含人臉標記）、GPS 地圖檢視、360° 全景、Chromecast 支援。GPLv2+ 授權[^photoqt]。

## 網頁式自架管理方案（不包含編輯）

### 9. PhotoPrism

AI 驅動的網頁照片管理員，支援物件/場景/色彩自動分類、人臉辨識、6 種高解析度世界地圖、重複偵測、RAW/影片支援。AGPLv3 授權，Docker 部署[^photoprism]。無內建編輯功能。

### 10. Immich

Google Photos 自架替代方案，提供 Android/iOS 自動備份、AI 物件/場景/人臉搜尋、反向地理編碼、標籤編輯、合作者分享。AGPLv3 授權，Docker 部署，開發非常活躍[^immich]。無內建編輯功能。

### 11. Piwigo

成熟（17+ 年）的網頁照片畫廊與 DAM，支援分層相簿與權限管理、批次管理工具、行動應用程式、外掛系統。GPLv2 授權[^piwigo]。

## 快速比較表

| 軟體 | 組織能力 | 編輯能力 | 平台 | 授權 |
|---|---|---|---|---|
| **digiKam** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | Win/Mac/Linux | GPL (FOSS) |
| **Darktable** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Win/Mac/Linux | GPL (FOSS) |
| **RawTherapee** | ⭐ | ⭐⭐⭐⭐ | Win/Mac/Linux | GPL (FOSS) |
| **GIMP** | 無 | ⭐⭐⭐⭐⭐ | Win/Mac/Linux | GPL (FOSS) |
| **Shotwell** | ⭐⭐⭐ | ⭐⭐ | Linux | LGPL (FOSS) |
| **Gwenview** | ⭐⭐ | ⭐⭐ | Linux (KDE) | GPL (FOSS) |
| **PhotoPrism** | ⭐⭐⭐⭐ (AI) | 無 | 網頁自架 | AGPL (FOSS) |
| **Immich** | ⭐⭐⭐⭐ (AI) | 無 | 網頁自架 | AGPL (FOSS) |
| **Piwigo** | ⭐⭐⭐ | 無 | 網頁自架 | GPL (FOSS) |
| **nomacs** | ⭐ | ⭐⭐ | Win/Mac/Linux | GPL (FOSS) |
| **PhotoQt** | ⭐⭐ | ⭐ | Linux/Win | GPL (FOSS) |

## 推薦工作流程

- **單一軟體完整取代 ACDSee** → **digiKam**（管理＋編輯兼備，三大平台支援，具 AI 功能）
- **最佳 RAW 編輯＋基本組織** → **Darktable**（RAW 開發）＋ **digiKam**（目錄管理）
- **自架網頁瀏覽（Google Photos 風格）** → **Immich** 或 **PhotoPrism**，搭配 GIMP/Darktable 編輯
- **Linux 輕量組織** → **Shotwell** 或 **Gwenview** ＋ **GIMP**

---

[^digikam-features]: digiKam. (n.d.). Features. Retrieved 2026-10-03, from https://www.digikam.org/about/features/
[^darktable-features]: darktable. (n.d.). Features. Retrieved 2026-10-03, from https://www.darktable.org/about/features/
[^rawtherapee]: RawTherapee. (n.d.). About. Retrieved 2026-10-03, from https://rawtherapee.com/
[^gimp]: GIMP. (n.d.). Features. Retrieved 2026-10-03, from https://www.gimp.org/features/
[^shotwell]: Shotwell Project. (n.d.). About. Retrieved 2026-10-03, from https://www.shotwell-project.org/
[^gwenview]: KDE. (n.d.). Gwenview. Retrieved 2026-10-03, from https://apps.kde.org/gwenview/
[^nomacs]: nomacs. (n.d.). Features. Retrieved 2026-10-03, from https://nomacs.org/docs/documentation/features/
[^photoqt]: PhotoQt. (n.d.). Features. Retrieved 2026-10-03, from https://photoqt.org/
[^photoprism]: PhotoPrism. (n.d.). Features. Retrieved 2026-10-03, from https://www.photoprism.app
[^immich]: Immich. (n.d.). Features. Retrieved 2026-10-03, from https://immich.app/
[^piwigo]: Piwigo. (n.d.). About. Retrieved 2026-10-03, from https://piwigo.org/