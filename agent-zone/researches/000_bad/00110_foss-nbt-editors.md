# FOSS NBT 編輯器研究與比較

> [!WARNING] 對齊失敗
> "editor" 是指 3D 編輯器，不是二進制檔案編輯器。

## 摘要

本報告調查可編輯 Minecraft NBT (.nbt) 檔案的開放原始碼（FOSS）編輯器，涵蓋桌面圖形介面工具、命令列工具、網頁版編輯器，以及具備 NBT 編輯能力的 Minecraft 世界編輯器。針對各工具的授權方式、平台支援、維護狀態與功能特性進行比較分析，協助使用者根據自身需求選擇合適工具。

## 1. 背景

NBT（Named Binary Tag）是 Minecraft 使用的二進位資料格式，用於儲存玩家資料（`level.dat`）、地圖區塊（region 檔案）、結構（schematic/schematics 檔案）及建築藍圖（`.mcstructure`）等。編輯 NBT 檔案的需求包括：修改地圖資料、調整玩家狀態、轉換建築格式、或開發 Minecraft 相關工具。

## 2. 桌面圖形介面編輯器

### 2.1 NBT Studio（作者：tryashtar）

NBT Studio 是功能最豐富的現代化 NBT 編輯器，支援 `.dat`、`.mca`（Anvil 格式）、`.mcr`（McRegion 格式）、`.mcstructure`（Bedrock 結構檔案）與 SNBT（文字格式）。最新版本 v1.15.3（2026 年 10 月釋出），在 GitHub 上獲得 820 顆星、301 次提交[^nbtstudio]。

- **授權**：原始碼公開於 GitHub，但**未附帶任何開放原始碼授權條款**。2021 年有 Issue #25 請求加入自由軟體授權，但已關閉且未處理[^nbtstudio-license]。因此 NBT Studio **並非真正的 FOSS**。
- **平台**：**僅限 Windows**（需 .NET Desktop Runtime 6.0）。
- **特色**：樹狀檢視器/編輯器、SNBT 支援、復原/重做、拖放檔案、多選編輯、十六進位編輯器、正則表達式搜尋、深色模式、完整的新增/刪除/編輯所有 NBT 標籤類型。
- **安裝**：Microsoft Store 或 GitHub Releases。

### 2.2 NBTExplorer（作者：jaquadro）

NBTExplorer 是最經典的 NBT 編輯器，支援 `.dat`、`.schematic`、`.mcr`、`.mca`、`.nbt` 及 Cubic Chunks 區域檔案。GitHub 上獲得 3,000 顆星、342 個分支[^nbtexplorer]。

- **授權**：**MIT 授權** ✅ 完全自由開源。
- **平台**：**Windows、Linux（透過 Mono）、macOS**（原生 UI）。
- **維護狀態**：**維護模式** —— 最後一次提交已是數年前。但仍可正常運作，可從 Flathub 安裝[^flathub-nbtexplorer]。
- **特色**：樹狀 NBT 檢視與編輯、支援所有 NBT 標籤類型、可開啟區域/Anvil 檔案、原生 macOS 版本。

### 2.3 jdNBTExplorer（作者：JakobDev）

jdNBTExplorer 是使用 Python/PyQt6 編寫的輕量級現代化 NBT 編輯器，支援 `.dat`、`.dat_old`、`.mca`、`.mcc` 格式[^jdnbtexplorer]。

- **授權**：**GPL-3.0** ✅ 完全自由開源。
- **平台**：**Linux**（Flatpak、AUR、AppImage）、**Windows**（winget、SourceForge）、**macOS**（透過 pip）。
- **維護狀態**：**積極維護中** —— 最新版本 3.0，73 次提交。
- **特色**：PyQt6 現代介面、支援 `.mcc` 檔案（Minecraft 區域格式的子格式）。
- **安裝**：`winget install JakobDev.jdNBTExplorer` 或 Flathub[^flathub-jdnbt]。

### 2.4 NBTEditor（作者：Howaner）

以 C++/Qt5 編寫的簡易 NBT 編輯器，支援 `.dat` 與 `.schematic`[^nbteditor]。

- **授權**：**BSD-3-Clause** ✅ 完全自由開源。
- **平台**：**Windows**（可攜版 + 安裝版）、**Linux**（Ubuntu .deb）。
- **維護狀態**：**停滯** —— 僅 11 次提交、72 顆星，許久未更新。
- **特色**：基本介面，可編輯 `.dat` 與 `.schematic`。功能陽春，不建議作為主要工具。

## 3. 命令列工具

### 3.1 nbted（作者：C4K3）

以 Rust 撰寫的命令列 NBT 編輯器，將 NBT 轉換為可編輯的文字格式。GitHub 上 89 次提交、140 顆星[^nbted]。

- **授權**：**CC0-1.0（公眾領域）** ✅ 完全自由開源。
- **平台**：**跨平台**（Rust 編譯，支援所有平台）。
- **維護狀態**：**積極維護中**。
- **用法**：
  ```bash
  nbted --print file.nbt > file.txt   # 導出為文字
  # 編輯 file.txt...
  nbted -r file.txt > file.nbt         # 導回 NBT
  ```
- **特色**：與任何 `$EDITOR` 整合、pipe-friendly、適合自動化腳本。
- **安裝**：`cargo install nbted`、Nix、或下載 Releases。
- **備註**：連 norbert 的 README 都推薦改用 nbted。

### 3.2 norbert（作者：DMBuce）

以 Python 撰寫的命令列 NBT 編輯器，提供 `vinbt` 腳本讓使用者透過 vim 編輯 NBT 檔案[^norbert]。

- **授權**：**GPL-2.0** ✅ 完全自由開源。
- **平台**：**跨平台**（Python）。
- **維護狀態**：**已停止維護** —— README 明確表示不再維護，僅 18 顆星。
- **備註**：作者推薦改用 nbted。

### 3.3 nbt2yaml（作者：zzzeek，即 SQLAlchemy 作者 Mike Bayer）

以 Python 撰寫的工具組，包含 `nbtedit` 命令（將 NBT 轉為 YAML 供編輯器編輯後存回）、`nbt2yaml` 與 `yaml2nbt` 轉換器[^nbt2yaml]。

- **授權**：**MIT** ✅ 完全自由開源。
- **平台**：**跨平台**（Python）。
- **維護狀態**：**停滯但仍可用** —— 8 顆星，最後釋出為 2021 年 3 月。
- **用法**：`pip install nbt2yaml` → `nbtedit myworld/level.dat`。
- **特色**：透過 YAML 進行結構化編輯，適合腳本化/自動化流程。

### 3.4 nbtlib CLI

以 Python 撰寫的 NBT 函式庫，附帶命令列工具，支援以 NBT path 方式讀取/編輯檔案[^nbtlib]。

- **授權**：**MIT** ✅ 完全自由開源。
- **平台**：**跨平台**（Python）。
- **備註**：主要為 Python 函式庫，但提供 CLI 工具。

## 4. 網頁版編輯器

### 4.1 bedrock-nbt-editor（作者：w1zardz）

功能最完整的網頁版 NBT 編輯器，100% 客戶端運作（不上傳任何檔案），支援 Bedrock 與 Java 雙版本。支援 `level.dat`、`.nbt`、`.mcstructure`、`.schem`、`.schematic`、`.dat`、以及 `.mca` 檔案中的區塊資料[^bedrocknbt]。

- **授權**：**MIT** ✅ 完全自由開源，原始碼在 GitHub[^bedrocknbt-github]。
- **平台**：**任何瀏覽器**（桌面 + 手機），支援離線運作。
- **維護狀態**：**積極維護中**。
- **特色**：自動偵測壓縮方式與 Byte Order、支援全部 13 種 NBT 標籤類型、64-bit BigInt 支援、SNBT 匯出、格式轉換、行動端友善 UI、可腳本化 API（`window.NBT`）。
- **網址**：https://w1zardz.github.io/bedrock-nbt-editor/

### 4.2 webNBT（作者：iRath96）

以 emscripten 編譯的簡易 HTML5 NBT 編輯器。GitHub 上 22 次提交、282 顆星[^webnbt]。

- **授權**：原始碼公開但**未附授權條款** ❌ 非 FOSS。
- **平台**：**任何瀏覽器**。
- **維護狀態**：**停滯**。
- **網址**：https://irath96.github.io/webNBT/
- **備註**：功能簡單，適合快速檢視。

### 4.3 其他非 FOSS 網頁工具

以下網頁工具雖然實用但**並非開源**：Mcgic NBT Editor、Nodecraft NBT Editor、wisehosting NBT Editor、Chunkweave NBT Editor、CraftMC NBT Editor。

## 5. Minecraft 世界編輯器（具備 NBT 能力）

### 5.1 MCEdit-Unified（作者：Podshot）

經典的 Minecraft 3D 世界編輯器，內建 NBT 編輯面板。支援 `.schematic`、NBT 世界資料與區域檔案[^mcedit]。

- **授權**：**ISC（等同 MIT）** ✅ 完全自由開源。
- **平台**：**Windows、Linux、macOS**。
- **維護狀態**：**已停止維護** —— 需要 Python 2.7，不支援現代 Minecraft（1.13+ 的 Blockstate 系統）。
- **備註**：對舊版本 Minecraft 仍可運作，但已被 Amulet Editor 完全取代。

### 5.2 Amulet Editor（作者：Amulet Team）

MCEdit 的繼承者，支援所有現代 Minecraft 格式（Java 1.12+、Bedrock 1.7+），具備 3D 世界編輯、跨版本世界轉換、NBT 編輯與 Schematic 匯出入等能力。GitHub 2,200 顆星[^amulet]。

- **授權**：**最新版本為付費軟體**（約 $10-20 USD）。最後一個**免費版本為 0.10.44**，採用 LGPL 授權[^amulet-free]。
- **平台**：**Windows、Linux、macOS**。
- **維護狀態**：**積極開發中**，但由付費授權支持。
- **備註**：底層的 `amulet-nbt` Python 函式庫仍為 **MIT 授權**且積極維護，適合程式化存取 NBT 資料。

## 6. 比較總表

| 工具 | FOSS？ | 平台 | 積極維護？ | 最適合用途 |
|---|---|---|---|---|
| **NBT Studio** | ❌（無授權） | Windows | ✅ 是 | Windows 桌面 NBT 編輯（不在意授權） |
| **NBTExplorer** | ✅ MIT | Win/Lin/Mac | ⚠️ 維護模式 | 經典桌面 NBT 編輯 |
| **jdNBTExplorer** | ✅ GPL-3.0 | Win/Lin/Mac | ✅ 是 | 現代跨平台桌面 NBT 編輯器 |
| **nbted** | ✅ CC0-1.0 | 跨平台 (Rust) | ✅ 是 | 命令列/腳本化 NBT 編輯 |
| **nbt2yaml** | ✅ MIT | 跨平台 (Python) | ⚠️ 停滯 | YAML 式 NBT 編輯 |
| **bedrock-nbt-editor** | ✅ MIT | 任何瀏覽器 | ✅ 是 | 快速網頁 NBT 編輯、Bedrock 支援 |
| **webNBT** | ❌（無授權） | 任何瀏覽器 | ❌ 停滯 | 簡易瀏覽器檢視 |
| **Amulet v0.10.44（免費版）** | ✅ LGPL | Win/Lin/Mac | ⚠️ 舊版 | 3D 世界編輯 + NBT |
| **amulet-nbt（Python 函式庫）** | ✅ MIT | 跨平台 | ✅ 是 | 程式化 NBT 存取 |

## 7. 建議

- **現代桌面圖形介面**：使用 **jdNBTExplorer**（完全自由開源、跨平台、積極維護），或 **NBT Studio**（Windows 限定、授權模糊）。
- **命令列/腳本化**：使用 **nbted**（Rust、跨平台、公眾領域）。
- **快速瀏覽器編輯**：使用 **bedrock-nbt-editor**（MIT、離線運作、功能完整）。
- **程式化 NBT 存取**：使用 **amulet-nbt** Python 函式庫（MIT、積極維護）。
- **完整 3D 世界編輯**：取得最後免費的 **Amulet Editor v0.10.44**（但可能無法開啟最新 Minecraft 世界）。

## 參考資料

[^nbtstudio]: tryashtar. (n.d.). NBT Studio. Retrieved 2026-09-22, from https://github.com/tryashtar/nbt-studio
[^nbtstudio-license]: tryashtar/nbt-studio. (2021). Issue #25: Please add a free software license. Retrieved 2026-09-22, from https://github.com/tryashtar/nbt-studio/issues/25
[^nbtexplorer]: jaquadro. (n.d.). NBTExplorer. Retrieved 2026-09-22, from https://github.com/jaquadro/NBTExplorer
[^flathub-nbtexplorer]: Flathub. (n.d.). NBTExplorer. Retrieved 2026-09-22, from https://flathub.org/en/apps/com.jaquadro.NBTExplorer
[^jdnbtexplorer]: JakobDev. (n.d.). jdNBTExplorer. Retrieved 2026-09-22, from https://codeberg.org/JakobDev/jdNBTExplorer
[^flathub-jdnbt]: Flathub. (n.d.). jdNBTExplorer. Retrieved 2026-09-22, from https://flathub.org/en/apps/page.codeberg.JakobDev.jdNBTExplorer
[^nbted]: C4K3. (n.d.). nbted. Retrieved 2026-09-22, from https://github.com/C4K3/nbted
[^norbert]: DMBuce. (n.d.). norbert. Retrieved 2026-09-22, from https://github.com/DMBuce/norbert
[^nbt2yaml]: zzzeek. (n.d.). nbt2yaml. Retrieved 2026-09-22, from https://github.com/zzzeek/nbt2yaml
[^nbtlib]: (n.d.). nbtlib. Retrieved 2026-09-22, from https://pypi.org/project/nbtlib/
[^bedrocknbt]: w1zardz. (n.d.). bedrock-nbt-editor. Retrieved 2026-09-22, from https://w1zardz.github.io/bedrock-nbt-editor/
[^bedrocknbt-github]: w1zardz. (n.d.). bedrock-nbt-editor (GitHub). Retrieved 2026-09-22, from https://github.com/w1zardz/bedrock-nbt-editor
[^webnbt]: iRath96. (n.d.). webNBT. Retrieved 2026-09-22, from https://github.com/iRath96/webNBT
[^mcedit]: Podshot. (n.d.). MCEdit-Unified. Retrieved 2026-09-22, from https://github.com/Podshot/MCEdit-Unified
[^amulet]: Amulet Team. (n.d.). Amulet Map Editor. Retrieved 2026-09-22, from https://github.com/Amulet-Team/Amulet-Map-Editor
[^amulet-free]: Amulet Team. (n.d.). Amulet Free. Retrieved 2026-09-22, from https://www.amuletmc.com/free