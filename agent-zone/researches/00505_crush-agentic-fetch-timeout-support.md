# Crush 的 `agentic_fetch` 是否支援逾時（timeout）？

## 摘要

**結論：不支援。** 截至本報告撰寫時（Crush v0.98.1，2026-10-09 發布），[Charm](https://charm.land) 的開源終端機 AI 編碼代理 **Crush**（GitHub 倉庫 `charmbracelet/crush`，約 28,596 顆星[^stars]）**並沒有針對 `agentic_fetch` 這個工具、或其背後子代理（sub-agent）的專屬逾時設定**。

`agentic_fetch` 是一個「代理式抓取」工具：它會啟動一個獨立的子代理，讓子代理自己去搜尋網路、抓取網頁、並萃取使用者要的資訊[^fetchdoc]。使用者關心的情境（「子代理為了解決一個不可能的任務而無限打轉，需要逾時來終止」）目前在 Crush 中**只能被間接、且不完整地緩解**，主要靠全域的 LLM 請求逾時 `request-timeout`、以及一個「重複工具呼叫偵測」的迴圈防護機制。

| 機制 | 是否存在 | 是否可設定 | 是否針對 `agentic_fetch` |
| --- | --- | --- | --- |
| 工具參數 `timeout` | 是（`fetch`、`sourcegraph`）| 是（呼叫時帶入，上限 120 秒）| 否 |
| `agentic_fetch` 專屬逾時 | 否 | 否 | — |
| 子代理整體執行時間上限 | 否 | 否 | 否 |
| 全域 LLM 請求逾時 `request-timeout` | 是 | 是（`0` 可停用）| 間接（子代理的模型呼叫）|
| 重複工具呼叫迴圈偵測 | 是 | 否（硬編碼）| 是（子代理共用同一代理迴圈）|

## 背景：`agentic_fetch` 的運作方式

`agentic_fetch` 工具的參數只有兩個欄位：`url`（選填）與 `prompt`（必填），**沒有任何逾時欄位**[^types]。其原始碼定義如下[^types]：

```go
// AgenticFetchParams defines the parameters for the agentic fetch tool.
type AgenticFetchParams struct {
	URL    string `json:"url,omitempty" description:"The URL to fetch content from (optional - if not provided, the agent will search the web)"`
	Prompt string `json:"prompt" description:"The prompt describing what information to find or extract"`
}
```

對照之下，**簡單版**的 `fetch` 工具確實有逾時參數，但那是「工具呼叫引數」而非設定檔選項[^types]：

```go
// FetchParams defines the parameters for the simple fetch tool.
type FetchParams struct {
	URL     string `json:"url" description:"The URL to fetch content from"`
	Format  string `json:"format" description:"The format to return the content in (text, markdown, or html)"`
	Timeout int    `json:"timeout,omitempty" description:"Optional timeout in seconds (max 120)"`
}
```

`agentic_fetch` 會建立一個子代理（sub-agent），給它 `web_fetch`、`web_search`、`glob`、`grep`、`sourcegraph`、`view` 等工具，並用「小模型」（small model）執行[^agenticfetch]。子代理的執行參數結構 `subAgentParams` 內**沒有任何逾時欄位**[^coordinator]：

```go
// subAgentParams holds the parameters for running a sub-agent.
type subAgentParams struct {
	Agent          SessionAgent
	SessionID      string
	AgentMessageID string
	ToolCallID     string
	Prompt         string
	SessionTitle   string
	SessionSetup   func(sessionID string)
}
```

## 一、硬編碼的 HTTP 逾時（不可設定）

`agentic_fetch` 及其子代理所用的網路工具，都建立了一個 **30 秒**的 `http.Client`，且該數值直接寫死在程式碼中，**無法透過設定調整**[^agenticfetch][^webfetch][^websearch][^sourcegraph]：

```go
client = &http.Client{
	Timeout:   30 * time.Second,
	Transport: transport,
}
```

這代表「單一次 HTTP 抓取」有 30 秒上限，但**子代理可以做很多次抓取**，整體執行時間仍沒有上限。

`fetch` 工具則另外用 `context.WithTimeout` 實作每次呼叫的逾時，並把上限硬性夾在 **120 秒**[^fetchcode]：

```go
// maxFetchTimeoutSeconds is the maximum allowed timeout for fetch requests (2 minutes)
const maxFetchTimeoutSeconds = 120

// Handle timeout with context
requestCtx := ctx
if params.Timeout > 0 {
	if params.Timeout > maxFetchTimeoutSeconds {
		params.Timeout = maxFetchTimeoutSeconds
	}
	var cancel context.CancelFunc
	requestCtx, cancel = context.WithTimeout(ctx, time.Duration(params.Timeout)*time.Second)
	defer cancel()
}
```

`sourcegraph` 工具也有相同設計（呼叫時可帶 `timeout`，上限 120 秒）[^sourcegraph]。

## 二、全域 LLM 請求逾時：`request-timeout`

Crush 唯一的**使用者可設定逾時**，是套用在每一次「LLM API 請求」上的 `request_timeout`（Bash 設定的鍵名為 `option request-timeout`，JSON 為 `options.request_timeout`）[^configdocs][^configgo]：

- 文件描述：*「seconds before an LLM request is aborted; streaming responses are only aborted after this much inactivity; 0 waits forever (default 60)」*[^configdocs]
- **實際 Go 預設值為 2 分鐘**（`const DefaultRequestTimeout = 2 * time.Minute`），與文件的 60 秒不一致，屬實作與文件不同步[^configgo]。
- **語意**：非串流請求是「硬性期限」；串流請求則是「閒置逾時」，只有在供應商完全停止輸出時才中止，緩慢但持續輸出的串流不會被殺掉[^reqtimeout]。

其包裝邏輯見 `internal/agent/request_timeout.go`[^reqtimeout]：

```go
// newRequestTimeoutModel bounds each request to m with the given timeout. A
// timeout of zero or less, or a nil model, returns m unchanged.
func newRequestTimeoutModel(m fantasy.LanguageModel, timeout time.Duration) fantasy.LanguageModel {
	if m == nil || timeout <= 0 {
		return m
	}
	return requestTimeoutModel{LanguageModel: m, timeout: timeout}
}
```

**對 `agentic_fetch` 的影響**：子代理是用「小模型」跑多次 LLM 請求，每次請求各自受 `request-timeout` 約束。因此這個設定**只能限制「單次模型請求」**，不能限制「子代理的整體任務時間」。子代理若不斷改變策略、每次都發出新的（且各自合法的）請求，仍可無限打轉下去。

使用者可這樣調整（`crushrc`）[^configdocs]：

```bash
option request-timeout 120   # 每次 LLM 請求 120 秒
option request-timeout 0     # 完全停用逾時
```

## 三、重複工具呼叫的迴圈偵測（硬編碼）

Crush 在代理迴圈中有一個 `StopWhen` 條件，會偵測「同一組工具呼叫＋結果」重複出現的情形[^loopdet]：

```go
const (
	loopDetectionWindowSize = 10
	loopDetectionMaxRepeats = 5
)

// hasRepeatedToolCalls checks whether the agent is stuck in a loop by looking
// at recent steps. It examines the last windowSize steps and returns true if
// any tool-call signature appears more than maxRepeats times.
func hasRepeatedToolCalls(steps []fantasy.StepResult, windowSize, maxRepeats int) bool {
	...
}
```

它在 `agent.go` 中被掛進 `StopWhen`[^agentgo]：

```go
StopWhen: []fantasy.StopCondition{
	func(_ []fantasy.StepResult) bool {
		// ... 上下文視窗自動摘要判斷 ...
	},
	func(steps []fantasy.StepResult) bool {
		return hasRepeatedToolCalls(steps, loopDetectionWindowSize, loopDetectionMaxRepeats)
	},
},
```

**限制**：這個機制只抓「完全相同的工具呼叫簽章」。若子代理每次都用不同關鍵字搜尋、抓不同網址（在解決「不可能的任務」時很常見），簽章就不同，**不會觸發**，子代理仍可持續執行。

## 四、相關的 GitHub 議題（Issue）

社群已多次反映此問題，但**目前尚無針對 `agentic_fetch` 或子代理的逾時功能**：

- **Issue #2066「Subagents go on forever and cannot be stopped」**（作者 taoeffect，2026-01-30 提出，**至今仍 open**）[^i2066]：使用者描述子代理在簡單任務中執行了 50 次以上工具呼叫，即使被指示「第 20 次呼叫前結束」也照樣無視。有留言者提出具體方案：「**Hard-enforced per-task timeout**，可在 config 設定並允許 per-task 覆寫；到期即終止子代理 session、把部分輸出與結束原因回傳父代理」，以及「**以硬性上限取代提示指令的步驟預算**，例如 `maxSteps: 50`」。**此提案未獲維護者回應。**

- **Issue #3582「Agentic Fetch gets cancelled wit no output if sourcegraph times out」**（作者 plutonium-239，2026-08-18 提出，**open**）[^i3582]：當 Sourcegraph 工具因網路問題逾時，整個 `agentic_fetch` 會被取消且沒有輸出。作者期望「工具單純報錯、讓子代理能繼續其他工具呼叫」。對應的 PR #3586「fix(tools): make network failures recoverable tool errors」**至今仍為 open、尚未合併**[^pr3586]。此 PR 將 `sourcegraph`、`fetch`、`download` 的網路失敗從「硬性 Go error」改為「可復原的工具錯誤」，但這**是錯誤處理修正，不是逾時功能**。

- **Issue #3658**（2026-08-27，closed）[^i3658]：使用者反映硬編碼逾時造成本地模型失敗。維護者 andreynering 回覆：「我們或許可以加一個 config 來改逾時或完全停用它。要開 PR 嗎？」最後由 **PR #3677（2026-09-04 合併）**實作——但那是**全域的 `request-timeout`**，並非子代理逾時。

- **Issue #2758**（open）[^i2758]：`agentic_fetch`／代理工具有時會對根目錄 `path=/` 執行 grep，*「顯然永遠不會在合理時間內結束」*。這與使用者擔心的「子代理做不可能的任務」高度相關，同樣尚無逾時解法。

- **Issue #3213**（open）[^i3213]：請求為「測試執行」增加可設定的預設逾時、更可靠的 Esc 取消、以及 `/stop` 指令——反映社群對「可中止長時間執行」的普遍需求。

## 五、目前實務上可行的緩解方式

既然沒有專屬逾時，使用者在 Crush 中只能間接緩解：

1. **降低 `request-timeout`**：讓子代理每次模型請求更快失敗（但無法限制總時間）。
2. **適時手動取消**：以 Esc／取消指令中止整個回合（會連帶中止子代理）。
3. **改用 `fetch` 工具並傳入 `timeout`**：若只需抓單一網址，避免使用 `agentic_fetch`，可取得每次呼叫 ≤ 120 秒的逾時。
4. **精簡 prompt**：明確限制子代理的搜尋範圍，降低其反覆搜尋的機率。

## 結論

**Crush 目前不支援針對 `agentic_fetch` 的逾時。** 使用者想用逾時「防止抓取子代理解決不可能的任務」的期待，在現行版本（v0.98.1）尚無法直接實現；唯一可設定的是全域 LLM 請求逾時 `request-timeout`（預設實作值 2 分鐘，0 可停用），而子代理的整體執行時間、步驟數上限則完全沒有硬性限制。社群在 Issue #2066 中提出的「per-task timeout + `maxSteps` 硬性上限」方案至今未被採納。

## 參考資料

[^stars]: GitHub. (2026). *charmbracelet/crush*. Retrieved 2026-10-11, from https://api.github.com/repos/charmbracelet/crush
[^fetchdoc]: GitHub. (n.d.). *internal/agent/templates/agentic_fetch.md*. Retrieved 2026-10-11, from https://raw.githubusercontent.com/charmbracelet/crush/main/internal/agent/templates/agentic_fetch.md
[^types]: GitHub. (n.d.). *internal/agent/tools/fetch_types.go*. Retrieved 2026-10-11, from https://raw.githubusercontent.com/charmbracelet/crush/main/internal/agent/tools/fetch_types.go
[^agenticfetch]: GitHub. (n.d.). *internal/agent/agentic_fetch_tool.go*. Retrieved 2026-10-11, from https://raw.githubusercontent.com/charmbracelet/crush/main/internal/agent/agentic_fetch_tool.go
[^coordinator]: GitHub. (n.d.). *internal/agent/coordinator.go*. Retrieved 2026-10-11, from https://raw.githubusercontent.com/charmbracelet/crush/main/internal/agent/coordinator.go
[^webfetch]: GitHub. (n.d.). *internal/agent/tools/web_fetch.go*. Retrieved 2026-10-11, from https://raw.githubusercontent.com/charmbracelet/crush/main/internal/agent/tools/web_fetch.go
[^websearch]: GitHub. (n.d.). *internal/agent/tools/web_search.go*. Retrieved 2026-10-11, from https://raw.githubusercontent.com/charmbracelet/crush/main/internal/agent/tools/web_search.go
[^sourcegraph]: GitHub. (n.d.). *internal/agent/tools/sourcegraph.go*. Retrieved 2026-10-11, from https://raw.githubusercontent.com/charmbracelet/crush/main/internal/agent/tools/sourcegraph.go
[^fetchcode]: GitHub. (n.d.). *internal/agent/tools/fetch.go*. Retrieved 2026-10-11, from https://raw.githubusercontent.com/charmbracelet/crush/main/internal/agent/tools/fetch.go
[^configdocs]: GitHub. (n.d.). *docs/config/README.md*. Retrieved 2026-10-11, from https://raw.githubusercontent.com/charmbracelet/crush/main/docs/config/README.md
[^configgo]: GitHub. (n.d.). *internal/config/config.go*. Retrieved 2026-10-11, from https://raw.githubusercontent.com/charmbracelet/crush/main/internal/config/config.go
[^reqtimeout]: GitHub. (n.d.). *internal/agent/request_timeout.go*. Retrieved 2026-10-11, from https://raw.githubusercontent.com/charmbracelet/crush/main/internal/agent/request_timeout.go
[^loopdet]: GitHub. (n.d.). *internal/agent/loop_detection.go*. Retrieved 2026-10-11, from https://raw.githubusercontent.com/charmbracelet/crush/main/internal/agent/loop_detection.go
[^agentgo]: GitHub. (n.d.). *internal/agent/agent.go*. Retrieved 2026-10-11, from https://raw.githubusercontent.com/charmbracelet/crush/main/internal/agent/agent.go
[^i2066]: taoeffect. (2026, January 30). *Subagents go on forever and cannot be stopped* [Issue #2066]. GitHub. Retrieved 2026-10-11, from https://github.com/charmbracelet/crush/issues/2066
[^i3582]: plutonium-239. (2026, August 18). *Agentic Fetch gets cancelled wit no output if sourcegraph times out* [Issue #3582]. GitHub. Retrieved 2026-10-11, from https://github.com/charmbracelet/crush/issues/3582
[^pr3586]: marwan562. (2026, August 18). *fix(tools): make network failures recoverable tool errors* [Pull request #3586]. GitHub. Retrieved 2026-10-11, from https://github.com/charmbracelet/crush/pull/3586
[^i3658]: GitHub. (2026, August 27). *Commit 0f403fe introduced a hardcoded timeout* [Issue #3658]. GitHub. Retrieved 2026-10-11, from https://github.com/charmbracelet/crush/issues/3658
[^i2758]: GitHub. (2026, April 30). *Prevent agent and agentic_fetch tools from grepping the entire drive* [Issue #2758]. GitHub. Retrieved 2026-10-11, from https://github.com/charmbracelet/crush/issues/2758
[^i3213]: GitHub. (2026, June 26). *Set default timeouts for test execution and improve process cancellation* [Issue #3213]. GitHub. Retrieved 2026-10-11, from https://github.com/charmbracelet/crush/issues/3213
