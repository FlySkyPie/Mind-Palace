# Cuberite 插件日誌機制

## 概述

Cuberite 的 Lua 插件 API 中不存在獨立的 `cLog` 類別來提供日誌功能，而是透過一系列**全域函數**來實現日誌輸出。所有日誌訊息會同時寫入伺服器主控台與日誌檔案。[^writing-plugin]

## 可用的日誌函數

Cuberite 提供以下全域日誌函數，無需實例化任何物件即可在任何插件程式碼中使用：

| 函數 | 日誌層級 | 主控台顏色 |
|------|----------|-----------|
| `LOG()` | Regular（一般） | 白色/灰色 |
| `LOGINFO()` | Info（資訊） | 淺藍色 |
| `LOGWARNING()` | Warning（警告） | 黃色 |
| `LOGERROR()` | Error（錯誤） | 紅色 |

`LOGWARN()` 是 `LOGWARNING()` 的別名，兩者對應到同一個 C++ 函式 `tolua_LOGWARN`。[^manual-bindings]

底層 C++ 的日誌層級枚舉定義為：[^logger-simple]

```cpp
enum class eLogLevel {
    Regular,
    Info,
    Warning,
    Error
};
```

## 函數簽名與使用方式

所有日誌函數共用相同的參數結構：

```lua
LOG(Message, ...)
LOGINFO(Message, ...)
LOGWARNING(Message, ...)
LOGERROR(Message, ...)
```

### 基本字串輸出

```lua
LOG("Initialised version " .. Plugin:GetVersion())
LOGINFO("Loading configuration...")
LOGWARNING("Deprecated setting detected, using default")
LOGERROR("Failed to open file: " .. filename)
```

`LOG` 會自動在訊息前加上插件名稱作為前綴，方便識別日誌來源。[^writing-plugin]

### Printf 風格格式化字串

日誌函數支援類似 C 語言 `printf` 的格式化字串語法：[^logger-simple]

```lua
LOG("Player %s joined from %s", PlayerName, IP)
LOGWARNING("Block type %d is not supported", BlockType)
LOGERROR("Failed after %d attempts: %s", attempts, errorMsg)
```

### `cCompositeChat` 物件支援

`LOG()` 函數可接受 `cCompositeChat` 物件作為參數，此時會使用該物件內嵌的 `MessageType` 來決定日誌的嚴重程度分級，達到自訂顯示樣式的效果。[^globals]

### 除錯專用變體

- **`LOGD(...)`** — 僅在除錯（Debug）版本中輸出日誌；在 Release 版本中編譯為空操作，完全不產生任何程式碼。[^logger-simple]

## C++ 內部的「F」變體（非 Lua API）

Cubrite 的 C++ 原始碼中還有一系列使用 Python 風格的 `{}` 格式化語法的日誌函數，但這些**沒有暴暴露給 Lua 插件**，僅供 C++ 內部使用：[^logger-simple]

```cpp
FLOG(...)      // Regular 層級
FLOGINFO(...)    // Info 層級
FLOGWARNING(...)  // Warning 層級
FLOGERROR(...)   // Error 層級
FLOGD(...)     // 除錯專用的 FLOG
```

## 底層實作

這些日誌函數是透過 Cubrite 原始碼中的 `ManualBindins.cpp` 註冊為 Lua 全域函數：[^manual-bindins]

```cpp
tolua_function(tolua_S, "LOG",        tolua_LOG);
tolua_function(tolua_S, "LOGINFO",    tolua_LOGINFO);
tolua_function(tolua_S, "LOGWARN",    tolua_LOGWARN);
tolua_function(tolua_S, "LOGWARNING", tolua_LOGWARN);
tolua_function(tolua_S, "LOGERROR",   tolua_LOGERROR);
```

## 標準使用模式

在插件中最典型的使用情境是在初始化階段記錄載入狀態：

```lua
function Initialize(Plugin)
    Plugin:SetName("MyPlugin")
    Plugin:SetVersion(1)
    LOG("Initialised version " .. Plugin:GetVersion())
    return true
end

function OnDisable()
    LOG("Shutting down...")
end
```

## 輸出目標

所有 `LOG*` 函數的輸出會同時送往兩個目的地：
1. **伺服器主控台**（即時顯示，依嚴重程度有不同的顏色標示）
2. **日誌檔案**（持久化儲存於伺服器目錄下的日誌檔案中，便於回溯排查）

## 總結

Cuberite 的 Lua 插件日誌系統設計簡潔，以全域函數取代物件導向的 API：

- 四種嚴重程度：Regular、Info、Warning、Error，各有對應的視覺樣式
- 支援純字串與 `cCompositeChat` 物件兩種輸入型態
- 支援 `printf` 風格格式化字串
- 自動附加插件名稱前綴，便於多插件環境下的除錯
- `LOGWARN()` 與 `LOGWARNING()` 功能完全相同，建議統一使用 `LOGWARNING()`

## 參考資料

[^writing-plugin]: Cuberite Project. (n.d.). *Writing a Cuberite Plugin*. Retrieved 2026-09-25, from https://api.cuberite.org/Writing-a-Cuberite-plugin.html

[^manual-bindings]: Cuberite Project. (n.d.). *ManualBindings.cpp — Lua LOG function bindings*. Retrieved 2026-09-25, from https://github.com/cuberite/cuberite/blob/master/src/Bindings/ManualBindings.cpp

[^logger-simple]: Cuberite Project. (n.d.). *LoggerSimple.h — Logging function declarations*. Retrieved 2026-09-25, from https://raw.githubusercontent.com/cuberite/cuberite/master/src/LoggerSimple.h

[^globals]: Cuberite Project. (n.d.). *Globals*. Retrieved 2026-09-25, from https://api.cuberite.org/Globals.html