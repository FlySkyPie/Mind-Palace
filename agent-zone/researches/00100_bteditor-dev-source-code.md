# bteditor.dev 原始碼搜尋報告

## 專案概述

[bteditor.dev](https://bteditor.dev/) 是一款基於 [Drawflow](https://github.com/jerosoler/Drawflow) 構建的瀏覽器端視覺化行為樹（Behavior Tree）編輯器與除錯工具，採用 MIT 授權釋出。主要功能包括拖拽式節點編輯、JSON/XML 雙向匯入匯出、NDJSON 日誌回放、WebSocket 即時監控、自訂節點等。

## 原始碼位置

**實際儲存庫：** [github.com/lingzolabs/bt-editor](https://github.com/lingzolabs/bt-editor)

該儲存庫託管於 GitHub 組織 `lingzolabs` 下，於 2025-11-08 建立，共 12 次提交，最後一次推送在 2026-05-15。透過 GitHub Pages 部署於 [lingzolabs.github.io/bt-editor/](https://lingzolabs.github.io/bt-editor/)，並以自訂域名 bteditor.dev 指向該站點[^1]。

## 關於 bteditor/bteditor 儲存庫

bteditor.dev 的 About 頁面與 Changelog 均標註原始碼儲存庫為 `https://github.com/bteditor/bteditor`，但該 URL 回傳 404（組織 `bteditor` 不存在）。此為預期的規範儲存庫位置但尚未建立，實際程式碼暫存於 lingzolabs/bt-editor[^2]。

## 專案結構

```
├── index.html                # 主頁面
├── server.js                 # 本地開發伺服器
├── css/
│   ├── main.css
│   └── drawflow.custom.css
├── js/
│   ├── main.js              # 應用入口
│   ├── app.js               # 全域狀態
│   ├── editor.js            # 編輯器核心
│   ├── behaviorTree.js      # 行為樹資料轉換
│   ├── nodeTemplates.js     # 節點模板定義
│   ├── logPlayer.js         # 日誌回放引擎
│   ├── wsViewer.js          # WebSocket 即時檢視
│   └── ui/                  # UI 控制器
├── examples/
│   ├── sample_tree.json
│   ├── sample_nodes.json
│   └── bt_log.jsonl
└── .github/workflows/
    └── deploy.yml           # CI/CD 部署
```

## 技術棧

- **前端：** 原生 HTML5 + CSS3 + JavaScript（ES6+），無需建構工具
- **視覺化節點編輯器：** Drawflow
- **部署：** GitHub Pages（靜態站點）
- **本地開發：** Node.js（可選）

## 授權與貢獻者

MIT License。貢獻者列表見 GitHub 儲存庫的 Contributors 頁面[^1]。

## 參考資料

[^1]: lingzolabs. (n.d.). bt-editor. GitHub. Retrieved 2026-09-20, from https://github.com/lingzolabs/bt-editor

[^2]: Behavior Tree Editor. (n.d.). About. Retrieved 2026-09-20, from https://bteditor.dev/about

[^3]: lingzolabs. (n.d.). bt-editor README. GitHub. Retrieved 2026-09-20, from https://raw.githubusercontent.com/lingzolabs/bt-editor/master/README.md