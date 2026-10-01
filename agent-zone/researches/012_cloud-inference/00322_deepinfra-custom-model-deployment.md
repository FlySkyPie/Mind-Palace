# DeepInfra 自訂模型部署研究

## 概述

DeepInfra 提供「Private Models」(自訂模型)功能，允許使用者將 HuggingFace 上的 LLM 部署至專用 GPU 上，並透過 OpenAI 相容 API 進行推論。但此功能**僅限於文字生成 LLM**，非任意 PyTorch 模型均可部署。

## 1. DeepInfra 是否支援目錄模型以外的自訂模型？

是，但有明確範疇限制。根據官方文件，可以部署：

- **自訂 LLM** — HuggingFace 上任何相容於 Transformers 的文字生成 LLM，部署於專用 GPU[^custom-llms]
- **LoRA Adapters** — 基於支援的基礎模型之上，部署 LoRA 微調的語言模型 adapter[^lora]
- **LoRA Image Models** — 來自 Civitai 的文字轉圖像 LoRA adapter[^lora]

對於**非 LLM 類型**的自訂 PyTorch 模型（如影像分類、物件偵測、自訂視覺模型等），DeepInfra 並未提供對應的部署路徑。

DeepInfra 的推論底層基於 **NVIDIA TensorRT-LLM**[^github]，這意味著模型必須與 TensorRT-LLM 支援的架構相容——通常是標準的 decoder-only Transformer LLM（Llama、Mistral、Qwen、DeepSeek 等）。若自訂 PyTorch 模型使用非標準架構，則無法運作。

## 2. 如何部署自訂模型？

### 前置步驟：模型上傳 HuggingFace

將模型上傳至 HuggingFace（公開或私有 repo），格式須與 HuggingFace Transformers 相容（`.safetensors` 或 PyTorch `.bin` + `config.json`）。如需使用私有 repo，建立 HuggingFace access token。

### 建立部署

**透過 Web UI：**
導覽至 Dashboard → New Deployment → Custom LLM[^custom-llms]

**透過 HTTP API：**

```bash
curl -X POST https://api.deepinfra.com/deploy/llm \
  -d '{
    "model_name": "my-custom-model",
    "gpu": "A100-80GB",
    "num_gpus": 2,
    "max_batch_size": 64,
    "hf": {
        "repo": "your-username/your-model-repo"
    },
    "settings": {
        "min_instances": 0,
        "max_instances": 1
    }
  }' \
  -H 'Content-Type: application/json' \
  -H "Authorization: Bearer $DEEPINFRA_API_KEY"
```

### 進行推論

部署完成後，使用 OpenAI 相容 API：

```bash
curl "https://api.deepinfra.com/v1/openai/chat/completions" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $DEEPINFRA_API_KEY" \
  -d '{
      "model": "YOUR_USERNAME/my-custom-model",
      "messages": [{"role": "user", "content": "Hello!"}]
    }'
```

模型完整名稱為 `YOUR_GITHUB_USERNAME/model-name`。

## 3. 需求與限制

### 需求

| 項目 | 說明 |
|---|---|
| **模型託管** | 模型必須在 HuggingFace 上（公開或私有 repo） |
| **格式** | 須與 HuggingFace Transformers 相容（`.safetensors` 或 PyTorch `.bin` + `config.json`） |
| **模型類型** | 主要支援文字生成 LLM（causal LM） |
| **GPU 選項** | A100-80GB、H100-80GB、H200-141GB、B200-180GB、B300-288GB |
| **帳號** | DeepInfra 帳號 + API key |
| **HF Token** | 私有 repo 需要 HuggingFace access token |

### 限制

- **每使用者 GPU 上限 4 顆**（例如 4×1GPU 或 1×4GPU），更多需聯絡支援[^custom-llms]
- **擴充時 GPU 可用性不保證** —僅按實際運行計費
- **量化目前不支援**（開發中）
- **按 GPU 時數計費**（非按 token）—需有足夠流量才符合成本效益；低流量時按 token 計費的 providers 可能更便宜
- **`deploy_id`** 在模型部署完成前可能無法即時取得
- **必須是 LLM** —自訂部署功能僅針對文字生成模型文件化

### 成本警示

若忘記關閉 deployment，費用會快速累積。例如：2 顆 GPU 在週末運行 64 小時，約 ~$2/GPU-hour × 2 × 64 = ~$256。

## 4. 與其他 Inference Providers 比較

| Provider | 自訂模型方式 | 計費模式 | 最適合 |
|---|---|---|---|
| **DeepInfra** | 透過 HuggingFace 部署自訂 LLM（專用 GPU） | 每 GPU 時數 | 共享模型低成本；簡易專用部署 |
| **Together AI** | 自訂模型 + 同平台微調（SFT） | 每 token | 最多模型目錄 + 自行微調 |
| **Fireworks AI** | API 上傳自訂權重 + 完整微調（SFT、LoRA、DPO） | 每 token | 速度 + 同平台微調；FireAttention 核心 |
| **Replicate** | 以 Docker 部署任何開源模型 | 每 GPU 秒 | 多種模型類型（不限 LLM）|
| **Baseten** | 基於 container 的自訂部署（程式碼優先） | 每 GPU 秒或每 token | 生產級可靠性；完整堆疊控制 |
| **RunPod / Modal** | 原始 GPU 存取 / serverless GPU | 每 GPU 秒 | 完全控制任何自訂模型、任何架構 |

### 關鍵取捨

- **DeepInfra 最簡潔** —只要指向 HuggingFace repo 即可，但**僅限 HuggingFace LLM**，無法自訂 serving 程式碼
- **Together AI 和 Fireworks** 除了自訂模型外還提供**同平台微調**功能
- **Replicate、Baseten、RunPod、Modal** 提供**完整 container 層級控制**，可執行任何 PyTorch 模型、任何架構、任何前處理，但設定與 DevOps 工作量較大

## 結論

DeepInfra 的自訂模型支援是**真實但有限範疇**的：它非常適合將 **HuggingFace 上的 LLM**（或 LoRA adapter）部署至專用 GPU，並提供 OpenAI 相容 API。但如果你的 PyTorch 專案涉及非 LLM 模型（如自訂視覺分類器、非標準架構），DeepInfra 不是合適的選擇——應考慮 Replicate、Baseten、Modal 或 GPU 雲端 provider。

## 參考資料

[^custom-llms]: DeepInfra. (n.d.). Custom LLMs. Retrieved 2026-01-10, from https://docs.deepinfra.com/private-models/custom-llms
[^lora]: DeepInfra. (n.d.). LoRA Adapters. Retrieved 2026-01-10, from https://docs.deepinfra.com/private-models/lora
[^overview]: DeepInfra. (n.d.). Private Models Overview. Retrieved 2026-01-10, from https://docs.deepinfra.com/private-models/overview
[^models]: DeepInfra. (n.d.). Models. Retrieved 2026-01-10, from https://docs.deepinfra.com/models
[^blog-custom]: DeepInfra. (n.d.). Custom LLMs on DeepInfra (Blog). Retrieved 2026-01-10, from https://deepinfra.com/blog/custom-llms
[^github]: DeepInfra. (n.d.). TensorRT-LLM fork on GitHub. Retrieved 2026-01-10, from https://github.com/deepinfra
[^huggingface-provider]: Hugging Face. (n.d.). DeepInfra Inference Provider. Retrieved 2026-01-10, from https://huggingface.co/docs/inference-providers/providers/deepinfra
[^comparison]: InfraBase. (n.d.). AI Inference API Providers Compared. Retrieved 2026-01-10, from https://infrabase.ai/blog/ai-inference-api-providers-compared