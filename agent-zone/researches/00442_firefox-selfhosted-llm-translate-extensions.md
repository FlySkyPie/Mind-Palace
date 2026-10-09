# Firefox 自架 LLM / OpenAI-Compatiable API 網頁翻譯擴充功能研究

## 概要

本報告調查 Firefox 上可使用自架 LLM（Self-hosted LLM）或 OpenAI-Compatible API 進行網頁翻譯的擴充功能，涵蓋功能、授權、活躍程度及優劣勢比較。

## 結論摘要

目前 Firefox 生態系已有 **至少 5 款活躍維護中** 的擴充功能支援自架 LLM 或 OpenAI-Compatible API 進行全頁翻譯。其中 **Local Browser Translator** 與 **MuxTranslator** 最為突出，前者專為本機 LLM 最佳化，後者功能最全面。

---

## 詳細調查結果

### 1. Local Browser Translator 🏆 *最佳自架 LLM 翻譯*

| 項目 | 內容 |
|------|------|
| AMO 連結 | [addons.mozilla.org/…/local-browser-translator/](https://addons.mozilla.org/en-US/firefox/addon/local-browser-translator/) |
| 版本 | v1.2.0 — 2026-10-01（極活躍維護） |
| 授權 | MIT |

**支援後端：**
- **本機：** llama.cpp、Ollama、vLLM、SGLang、LM Studio、llama-swap
- **雲端 OpenAI-compatible：** OpenAI、OpenRouter、Groq、DeepSeek、Together 及其他 OpenAI-compatible 提供者

**主要特色：**
- **視口優先翻譯（Viewport priority）** — 先翻譯可見內容
- **串流輸出（Streaming）** — 模型產出片段後立即顯示
- **在地翻譯記憶** — 重複文字不重複翻譯（跨頁面選單/標題無成本）
- **容錯修復** — 格式異常輸出會自動修復、縮小重試或保留原文
- **逐站選擇加入** — 安裝後不影響任何網站
- **鍵盤快捷鍵** — 翻譯頁面、還原、翻譯選取、顯示模式切換（取代/雙語/懸浮）
- **無遙測** — 僅與設定端點通訊
- 支援非同步 paywall 後載入內容與 HTML `<details>` 元素

**設定方式：** 指向任意 OpenAI-compatible 端點，預設 `http://localhost:8080/v1`。

**優點：** 本機 LLM 最佳化最佳；串流輸出；翻譯記憶；活躍維護
**缺點：** 未發現 GitHub 儲存庫；較新（尚無大量評分）

---

### 2. MuxTranslator 🏆 *功能最全面*

| 項目 | 內容 |
|------|------|
| AMO 連結 | [addons.mozilla.org/…/muxtranslator/](https://addons.mozilla.org/en-US/firefox/addon/muxtranslator/) |
| GitHub | [github.com/kb5000/muxtranslator](https://github.com/kb5000/muxtranslator) |
| 星數 | **1** ⭐ |
| 版本 | v1.9.1 — 2026-05-06 |
| 授權 | MIT |

**支援後端：**
- **OpenAI-compatible API**（OpenAI、OpenRouter、Groq 等任意端點）
- **Ollama**（本機，不需 API 金鑰）
- Google Translate API
- DeepL API
- LibreTranslate（自架或公用）
- **功能綁定：** 可將不同提供者指派給頁面/選取/手動/PDF 翻譯

**主要特色：**
- **全頁翻譯**（原位取代，保留版面）
- **PDF 翻譯**（內建 PDF 閱讀器疊加層）
- **選取翻譯**（反白 → 浮動徽章 → 行內翻譯）
- **手動翻譯**（在彈出視窗貼上文字）
- **雙語顯示：** 翻譯取代、左右並排、懸浮提示三種模式
- **串流輸出** — 逐詞顯示
- **自動偵測** — 偵測外語頁面時提示
- **視口優先** — 先翻譯可見元素
- **智慧批次處理** — 合併文字節點減少 API 呼叫
- **IndexedDB 快取** — 逐站或全域，跨次瀏覽重用
- **SPA 支援** — 監聽 DOM 變異，支援 React/Vue/無限滾動
- **代幣用量統計** — 追蹤各提供者/工作階段的 prompt 與 completion 代幣
- **詞彙表（Glossary）** — 逐提供者定義固定譯詞
- **頁面上下文** — 在 system prompt 中納入頁面標題與 meta 描述
- **工具呼叫模式（Tool-call）** — 函式呼叫取得更可靠 JSON
- **站台規則** — 逐主機名稱設定行為（永遠翻譯/跳過/詢問）
- **行動版支援** — 響應式設計
- 介面支援英文與簡體中文

**優點：** 功能最全面；PDF 翻譯；SPA 支援；詞彙表；快取；活躍維護
**缺點：** 星數少（新專案）；設定項目較多

---

### 3. Local LLM Translator（Eldoprano）

| 項目 | 內容 |
|------|------|
| AMO 連結 | [addons.mozilla.org/…/local-llm-translator/](https://addons.mozilla.org/en-US/firefox/addon/local-llm-translator/) |
| GitHub | [github.com/Eldoprano/offline-browser-translate](https://github.com/Eldoprano/offline-browser-translate) |
| 星數 | **32** ⭐ |
| 版本 | v1.7.0 — 2026-08-12 |
| 授權 | MIT |

**支援後端：**
- Ollama（預設 `localhost:11434`）
- LM Studio（預設 `localhost:1234`）
- 任意 OpenAI-compatible 伺服器（llama.cpp、vLLM 等）

**主要特色：**
- **100% 離線** — 資料永不離開本機
- 優先翻譯可見內容＋標題
- **翻譯快取**（預設關閉；可選工作階段或持久模式）
- **詞彙表（Glossary）**：TSV 詞典強制一致翻譯專有名詞與 UI 標籤
- **自動翻譯**：可選頁面載入時自動翻譯，支援逐語言與逐站排除
- 多語言支援
- 純 vanilla JS，無建置步驟或依賴

**優點：** 真正隱私（僅本機連線）；詞彙表獨特且強大；簡潔程式碼；自動翻譯
**缺點：** 未提及串流輸出；不支援雲端提供者（刻意本機優先）

---

### 4. Local Translate AI

| 項目 | 內容 |
|------|------|
| AMO 連結 | [addons.mozilla.org/…/local-translate-ai/](https://addons.mozilla.org/en-US/firefox/addon/local-translate-ai/) |
| GitHub | [github.com/roflsunriz/local-translate-ai](https://github.com/roflsunriz/local-translate-ai) |
| 星數 | **4** ⭐ |
| 版本 | v1.8.0 — 2026-09-16 |
| 授權 | MIT |

**支援後端：**
- OpenAI-compatible API（Ollama、llama.cpp、LM Studio、OpenRouter、Cerebras 等）
- Anthropic Messages API
- Google Translate

**主要特色：**
- **三種翻譯模式：** 選取文字、全頁、側欄翻譯
- **串流翻譯：** 全頁翻譯逐段落串流顯示
- 翻譯歷史（可從設定/側欄保存重用）
- 暗色模式
- 鍵盤快捷鍵：`Alt+W` 快速翻譯選取、`Alt+Q` 切換側欄
- 可拖曳彈出視窗
- Google Translate 後備（不需 API 金鑰或 LLM 伺服器）

**設定方式（本機）：** 執行 llama.cpp 伺服器，設定端點為 `http://localhost:3002/v1/chat/completions`，model 使用 `plamo-2-translate-gguf`。

**優點：** 功能豐富；三種翻譯模式；串流；Google Translate 後備；llama.cpp 設定文件完善
**缺點：** 文件主要為日文；星數較少

---

### 5. Translator OpenAI compatible

| 項目 | 內容 |
|------|------|
| AMO 連結 | [addons.mozilla.org/…/translator-openai-compatible/](https://addons.mozilla.org/en-US/firefox/addon/translator-openai-compatible/) |
| 版本 | v1.2.3 — 2026-09-08 |
| 授權 | MIT |
| 尺寸 | 34.99 KB |

**說明：** 「使用自訂 OpenAI-compatible LLM（Ollama、LM Studio）引擎，在頁面上就地翻譯，如同瀏覽器內建翻譯功能。」

**支援後端：** Ollama、LM Studio 等 OpenAI-compatible LLM

**特色：**
- 頁面原位翻譯（類似 Firefox 內建翻譯）
- 使用自訂端點
- 極輕量（35KB）

**優點：** 輕量；簡潔；無資料收集
**缺點：** 無 GitHub 儲存庫；AMO 說明極簡；尚無評分

---

### 6. FO Translator（Agent 反思式翻譯）

| 項目 | 內容 |
|------|------|
| AMO 連結 | [addons.mozilla.org/…/fo-translator/](https://addons.mozilla.org/en-US/firefox/addon/fo-translator/) |
| GitHub | [github.com/snekkenull/FO-Translator](https://github.com/snekkenull/FO-Translator) |
| 星數 | **4** ⭐ |
| 版本 | v0.1.3 — 2024-08-03 |
| 授權 | MPL-2.0 |

**特色：**
- **Agent 式翻譯：** 靈感來自 Andrew Ng 的 translation-agent，使用反思（reflection）流程提升翻譯自然度與細膩度
- 選取文字 → 點擊 FO Translator 圖示 → 得到改善翻譯
- **可自訂提示：** 使用變數（`{source_lang}`、`{target_lang}`、`{translation_1}`、`{reflection}`）
- 隱私優先：不記錄/收集資料，設定本機保存，開源

**優點：** 獨特的 Agent 反思式流程提升品質；可自訂提示；極輕量（50KB）
**缺點：** 兩年未更新；僅選取翻譯（無全頁）；功能有限

---

## 比較總表

| 擴充功能 | 自架 LLM | OpenAI-API | 全頁翻譯 | 串流 | 開源 | 最近更新 |
|----------|:--------:|:----------:|:--------:|:----:|:----:|:--------:|
| **Local Browser Translator** | ✅ | ✅ | ✅ | ✅ | ✅ MIT | 2026-10 |
| **Local LLM Translator** | ✅ | ✅ | ✅ | ❌ | ✅ MIT | 2026-08 |
| **Local Translate AI** | ✅ | ✅ | ✅ | ✅ | ✅ MIT | 2026-09 |
| **MuxTranslator** | ✅ | ✅ | ✅ | ✅ | ✅ MIT | 2026-05 |
| **Translator OpenAI compatible** | ✅ | ✅ | ✅ | ❌ | ✅ MIT | 2026-09 |
| **FO Translator** | ❌ | ✅ | ❌ | ❌ | ✅ MPL-2.0 | 2024-08 |

---

## 推薦

- **🏆 自架 LLM 翻譯最佳：** **Local Browser Translator** — 為本機 LLM 最完善設計，具視口優先、串流、翻譯記憶，且極活躍維護（2026-10-01）。
- **🏆 功能最全面：** **MuxTranslator** — PDF 翻譯、SPA 支援、詞彙表、快取、代幣統計、功能綁定，支援 Ollama、OpenAI-compatible、DeepL、Google Translate、LibreTranslate 多種後端。
- **🏆 純隱私（完全不連雲端）：** **Local LLM Translator**（Eldoprano）— 刻意僅連本機，具詞彙表與自動翻譯。

---

## 重要設定注意事項

1. **本機 LLM 須執行推論伺服器：**
   - **Ollama：** `ollama serve`（預設 `http://localhost:11434`）
   - **llama.cpp：** `./llama-server -m model.gguf --port 3002`（或 8080）
   - **LM Studio：** 從 GUI 啟動伺服器（預設 `http://localhost:1234`）

2. **推薦翻譯模型：**
   - `plamo-2-translate.Q4_K_M.gguf`（日↔英，Local Translate AI 推薦）
   - `TranslateGemma`（Google 翻譯調校模型）
   - `tencent.hunyuan-mt`（騰訊翻譯模型）
   - `qwen3`（通用模型，翻譯表現佳）
   - 透過 Ollama/LM Studio 可選任意模型

3. **API 端點格式：** 多數擴充功能預期指向 `/v1/chat/completions`（例如 `http://localhost:3002/v1/chat/completions` 或 Ollama 的 `http://localhost:11434/v1/chat/completions`）。

4. **API 金鑰：** 本機設定通常不需金鑰（可使用佔位字元如 `test`）；雲端提供者則須實際金鑰。

5. **權限：** 多數擴充功能須要求「存取所有網站資料」以讀取頁面內容進行翻譯。僅本機的擴充功能可能僅需 localhost 存取。

---

## 參考資料

Local Browser Translator. (n.d.). Firefox Add-on. Retrieved 2026-10-09, from https://addons.mozilla.org/en-US/firefox/addon/local-browser-translator/ [^lbt]

Eldoprano. (2026). offline-browser-translate [GitHub repository]. Retrieved 2026-10-09, from https://github.com/Eldoprano/offline-browser-translate [^llt]

roflsunriz. (2026). local-translate-ai [GitHub repository]. Retrieved 2026-10-09, from https://github.com/roflsunriz/local-translate-ai [^lta]

kb5000. (2026). muxtranslator [GitHub repository]. Retrieved 2026-10-09, from https://github.com/kb5000/muxtranslator [^mux]

Translator OpenAI compatible. (n.d.). Firefox Add-on. Retrieved 2026-10-09, from https://addons.mozilla.org/en-US/firefox/addon/translator-openai-compatible/ [^toc]

snekkenull. (2024). FO-Translator [GitHub repository]. Retrieved 2026-10-09, from https://github.com/snekkenull/FO-Translator [^fo]

AI Page Translator. (n.d.). Firefox Add-on. Retrieved 2026-10-09, from https://addons.mozilla.org/en-US/firefox/addon/ai-page-translator/ [^apt]

NextAI Translator. (n.d.). Firefox Add-on. Retrieved 2026-10-09, from https://addons.mozilla.org/en-US/firefox/addon/nextai-translator/ [^nextai]

ollykel. (2026). firefox_llm_translator [GitHub repository]. Retrieved 2026-10-09, from https://github.com/ollykel/firefox_llm_translator [^olly]

[^lbt]: Local Browser Translator. (n.d.). Firefox Add-on. Retrieved 2026-10-09, from https://addons.mozilla.org/en-US/firefox/addon/local-browser-translator/
[^llt]: Eldoprano. (2026). offline-browser-translate [GitHub repository]. Retrieved 2026-10-09, from https://github.com/Eldoprano/offline-browser-translate
[^lta]: roflsunriz. (2026). local-translate-ai [GitHub repository]. Retrieved 2026-10-09, from https://github.com/roflsunriz/local-translate-ai
[^mux]: kb5000. (2026). muxtranslator [GitHub repository]. Retrieved 2026-10-09, from https://github.com/kb5000/muxtranslator
[^toc]: Translator OpenAI compatible. (n.d.). Firefox Add-on. Retrieved 2026-10-09, from https://addons.mozilla.org/en-US/firefox/addon/translator-openai-compatible/
[^fo]: snekkenull. (2024). FO-Translator [GitHub repository]. Retrieved 2026-10-09, from https://github.com/snekkenull/FO-Translator
[^apt]: AI Page Translator. (n.d.). Firefox Add-on. Retrieved 2026-10-09, from https://addons.mozilla.org/en-US/firefox/addon/ai-page-translator/
[^nextai]: NextAI Translator. (n.d.). Firefox Add-on. Retrieved 2026-10-09, from https://addons.mozilla.org/en-US/firefox/addon/nextai-translator/
[^olly]: ollykel. (2026). firefox_llm_translator [GitHub repository]. Retrieved 2026-10-09, from https://github.com/ollykel/firefox_llm_translator