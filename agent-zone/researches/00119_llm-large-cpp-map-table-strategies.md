# LLM 處理大型 C++ Map Table 的策略

## 問題概述

LLM 在處理大型 C++ map table（如靜態查找表、enum 對應表、配置映射表）時表現不佳，主要原因有二：

1. **上下文污染**：大量重複性資料佔用 token 配額，稀釋 LLM 對關鍵邏輯的注意力
2. **結構性盲點**：LLM 對大型表格的結構理解有限，即使 GPT-4 在表格結構任務上也僅有 ~65% 的整體準確率[^suctable]

這份報告整理目前學術界與業界的最佳解決方案。

---

## 1. 根本原則：資料與邏輯分離

最核心的解決策略是**不讓 LLM 直接處理大型原始資料**，而是讓 LLM 專注於撰寫邏輯，資料則由外部腳本處理。

### 腳本式程式碼生成（Script-Based Code Generation）

最實證有效的方法是使用 Python 腳本從結構化資料（CSV、JSON、YAML）讀取數據，透過模板引擎（如 Jinja2）生成 C++ 程式碼[^jsonschemacodegen][^jinja2codegen]。

```
資料來源 (CSV/JSON/YAML)
      ↓
Python 腳本 (讀取 + 迭代)
      ↓
Jinja2 模板 (C++ 程式碼樣板)
      ↓
生成 .h / .cpp 檔案
```

這種模式的優點：
- LLM 只需撰寫**模板邏輯**（資料迭代方式），不需要塞入數千行原始資料
- 資料修改時只需更新 CSV/JSON 檔案，不需要重新生成程式碼
- 模板輸出是確定性的（deterministic），不會有 LLM 常見的疏忽或遺漏

現有工具：
- **json-schema-codegen**[^jsonschemacodegen]：從 JSON Schema 生成 C++ 型別定義（含 enum、struct、序列化）
- **csnake**[^csnake]：完整的 Python API，可程式化生成 C 語言程式碼
- **code-generation**（PyPI）[^codegenpypi]：提供 `CppFile` 類別，可用 OOP 方式組合 C++ 程式碼
- **Fips Code Generation**[^fipscodegen]：框架內呼叫 Python 腳本生成 C/C++ 檔案

### 混合架構（Hybrid Approach）

LLM 只負責生成**演算法邏輯**，大型靜態資料存在獨立的 JSON/YAML/CSV 檔案中，執行時期由應用程式自行載入[^langchain][^llamaindex]。這種模式不僅規避了 LLM 的資料處理瓶頸，也讓 C++ 程式本身更模組化。

---

## 2. 當 LLM 必須接觸表格資料時的優化策略

### 2.1 最佳輸入格式：HTML

微軟「Table Meets LLM」基準測試（Sui et al., WSDM 2024）是目前最全面的表格格式比較研究[^suctable]，結果顯示 **HTML 在所有格式中表現最佳**：

| 格式 | 平均準確率（7 項任務） |
|------|----------------------|
| **HTML** | **65.43%** |
| XML | ~59-60% |
| JSON | ~58-59% |
| Markdown | ~57-58% |
| CSV | ~58% |

研究指出：「分隔符號格式（如 CSV、TSV）的表現比 HTML 低了 6.76 個百分點。」[^suctable] 原因是 LLM 的訓練資料大量來自網頁（HTML）。

### 2.2 TOON 格式：兼顧 Token 效率與準確率

TOON（Token-Oriented Object Notation）是一種新型資料格式[^toon]，結合了 YAML 的縮排結構與 CSV 的表格形式：

- **Token 量比 JSON 少 42.6%**
- **檢索準確率 72.2%**（JSON 為 71.4%）
- 使用 `[N]` 明確標記行數，防止截斷導致資料遺漏
- 已通過 5,856 次 LLM 呼叫的基準測試

TOON 的設計特別適合 C++ map table 場景，因為其結構化行數標記讓 LLM 能「意識到」表格的完整規模，不會因為只看見部分資料就產生錯誤假設。

### 2.3 語意取樣（Semantic Sampling）

TAP4LLM 框架（Sui et al., EMNLP 2024）提出了表格處理的最佳實踐[^tap4llm]：

當表格過大時，**不應該直接截斷**，而應該進行語意取樣：

1. **嵌入向量取樣**：將資料列/欄映射到向量空間，選取與查詢最相關的 top-k 項目
2. **欄位接地**：同時選取相關的列**與**欄，效果最佳（提升約 5%）
3. **聚心取樣**：使用 K-Means 分群後取每群中心點，保留資料多樣性
4. **混合取樣**：結合語意相關性（查詢相似度）與多樣性（群中心點）

關鍵發現：**即使擁有 32K 上下文視窗，直接餵入整張表格的表現仍然不如策略性取樣 + 增強資訊**。

### 2.4 Token 配額的最佳比例

TAP4LLM 發現了關鍵的 **token 配額法則**：表格內容與增強資訊的 token 分配比例應接近 **5:5 或 4:6**[^tap4llm]。也就是說，約一半的 token 預算應該用在：

- 表格維度標註（幾行幾列）
- 欄位名稱與型別說明
- 統計摘要（min、max、分佈）
- 領域術語解釋

而不是全部塞入原始資料。研究明確指出：「當過多 token 分配給增強資訊時，會出現報酬遞減現象。」

### 2.5 提示詞結構（最佳順序）

根據上述研究，最有效的提示詞結構為[^suctable][^tap4llm]：

1. **角色提示**：如「你是一位 C++ 程式碼生成專家」
2. **任務描述**：外部資訊（問題/查詢）應放在**表格之前**，如此可提升 6.81% 準確率
3. **結構性元資料**：表格大小、欄位名稱、型別
4. **表格資料**：使用 HTML 或 TOON 格式
5. **單次示範（One-shot）**：移除範例會導致 30.38% 準確率下降
6. **自增強提示（Self-Augmented Prompting）**：要求 LLM 先分析關鍵數值與範圍
7. **最終輸出要求**

### 2.6 關鍵的兩階段自增強提示

這是研究中回報效果最好的提示策略[^suctable]：

- **第一階段**：要求 LLM 分析表格，生成中間結構知識。三種指令類型的效果：
  - 關鍵數值與範圍標識：**+3.26%**（最佳）
  - 結構資訊描述：+2.11%
  - 自我格式解釋：+1.10%
- **第二階段**：將 LLM 自己生成的分析結果重新餵入，要求進行程式碼生成

這種方法的效果優於人工編寫的格式說明，因為「自增強提示能獨立學習模式，並生成更全面有用的提示。」[^suctable]

---

## 3. 文法約束生成（Grammar-Constrained Generation）

對於生成 C++ 程式碼的場景，文法約束可以直接確保 LLM 輸出符合語法規範[^syncode][^guidance][^outlines]：

| 工具 | 說明 | 適用場景 |
|------|------|---------|
| **SynCode**[^syncode] | 語法引導生成，支援 EBNF 自訂文法，比無約束生成快 10-20% | 最適合 C++ 程式碼生成，可先定義 C++ 子集的 CFG |
| **Guidance**[^guidance] | Microsoft 開發，支援 `select()`、`gen(regex=...)`、token 快轉 | 適合組合可複用的語法函數 |
| **Outlines**[^outlines] | 支援 Pydantic 模型、CFG、JSON Schema 約束 | 適合生成結構化資料後再填入模板 |
| **SGLang**[^sglang] | 高效能推論框架，內建結構化輸出 | 適合高吞吐量場景 |

然而，對於大型靜態資料表，**文法約束不能取代腳本生成**——文法約束解決的是語法正確性問題，而不是資料量問題。

---

## 4. RAG 模式用於查表

對於需要在生成程式碼時「查詢」大型對應表的場景，RAG（Retrieval-Augmented Generation）模式是適當的架構選擇[^langchain][^llamaindex]：

1. 將 C++ map table 的資料索引到向量資料庫
2. LLM 生成程式碼時，先從向量資料庫檢索相關的幾筆對應關係
3. 只將檢索到的部分注入 LLM 上下文

這種方法本質上實現了 TAP4LLM 的語意取樣概念，但以更通用的 RAG 架構實現。

---

## 5. 綜合建議

### 針對不同場景的解決方案選擇

| 場景 | 推薦方案 | 理由 |
|------|---------|------|
| 靜態對應表（幾百筆以上） | **Python + Jinja2 腳本生成** | 完全規避 LLM 限制，確定性輸出 |
| 中型對應表（幾十筆） | **TOON 格式 + 自增強提示** | Token 效率最佳，結構清晰 |
| 動態查表（執行時期決定 key） | **RAG + 向量檢索** | 只在需要時檢索相關筆數 |
| 需要 LLM 推理對應邏輯 | **HTML 格式 + 兩階段提示** | 最高準確率格式 + 自增強推理 |

### 一般原則

1. **盡量讓 LLM 只寫邏輯**：使用外部腳本處理大型靜態資料
2. **如果必須給 LLM 看資料**：使用 TOON 或 HTML 格式，不要用 CSV
3. **保留 40-50% token 給元資料**：表格摘要比原始資料更有價值
4. **永遠給出單次示範（One-shot）**：30% 的差異
5. **使用兩階段提示**：先分析，再生成
6. **考慮使用 TOON**[^toon]：其 `[N]` 行數標記機制能讓 LLM 意識到資料的完整規模，降低遺漏風險

### 關於你提到的「需要額外撰寫 Python 腳本」

這其實是目前最好的解法。如 TAP4LLM 研究所指出，連 GPT-4 處理表格任務都有 ~35% 的錯誤率。與其花時間除錯 LLM 的輸出，不如將這個步驟**正式化為開發流程的一部分**：建立一個標準的 Python 腳本模板，從 YAML/CSV 生成 C++ map table 程式碼，用 CI/CD 確保資料同步正確。這不是 LLM 的限制，而是對工具的合理分工運用。

---

[^suctable]: Sui, Y., et al. (2024). Table Meets LLM: Can Large Language Models Understand Structured Table Data? A Benchmark and Empirical Study. *WSDM 2024*. Retrieved 2026-09-24, from https://arxiv.org/abs/2305.13062

[^tap4llm]: Sui, Y., et al. (2024). TAP4LLM: Table Provider on Sampling, Augmenting, and Packing Tabular Data for Large Language Models. *EMNLP 2024*. Retrieved 2026-09-24, from https://arxiv.org/abs/2312.09039

[^toon]: TOON Format. (2025). Token-Oriented Object Notation. Retrieved 2026-09-24, from https://github.com/toon-format/toon

[^jsonschemacodegen]: pearmaster. (n.d.). json-schema-codegen. Retrieved 2026-09-24, from https://github.com/pearmaster/json-schema-codegen

[^jinja2codegen]: MarkV Tech Blog. (2024). Code Generation in Python with Jinja2. Retrieved 2026-09-24, from https://markvtechblog.wordpress.com/2024/04/28/code-generation-in-python-with-jinja2/

[^csnake]: csnake. (n.d.). Python Package for C Code Generation. Retrieved 2026-09-24, from https://andrejr.gitlab.io/csnake/

[^codegenpypi]: code-generation. (n.d.). PyPI. Retrieved 2026-09-24, from https://pypi.org/project/code-generation/

[^fipscodegen]: Fips Code Generation. (n.d.). Retrieved 2026-09-24, from https://floooh.github.io/fips/docs/codegen/

[^langchain]: LangChain. (n.d.). Structured Outputs and Tool Calling. Retrieved 2026-09-24, from https://github.com/langchain-ai/langchain

[^llamaindex]: LlamaIndex. (n.d.). Structured Output Modules. Retrieved 2026-09-24, from https://github.com/run-llama/llama_index

[^syncode]: Ugare, S., et al. (2024). SynCode: LLM Generation with Grammar Augmentation. *arXiv:2403.01632*. Retrieved 2026-09-24, from https://github.com/uiuc-focal-lab/syncode

[^guidance]: Guidance AI. (n.d.). Guidance: A Programming Paradigm for Steering LLMs. Retrieved 2026-09-24, from https://github.com/guidance-ai/guidance

[^outlines]: Outlines. (n.d.). Structured Outputs from Any LLM. Retrieved 2026-09-24, from https://github.com/outlines-dev/outlines

[^sglang]: SGLang. (n.d.). Efficient Structured Output Generation. Retrieved 2026-09-24, from https://github.com/sgl-project/sglang