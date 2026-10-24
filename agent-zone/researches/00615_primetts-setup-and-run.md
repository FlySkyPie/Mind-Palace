# PrimeTTS 的安裝與執行指南

本報告說明如何在自有環境中安裝並執行 Hugging Face 上的文字轉語音模型 `Luigi/PrimeTTS`[^hfmodel]。內容涵蓋模型家族組成、依賴需求、逐步安裝指令、各變體的推論程式碼、串流模式，以及部署執行環境的選擇。所有事實均標註引用來源，並以改良版 APA 格式（ISO 8601 日期）標示。

---

## 1. PrimeTTS 是什麼

PrimeTTS 是一組為「裝置端（on-device）」使用情境設計的台灣華語（zh-TW）加英文文字轉語音模型家族，鎖定客服中心、導航、交通運輸等場域。其單一前端（frontend）可同時處理中文、英文與「中英夾雜（code-mix）」文本，無需切換語言路徑，並能正確朗讀實體詞（entities），例如電話號碼、電子郵件、地址、價格、日期、溫度、百分比與序號[^readme]。

模型的授權條款為 Apache-2.0，語言標記為 `zh` 與 `en`，推論框架為 ONNX Runtime，並同時提供 GGUF 格式[^hfmodel]。截至本報告擷取時，該模型倉庫的累計下載量為 163 次、按讚數為 5，最後更新時間為 2026-07-06[^hfapi]。

### 1.1 模型家族

PrimeTTS 並非單一模型，而是多個變體的集合。依官方模型卡的整理，其主要成員如下[^readme]：

| 模型 | 目錄 | 參數（部署） | 架構 | 適用情境 |
|---|---|---|---|---|
| **v2.1**（旗艦） | `v21_mbistft_16k/` | 37.9M | MB-iSTFT-VITS，3 種音色 | 需要多音色可選 |
| v2 | `v2_mbistft_16k/` | 34.7M（17.5M） | MB-iSTFT-VITS，單一音色 | 追求最乾淨的單一音色 |
| V2 Lite | `v2lite_mbistft_16k/` | 24.8M（17.5M） | 同 v2，體積較小 | GPU 資源較緊時 |
| **v1** | `v1b_16k/`、`v1b_8k/` | 約 5M | FastSpeech + Snake-HiFiGAN | 純 CPU、即時推論 |

官方指出，v2.1 的 3 個音色以整數 `sid` 輸入選擇：`0` 為 Xinran（女聲）、`1` 為 Anchen（男聲）、`2` 為 Bowen（男聲）[^readme]。而 v1 則為單一年輕女聲，涵蓋 zh／en／中英夾雜，並以「純 CPU、可在 Jetson Nano 上即時執行」為賣點[^readme]。官方建議：需要多音色時選 v2.1；若預算僅限 CPU 且極度吃緊，選 v1[^readme]。

---

## 2. 環境需求

### 2.1 執行時依賴（v2.1 與 v1）

官方 Quickstart 所列的 Python 套件如下[^readme]：

```bash
pip install onnxruntime numpy soundfile g2pw g2p_en cn2an
```

其中：

- `onnxruntime`（亦寫作 ort）負責載入並執行 `.onnx` 模型[^readme]。
- `numpy` 與 `soundfile` 分別用於張量運算與 WAV 檔寫入[^readme]。
- `g2pw` 提供台灣讀音的注音符號轉換與破音字消歧[^readme]。該套件在 PyPI 上最新版本為 0.1.2（2026-10-19 上傳），依賴 `requests`、`tqdm`、`onnxruntime`、`transformers`[^g2pw]。
- `g2p_en` 提供英文的 ARPAbet 音素轉換[^readme]。該套件最新版本為 2.1.0，依賴 `numpy`、`nltk`、`inflect`、`distance`[^g2pen]。
- `cn2an` 用於中文數字轉換[^readme]。前端程式碼中的實體正規化（`text_norm.py`）即使用 `cn2an` 與 `inflect`[^textnorm]。

需注意，`g2p_en` 首次執行時會自動下載 nltk 資料（包含 `averaged_perceptron_tagger` 與 `cmudict`）[^g2pen]。前端 `frontend_bopomofo.py` 匯入了 `g2pw.G2PWConverter`、`g2p_en.G2p` 與 `cn2an`，並在首次呼叫時以延遲載入方式初始化這些模型[^frontend]。

### 2.2 硬體與平台

- v1 在 Jetson Nano（Tegra X1，ARMv8.0 Cortex-A57）上，以 8 kHz 模型單執行緒可達 RTF 0.35（即約 2.9 倍即時）[^readme]。
- v2／v2.1 在相同硬體上，CPU fp32 四執行緒 RTF 為 0.52，GPU（RapidSpeech.cpp ggml-CUDA）RTF 為 0.42[^readme]。
- 官方強調在此類 ARMv8.0 核心上 **fp32 是最快的格式**：int8 並非加速手段（靜態 int8 會改變音色、動態 int8 比 fp32 更慢），fp16 會被轉為 fp32 而無加速效果[^readme]。

---

## 3. 安裝步驟

### 3.1 建立環境並安裝套件

建議在獨立的 Python 虛擬環境中進行。以 v2.1／v1 的推論為例[^readme]：

```bash
python -m venv primetts-env
source primetts-env/bin/activate
pip install onnxruntime numpy soundfile g2pw g2p_en cn2an
```

### 3.2 下載模型權重

官方提供兩種下載方式。使用 Hugging Face CLI 將整個倉庫抓取到本地目錄[^readme]：

```bash
huggingface-cli download Luigi/PrimeTTS --local-dir PrimeTTS
```

由於 `scripts/` 目錄與 `v1b_16k/`、`v21_mbistft_16k/` 等子目錄皆位於同一倉庫內，一次下載即可取得推論程式碼、前端、meta 設定與模型檔[^hfapi][^readme]。倉庫根目錄另含舊版 24 kHz 變體（`v3_4.6M/` 與頂層 `*.onnx`），官方標明僅保留作為溯源之用[^readme]。

> 注意：倉庫中的 `scripts/synth_from_text.py`、`synth_long.py`、`onnx_stream.py` 等程式含有作者本機的絕對路徑（例如 `/home/luigi/jetson-tts/...`），直接執行前必須先修改 `sys.path.insert` 與預設參數路徑[^synthfromtext][^synthlong][^onnxstream]。

---

## 4. 執行 v2.1（旗艦，3 音色）

以下為官方 Quickstart 的完整範例，改以本地下載目錄 `PrimeTTS/` 取代相對路徑[^readme]：

```python
import sys; sys.path.insert(0, "PrimeTTS/scripts")
import numpy as np, onnxruntime as ort, soundfile as sf
import frontend_bopomofo as F                       # g2pw 注音 + g2p_en，單次處理

sess = ort.InferenceSession("PrimeTTS/v21_mbistft_16k/primetts_v21_3voice.onnx",
                            providers=["CPUExecutionProvider"])
o = F.text_to_ids("您好，歡迎使用 PrimeTTS。Thank you for calling.")
blank = lambda s: np.array([[0] + [v for x in s for v in (x, 0)]], np.int64)   # add_blank=true
sid = 0                                             # 0 Xinran 女 · 1 Anchen 男 · 2 Bowen 男
wav = sess.run(None, {
    "x": blank(o["phone_ids"]), "tone": blank(o["tone_ids"]), "lang": blank(o["lang_ids"]),
    "x_lengths": np.array([2*len(o["phone_ids"])+1], np.int64),
    "sid": np.array([sid], np.int64),
    "noise_scale": np.array([0.667], np.float32),
    "length_scale": np.array([1.0], np.float32)})[0].reshape(-1)
sf.write("out.wav", wav, 16000)
```

### 4.1 關鍵輸入說明

- `add_blank`：模型於每個音素符號之間插入一個 `0` 值空白符，因此序列長度為 `2 * token 數 + 1`；範例中的 `blank` 函式即負責此交錯[^readme]。
- `sid`：v2.1 的 3 音色選擇整數[^readme]。
- `noise_scale`：控制取樣的隨機程度，預設 0.667[^readme]。
- `length_scale`：控制語速，1.0 為原速；數值越大語速越慢[^readme]。
- 輸出為 16 kHz 的 float 波形，直接寫入 WAV[^readme]。

### 4.2 前端輸出格式

`frontend_bopomofo.text_to_ids()` 會回傳字典，含三組等長序列：`phone_ids`（音素 id）、`tone_ids`（聲調／輕重音）、`lang_ids`（每個音素的語言 id，0 為 ZH、1 為 EN）[^frontend][^readme]。符號表共 88 個（含特殊符號、37 個注音符號、ARPAbet 音素、標點與一個補充的「ㄭ」韻母符號）[^frontend]。v2.1 與 v1 兩個家族共用同一份前端輸出[^readme]。

---

## 5. 執行 v1（精簡、純 CPU）

v1 的管線為 `encoder → numpy 長度調節器 → decoder → vocoder`，亦即編碼器輸出音框條件、時長與音高後，需在 host 端以 `host_regulate()` 進行時長展開，再交由解碼器與聲碼器[^readme][^synthfromtext]。官方範例[^readme]：

```python
import sys; sys.path.insert(0, "PrimeTTS/scripts")
import json, numpy as np, onnxruntime as ort, soundfile as sf
import frontend_bopomofo as F
from synth_from_text import host_regulate

D = "PrimeTTS/v1b_16k"                               # 或 v1b_8k 取得最省資源版本
meta = json.load(open(f"{D}/meta.json"))
enc = ort.InferenceSession(f"{D}/acoustic_encoder.onnx", providers=["CPUExecutionProvider"])
dec = ort.InferenceSession(f"{D}/acoustic_decoder.onnx", providers=["CPUExecutionProvider"])
voc = ort.InferenceSession(f"{D}/vocoder.onnx",          providers=["CPUExecutionProvider"])
o = F.text_to_ids("您好，歡迎使用 PrimeTTS。Thank you for calling.")
ph, tn, lg = (np.array([o[k]], np.int64) for k in ("phone_ids", "tone_ids", "lang_ids"))
cond, dur, pitch = enc.run(None, {"phone": ph, "tone": tn, "lang": lg, "speaker": np.zeros(1, np.int64)})
reg = host_regulate(cond, dur, pitch, meta["abs_frame_bins"], meta["max_frames"])
mel = dec.run(None, {k: reg[k] for k in ["frames","frame_meta","local_ctx_raw","abs_pos","pitch_frame","frame_mask"]})[0]
wav = voc.run(None, {"mel": mel.astype(np.float32)})[0].reshape(-1)
sf.write("out.wav", wav, meta["sample_rate"])
```

### 5.1 meta.json 參數

各 v1 目錄均附有 `meta.json`，記載取樣率、音框上限、hop size 等參數。例如 `v1b_16k/meta.json` 為 `sample_rate=16000`、`abs_frame_bins=512`、`max_frames=1000`、`hop_size=256`、`n_mels=80`[^meta16k]；`v1b_8k/meta.json` 則為 `sample_rate=8000` 與 `hop_size=128`[^meta8k]。

### 5.2 長文本處理

官方說明，聲學模型在超過約 `max_frames`（約 15 秒）後會退化，因為絕對位置編碼飽和，導致長句後半段出現亂音[^synthlong]。解法為使用 `scripts/synth_long.py`，在標點處切分文句，貪婪地打包成不超過安全音框預算（預設 1100 框）的區塊，逐塊合成後以短靜音（預設 70 毫秒）串接，無需重新訓練[^synthlong]。

```python
from synth_long import LongSynth
ls = LongSynth("PrimeTTS/v1b_16k", frame_budget=1100, gap_ms=70)
wav, chunks = ls.synth("（長文本……）")
```

---

## 6. 串流模式

PrimeTTS 另提供串流變體。`v2stream_streaming/` 的架構為「band-attention 編碼器 + 因果 FFN + 因果 MB-iSTFT 聲碼器」，可固定區塊增量輸出音訊，整句推論與分塊推論結果位元級一致[^streamreadme]。

串流參數為：區塊 = 24 音框（約 384 毫秒）、左快取 64 框、右前瞻 4 框，1 音框 = 16 kHz 下的 256 個取樣點[^streamreadme]。

Python（ONNX Runtime，免 torch）用法[^streamreadme]：

```python
from onnx_stream import StreamingTTS      # 位於同一目錄
tts = StreamingTTS("v2stream_streaming/v2stream_enc.onnx",
                   "v2stream_streaming/v2stream_dec.onnx")
z = tts.encode(phone_ids, tone_ids, lang_ids)   # 執行一次
for pcm_chunk in tts.stream(z):                  # 每 24 音框輸出一次
    play(pcm_chunk)                              # 16 kHz float32
```

`v2stream_streaming/meta.json` 進一步記載串流參數（`chunk_frames=24`、`left_cache_frames=64`、`right_lookahead_frames=4`、`samples_per_frame=256`）與前端慣例（`add_blank=true`、`n_symbols=88`、`n_tones=6`、`n_langs=2`）[^streammeta]。

另有一個改良版本 `v2streamclean_streaming/`，改用 v2 的非因果「乾淨」聲碼器以消除雜訊，其右前瞻需 16 框（因果版僅需 4 框），其餘參數相同[^streamclean]。

---

## 7. 部署執行環境選擇

官方文件列出數種推論後端[^readme][^streamreadme]：

1. **ONNX Runtime（Python）**：跨平台、免 torch，適合原型與伺服器端。上述 Quickstart 即屬此類[^readme]。
2. **sherpa-onnx（C++／ORT）**：串流變體提供 `OfflineTtsMbistftStreamModel(enc, dec, num_threads, right_lookahead)` 介面，可透過回呼函式逐區塊輸出音訊，程式碼位於 `vieenrose/sherpa-onnx` 的 `mbistft-streaming-tts` 分支[^streamreadme][^sherpabranch]。
3. **RapidSpeech.cpp（ggml，GPU／Jetson）**：以 `MBISTFT_STREAM=1 MBISTFT_STREAM_LOOKAHEAD=5` 執行 `mbistft-parity`，載入 GGUF 單圖檔；官方在 Jetson Nano 上驗證 parity cosine 大於 0.99、首次出聲約 285 毫秒[^streamreadme]。RapidSpeech.cpp 為 RapidAI 專案的 fork，原生倉庫為純 C++ 的裝置端語音執行框架[^rapidspeech]。
4. **llama.cpp 生態**：Hugging Face 模型頁面亦提供以 `llama serve -hf Luigi/PrimeTTS:F32` 等指令載入的方式，但此屬平台自動生成的通用樣板，並非 PrimeTTS 本身的官方推論路徑[^hfmodel]。

另有兩個純前端（WASM）示範 Space：`Luigi/PrimeTTS-vs-Inflect-Nano-v1` 提供 7 模型 A/B 比較，`Luigi/PrimeTTS-Streaming` 提供瀏覽器端逐區塊串流播放；兩者皆以 `onnxruntime-web` 在瀏覽器內推論，權重於首次使用時才從模型倉庫延遲下載[^spaces1][^spaces2]。

---

## 8. 常見問題與注意事項

- **音色挑選**：v2.1 的 3 音色中，Xinran（`sid=0`）CER 最低（0.059），Anchen 與 Bowen 分別為 0.069 與 0.066，官方標註兩男聲帶有些微口音[^readme]。
- **長度上限**：非串流模型在約 15 秒後音質下降，須改用 `synth_long.py` 分塊[^synthlong]。
- **程式碼中的絕對路徑**：倉庫內腳本多處寫死作者本機路徑，執行前務必修改[^synthfromtext][^synthlong][^onnxstream]。
- **量化非加速手段**：在 ARMv8.0 Cortex-A57 上，量化只影響下載體積，並非速度槓桿[^readme]。
- **前端資源**：`g2pw` 會使用 BERT 模型進行破音字消歧，需有 `transformers` 與對應權重；`g2p_en` 首次執行需下載 nltk 資料[^g2pw][^g2pen]。

---

## 9. 從零重建（選讀）

若需以自有聲音重建模型，倉庫提供完整訓練腳本。官方列出兩條路徑[^readme]：

- **v1**：使用 `inflect_nano/` 訓練器。流程為產生語料文本 → 以教師模型（VoxCPM2 複製 CC0 zh-TW 參考音）合成 → 以 `asr_filter.py` 進行 ASR 品質閘門 → 以 `align_durations_v4.py` 進行音素級對齊 → 訓練聲學與聲碼器 → 以 `export_8k.py` 匯出。官方另提供一鍵腳本 `scripts/rebuild_voice.sh`，只需換入約 10 秒的參考音檔[^readme][^rebuild]。
- **v2.1**：使用上游 MB-iSTFT-VITS 訓練器。以 VibeVoice-Large 教師合成語料，進行 CER 閘門（X-ASR 正規化 CER < 0.05），訓練 3 講者模型後以 `export_onnx_primetts_v21.py` 匯出[^readme]。該匯出腳本以精確的 irFFT 矩陣乘法加上窗函數 overlap-add 取代 `torch.istft`（因 ONNX 無此運算子），並在匯出前與 `torch.istft` 數值比對[^exportonnx]。

---

## 10. 來源

[^hfmodel]: Luigi. (2026). PrimeTTS. Retrieved 2026-10-23, from https://huggingface.co/Luigi/PrimeTTS
[^hfapi]: Hugging Face. (2026). Model API record for Luigi/PrimeTTS. Retrieved 2026-10-23, from https://huggingface.co/api/models/Luigi/PrimeTTS
[^readme]: Luigi. (2026). PrimeTTS — on-device zh-TW + English TTS (model card). Retrieved 2026-10-23, from https://huggingface.co/Luigi/PrimeTTS/raw/main/README.md
[^frontend]: Luigi. (2026). scripts/frontend_bopomofo.py. Retrieved 2026-10-23, from https://huggingface.co/Luigi/PrimeTTS/raw/main/scripts/frontend_bopomofo.py
[^textnorm]: Luigi. (2026). scripts/text_norm.py. Retrieved 2026-10-23, from https://huggingface.co/Luigi/PrimeTTS/raw/main/scripts/text_norm.py
[^synthfromtext]: Luigi. (2026). scripts/synth_from_text.py. Retrieved 2026-10-23, from https://huggingface.co/Luigi/PrimeTTS/raw/main/scripts/synth_from_text.py
[^synthlong]: Luigi. (2026). scripts/synth_long.py. Retrieved 2026-10-23, from https://huggingface.co/Luigi/PrimeTTS/raw/main/scripts/synth_long.py
[^exportonnx]: Luigi. (2026). scripts/export_onnx_primetts_v21.py. Retrieved 2026-10-23, from https://huggingface.co/Luigi/PrimeTTS/raw/main/scripts/export_onnx_primetts_v21.py
[^rebuild]: Luigi. (2026). scripts/rebuild_voice.sh. Retrieved 2026-10-23, from https://huggingface.co/Luigi/PrimeTTS/raw/main/scripts/rebuild_voice.sh
[^meta16k]: Luigi. (2026). v1b_16k/meta.json. Retrieved 2026-10-23, from https://huggingface.co/Luigi/PrimeTTS/raw/main/v1b_16k/meta.json
[^meta8k]: Luigi. (2026). v1b_8k/meta.json. Retrieved 2026-10-23, from https://huggingface.co/Luigi/PrimeTTS/raw/main/v1b_8k/meta.json
[^streamreadme]: Luigi. (2026). PrimeTTS v2-Stream — token-level streaming zh-TW/English TTS. Retrieved 2026-10-23, from https://huggingface.co/Luigi/PrimeTTS/raw/main/v2stream_streaming/README.md
[^streammeta]: Luigi. (2026). v2stream_streaming/meta.json. Retrieved 2026-10-23, from https://huggingface.co/Luigi/PrimeTTS/raw/main/v2stream_streaming/meta.json
[^streamclean]: Luigi. (2026). PrimeTTS v2-Stream-Clean. Retrieved 2026-10-23, from https://huggingface.co/Luigi/PrimeTTS/raw/main/v2streamclean_streaming/README.md
[^onnxstream]: Luigi. (2026). v2stream_streaming/onnx_stream.py. Retrieved 2026-10-23, from https://huggingface.co/Luigi/PrimeTTS/raw/main/v2stream_streaming/onnx_stream.py
[^spaces1]: Luigi. (2026). PrimeTTS vs Inflect-Nano WASM. Retrieved 2026-10-23, from https://huggingface.co/api/spaces/Luigi/PrimeTTS-vs-Inflect-Nano-v1
[^spaces2]: Luigi. (2026). PrimeTTS Streaming WASM. Retrieved 2026-10-23, from https://huggingface.co/api/spaces/Luigi/PrimeTTS-Streaming
[^g2pw]: Chen, Y.-C. (2026). g2pW (Version 0.1.2) [Software]. Retrieved 2026-10-23, from https://pypi.org/project/g2pw/
[^g2pen]: Park, K., & Kim, J. (2019). g2p_en (Version 2.1.0) [Software]. Retrieved 2026-10-23, from https://pypi.org/project/g2p-en/
[^sherpabranch]: Vieenrose. (2026). sherpa-onnx branch mbistft-streaming-tts. Retrieved 2026-10-23, from https://github.com/vieenrose/sherpa-onnx/tree/mbistft-streaming-tts
[^rapidspeech]: RapidAI. (2026). RapidSpeech.cpp. Retrieved 2026-10-23, from https://github.com/RapidAI/RapidSpeech.cpp
