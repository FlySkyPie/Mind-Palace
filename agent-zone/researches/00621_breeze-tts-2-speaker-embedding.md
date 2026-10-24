# Breeze TTS 2 是否能像 Qwen3-TTS 一樣把語音轉成「語者嵌入向量」而非直接複製

## 摘要

結論先說：**不能。** Qwen3-TTS 的 Base 系列模型在架構中內建了一個獨立的語者編碼器(speaker encoder)，可以把一段參考語音壓縮成固定維度的「語者嵌入向量」(speaker embedding，又稱 x-vector)，之後只需帶著這個向量就能合成語音，不必再保留原始音檔。相對地，Breeze TTS 2 的模型架構中**完全不存在任何語者編碼器或語者嵌入模組**，它的語音複製(voice clone)是採用「以情境學習(in-context learning)方式，把參考音訊的離散編碼器代碼(codec token)直接串進提示序列」的做法，因此無法輸出或重用一個可攜的語者向量。

## 背景：兩種「語音複製」在技術上的差異

在討論哪個模型能做到什麼之前，需要先釐清兩種常被混為一談的做法：

| 做法 | 機制 | 是否能產出可攜的向量 |
| --- | --- | --- |
| 情境式複製(In-context / codec-token conditioning) | 把參考音訊先轉成離散音訊代碼(codec tokens)，與目標文字一起餵進自迴歸模型，由模型「看著」參考語音繼續生成 | 否。語者資訊隱含在整段代碼序列中，無法壓縮成單一向量 |
| 語者嵌入複製(Speaker embedding / x-vector) | 用一個獨立的語者編碼器把參考音訊壓成固定維度向量，推論時只需帶入該向量 | 是。向量可存檔、可重複使用、可做算術運算 |

Qwen3-TTS 兩者兼具；Breeze TTS 2 只有前者。

## Qwen3-TTS 為何能做到：架構證據

### 1. 官方設定檔中明載語者編碼器

`Qwen/Qwen3-TTS-12Hz-1.7B-Base` 的 `config.json` 內含一個獨立的 `speaker_encoder_config` 區塊，其中 `enc_dim` 為 2048、`sample_rate` 為 24000[^cfg]。這代表語者編碼器是模型的正式組成部分，而非外部工具。

### 2. 官方推論介面支援「純向量模式」

Qwen3-TTS 官方說明文件指出，在 `generate_voice_clone` 中若設定 `x_vector_only_mode=True`，模型就只使用語者嵌入向量，此時不需要提供參考文字稿(`ref_text`)[^readme]。此外，`create_voice_clone_prompt` 可先建立一次可重複使用的複製提示(prompt)，避免每次生成都重新計算參考特徵[^readme]。這正是「先轉成向量、再重複使用」的工作流程。

### 3. 社群已把它拆成獨立向量抽取模型

社群模型 `marksverdhei/Qwen3-Voice-Embedding-12Hz-1.7B` 直接從 `Qwen/Qwen3-TTS-12Hz-1.7B-Base` 抽出其中的 ECAPA-TDNN 語者編碼器，可單獨輸出 **2048 維**的 x-vector，並支援存成 SafeTensors 檔案、之後只憑向量驅動 TTS 生成[^embed]。該模型卡亦註明：只有 `*-Base` 變體含有語者編碼器，`*-CustomVoice` 與 `*-VoiceDesign` 變體不支援語者嵌入[^embed]。

### 4. 可搭配 vLLM-Omni 服務化 API 直接傳入向量

當以 vLLM-Omni 部署 Qwen3-TTS 時，請求中可直接帶入預先算好的 `speaker_embedding` 欄位(2048 個浮點數的扁平陣列)，完全不需要參考音檔[^embed]。

## Breeze TTS 2 為何做不到：架構證據

### 1. 模型設定檔中沒有語者編碼器

Breeze TTS 2 的 `config.json` 所列出的子模組僅有：文字編碼器(`text_encoder_config`，T5Gemma2TextEncoder)、自迴歸主幹(`backbone_config`，Qwen3-1B)、深度解碼器(`depth_decoder_config`)與音訊編碼解碼器(`codec_config`)[^bcfg]。全檔**沒有** `speaker_encoder`、`x_vector`、`ecapa` 或任何等價欄位。對照前面 Qwen3-TTS 的 `speaker_encoder_config`，這個缺席是決定性的差異。

### 2. 參考音訊只被轉成離散代碼，而非向量

Breeze 官方的推論程式碼在載入參考音訊時，走的是 `encode_prompt_audio` 函式：它讀入波形後呼叫 `audio_tokenizer.encode(...)`，取出的是一組 2 維的離散音訊代碼(`audio_codes`)，型別為 int16，作為後續提示的一部分[^audio]。整條路徑上沒有任何「輸出固定維度向量」的步驟。

### 3. 複製流程是「文字－音訊－文字」的情境式拼接

Breeze 的提示模板 `ref_clone_tata` 會依序拼出三段：參考文字稿 → 參考音訊(佔位符 `<|AUDIO|>` 序列) → 目標文字[^tmpl]。這證實其複製機制是把參考音訊的 codec token 當作情境直接串入序列，由主幹模型看著它生成，而不是抽取一個語者向量。

### 4. 官方文件與 API 皆未提供向量欄位

Breeze TTS 2 的官方說明僅列出三種模式：語音複製(需要參考音訊與其逐字稿，兩者必須成對提供)、語音設計(無需參考音訊)、語音導向(複製並以指令控制風格)[^readme2]。命令列介面只輸出 WAV，串流 API `/v1/audio/speech` 只回傳音訊 PCM，兩者都沒有任何「抽取嵌入」或「帶入嵌入」的參數[^readme2]。官方詞彙中也不存在「speaker embedding」這個概念。

## 兩者對照總表

| 能力 | Qwen3-TTS(`*-Base` 開源權重) | Breeze TTS 2 |
| --- | --- | --- |
| 從參考音訊複製語音 | 支援(3 秒即可)[^readme] | 支援(需參考音訊+逐字稿)[^readme2] |
| 內建語者編碼器 | 有(ECAPA-TDNN)[^cfg][^embed] | 無[^bcfg] |
| 產出可攜的語者嵌入向量 | 可(2048 維 / 0.6B 為 1024 維)[^embed] | 不可 |
| 只用向量、不需原始音檔即可合成 | 可(`x_vector_only_mode=True`)[^readme] | 不可 |
| 向量可存檔並重複使用 | 可(SafeTensors / JSON)[^embed] | 不適用 |
| 複製機制 | 語者嵌入 + 情境式(ICL)雙模式[^readme] | 僅情境式(codec token 串接)[^tmpl] |

## 若你的目標是「把聲音存成向量」

如果你的使用情境需要「一段聲音 → 一個固定長度向量 → 之後可重複用、可存資料庫、可做向量插值」，那麼：

1. **不要選 Breeze TTS 2**，因為它的架構設計上就不會產生這種向量。
2. **選 Qwen3-TTS 的 `*-Base` 開源模型**，可直接使用社群抽取出的 ECAPA-TDNN 編碼器取得 2048 維(1.7B)或 1024 維(0.6B)的語者向量，並用 SafeTensors 存放[^embed]。
3. **注意限制**：0.6B 與 1.7B 產生的向量維度不同，兩者不可互換；且只有 `*-Base` 變體支援語者嵌入，`*-CustomVoice`、`*-VoiceDesign` 不行[^embed]。

## 來源反思

- **時效性**：本文所依據的官方設定檔與程式碼皆為研究當日直接抓取，Breeze TTS 2 於 2026-08-25 才發布[^readme2]，屬相當新的專案，未來版本可能新增語者編碼器，屆時結論需重新確認。
- **權威性**：關於 Qwen3-TTS 語者編碼器的關鍵證據來自官方 `config.json` 與官方說明文件；社群抽取模型(`marksverdhei/...`)雖非官方，但其模型卡明確標示來源並提供可重現的架構描述，可信度可接受。
- **佐證**：Breeze 的「無語者編碼器」結論同時由三條獨立證據支持——設定檔、音訊載入程式碼、提示模板——彼此一致，因此判斷相當穩固。
- **偏誤**：本主題的兩造當事產品文件皆為各自廠商撰寫，可能有宣傳取向；但本文引用的均為技術性設定與原始碼，而非行銷文案，偏誤風險較低。

[^cfg]: Qwen. (n.d.). *config.json — Qwen/Qwen3-TTS-12Hz-1.7B-Base*. Hugging Face. Retrieved 2026-10-23, from https://huggingface.co/Qwen/Qwen3-TTS-12Hz-1.7B-Base/raw/main/config.json

[^readme]: Qwen. (2026). *Qwen3-TTS — Voice Clone, x_vector_only_mode and create_voice_clone_prompt*. GitHub. Retrieved 2026-10-23, from https://github.com/QwenLM/Qwen3-TTS

[^embed]: marksverdhei. (n.d.). *Qwen3-Voice-Embedding-12Hz-1.7B — ECAPA-TDNN Voice Encoder*. Hugging Face. Retrieved 2026-10-23, from https://huggingface.co/marksverdhei/Qwen3-Voice-Embedding-12Hz-1.7B

[^bcfg]: BreezeBlue. (2026). *config.json — BreezeBlue/Breeze-TTS-2*. Hugging Face. Retrieved 2026-10-23, from https://huggingface.co/BreezeBlue/Breeze-TTS-2/raw/main/config.json

[^audio]: BreezeBlue. (2026). *breeze_infer/audio.py — encode_prompt_audio*. GitHub. Retrieved 2026-10-23, from https://github.com/breezeblue-ai/breeze-tts

[^tmpl]: BreezeBlue. (2026). *breeze_infer/templates.py — ref_clone_tata template*. GitHub. Retrieved 2026-10-23, from https://github.com/breezeblue-ai/breeze-tts

[^readme2]: BreezeBlue. (2026, August 25). *Breeze TTS 2 — Voice Clone, Voice Design, Voice Direction*. GitHub. Retrieved 2026-10-23, from https://github.com/breezeblue-ai/breeze-tts
