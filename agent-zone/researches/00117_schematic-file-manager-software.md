# Minecraft `.schematic` 檔案管理軟體調查

本報告調查自由／開源（FOSS）軟體中，專注於**管理、分類、瀏覽、搜尋** Minecraft `.schematic`（及其他格式）建築示意檔案的解決方案。排除純編輯器（如 WorldEdit、MCEdit）與純預覽工具。

## 完整目錄／管理系統

### 1. mc-schematic-manager（自託管 Web 應用）

- **類型**：Web 前端 + 後端伺服器
- **倉庫**：https://github.com/NikolaMilinkovic/mc-schematic-manager（前端）、https://github.com/NikolaMilinkovic/mc-schematic-manager-server（後端）
- **功能**：完整網頁版示意圖資產管理器，提供儲存、顯示與存取管理。使用 FAWE 轉換示意圖後上傳至伺服器，具備使用者認證、目錄瀏覽、選取與上傳流程。
- **技術棧**：React/Vite 前端、Node.js/Express/MongoDB 後端、Docker 部署
- **授權**：原始碼公開（未明確標示授權條款）
- **適合場景**：需要完整網頁目錄系統的使用者[^mc-schematic-manager]

### 2. LitematicDownloader（Fabric Mod，遊戲內）

- **類型**：用戶端模組
- **倉庫**：https://github.com/Choculaterie/LitematicDownloader
- **Modrinth**：https://modrinth.com/mod/litematicdownloader
- **功能**：遊戲內示意圖瀏覽器與檔案管理器。可直接從多個線上來源搜尋／下載示意圖。內建**檔案管理器**可組織、重新命名、刪除、甚至修改 `.litematic` 內的方塊類型。另提供 Quick-Share 功能。
- **技術**：Fabric Mod（客戶端），支援 Minecraft 1.21+
- **授權**：**MIT**
- **適合場景**：想在不離開遊戲的情況下管理示意圖的使用者[^litematicdownloader]

### 3. SchemFlow（Paper/Spigot 伺服器插件）

- **類型**：伺服端插件
- **倉庫**：https://github.com/c4g7-dev/SchemFlow
- **功能**：雲端原生示意圖管理器。提供指令清單、上傳、下載、貼上、刪除、分組、管理示意圖，支援 S3/MinIO 雲端儲存與本地儲存。具備群組管理（類似分類／資料夾）、回收站、快取、Skript API。
- **技術**：Java、Paper 1.21+、WorldEdit/FAWE、S3/MinIO
- **授權**：**Apache-2.0**
- **適合場景**：多伺服器網路需要雲端目錄系統的管理員[^schemflow]

### 4. organize-mc-schematics（Python CLI）

- **類型**：命令列工具
- **倉庫**：https://github.com/wregR/organize-mc-schematics
- **功能**：確定性組織工具。驗證格式（`.litematic`、`.schem`、`.schematic`、`.nbt`、`.mcstructure`），依據規則分類檔案（如「紅石裝置」、「農場」、「儲存技術」），透過 SHA-256 去重，檢查壓縮檔安全性，並將檔案移至分類資料夾。使用**預覽 → 套用 → 驗證**工作流程。
- **技術**：Python 3.10+，Windows
- **授權**：原始碼公開
- **適合場景**：批次整理既有示意圖收藏的使用者[^organize-mc-schematics]

## 專用／輔助工具

### 5. Litematic Organizer（瀏覽器擴充功能）

- **類型**：瀏覽器擴充功能
- **倉庫**：https://github.com/MCodex-org/litematic-organizer（Chrome）、https://github.com/emerald0s/Litematica-File-Organizer（Firefox 分支）
- **功能**：自動將下載的 `.litematic` 檔案導向專屬資料夾（如 `~/Downloads/schematics/`），設定 Litematica 使用該資料夾後，示意圖便會自動在遊戲中出現。
- **授權**：**MIT**
- **適合場景**：自動化下載流程的一環[^litematic-organizer]

### 6. SchematicUpload（Spigot/Paper 插件，已封存）

- **類型**：伺服端插件
- **倉庫**：https://github.com/WiIIiam278/SchematicUpload
- **功能**：網頁面板提供上傳／下載示意圖至伺服器。玩家使用 `/schematicupload` 取得連結，透過瀏覽器上傳檔案。含速率限制與下載支援。**已封存**（2024年6月）。
- **授權**：**Apache-2.0**
- **適合場景**：已被封存，但概念可供參考[^schematicupload]

### 7. Minemev Desktop（Electron 應用程式）

- **類型**：桌面應用程式
- **倉庫**：https://github.com/Dalinnar/Minemev-desktop
- **功能**：Minemev 示意圖檔案庫的桌面伴侶應用程式，可將 litematics 與世界檔直接下載至 Minecraft 資料夾。相當於**下載管理器**。
- **技術**：ElectronJS
- **授權**：原始碼公開
- **適合場景**：Minemev 平台的使用者[^minemev-desktop]

### 8. Pugtools Schematic Organizer（網頁工具）

- **類型**：網頁工具（閉源）
- **網址**：https://www.pugtools.com/tools/schematic-organizer/
- **功能**：瀏覽器內組織工具。拖拉資料夾後可檢視 3D 預覽、標記收藏、拖拉分組、批次重新命名、匯出含資料夾結構的壓縮檔。**不上傳任何檔案**，全部在客戶端執行。支援 `.litematic`、`.schem`、`.schematic`、`.nbt`。
- **授權**：免費網頁工具，原始碼未公開
- **適合場景**：快速在瀏覽器中整理示意圖[^pugtools]

## 總覽比較

| 工具 | 類型 | 適合場景 |
|------|------|----------|
| mc-schematic-manager | Web 應用（自託管） | 完整網頁示意圖資產目錄 |
| LitematicDownloader | 遊戲內 Fabric Mod | 不離開遊戲管理示意圖 |
| SchemFlow | 伺服器插件 | 多伺服器雲端目錄系統 |
| organize-mc-schematics | Python CLI | 批次整理既有收藏 |
| Litematic Organizer | 瀏覽器擴充功能 | 自動分類下載 |
| Minemev Desktop | Electron 桌面程式 | Minemev 平台下載管理 |

## 結論

若需求是「像檔案總管／圖庫／目錄般管理 `.schematic` 檔案」：

- **最接近完整目錄系統**：**mc-schematic-manager**（網頁端完整目錄 UI）與 **SchemFlow**（伺服端群組管理）
- **遊戲內管理**：**LitematicDownloader** 內建檔案管理器功能最全面
- **批次整理**：**organize-mc-schematics** 適合清理大量散亂示意圖

目前 FOSS 生態中缺乏一個成熟、專注於「本地示意圖圖庫瀏覽器」的桌面應用——多數方案傾向網頁版或遊戲內 Mod 形式。

---

[^mc-schematic-manager]: NikolaMilinkovic. (n.d.). mc-schematic-manager. GitHub. Retrieved 2026-09-22, from https://github.com/NikolaMilinkovic/mc-schematic-manager
[^litematicdownloader]: Choculaterie. (n.d.). LitematicDownloader. Modrinth. Retrieved 2026-09-22, from https://modrinth.com/mod/litematicdownloader
[^schemflow]: c4g7-dev. (n.d.). SchemFlow. GitHub. Retrieved 2026-09-22, from https://github.com/c4g7-dev/SchemFlow
[^organize-mc-schematics]: wregR. (n.d.). organize-mc-schematics. GitHub. Retrieved 2026-09-22, from https://github.com/wregR/organize-mc-schematics
[^litematic-organizer]: MCodex-org. (n.d.). Litematic Organizer. GitHub. Retrieved 2026-09-22, from https://github.com/MCodex-org/litematic-organizer
[^schematicupload]: WiIIiam278. (n.d.). SchematicUpload. GitHub. Retrieved 2026-09-22, from https://github.com/WiIIiam278/SchematicUpload
[^minemev-desktop]: Dalinnar. (n.d.). Minemev Desktop. GitHub. Retrieved 2026-09-22, from https://github.com/Dalinnar/Minemev-desktop
[^pugtools]: PugTools. (n.d.). Schematic Organizer. PugTools. Retrieved 2026-09-22, from https://www.pugtools.com/tools/schematic-organizer/
