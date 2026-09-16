# Cuberite Plugin 資料夾結構與安裝方式

Cuberite 是一個以 C++ 撰寫、相容於 Minecraft Java Edition 的輕量高效能遊戲伺服器，其 Plugin 系統使用 Lua 語言開發[^cuberite-repo]。本文說明 Cuberite Plugin 應遵守的資料夾結構、必要檔案、以及安裝與啟用方式。

## Plugin 資料夾結構

每個 Plugin 必須放在 `Server/Plugins/` 目錄下各自獨立的子資料夾中。Cuberite 在啟動時掃描此目錄來發現所有 Plugin[^plugin-tutorial]。

```
Server/
└── Plugins/
    ├── Core/                     ← 必要核心 Plugin（處理基本遊戲體驗）
    │   ├── main.lua
    │   ├── Info.lua
    │   └── ...（其他 .lua 檔案、子資料夾、設定檔）
    ├── MyNewPlugin/              ← 你的 Plugin
    │   ├── main.lua
    │   ├── Info.lua
    │   └── ...
    ├── InfoReg.lua               ← 共用函式庫（隨 Cuberite 附贈）
    └── APIDump/                  ← 內建 API 文件產生 Plugin
```

### 載入機制

Cuberite 在執行時期會將 Plugin 資料夾內**所有 `.lua` 檔案**依**字母順序**合併載入，其中 `Info.lua` 固定**最後**載入[^plugin-tutorial][^deepwiki]。

## 必要檔案

### `main.lua` — Plugin 進入點

每個 Plugin **至少需要一個 Lua 檔案**包含 `Initialize(Plugin)` 函式。慣例上命名為 `main.lua`。`Initialize` 函式**必須回傳 `true`**，否則 Cuberite 認為 Plugin 載入失敗並記錄錯誤[^plugin-tutorial]。

最小必要結構：

```lua
PLUGIN = nil

function Initialize(Plugin)
    Plugin:SetName("MyNewPlugin")
    Plugin:SetVersion(1)

    -- 註冊 Hook（事件監聽）
    -- cPluginManager.AddHook(cPluginManager.HOOK_PLAYER_JOINED, OnPlayerJoined)

    -- 綁定指令
    -- cPluginManager.BindCommand("/cmd", "myplugin.cmd", HandleCmd, " - Does something")

    PLUGIN = Plugin
    LOG("Initialised version " .. Plugin:GetVersion())
    return true   -- ← 必要
end

function OnDisable()
    LOG("Shutting down...")
end
```

- **`Initialize(Plugin)`** — 必要。Plugin 啟動時呼叫，必須回傳 `true`
- **`OnDisable()`** — 選擇性。Plugin 關閉或卸載時呼叫，用於清理資源
- **`Plugin:SetName("Name")`** — 設定 Plugin 顯示名稱
- **`Plugin:SetVersion(1)`** — 設定版本號（必須為整數）
- **全域變數 `PLUGIN`** — 可選，儲存 Plugin 物件以便在 `OnDisable()` 中使用

### `Info.lua` — Plugin 後設資料

`Info.lua` 是一個結構化的後設資料檔案，集中定義指令、權限、說明文件等資訊。**非必要**但強烈建議使用，特別是當 Plugin 有定義指令時[^info-file]。

結構範例：

```lua
g_PluginInfo =
{
    Name = "Example Plugin",
    Date = "2024-06-12",
    Description = "This is an example plugin that shows how to use the Info.lua file",

    AdditionalInfo =
    {
        {
            Title = "Setup",
            Contents = "Describe setup steps here",
        },
    },

    Commands =
    {
        ["/cmd1"] =
        {
            HelpString = "Performs the first action",
            Permission = "firstplugin.cmd.1",
            Handler = HandleCmd1,
            ParameterCombinations =
            {
                { Params = "x y z", Help = "Performs action at coords" },
            },
        },
    },

    ConsoleCommands =
    {
        concmd =
        {
            HelpString = "Performs a console action",
            Handler = HandleConCmd,
        },
    },

    Permissions =
    {
        ["firstplugin.cmd.1"] =
        {
            Description = "Allows the player to use /cmd1",
            RecommendedGroups = "players",
        },
    },

    Categories = {},
}
```

要在 Plugin 程式碼中啟用 `Info.lua` 的定義，需在 `Initialize()` 中加入[^info-file]：

```lua
dofile(cPluginManager:GetPluginsPath() .. "/InfoReg.lua")
RegisterPluginInfoCommands()        -- 註冊 Info.lua 中的遊戲內指令
RegisterPluginInfoConsoleCommands() -- 註冊 Info.lua 中的控制台指令
```

## 安裝 Plugin 的方式

### 步驟一：放置 Plugin 資料夾

將 Plugin 的完整資料夾（包含 `main.lua` 等檔案）複製到 Cuberite 伺服器的 `Server/Plugins/` 目錄下[^plugin-forum]。

### 步驟二：在 `settings.ini` 中啟用 Plugin

編輯伺服器根目錄的 `settings.ini`，在 `[Plugins]` 區段中加入項目[^cuberite-book]：

```ini
[Plugins]
Core=1
MyNewPlugin=1
```

- **`PluginName=1`** — 啟用該 Plugin
- **`PluginName=0`** — 停用該 Plugin

Plugin 會依照 `[Plugins]` 中列出的順序載入。`Core` 應永遠列於首位並保持啟用。

### 步驟三：重新啟動伺服器或熱載入

- **重新啟動伺服器** — 最直接的方式
- **使用控制台指令** — 不需重啟伺服器[^plugin-manager-api]

| 指令 / 函式 | 說明 |
|---|---|
| `/reload` | 重新載入所有啟用中的 Plugin |
| `/reload <PluginName>` | 重新載入指定的 Plugin |
| `cPluginManager:ReloadPlugin("Name")` | 非同步佇列重新載入 Plugin |

## Plugin 狀態

Cuberite 的 `cPluginManager` API 定義了五種 Plugin 狀態[^plugin-manager-api]：

| 狀態 | 數值 | 說明 |
|---|---|
| `psLoaded` | 0 | 已啟用且成功載入 |
| `psDisabled` | 1 | 未在 `settings.ini` 中啟用 |
| `psUnloaded` | 2 | 已在設定中啟用但被手動卸載 |
| `psError` | 3 | 已啟用但載入失敗（請檢查錯誤紀錄） |
| `psNotFound` | 4 | 曾載入過但資料夾已不存在 |

## 總結

| 項目 | 必要？ | 說明 |
|---|---|---|
| Plugin 資料夾位於 `Server/Plugins/` 下 | ✅ 是 | 資料夾名稱即為 Plugin 識別名稱 |
| `main.lua` 包含 `Initialize()` | ✅ 是 | 必須 `return true` |
| `Info.lua` | ❌ 否但建議 | 定義指令、權限、後設資料 |
| `OnDisable()` | ❌ 否 | 關閉時用於清理 |
| 在 `settings.ini` 的 `[Plugins]` 加入項目 | ✅ 是 | `PluginName=1` 啟用 |
| 使用 `InfoReg.lua` | 若有 `Info.lua` 則需 | `dofile(cPluginManager:GetPluginsPath() .. "/InfoReg.lua")` |

## 參考資料

[^cuberite-repo]: Cuberite Contributors. (n.d.). *Cuberite: A lightweight, fast and extensible game server for Minecraft*. GitHub. Retrieved 2026-09-16, from https://github.com/cuberite/cuberite

[^plugin-tutorial]: Cuberite Contributors. (n.d.). *Writing a Cuberite plugin*. Cuberite API Documentation. Retrieved 2026-09-16, from https://api.cuberite.org/Writing-a-Cuberite-plugin.html

[^info-file]: Cuberite Contributors. (n.d.). *Using the Info.lua file*. Cuberite API Documentation. Retrieved 2026-09-16, from https://api.cuberite.org/InfoFile.html

[^plugin-manager-api]: Cuberite Contributors. (n.d.). *cPluginManager Class*. Cuberite API Documentation. Retrieved 2026-09-16, from https://api.cuberite.org/cPluginManager.html

[^deepwiki]: DeepWiki Contributors. (n.d.). *Cuberite - Plugin System*. Retrieved 2026-09-16, from https://deepwiki.com/cuberite/cuberite/6-plugin-system

[^cuberite-book]: Cuberite Contributors. (n.d.). *Cuberite User's Manual - Configuration*. Retrieved 2026-09-16, from https://book.cuberite.org/

[^plugin-forum]: Cuberite Forum. (n.d.). *Plugin installation instructions*. Retrieved 2026-09-16, from https://forum.cuberite.org/thread-1590.html