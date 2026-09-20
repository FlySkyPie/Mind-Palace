# BehaviorTree.CPP V3 與 V4 XML 格式差異

## 概述

BehaviorTree.CPP 是一個廣受使用的 C++ 行為樹函式庫。V4 版本在 XML 格式上引入了向後相容但破壞性的變更，主要透過版本標記 `BTCPP_format` 來區分。本文件完整比較 V3 與 V4 的 XML 格式差異。

## 1. 根標籤變更

V3 的根標籤為 `<root main_tree_to_execute="MainTree">`，而 V4 **要求**在 `<root>` 加上 `BTCPP_format="4"` 屬性[^migration]。

```xml
<!-- V3 -->
<root main_tree_to_execute="MainTree">
  ...
</root>

<!-- V4 -->
<root BTCPP_format="4" main_tree_to_execute="MainTree">
  ...
</root>
```

若省略 `BTCPP_format`，解析器將發出警告。`BTCPP_format="3"` 則觸發舊版埠語法解析模式[^v4format]。

## 2. 節點/類別重新命名

| V3 名稱 | V4 名稱 | 影響範圍 |
|---|---|---|
| `SequenceStar` | `SequenceWithMemory` | C++ 及 XML |
| `AsyncActionNode` | `ThreadedAction` | C++ |
| `NodeConfiguration` | `NodeConfig` | C++ |
| `Optional<T>` | `Expected<T>` | C++ |

XML 中最直接的影響是 `<SequenceStar>` 必須改為 `<SequenceWithMemory>`[^migration]。

## 3. SubTree 語意變更

這是 V3→V4 最大的變動之一：

- **V3 的 `<SubTree>` 已被棄用**，取而代之的是原本稱為 `SubTreePlus` 的行為。V4 直接將這個新行為命名為 `<SubTree>`[^migration]。
- **`__shared_blackboard` 屬性棄用**：V3 的 `__shared_blackboard="true"` 在 V4 中改為 `_autoremap="1"`，自動將 SubTree 所有同名的埠與父 blackboard 的鍵進行映射[^convert_script]。
- **埠映射語法擴充**：V4 支援 `{=}` 縮寫（`port="{=}"`），表示使用與埠名稱相同的 blackboard 鍵，減少冗餘寫法[^v460]。

## 4. 新 XML 屬性

### 前置/後置條件屬性（V4 新增）

所有節點都可使用以下屬性，無需額外的條件節點[^pre_post]：

| 屬性 | 說明 |
|---|---|
| `_failureIf` | 當條件式為真時，節點回傳 FAILURE |
| `_successIf` | 當條件式為真時，節點回傳 SUCCESS |
| `_skipIf` | 當條件式為真時，節點回傳 SKIPPED |
| `_while` | 每次 tick 都檢查（前三個只在 IDLE 狀態檢查） |
| `_onSuccess` | 節點成功時執行的腳本 |
| `_onFailure` | 節點失敗時執行的腳本 |
| `_onHalted` | 節點被中斷時執行的腳本 |
| `_post` | 節點完成（不論成敗）時執行的腳本 |

### 其他新屬性

- **`_autoremap`**：用於 `<SubTree>`，取代 V3 的 `__shared_blackboard`[^convert_script]。
- **`_description`**：用於 `<BehaviorTree>`，人類可讀的描述文字[^v4format]。

## 5. 新節點元素

V4 引入了腳本語言及一系列新節點[^scripting]：

- **`<Script>`**：執行內嵌腳本語言（賦值、算術、位元運算、邏輯運算、三元運算子、列舉），永遠回傳 SUCCESS。
- **`<ScriptCondition>`**：評估運算式並回傳 SUCCESS 或 FAILURE。
- **`<Precondition>`**：裝飾節點，每次 tick 檢查條件，支援 `if`/`else` 屬性。
- **`<AsyncSequence>` 與 `<AsyncFallback>`**：在每個同步子節點完成後回傳 RUNNING，使純同步序列也可被中斷[^sequence]。
- **`<ParallelAll>`** (4.3.2)、**`<RunOnce>`** (4.2)、**`<SkipUnlessUpdated>`**、**`<WaitValueUpdate>`**、**`<WasEntryUpdated>`** (4.6)、**`<TryCatch>`** (4.9) 等。

> **注意**：V4.0 中 Sequence/Fallback 曾短暫在每個子節點後回傳 RUNNING，但 **V4.2 把這行為還原回 V3.8 樣式**，並以 `<AsyncSequence>/<AsyncFallback>` 作為選擇性的非同步替代方案[^v420]。

## 6. 已棄用/移除的功能

- **`<SetBlackboard>` 及 `<BlackboardCheckInt>` 等**：被 `<Script>` 取代[^migration]。
  - 舊寫法：`<SetBlackboard output_key="port_A" value="42" />`
  - 新寫法：`<Script code="port_A:=42; port_B:=69" />`
- **`<SubTreePlus>` 標籤**：直接改名為 `<SubTree>`，V3 的舊 `<SubTree>` 被棄用。
- **`__shared_blackboard`**：被 `_autoremap` 取代。
- **保留屬性名稱衝突**：若節點原有的埠名稱與腳本指令保留字（`_successIf`、`_failureIf`、`_skipIf`、`_while`、`_onSuccess`、`_onFailure`、`_onHalted`、`_post`）衝突，轉換腳本會報錯[^convert_script]。

## 7. 新的 NodeStatus：SKIPPED

V4 新增 `SKIPPED` 狀態，當前置條件不滿足時回傳（例如 `_skipIf`）。ControlNode 與 Decorator 需處理這個第三種子節點結果[^migration]。

## 8. 埠系統與型別驗證強化

- V3 中不在 `{...}` 內的值以字串傳遞，**轉換發生在 tick 時**。
- V4 在**樹建立時即進行型別驗證**（`convertFromString` 在解析時就被呼叫），型別不符在執行前就會拋出 `RuntimeError`[^v4format]。
- 埠名稱必須符合 `providedPorts()` 註冊的埠。
- 以底線開頭且不屬於保留清單的屬性（如 `_description`），會流入 `NodeConfig::other_attributes`。

## 9. 結構與驗證強化

V4 在執行樹前先執行 `VerifyXML()`，強制以下規則[^deepwiki]：

- 各節點類別的子節點數量限制（`TryCatch` ≥2、裝飾節點恰好 1、動作/條件節點 0、`<SubTree>` 0）
- 最大巢狀深度 256 層
- 自 V4.8.4 起，節點模型/實例/埠名稱有更嚴格的驗證規則（禁止特殊字元、保留名稱、重複 fullpath）[^v484]

## 10. 遷移工具

官方提供 Python 轉換腳本 `convert_v3_to_v4.py`，位於專案根目錄[^convert_script]：

```bash
python3 convert_v3_to_v4.py -i tree_v3.xml -o tree_v4.xml
```

腳本自動處理：
- 加入 `BTCPP_format="4"`
- 重新命名 `SequenceStar` → `SequenceWithMemory`
- 轉換 V3 SubTree 埠映射、`__shared_blackboard` → `_autoremap`
- 重新命名 `<SubTreePlus>` → `<SubTree>`

腳本不提供 V4→V3 的反向轉換。

## 總結對照表

| 面向 | V3 | V4 |
|---|---|---|
| 根標籤 | `<root main_tree_to_execute="...">` | `<root BTCPP_format="4" ...>` |
| Sequence 變體 | `SequenceStar` | `SequenceWithMemory` |
| 腳本 | `SetBlackboard` / `BlackboardCheck*` | `<Script>` 內嵌語言 |
| SubTree 共享 blackboard | `__shared_blackboard="true"` | `_autoremap="1"` |
| 子樹埠映射縮寫 | 無 | `{=}` |
| 前置/後置條件 | 需組合控制節點 | 節點屬性 `_successIf` 等 |
| NodeStatus | SUCCESS / FAILURE | 新增 SKIPPED |
| 非同步序列 | 無（節點自行決定） | `<AsyncSequence>` / `<AsyncFallback>` |
| 型別檢查 | tick 時 | 樹建立時 |
| 遷移工具 | 無 | `convert_v3_to_v4.py` |

---

[^migration]: BehaviorTree.CPP. (n.d.). Migration from BT.CPP 3.x.  Retrieved 2026-09-20, from https://www.behaviortree.dev/docs/migration/
[^v4format]: BehaviorTree.CPP. (n.d.). XML Format. Retrieved 2026-09-20, from https://www.behaviortree.dev/docs/learn-the-basics/xml_format/
[^pre_post]: BehaviorTree.CPP. (n.d.). Pre/Post Conditions. Retrieved 2026-09-20, from https://www.behaviortree.dev/docs/guides/pre_post_conditions/
[^scripting]: BehaviorTree.CPP. (n.d.). Scripting. Retrieved 2026-09-20, from https://www.behaviortree.dev/docs/guides/scripting/
[^sequence]: BehaviorTree.CPP. (n.d.). SequenceNode. Retrieved 2026-09-20, from https://www.behaviortree.dev/docs/nodes-library/SequenceNode/
[^convert_script]: BehaviorTree.CPP. (n.d.). convert_v3_to_v4.py. Retrieved 2026-09-20, from https://github.com/BehaviorTree/BehaviorTree.CPP/blob/master/convert_v3_to_v4.py
[^v420]: BehaviorTree.CPP. (2023). Release 4.2.0. Retrieved 2026-09-20, from https://github.com/BehaviorTree/BehaviorTree.CPP/releases/tag/4.2.0
[^v460]: BehaviorTree.CPP. (2023). Release 4.6.0. Retrieved 2026-09-20, from https://github.com/BehaviorTree/BehaviorTree.CPP/releases/tag/4.6.0
[^v484]: BehaviorTree.CPP. (2024). Release 4.8.4. Retrieved 2026-09-20, from https://github.com/BehaviorTree/BehaviorTree.CPP/releases/tag/4.8.4
[^deepwiki]: DeepWiki. (n.d.). BehaviorTree.CPP XML Definition and Parsing. Retrieved 2026-09-20, from https://deepwiki.com/BehaviorTree/BehaviorTree.CPP/5-xml-definition-and-parsing