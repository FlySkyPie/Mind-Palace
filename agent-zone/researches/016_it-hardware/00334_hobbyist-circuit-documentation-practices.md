# 業餘電子／電腦 DIY 的電路資料管理方法

## 概述

業餘愛好者（Hobbyist）在進行電子或電腦 DIY 專案時，面臨的核心挑戰與專業工程師相同：**如何記錄電路連接資訊（哪條線接到哪個 pin）、如何跨專案追蹤硬體設定、以及如何讓自己（或他人）日後還能理解這些記錄**。本報告根據網路資料，整理常見的做法、工具、與社群資源。

## 1. 接腳（Pinout）與連接器文件記錄方法

最詳盡的實用指南來自 TechOverflow，其推薦一套六步驟流程[^techoverflow]：

1. **拍照** — 用手機拍攝連接器在其原本環境中的照片。
2. **標註** — 使用向量繪圖軟體（如 **draw.io** 或 **Inkscape**）在照片上標示 pin 的功能，建議用醒目顏色（如紅色）標功能，綠色標線色。
3. **加入後設資料** — 專案名稱、連接器名稱與類型、修訂編號（如 R1.0）、ISO8601 日期。
4. **雙格式儲存** — 一個不可編輯版本（PDF/PNG）和一個可編輯原始檔（drawio XML）。
5. **納入版本管理** — 將檔案放入 git 或專案管理軟體中。
6. **核心哲學**：你現在覺得理所當然的細節，明天（或對其他人來說）將完全不清晰。

## 2. 跨專案硬體設定追蹤方法

根據 EEVblog 論壇長達十年以上的討論[^eevblog]，業餘愛好者最常用的方法包含：

- **工程筆記本（Engineering Notebook）** — 使用方格筆記本，一本一個專案，或一本涵蓋兩個專案（正面寫一個，翻過來從背面寫第二個）。按時間順序記錄所有內容：初始電路圖、失敗嘗試、量測數據。
- **活頁 binder 系統** — 使用活頁檔案夾搭配專案標籤，可重新排列與插入資料手冊。
- **混合方法** — 紙本筆記本用於每日記錄，後續整理到數位 wiki 或文件中長期保存。
- **Git 儲存庫** — 許多愛好者將完整硬體專案（含 KiCad 原始檔、物料清單 BOM、Gerber 匯出檔）全部放在一個 GitHub 儲存庫中。
- **數位工具** — 部分人使用 Evernote、OneNote、或專屬 wiki，但紙本筆記本因其即時性仍是主流。

## 3. 電路圖、麵包板佈局與接線記錄

- **先記錄邏輯電路圖**（schematic capture），再處理實體佈局[^breadboard]。
- **標準麵包板（breadboard）佈局作法**：先建立主要電源軌道，再配電源分佈，接著才佈訊號線；使用一致顏色配置（紅=正電、黑=GND、其他顏色=訊號）。
- **物料清單（BOM）** 始終與電路圖並存，通常以試算表或 PCB 設計軟體內嵌方式維護[^pcbdesign]。
- **Fritzing** 是專為此設計的工具，提供直觀的「麵包板視圖」，讓使用者以虛擬方式擺放元件與接線，再轉換為電路圖與 PCB 佈局[^fritzing]。

## 4. 常用軟體工具

### PCB 設計與電路圖捕捉

| 工具 | 類型 | 價格 (2026) | 適合對象 |
|------|------|-------------|----------|
| **KiCad 8** | 開源 EDA 套裝 | 免費 (GPL-3) | **業餘愛好者標準工具**。含電路圖編輯器、PCB 佈局、3D 檢視器、SPICE 模擬。CERN、System76 等機構使用。社群符號庫龐大。[^kicad] |
| **Fritzing** | 開源硬體工具 | 免費 | **麵包板轉 PCB 流程**。初學者友善 — 從麵包板視圖開始，再轉成電路圖與 PCB。[^fritzing] |
| **EasyEDA** | 雲端（瀏覽器） | 免費（付費 $40-200/年） | **JLCPCB 工廠整合**。一鍵訂購。內建 1M+ LCSC 零件庫。學習曲線最平緩。[^easyeda] |
| **Autodesk EAGLE**（已整合進 Fusion 360） | ECAD 工作空間 | **$545/年** | 舊工具 — 獨立 EAGLE 已於 2026 年 6 月終止。除非需要 ECAD/MCAD 雙向同步，否則不值得。[^pcbdesign] |

### 圖示與標註工具

- **draw.io**（diagrams.net） — 廣泛推薦用於標註連接器照片
- **Inkscape** — 向量繪圖用於 pinout 圖示
- **GIMP** — 像素級照片標註

### 零件庫資源

- **SnapEDA（SnapMagic Search）** — 1,000 萬+ 免費腳印/符號，支援所有主流 EDA 工具（KiCad、EAGLE、Altium 等）[^snapeda]
- **UltraLibrarian** — 全球最大免費 PCB CAD 函式庫，30+ 格式[^ultralibrarian]
- **KiCad 官方函式庫** — 約 25,000 個開源符號
- **LCSC 目錄**（via EasyEDA） — 1M+ 庫存元件，含即時價格

## 5. 實體標記方法

根據 2026 年針對電子工作坊專用標籤印表機的評測[^labelmaker]：

### 電子 DIY 必備功能
- **熱縮管相容** — 焊接前標記線纜，標籤能承受高溫與化學清潔劑
- **護膜輸出** — 不受 flux、清潔劑或頻繁接觸影響而褪色
- **纜線旗標模板** — 用於標記個別線纜

### 2026 年推薦機種
- **Brother P-Touch PT-D610BTVP**（編輯首選） — 藍牙、300dpi、175 種模板
- **Brady M210**（最佳性價比） — 軍事級耐震、支援熱縮、QR/條碼列印
- **DYMO Rhino 4200** — 工業級，具備專用熱縮模式

### 常見實體標記方法
- **色碼線纜** — 跨專案保持顏色一致（紅=正電、黑=GND、黃=訊號等）
- **線纜端點附近使用熱縮標籤**
- **元件收納箱以 QR code 標記**，連結數位庫存
- **麵包板行列以英數字座標標記**

## 6. 跨電腦型號／主機板的交叉參考

DIY 愛好者使用以下方法來追蹤不同裝置的連接資訊：

- **PinoutGuide.com** — 3,211 份文件，涵蓋 PCI Express（1x/4x/8x/16x）、M.2 NGFF、Mini PCIe、ATX 電源、SATA、USB-C 等，社群貢獻與維護，自 2000 年起運作[^pinoutguide]。
- **PinoutDatabase.org** — 276+ 元件的互動式接腳圖，含開發板、模組、晶片、連接器。提供可點選圖示及資料手冊連結[^pinoutdb]。
- **製造商資料手冊** — 最權威來源；愛好者將其按專案資料夾或 binder 分類整理。
- **GitHub 儲存庫** — 公開硬體文件儲存庫，愛好者為特定型號家族彙整接腳資料（如所有 Raspberry Pi Compute Module 版本）。
- **個人試算表資料庫** — 部分愛好者自行維護交叉參照表，對應連接類型與主機板型號。

## 7. 線上資料庫與社群資源

### 接腳資料庫
- **PinoutGuide.com** — 3,211 份文件，涵蓋 PC 硬體、消費電子、汽車音響、工業連接器。最古老且最大型的集合[^pinoutguide]。
- **PinoutDatabase.org** — 互動式接腳圖，276+ 元件涵蓋微控制器、開發板、IC、標準介面[^pinoutdb]。
- **PinoutLab.com** — 連接器圖鑑，提供詳細規格[^pinoutlab]。
- **PinoutSearch.com** — USB-C 與 RJ45 接腳參考。

### 社群平台
- **Hackster.io** — 硬體學習社群，分享含完整文件的專案[^hackster]。
- **OSHWHub / OSHWLab** — 開源硬體專案託管（與 EasyEDA/JLCPCB 生態系整合）。
- **GitHub** — 大量業餘硬體專案，含電路圖、BOM、KiCad 檔案。
- **EEVblog 論壇** — 長期運作的電子社群，詳細討論文件記錄方法[^eevblog]。
- **r/electronics、r/AskElectronics**（Reddit）。

## 總結

業餘電子 DIY 的電路資料管理並無單一標準，而是呈現**混合生態**：

- **記錄即時想法**用紙本方格筆記本
- **長期保存**用 git + KiCad 原始檔
- **接腳查詢**依賴 PinoutGuide 等線上資料庫
- **實體標記**依靠標籤印表機與色碼線纜
- **照片標註**使用 draw.io 或 Inkscape

關鍵心法是：**不要依賴記憶，記錄時假設自己三個月後會忘記所有細節**。

---

[^techoverflow]: TechOverflow. (2022-10-06). How to properly document electronics connectors — an easy guideline. Retrieved 2026-10-01, from https://techoverflow.net/2022/10/06/how-to-properly-document-electronics-connectors-an-easy-guideline/
[^eevblog]: EEVblog Forum. (n.d.). Project/Lab Notebook for documentation, note taking etc. Retrieved 2026-10-01, from https://www.eevblog.com/forum/chat/projectlab-notebook-for-documentation-note-taking-etc/
[^breadboard]: RayPCB. (n.d.). Breadboard Layout: Tips and Best Practices. Retrieved 2026-10-01, from https://www.raypcb.com/breadboard-layout/
[^pcbdesign]: Raphael Staebler. (n.d.). PCB Design — A Hobbyist's Perspective. Retrieved 2026-10-01, from https://raphaelstaebler.info/en/blog/pcb-design-a-hobbyists-perspective/
[^kicad]: KiCad. (n.d.). KiCad EDA Suite. Retrieved 2026-10-01, from https://www.kicad.org
[^fritzing]: Fritzing. (n.d.). Fritzing — Electronics Design Made Easy. Retrieved 2026-10-01, from https://fritzing.org
[^easyeda]: EasyEDA. (n.d.). EasyEDA — Free PCB design software. Retrieved 2026-10-01, from https://easyeda.com
[^snapeda]: SnapMagic Search. (n.d.). SnapEDA / SnapMagic Search. Retrieved 2026-10-01, from https://snapeda.com
[^ultralibrarian]: UltraLibrarian. (n.d.). World's Largest Free PCB CAD Library. Retrieved 2026-10-01, from https://www.ultralibrarian.com/
[^labelmaker]: Logix4U. (n.d.). Best Label Makers for Electronics Workshops 2026. Retrieved 2026-10-01, from https://www.logix4u.net/best-label-makers-for-electronics-workshops/
[^pinoutguide]: PinoutGuide.com. (n.d.). Handbook of hardware schemes, cables and connectors layouts pinouts. Retrieved 2026-10-01, from https://pinoutguide.com
[^pinoutdb]: PinoutDB.org. (n.d.). Interactive pinouts for components. Retrieved 2026-10-01, from https://pinoutdb.org/en
[^pinoutlab]: PinoutLab.com. (n.d.). Connector Atlas. Retrieved 2026-10-01, from https://pinoutlab.com/
[^hackster]: Hackster.io. (n.d.). Hackster — Learn, Share, Get Inspired. Retrieved 2026-10-01, from https://www.hackster.io