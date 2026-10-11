# 如何在多台 TriliumNext 伺服器之間搬移筆記資料

本報告說明如何在 **TriliumNext**（原名 Trilium，GitHub 上約 38.3k 顆星[^stars]）的多台伺服器實例之間搬移、同步或備份筆記資料。TriliumNext 是 Trilium 的社群維護分支，用於個人知識管理與筆記樹的組織。

## 重要前提：架構上的限制

TriliumNext 的同步機制是**星形拓撲（star-shaped topology）**：一台中央「同步伺服器」與多台「桌面用戶端」雙向同步。**兩台伺服器實例之間沒有直接的伺服器對伺服器同步功能**[^sync-topo]。因此，若要在兩台伺服器之間搬移資料，必須使用下列其中一種方式。

---

## 方法一：使用匯出 / 匯入（Export / Import）

TriliumNext 內建支援三種匯出格式[^import-export]：

| 格式 | 說明 |
|---|---|
| **HTML** | Trilium 主要格式，匯出後可用於靜態網站發佈 |
| **Markdown** | 以 CommonMark 為基礎，可匯出單一筆記或整棵子樹（ZIP 壓縮） |
| **OPML** | 支援 OPML v1.0（純文字）與 v2.0（可含 HTML），適合與大綱工具交換 |

流程：
1. 在來源伺服器的筆記樹上**按右鍵 → Export**（或 Note Actions → Export note），選擇格式並匯出
2. 將匯出的檔案（單一 `.md` 或 ZIP）移動到目標伺服器
3. 在目標伺服器「匯入」：直接**拖曳檔案**到筆記樹，或使用 Import 對話框

**注意事項**：
- 受保護（protected）的筆記在匯出時會被**未加密匯出**，必須先在作用中的受保護工作階段才能匯出
- 匯出會遺失部分後設資料（metadata）
- 單一項目匯入大小上限約 **374 MiB**（由同步協定決定）；整體上傳限制已於 v0.104.0 移除[^import-export]

另可透過 **ETAPI（REST API）** 進行程式化匯出與自動化：

```
GET /etapi/notes/{noteId}/export?format={html|markdown}
```

以 `noteId` 設為 `root` 即可匯出整個文件。範例（以 curl 匯出 Markdown ZIP）[^etapi]：

```bash
curl -H "Authorization: $TRILIUM_ETAPI_TOKEN" \
     -X GET "https://<伺服器>/etapi/notes/root/export?format=markdown" \
     --output export.zip
```

授權方式：`Authorization: <token>`（純 token）、`Authorization: Bearer <token>`（v0.93.0 起）或 Basic Auth[^etapi]。可透過 cron（Linux）或工作排程器（Windows）排程執行以自動化備份。注意：大型資料庫的匯出會先在記憶體中組建再串流輸出，記憶體受限的伺服器可能卡住[^export-auto]。

---

## 方法二：使用內建備份 / 還原（Backup / Restore）

TriliumNext 內建備份系統，是最安全的**整機搬移**方式[^backup]。

### 自動備份
系統會自動在以下時機建立備份，並保留最多 4 份不同時間點的備份：
- 每天（daily）
- 每週（weekly）
- 每月（monthly）
- 資料庫遷移（migration）之前

預設存於資料目錄下的 `backup/` 子目錄。

### 備份檔案格式
| 副檔名 | 格式 | 說明 |
|---|---|---|
| `.db` | 原始 SQLite 資料庫副本 | 支援手動還原方式 |
| `.tnbackup` | Trilium 備份容器 | 支援 AES-256-GCM 加密（scrypt 金鑰衍生）及 GZIP 壓縮，僅能透過 GUI 還原 |

### 還原方式

**方式 A：GUI 還原（推薦，支援 .db 與 .tnbackup）**
- 新實例：設定選單中選「Restore from backup」
- 既有實例：**Settings → Backup → Restore a backup**，重啟後依提示操作，若為加密備份需輸入密碼

**方式 B：手動還原（僅支援 .db）**
1. 找到資料目錄，並找出 `backup/backup-weekly.db`
2. **停止 / 關閉** TriliumNext
3. 刪除 `document.db`、`document.db-wal`、`document.db-shm`
4. 將備份檔複製到資料目錄並改名為 `document.db`
5. 確保可寫入：`chmod 600 document.db`
6. 重新啟動

---

## 方法三：直接搬移資料庫檔案（伺服器對伺服器）

由於兩台伺服器之間無內建同步，**直接搬移 SQLite 資料庫檔案**是伺服器間整機搬移的可行方法。

TriliumNext 的資料庫是標準 **SQLite** 檔案 `document.db`，位於資料目錄中（預設路徑依平台而異，可用 `TRILIUM_DATA_DIR` 環境變數自訂）[^data-dir]：

| 平台 | 預設資料目錄 |
|---|---|
| Linux | `/home/[user]/.local/share/trilium-data` |
| Linux (Flathub) | `/home/[user]/.var/app/org.triliumnotes.Trilium/data/trilium-data` |
| Windows | `C:\Users\[user]\AppData\Roaming\trilium-data` |
| macOS | `/Users/[user]/Library/Application Support/trilium-data` |
| Docker / 其他 | 使用者家目錄 |

資料目錄內容包括 `document.db`、`config.ini`、`backup/`、`log/`。

### 搬移步驟
1. **停止**來源與目標的 TriliumNext 伺服器程序（避免資料庫被寫入）
2. **刪除**目標資料目錄中的 `document.db`、`document.db-wal`、`document.db-shm`
3. **複製**來源的 `document.db`（或備份檔）到目標資料目錄並改名為 `document.db`
4. `chmod 600 document.db`
5. **重新啟動**目標伺服器

### 重要警告
- **務必刪除 `-wal` 與 `-shm` 檔案**，否則可能造成重啟迴圈（restart loops）[^backup]
- SQLite 以 WAL 模式運作時，最近寫入可能先存在 `document.db-wal` 側車檔中，尚未合併回主檔；直接用 `cp`/`rsync` 複製主檔會漏掉這些未認可（uncommitted）的資料列[^sqlite-wal]。正規做法是**先停止伺服器**使其完成 checkpoint，或改用 `sqlite3 document.db ".backup 目標檔"` 產生一致的快照
- 來源與目標伺服器最好**使用相同 TriliumNext 版本**，以避免資料庫不相容；升級時 TriliumNext 會在啟動時自動遷移資料庫，而遷移後**舊版本將無法讀取升級過的資料庫**[^upgrade]

---

## 方法四：多伺服器資料同步（星形拓撲）

若需求是**持續、雙向地**在多個位置間同步（例如一台伺服器加多台桌面用戶端），則應使用內建同步機制[^sync-topo]：

- **星形拓撲**：一台中央同步伺服器 + 多台桌面用戶端（亦可桌面對桌面，不需伺服器）
- 離線優先（offline-first）：每個桌面用戶端本地保存資料
- 雙向同步：任一端更動會推播到伺服器，其他端拉取
- 使用**版本化同步協定**，所有叢集成員必須同步升級到相同協定版本

因為**兩台伺服器之間無法直接互相同步**，若要將伺服器 A 的資料持續同步到伺服器 B，常見作法是將兩台伺服器都當成「桌面用戶端」掛到同一個同步伺服器，或直接採用整機搬移（方法二 / 三）。

其他同步需求/限制：
- 兩端時間必須一致（容差約 5 分鐘）
- 強烈建議使用 HTTPS（有效 TLS 憑證）；自簽憑證需設定 `NODE_TLS_REJECT_UNAUTHORIZED=0`（不建議）
- 衝突處理：同一筆記在不同端被編輯時，**較新的版本勝出**，舊版保留在筆記修訂紀錄（note revisions）中
- 每次同步後會比對兩端雜湊，不一致時自動觸發恢復[^sync-topo]

---

## 方法比較總表

| 方法 | 最佳用途 | 注意事項 |
|---|---|---|
| 匯出 / 匯入（HTML/MD/OPML） | 部分搬移、跨應用互通 | 遺失部分後設資料；受保護筆記未加密匯出 |
| 內建備份 / 還原（.db/.tnbackup） | 災難復原、新實例初始化 | 保留完整狀態；支援加密 |
| 資料庫檔直接搬移 | 伺服器對伺服器整機搬移 | 需相同版本；務必刪除 WAL/SHM 檔案 |
| 同步（星形拓撲） | 持續雙向多端同步 | 僅桌面↔中央伺服器；需相同協定版本 |

---

## 引用來源

[^stars]: TriliumNext. (n.d.). Trilium - Build your personal knowledge base with TriliumNext. Retrieved 2026-10-10, from https://github.com/TriliumNext/Trilium

[^sync-topo]: TriliumNext. (n.d.). Synchronization - User Guide. Retrieved 2026-10-10, from https://docs.triliumnotes.org/user-guide/setup/synchronization

[^import-export]: TriliumNext. (n.d.). Import & Export - User Guide. Retrieved 2026-10-10, from https://docs.triliumnotes.org/user-guide/concepts/import-export

[^etapi]: TriliumNext. (n.d.). ETAPI - User Guide. Retrieved 2026-10-10, from https://docs.triliumnotes.org/user-guide/advanced-usage/etapi

[^backup]: TriliumNext. (n.d.). Backup - User Guide. Retrieved 2026-10-10, from https://docs.triliumnotes.org/user-guide/setup/backup

[^data-dir]: TriliumNext. (n.d.). Trilium data directory - User Guide. Retrieved 2026-10-10, from https://docs.triliumnotes.org/user-guide/setup/data-directory

[^upgrade]: TriliumNext. (n.d.). Upgrading Trilium - User Guide. Retrieved 2026-10-10, from https://docs.triliumnotes.org/user-guide/setup/upgrading

[^sqlite-wal]: 全球知識庫 (Global Knowledge Commons). (2026). SQLite WAL 模式下資料快照注意事項. Retrieved 2026-10-10, from 本機知識庫 ku_8b4aa929d15c4325b92d670129f21325

[^export-auto]: TriliumNext. (n.d.). GitHub Discussions: Automated export via ETAPI. Retrieved 2026-10-10, from https://github.com/orgs/TriliumNext/discussions/6601
