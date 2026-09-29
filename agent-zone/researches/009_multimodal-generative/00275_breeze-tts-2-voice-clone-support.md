# Breeze-TTS-2 語音複製（Voice Clone）功能研究報告

## 核心結論

**是，Breeze-TTS-2 支援語音複製（Voice Clone）。** 語音複製是其四大核心語音功能之一，並已於 Hugging Face 公開模型權重與推理程式碼。

## 功能概述

Breeze-TTS-2 提供四種語音生成模式[^hf-model]：

| 模式 | 功能說明 | 所需輸入 |
|------|---------|---------|
| **Voice Clone（語音複製）** | 從參考音訊複製說話者的音色、節奏、情緒與風格 | 參考音訊 + 參考音訊的逐字逐句文字稿 |
| **Voice Design（語音設計）** | 以自然語言描述創造全新聲音，不需要參考音訊 | 文字描述（如「一個溫暖、思緒清晰的年輕女性」） |
| **Voice Direction（語音導向）** | 複製聲音的同時，用自然語言指令調整語氣、情緒、語速 | 參考音訊 + 參考文字稿 + 風格指令 |
| **Vocal Events（人聲事件）** | 在文字中以括號嵌入笑、嘆氣等非語言聲音 | 文字中嵌入 `(laugh)`、`(sigh)` 等標記 |

語音複製之外的兩種模式（Voice Design 與 Voice Direction）進一步擴展了語音合成的靈活性，特別是不須任何參考音訊即可憑空生成聲音的 Voice Design，在開源 TTS 中較為罕見。

## 語音複製的技術實作

### 運作流程

1. 使用者提供一段參考音訊（wav）及其「逐字逐句的準確文字稿」
2. 參考音訊經由 Audio Codec 編碼為離散的音訊 token
3. 參考文字稿經由 Text Encoder 編碼為文字 token
4. 兩者以特殊 token（`<ins_bos>`、`<ins_eos>`、`<|AUDIO|>`、`<|audio_eos|>`）組成多模態 prompt 序列
5. Autoregressive Backbone 模型在此條件下生成新的音訊 token
6. Depth Decoder 將 backbone 輸出轉換為多碼本（multi-codebook）token
7. Audio Codec 解碼為最終的 24kHz PCM 波形[^deepwiki-arch]

### CLI 使用範例

```bash
python infer.py ../breeze-tts-2 \
  --ref-audio reference_en.wav \
  --ref-text "This is the exact transcript of the English reference audio." \
  --text "(sigh) It is good to hear your voice again after all this time." \
  --output outputs/voice_clone_en.wav
```

**重要限制**：若在 Voice Clone 指令中加入 `--instruction`，模式會自動切換為 **Voice Direction**[^hf-model]。

### 參考音訊要求

參考音訊必須是「乾淨的語音」（clean speech with minimal background noise），且必須提供「完全對應」的文字稿。這是因為模型需要從「說了什麼」（what）與「如何說的」（how）中分離出可遷移的語音特徵。

## 模型架構

Breeze-TTS-2 的整體架構約 3.5B 參數，由四個元件組成[^deepwiki-arch][^deepwiki-config]：

1. **Text Encoder**：基於 `t5gemma2` 架構的自訂編碼器，支援 Flash Attention 2，詞彙量 262,208
2. **Autoregressive Backbone**：28 層 Transformer，hidden size 2,048，GQA（16 query heads / 8 KV heads），支援原生 Breeze 骨架與外部 LLM 適配器（Qwen3、Llama3）
3. **Depth Decoder**：32 codebooks，vocab size 2,051，將 backbone 輸出轉為多碼本音訊 token
4. **Audio Codec**：基於 Qwen3-TTS 的音訊 tokenizer（Apache 2.0），將 token 解碼為 24kHz PCM

## 效能

- **Time to First Audio (TTFA)**：H100 上使用 fast path <40ms
- **RTF**：0.32（約 3.1 倍即時生成）
- **GPU 記憶體**：eager 模式 ~7.7 GiB（最低 12 GB GPU），fast path ~14.4 GiB（最低 24 GB GPU）[^hf-model]

## 授權條款

- **推理程式碼**：Apache 2.0
- **模型權重**：僅限研究與非商業用途。商業使用需取得 RESONIA, INC. 書面授權[^hf-model]

## 參考來源列表

[^hf-model]: BreezeBlue. (2026). Breeze-TTS-2. Hugging Face. Retrieved 2026-09-27, from https://huggingface.co/BreezeBlue/Breeze-TTS-2

[^deepwiki-arch]: DeepWiki. (2026). Breeze TTS Model Architecture Overview. Retrieved 2026-09-27, from https://deepwiki.com/breezeblue-ai/breeze-tts/4-model-architecture

[^deepwiki-config]: DeepWiki. (2026). Breeze Model and Configs. Retrieved 2026-09-27, from https://deepwiki.com/breezeblue-ai/breeze-tts/4.1-breeze-model-and-configs

[^blog]: BreezeBlue. (2026-08-07). Introducing Breeze TTS 2. Retrieved 2026-09-27, from https://breezeblue.ai/breeze-tts-2

[^github]: BreezeBlue. (2026). Breeze TTS. GitHub. Retrieved 2026-09-27, from https://github.com/breezeblue-ai/breeze-tts