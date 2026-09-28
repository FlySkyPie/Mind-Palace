# Breeze-TTS-2 語音嵌入（Voice Embedding）支援分析

## 摘要

Breeze-TTS-2 是 BreezeBlue 於 2026 年 8 月 25 日發布的開源文字轉語音模型（3.47B 參數），支援語音複製（Voice Clone）、語音設計（Voice Design）、語音導向（Voice Direction）與行內聲音事件（Vocal Events）四種模式。本文探討 Breeze-TTS-2 是否支援語音嵌入（voice embedding），亦即能否將語音特徵轉換為向量，並以向量形式作為語音複製的輸入。

## 結論：Breeze-TTS-2 不支援語音嵌入向量

**Breeze-TTS-2 沒有任何 speaker encoder、speaker embedding 或 voice embedding vector 機制**。模型內部不存在可將語音特徵萃取為固定維度向量的模組。語音複製的實作方式並非透過嵌入向量，而是透過 **in-context prompt prefixing（上下文提示前綴）**。[^source-hf]

## 語音複製的實際機制：Codec Token 前綴

Breeze-TTS-2 的語音複製流程如下：

1. **參考音訊經由 Qwen3-TTS Mimi audio codec 編碼為離散 codec tokens**（而非連續 embedding vector）。`encode_prompt_audio()` 函式載入參考 WAV 檔案，經由音訊 tokenizer 產生形狀為 `[T, 16]` 的離散 codec tokens（T 個時間幀 × 16 個殘差 codebooks）。[^source-audio]

2. **這些 codec tokens 被直接插入提示序列中**。語音複製範本（`ref_clone_tata`）的建構順序為：說話者前綴 → 參考文字 → 參考音訊 tokens（以 `<|AUDIO|>` 佔位符表示）→ 目標文字。[^source-templates]

3. **每個 `<|AUDIO|>` 佔位符的嵌入是 16 個 per-codebook lookup vectors 的總和**，來自單一 `nn.Embedding` 層——這仍然不是連續的說話者嵌入向量。[^source-arch]

```python
# 簡化自 models/breeze.py
self.embed_audio_tokens = nn.Embedding(config.num_codebooks * config.vocab_size, hidden_size)
input_embeds = self.embed_audio_tokens(input_ids + self.audio_tokens_offsets).sum(dim=2)
```

## 語音導向（Voice Direction）：最接近向量調控的機制

Voice Direction 模式使用 **Classifier-Free Guidance (CFG)** 雙分支設計：

- **正向分支**：參考音訊 + 參考文字 + 指令 + 目標文字
- **負向分支（無條件）**：參考音訊 + 參考文字 + 目標文字（不含指令）
- **結果**：`uncond + scale × (cond − uncond)` —— 僅放大**指令帶來的差異**，保留說話者身份

負向分支的提示實際上與 Voice Clone 完全相同。[^source-editing]

推薦的 `--cfg-scale 4` 意味著「將指令效果放大 4 倍」。此模式需雙倍計算量（`branch_batch_size = 2`）。

## 外部說話者相似度評估

BreezeBlue 發表的 **SPK_SIM (Speaker Similarity)** 0.67 分數來自**外部的說話者驗證流程**，非 Breeze-TTS-2 內建功能：[^source-benchmark]

```python
model = ECAPA_TDNN_SMALL(feat_dim=1024, feat_type="wavlm_large")
# 萃取 192-dim speaker embeddings，然後計算 cosine similarity
```

流程：`WavLM-Large (1024-dim 幀特徵) → ECAPA-TDNN (192-dim speaker embedding) → L2 正規化 → cosine similarity`

**此流程完全獨立於 Breeze-TTS-2 之外**，僅作為評估工具使用。

## 功能支援對照表

| 功能 | Breeze-TTS-2 支援 |
|---|---|
| 語音嵌入（稠密向量） | ❌ 不支援 |
| Speaker encoder 模型 | ❌ 不存在 |
| 說話者向量萃取 | ❌ 不支援 |
| 從參考音訊複製語音 | ✅ 支援（透過 codec token 前綴） |
| 從預計算向量複製語音 | ❌ 不支援（需原始音訊） |
| 透過嵌入向量進行說話者適配 | ❌ 不支援 |
| Voice Design（純文字描述） | ✅ 支援（無需參考音訊） |
| Voice Direction（複製 + 調整） | ✅ 支援（參考音訊 + 指令） |

## 模型架構一覽

| 組件 | 模型 | 參數量 | 細節 |
|---|---|---|---|
| 文字編碼器 | Gemma-3-1B (T5Gemma2) | ~1.0B | 26 層, 1152 hidden, 雙向, cross-attention 至 backbone |
| 主幹網路 | Qwen3-1.7B | ~1.41B | 28 層, 2048 hidden, GQA 16/8, 預測 codebook-0 tokens |
| Depth Decoder | 自訂 12 層 | ~434M | 預測 codebooks 1-15（15 步迴圈）, 1024 hidden |
| Audio Codec | Qwen3-TTS (Mimi-based) | ~170M | 12.5 Hz 幀率, 16 RVQ codebooks × 2048 詞表, 24 kHz 輸出 |
| **總計** | | **3.47B** | 磁碟 7.12 GiB |

## 總結

Breeze-TTS-2 的語音複製採用 **in-context conditioning** 方式——將參考音訊的 codec tokens 前綴至提示序列——而非傳統的 speaker embedding / adaptation 方法。這代表每次語音複製都需要原始音訊檔案，無法萃取可重複使用的「語音向量」。

如欲使用向量式語音複製，需尋找其他支援 speaker encoder 的 TTS 模型（如 Coqui TTS 的 speaker encoder、YourTTS、或 Meta 的 Voicebox 等）。

---

[^source-hf]: BreezeBlue. (2026). Breeze-TTS-2 - Hugging Face Model Page. Retrieved 2026-09-27, from https://huggingface.co/BreezeBlue/Breeze-TTS-2
[^source-audio]: BreezeBlue. (2026). breeze_infer/audio.py - Breeze-TTS GitHub Repository. Retrieved 2026-09-27, from https://github.com/breezeblue-ai/breeze-tts/blob/main/breeze_infer/audio.py
[^source-templates]: BreezeBlue. (2026). breeze_infer/templates.py - Breeze-TTS GitHub Repository. Retrieved 2026-09-27, from https://github.com/breezeblue-ai/breeze-tts/blob/main/breeze_infer/templates.py
[^source-arch]: BreezeBlue. (2026). Model Architecture - DeepWiki. Retrieved 2026-09-27, from https://deepwiki.com/breezeblue-ai/breeze-tts/4-model-architecture
[^source-editing]: BreezeBlue. (2026). Synthesis Modes and Prompting - DeepWiki. Retrieved 2026-09-27, from https://deepwiki.com/breezeblue-ai/breeze-tts/1.2-synthesis-modes-and-prompting
[^source-benchmark]: BreezeBlue. (2026). WavLM-ECAPA Speaker Embedding Extraction - TTS-Voice-Direction-Benchmark. Retrieved 2026-09-27, from https://github.com/breezeblue-ai/TTS-Voice-Direction-Benchmark/blob/main/speaker_verification/wavlm_ecapa.py