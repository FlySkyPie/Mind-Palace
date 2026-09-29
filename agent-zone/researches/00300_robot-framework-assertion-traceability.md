# Robot Framework 斷言失敗追蹤性改善方案

## 問題描述

Robot Framework 在斷言（assertion）失敗時，預設的行為是僅輸出錯誤訊息（error message）與失敗的關鍵字（keyword）名稱，但**不會輸出 stack trace**。這導致在以下情境中難以追蹤具體的斷點位置：

- 測試案例（Test Case）包含多個檢查點
- 使用多層自訂關鍵字（custom keyword）時，無法知道確切的失敗行號
- 關鍵字中未添加獨特的失敗訊息時，無法區分哪一行斷言失敗
- CI/CD 環境中無法重現時，追蹤困難

此問題在 Robot Framework 官方 GitHub Issue [#5126](https://github.com/robotframework/robotframework/issues/5126) 中有詳細討論，但該 Issue 已被標記為 **Closed as not planned**，表示官方不打算在核心框架中加入此功能，而是由社群提供解決方案。[^issue5126]

---

## 改善方案

本節整理六種可行的改善方式，從最簡單的安裝套件到進階的自訂工具，以及程式碼層面的最佳實踐。

---

### 方案一：使用 `robotframework-stacktrace` 函式庫

這是最直接的解決方案，只需安裝並加上 `--listener` 參數即可獲得包含行號與變數值的追蹤路徑。[^stacktrace_pypi]

**安裝**：

```bash
pip install robotframework-stacktrace
```

**使用**：

```bash
robot --listener RobotStackTracer <your file.robot>
```

**效果**：原本的 Console 輸出僅顯示 `FAIL | TimeoutError: ...`，加入後會顯示完整的追蹤路徑，包含從 Test Case → Keyword → Resource File 的完整呼叫鏈，以及行號和變數值。[^stacktrace_gh]

```
Configure Car with Pass                                               ...
  Traceback (most recent call last):
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    File  /path/to/01_CarConfig.robot:23
    T:  Configure Car with Pass
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    File  /path/to/01_CarConfig.robot:28
      Select aMinigolf as model
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    File  /path/to/functional_keywords.resource:14
      Select Options By    ${select_CarBaseModel}    text    ${basemodel}
      |  ${select_CarBaseModel} = "Basismodell" (str)
      |  ${basemodel} = aMinigolf (str)
______________________________________________________________________________
```

---

### 方案二：自訂 Listener 擷取完整 Traceback

透過 Robot Framework 的 Listener API，在關鍵字失敗時使用 `robot.utils.error.get_error_details()` 取得完整 traceback 並寫入 log。[^custom_listener]

```python
from robot import running, result
from robot.api import logger as robot_logger
from robot.utils.error import get_error_details


class ExceptionTracebackListener:
    ROBOT_LIBRARY_SCOPE = "GLOBAL"
    ROBOT_LISTENER_API_VERSION = 3

    def __init__(self, log_level="INFO"):
        self.logger_func = getattr(robot_logger, log_level.lower())

    def end_keyword(self, data, result):
        if result.status == "FAIL":
            error_message, error_traceback = get_error_details(full_traceback=True)
            trace_details = (
                f"keyword failed: {error_message}\n{error_traceback}"
            )
            self.logger_func(msg=trace_details)
```

**使用**：

```bash
robot --listener path/to/ExceptionTracebackListener.py tests/
```

效果：在 `log.html` 中以指定 log level 輸出完整的 Python traceback，同時保留原本的資訊層級（INFO）日誌。

---

### 方案三：將 Log Level 設為 DEBUG 或 TRACE

Robot Framework 在 DEBUG 或 TRACE 層級時，**預設就會輸出完整 traceback**。這是最簡單但會讓日誌變得更加龐大的方式。[^debug_mode]

```bash
robot --loglevel DEBUG tests/
# 或
robot --loglevel TRACE tests/
```

亦可透過 `robot.toml` 設定：

```toml
log-level = "DEBUG"
```

---

### 方案四：使用 `robotframework-trace-report`

這是一個基於 OpenTelemetry 的進階方案，不只顯示 stack trace，還提供 **Gantt 圖、Timeline、並行執行檢視、failure triage** 等功能。[^trace_report]

```bash
pip install robotframework-trace-report

# 從 output.xml 直接產生互動式 HTML 報告
rf-trace-report output.xml -o report.html
```

特色功能：

- 階層式 Suite → Test → Keyword 導覽
- Keyword 展開可看到完整執行樹（含型別標籤、參數值）
- 失敗關鍵字有 error breadcrumbs
- Live mode：測試執行中即時更新
- 支援 SigNoz、Jaeger 等 OpenTelemetry 後端

---

### 方案五：使用 `robotframework-reportlens`

這是一個將 `output.xml` 轉換為現代化互動式 HTML 報告的工具，專注於改善除錯體驗。[^reportlens]

```bash
pip install robotframework-reportlens
reportlens output.xml -o report.html
```

特色功能：

- 以 failure 為優先的使用者介面設計
- 可擴展的關鍵字檢視
- 日誌過濾與快速搜尋
- 不需要額外依賴套件

---

### 方案六：在關鍵字中加入明確的失敗訊息（防禦性寫法）

這是在程式碼層面的最佳實踐，在每個斷言中加入有意義的訊息，使得失敗時能立即知道錯誤來源。[^builtin]

```robot
Should Be Equal    ${actual}    ${expected}    Price mismatch on checkout page
Should Contain    ${page_source}    ${text}    Product text not found after search
Run Keyword And Continue On Failure    Should Be Equal    ${a}    ${b}
```

搭配 `Run Keyword And Continue On Failure` 可以讓一個測試案例收集多個斷言結果，而不會在第一次失敗就中止執行。

---

## 方案比較

| 方案 | 難度 | 安裝方式 | 核心效果 | 適合場景 |
|------|------|----------|----------|----------|
| **robotframework-stacktrace** | 低 | `pip install` | Console traceback + 行號 + 變數值 | 日常開發除錯 |
| **自訂 Listener** | 中 | 自寫 Python | 自訂日誌內容與層級 | 需要客製化輸出 |
| **Log Level DEBUG/TRACE** | 低 | 無需安裝 | 完整 Python traceback | 臨時除錯 |
| **robotframework-trace-report** | 中高 | `pip install` | Timeline + Gantt + 互動式報告 | 大專案、CI/CD 分析 |
| **robotframework-reportlens** | 低 | `pip install` | 現代化 HTML 報告 | 替代預設 report.html |
| **Failure Message 防禦** | 低 | 無需安裝 | 語意化錯誤定位 | 長期專案維護 |

---

## 總結建議

1. **立即改善**：安裝 `robotframework-stacktrace` 並搭配 `--listener RobotStackTracer` 參數，立即獲得行號與變數值的 traceback。
2. **更完整的日誌記錄**：加上自訂 Listener 或在 CI 中使用 `--loglevel DEBUG`。
3. **長期最佳實踐**：在所有斷言中加入有意義的 failure message，並定期使用 `robotframework-trace-report` 或 `reportlens` 產生更易讀的報告。
4. **大型團隊/CI 環境**：導入 `robotframework-trace-report` 搭配 OpenTelemetry，獲得 timeline 與 failure triage 的完整除錯體驗。

---

[^issue5126]: Robot Framework Foundation. (n.d.). *Support for stack trace in outputs* [Feature Request #5126]. Retrieved 2026-09-27, from https://github.com/robotframework/robotframework/issues/5126

[^stacktrace_pypi]: MarketSquare. (n.d.). *robotframework-stacktrace* [Python package]. Retrieved 2026-09-27, from https://pypi.org/project/robotframework-stacktrace/

[^stacktrace_gh]: MarketSquare. (n.d.). *robotframework-stacktrace* [GitHub repository]. Retrieved 2026-09-27, from https://github.com/MarketSquare/robotframework-stacktrace

[^custom_listener]: juliusunscripted. (2023). *Robot Framework – Logging Full Traceback of Errors and Exceptions*. Retrieved 2026-09-27, from https://www.juliusunscripted.com/posts/robotframework-logging-full-traceback-of-errors-and-exceptions/

[^debug_mode]: The Pi Guy. (n.d.). *Debugging Robot Framework Tests with Debug Mode and Logging*. Retrieved 2026-09-27, from https://the-pi-guy.com/blog/debugging_robot_framework_tests_with_debug_mode_and_logging/

[^trace_report]: xtergo. (n.d.). *robotframework-trace-report* [GitHub repository]. Retrieved 2026-09-27, from https://github.com/xtergo/robotframework-trace-report

[^reportlens]: (n.d.). *robotframework-reportlens* [Python package]. Retrieved 2026-09-27, from https://pypi.org/project/robotframework-reportlens/

[^builtin]: Robot Framework Foundation. (n.d.). *BuiltIn Library Documentation*. Retrieved 2026-09-27, from https://robotframework.org/robotframework/latest/libraries/BuiltIn.html