# Sketchfab 的自由開源替代方案研究報告

## 摘要

Sketchfab 是知名的線上 3D 模型發佈、分享與內嵌平台，但不是開源軟體。本報告調查自由開源（FOSS）的替代方案，區分為三大類別：（1）可自行架設的 3D 模型資產管理平台、（2）可嵌入網頁的 3D 檢視器元件，以及（3）開放/社群性質的 3D 模型倉儲服務。結論是：最接近 Sketchfab 完整功能的開源替代品是 **Manyfold**（自行架設、AGPL-3.0、支援 ActivityPub 聯邦）；若只需「檢視器」部分，則以 **Online 3D Viewer** 與 **Google `<model-viewer>`** 最為成熟。

## 背景說明

Sketchfab 提供使用者上傳、檢視、分享並在網頁中嵌入 3D 模型，支援 glTF、OBJ、FBX 等多種格式，並內建線上編輯與擴增實境（AR）檢視。然而其平台軟體並未開源，且商業授權模式可能不適合需要自主掌控資料與部署的個人或組織[^sf]。

自由開源替代方案因此可從兩個層面思考：**「管理與發佈」**層面（等同於 Sketchfab 的網站功能），以及**「檢視與嵌入」**層面（等同於其播放器/內嵌元件）。以下依此分類陳述。

## 一、可自行架設的 3D 模型資產管理平台

這類方案最接近「整套 Sketchfab」的定位：可以自行架設、管理模型庫、瀏覽、組織並分享模型。

### 1. Manyfold（最推薦）

| 項目 | 內容 |
|------|------|
| 定位 | 面向 3D 列印的數位資產管理平台，可瀏覽、檢視、組織並分享模型 |
| 授權 | AGPL-3.0 |
| 儲存庫 | [github.com/manyfold3d/manyfold](https://github.com/manyfold3d/manyfold) |
| GitHub 星數 | 約 2,200 ★ |
| 自行架設 | 是（Docker、Ruby on Rails、PostgreSQL） |
| 特點 | 內建 THREE.js 互動式 3D 檢視器，支援 ActivityPub（可與其他實例聯邦，成為去中心化方案）；受 NLnet/NGI0 資助，開發活躍；提供線上試用 demo[^manyfold] |

Manyfold 是目前最成熟、星數最高且開發活躍的開源「完整平台」類替代品。

### 2. Thingport

| 項目 | 內容 |
|------|------|
| 定位 | 自行架設的個人 3D 模型庫，著重 3D 列印 |
| 授權 | AGPL-3.0 |
| 儲存庫 | [github.com/TautvydasDerzinskas/Thingport](https://github.com/TautvydasDerzinskas/Thingport) |
| GitHub 星數 | 約 141 ★ |
| 自行架設 | 是（Docker Compose、PostgreSQL） |
| 特點 | 可藉由瀏覽器擴充功能直接從 MakerWorld、Printables、Thingiverse 匯入；支援 3D 預覽、集合、標籤、多使用者，以及切片器整合[^thingport] |

### 3. GyroidVault

| 項目 | 內容 |
|------|------|
| 定位 | 自行架設的 3D 模型庫與列印管理平台 |
| 授權 | AGPL-3.0 |
| 儲存庫 | [github.com/TeeCodeDev/GyroidVault](https://github.com/TeeCodeDev/GyroidVault) |
| GitHub 星數 | 約 157 ★ |
| 自行架設 | 是（Docker Compose、Node.js、SQLite） |
| 特點 | 多部件 3D 組裝檢視器、即時公釐尺寸、G-Code 預覽、Moonraker/Klipper 整合、角色權限管理、可設定過期時間的公開分享連結[^gyroidvault] |

### 4. MeshSilo

| 項目 | 內容 |
|------|------|
| 定位 | 自行架設的 3D 數位資產管理工具（DAM） |
| 授權 | AGPL-3.0 |
| 儲存庫 | [github.com/meshsilo/meshsilo](https://github.com/meshsilo/meshsilo) |
| GitHub 星數 | 極新，研究時約 0 ★（專案尚在早期） |
| 自行架設 | 是（Docker、PHP、MySQL/SQLite） |
| 特點 | 支援 STL、3MF、OBJ、PLY、GLTF/GLB、FBX、STEP、IGES 等多種格式；具 Web 3D 檢視器與 API、檔案去重[^meshsilo] |

### 5. STL Shelf

| 項目 | 內容 |
|------|------|
| 定位 | 3D 可列印模型的個人庫管理器 |
| 授權 | Apache 2.0 |
| 儲存庫 | [github.com/CQ-Fabrication/stl-shelf](https://github.com/CQ-Fabrication/stl-shelf) |
| GitHub 星數 | 約 27 ★ |
| 自行架設 | 是（Bun、PostgreSQL、S3 儲存）；另提供付費託管服務 |
| 特點 | 上傳、整理、版本控制、標籤、預覽（Three.js）；團隊協作與角色權限、版本歷史[^stlshelf] |

### 6. STLVault

| 項目 | 內容 |
|------|------|
| 定位 | 容器化的 3D 模型庫管理器 |
| 授權 | MIT |
| 儲存庫 | [github.com/moddroid94/STLVault](https://github.com/moddroid94/STLVault) |
| GitHub 星數 | 約 44 ★ |
| 自行架設 | 是（Docker Compose、FastAPI/Python、React、SQLite） |
| 特點 | 巢狀資料夾、列印群組、從 Printables/MakerWorld 的 URL 匯入、3D 預覽（STL、3MF、STEP）[^stlvault] |

### 7. GNU MediaGoblin

| 項目 | 內容 |
|------|------|
| 定位 | 去中心化、自行架設的媒體發佈平台（相片、影片、音訊、**3D 模型**） |
| 授權 | AGPL-3.0（GNU 計畫） |
| 儲存庫 | GNUSavannah（[savannah.gnu.org](https://savannah.gnu.org/git/?group=mediagoblin)），GitHub 上有鏡像 |
| GitHub 星數 | 不適用（主要不在 GitHub） |
| 自行架設 | 是（Python，需手動部署） |
| 特點 | 原定位為 FOSS 的 Flickr/YouTube 替代品，藉由與 LulzBot 合作支援 STL/3D 模型；可惜近年維護動能偏低[^mediagoblin] |

## 二、可嵌入網頁的 3D 檢視器元件

若只需要 Sketchfab 的「檢視器/內嵌播放器」部分，可採用以下開源元件，搭配自己的模型檔案託管。

### 8. Online 3D Viewer

| 項目 | 內容 |
|------|------|
| 定位 | 自由開源的 Web 3D 模型檢視解決方案，可直接在瀏覽器探索模型 |
| 授權 | MIT |
| 儲存庫 | [github.com/kovacsv/Online3DViewer](https://github.com/kovacsv/Online3DViewer) |
| GitHub 星數 | 約 3,700 ★ |
| 自行架設 | 是（靜態網頁放置於任意 Web Server，或作為 npm 套件嵌入） |
| 特點 | 可匯入 18+ 種格式（3dm、3ds、3mf、amf、dae、fbx、gltf、step、stl、obj、ply 等）、匯出 7 種格式；官方示範網站為 3dviewer.net；底層使用 Three.js[^online3dviewer] |

### 9. Google `<model-viewer>`

| 項目 | 內容 |
|------|------|
| 定位 | 用於在網頁與擴增實境（AR）中顯示互動式 3D 模型的 Web Component |
| 授權 | Apache 2.0 |
| 儲存庫 | [github.com/google/model-viewer](https://github.com/google/model-viewer) |
| GitHub 星數 | 約 8,300 ★ |
| 自行架設 | 是（npm 套件或 CDN，可直接內嵌） |
| 特點 | 以一個 `<model-viewer>` HTML 標籤即可顯示 glTF/GLB，支援自動旋轉、環境光照、AR 模式；已被 Google 搜尋等地大量採用；提供線上編輯器[^modelviewer] |

純粹是檢視元件，非完整平台；需自行託管 GLB 檔案。

### 10. ThreePipe

| 項目 | 內容 |
|------|------|
| 定位 | 用較簡單方式建立 Three.js 3D Web 體驗的框架 |
| 授權 | MIT |
| 儲存庫 | [github.com/repalash/threepipe](https://github.com/repalash/threepipe) |
| GitHub 星數 | 成長中的專案 |
| 自行架設 | 是（npm 套件，可嵌入） |
| 特點 | 提供視覺化編輯器、檢視器設定與外掛系統，比直接使用原始 Three.js 更容易建構自訂 3D 檢視器[^threepipe] |

## 三、開放/社群性質的 3D 模型倉儲

這些是「公眾可瀏覽、下載模型」的網站，具部落格的社群分享性質。

- **Open3DLab / SFMLab / Smutba.se**：由社群營運的 3D 資源分享網站家族，服務免費且可下載模型，但**平台軟體本身為專屬（非開源）**，僅其中部分工具腳本以 MIT 授權釋出。不是可自行架設的開源平台[^open3dlab]。

- **Thingiverse**：MakerBot 原創的開放硬體/3D 列印模型倉儲，模型常為 Creative Commons 授權，但平台本身為專屬軟體，不可自行架設[^thingiverse]。

> 值得注意的是：其他各大模型倉儲（Printables、MakerWorld、Cults3D、MyMiniFactory、Thangs、CGTrader、TurboSquid）皆為**專屬平台**，原始碼不公開。因此「公開模型倉儲 + 開源」的組合並不存在於現有的成熟大型服務中；要走 FOSS 路線，多數情況是自行架設上文的管理平台，並透過聯邦機制彼此分享[^alternativeto]。

## 比較總覽

| 方案 | 類型 | 授權 | GitHub 星數 | 可自行架設 | 最適合 |
|------|------|------|-------------|------------|--------|
| **Manyfold** | 完整平台 | AGPL-3.0 | ~2,200 ★ | 是（Docker） | 最接近 Sketchfab 的完整替代；3D 列印模型庫 |
| **Thingport** | 完整平台 | AGPL-3.0 | ~141 ★ | 是（Docker） | 從其他平台匯入；多使用者模型庫 |
| **GyroidVault** | 完整平台 | AGPL-3.0 | ~157 ★ | 是（Docker） | 列印農場管理；多部件組裝檢視器 |
| **MeshSilo** | 完整平台 | AGPL-3.0 | 新專案 | 是（Docker） | 輕量 DAM；廣泛格式支援 |
| **STL Shelf** | 完整平台 | Apache 2.0 | ~27 ★ | 是（Docker） | 版本追蹤；團隊協作 |
| **STLVault** | 完整平台 | MIT | ~44 ★ | 是（Docker） | 簡單自行架設庫；URL 匯入 |
| **MediaGoblin** | 媒體平台 | AGPL-3.0 | 不適用（GNU） | 是（手動） | 通用媒體分享、含 3D 支援 |
| **Online 3D Viewer** | 可嵌入檢視器 | MIT | ~3,700 ★ | 是（靜態檔） | 在自家網站取代 Sketchfab 的檢視/嵌入 |
| **Google model-viewer** | Web Component | Apache 2.0 | ~8,300 ★ | 是（npm/CDN） | 以 HTML 標籤在任意網頁嵌入並支援 AR |
| **Open3DLab/SFMLab** | 公開倉儲 | 平台為專屬 | 不適用 | 否 | 瀏覽/公開分享 3D 模型（非開源平台） |

## 建議與批判性反思

**若想擁有一套可自行架設的「完整 Sketchfab 替代品」**：首選 **Manyfold**（最成熟、星數最高、開發活躍、支援去中心化聯邦）；次選 **Thingport**（擅長從既有平台匯入模型）。

**若只需要「檢視器/嵌入」部分**：使用 **Online 3D Viewer**（完整檢視器應用）或 **Google `<model-viewer>`**（簡單的 HTML 標籤嵌入、支援 AR）。

**若需要「公開模型倉儲」**：目前的成熟大型倉儲幾乎都是專屬平台。唯一務實的 FOSS 路徑是自行架設 **Manyfold** 並透過 ActivityPub 與他人聯邦分享，而非依賴某個開源的公開服務[^alternativeto][^manyfold]。

**資料來源的批判性提醒**：本報告中 GitHub 星數與專案細節來自網路搜尋彙整，星數會隨時間變動；部分新專案（如 MeshSilo）尚在早期，採用前應再確認其維護狀態與功能成熟度。許多「替代方案清單」網站（如 AlternativeTo）對「開源」的判定與實際授權狀態可能存在落差，故以各專案官方 GitHub 授權標示為準[^alternativeto]。

## 參考文獻

[^sf]: Sketchfab. (n.d.). *Sketchfab 官方網站*. Retrieved 2026-10-09, from https://sketchfab.com/

[^manyfold]: Manyfold. (n.d.). *Manyfold GitHub 儲存庫*. Retrieved 2026-10-09, from https://github.com/manyfold3d/manyfold

[^thingport]: Derzinskas, T. (n.d.). *Thingport GitHub 儲存庫*. Retrieved 2026-10-09, from https://github.com/TautvydasDerzinskas/Thingport

[^gyroidvault]: TeeCodeDev. (n.d.). *GyroidVault GitHub 儲存庫*. Retrieved 2026-10-09, from https://github.com/TeeCodeDev/GyroidVault

[^meshsilo]: meshsilo. (n.d.). *MeshSilo GitHub 儲存庫*. Retrieved 2026-10-09, from https://github.com/meshsilo/meshsilo

[^stlshelf]: CQ Fabrication. (n.d.). *STL Shelf GitHub 儲存庫*. Retrieved 2026-10-09, from https://github.com/CQ-Fabrication/stl-shelf

[^stlvault]: moddroid94. (n.d.). *STLVault GitHub 儲存庫*. Retrieved 2026-10-09, from https://github.com/moddroid94/STLVault

[^mediagoblin]: GNU Project. (n.d.). *GNU MediaGoblin*. Retrieved 2026-10-09, from https://mediagoblin.org/

[^online3dviewer]: kovacsv. (n.d.). *Online3DViewer GitHub 儲存庫*. Retrieved 2026-10-09, from https://github.com/kovacsv/Online3DViewer

[^modelviewer]: Google. (n.d.). *model-viewer GitHub 儲存庫*. Retrieved 2026-10-09, from https://github.com/google/model-viewer

[^threepipe]: repalash. (n.d.). *ThreePipe 官方網站*. Retrieved 2026-10-09, from https://threepipe.org/

[^open3dlab]: Open3DLab. (n.d.). *Open3DLab 官方網站*. Retrieved 2026-10-09, from https://open3dlab.com/

[^thingiverse]: MakerBot. (n.d.). *Thingiverse 官方網站*. Retrieved 2026-10-09, from https://www.thingiverse.com/

[^alternativeto]: AlternativeTo. (n.d.). *Sketchfab open source 替代軟體*. Retrieved 2026-10-09, from https://alternativeto.net/software/sketchfab/?license=opensource
