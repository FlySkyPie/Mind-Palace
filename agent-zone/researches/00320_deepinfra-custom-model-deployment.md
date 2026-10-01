# DeepInfra 自訂模型部署完整指南

## 概述

DeepInfra 提供三種自訂模型（Private Models）方案，讓使用者部署自己的模型到專用 GPU 上[^private-overview]：

1. **Custom LLMs** — 部署任何 Hugging Face 上的 LLM 至專用 GPU，具備自動擴展功能，並提供 OpenAI 相容 API 端點
2. **LoRA Adapters** — 在 DeepInfra 既有支援的基礎模型上部署 LoRA 微調語言模型
3. **LoRA Image Models** — 從 Civitai 部署 LoRA adapter 用於圖片生成

## 支援的模型格式

DeepInfra 要求模型必須存放在 **Hugging Face**（公開或私有儲存庫皆可）。底層引擎為 **vLLM**[^custom-llms]，支援 Hugging Face Transformers 相容格式：

- **SafeTensors**（主要/推薦格式）
- **PyTorch** checkpoint 格式（`pytorch_model.bin`）

GGUF 格式不直接支援；使用者需先將 GGUF 轉換回 SafeTensors 才可部署。

## 部署步驟

### 前置需求

1. DeepInfra 帳號與 API key
2. 模型存放於 Hugging Face（公開或私有 repo）
3. 私有 repo 需準備 Hugging Face token

### 方法 A：Web UI

1. 前往 **Dashboard → [Deployments](https://deepinfra.com/dash/deployments)**
2. 點擊 **New Deployment** → **Custom LLM**
3. 填寫：
   - **Model Name** — 推論時使用的名稱（如 `my-finetuned-llm`）
   - **GPU** — 選擇 A100-80GB、H100-80GB、H200-141GB、B200-180GB 或 B300-288GB
   - **Number of GPUs** — 每 instance 使用的 GPU 數量
   - **Max Batch Size** — 最大平行請求數
   - **Weights** — Hugging Face repo 路徑（如 `your-username/your-model`）
   - **Hugging Face Token** — 私有 repo 需要
4. 設定擴展參數（min/max instances）
5. 點擊 **Upload/Deploy**

### 方法 B：HTTP API

```bash
curl -X POST https://api.deepinfra.com/deploy/llm \
  -d '{
    "model_name": "my-finetuned-llm",
    "gpu": "A100-80GB",
    "num_gpus": 2,
    "max_batch_size": 64,
    "hf": {
        "repo": "your-username/your-model"
    },
    "settings": {
        "min_instances": 0,
        "max_instances": 1
    }
  }' \
  -H 'Content-Type: application/json' \
  -H "Authorization: Bearer $DEEPINFRA_API_KEY"
```

部署完成後，模型全名為 `YOUR_GITHUB_USERNAME/my-finetuned-llm`。

### 進階引擎參數

可透過 `standard_args` 傳遞 vLLM 引擎參數：

```json
"standard_args": {
    "max_context_size": 32768,
    "max_concurrent_requests": 128,
    "kv_cache_dtype": "fp8",
    "enable_prefix_caching": true,
    "quantization": "awq"
}
```

支援的量化方式：`fp8`、`awq`、`gptq`、`awq_marlin`、`gptq_marlin`、`compressed-tensors`、`bitsandbytes`。

### 監控部署狀態

```bash
curl https://api.deepinfra.com/deploy/list \
  -H "Authorization: Bearer $DEEPINFRA_API_KEY"
```

狀態變化：`Initializing` → `Deploying` → `Running`

### 呼叫部署後的模型

API 端點與一般模型**完全相同**：

```bash
curl "https://api.deepinfra.com/v1/openai/chat/completions" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $DEEPINFRA_API_KEY" \
  -d '{
      "model": "YOUR_USERNAME/my-finetuned-llm",
      "messages": [{"role": "user", "content": "Hello!"}]
    }'
```

## API 端點比較：自訂模型 vs 內建模型

| 面向 | 內建（共享）模型 | 自訂（私有）模型 |
|------|-----------------|-----------------|
| **Base URL** | `https://api.deepinfra.com/v1/openai` | 相同 |
| **模型名稱格式** | `org/model-name` | `YOUR_USERNAME/your-model` |
| **計費方式** | 每 token（~$0.02–$2.85/1M input） | 每 GPU 小時（$0.89–$4.89/hr）[^pricing] |
| **API 相容性** | OpenAI 相容（chat/completions/embeddings） | **完全相同** |
| **服務層級** | Flex (0.8x)、Standard (1x)、Priority (1.5x) | 不適用 |
| **資料隔離** | 與其他使用者共享 | 專屬端點 |

## GPU 選項與價格

| GPU | 記憶體 | 價格/GPU-hr |
|-----|--------|-------------|
| A100 | 80GB | $0.89 |
| H100 | 80GB | $2.20 |
| H200 | 141GB | $2.69 |
| B200 | 180GB | $3.69 |
| B300 | 288GB | $4.89 |

## 限制與注意事項

### 限制

- **GPU 配額**：每位使用者預設 4 GPU（如 4×1GPU 或 1×4GPU），可聯絡業務提高
- **GPU 可用性**：擴展時不保證立即有可用 GPU
- **帳單週期**：按週計費（與 per-token 計費分開）
- **無閒置折扣**：即使無流量也需支付 uptime 費用
- **擴展延遲**：`deploy_id` 在模型部署完成前可能無法立即使用

### 成本警示

> DeepInfra 文件提醒：「忘記關閉自訂部署可能快速累積成本。例如，忘記在週末（64 小時）關閉一個 2-GPU 部署，約花費 $256 USD。」

**務必在 [payment settings](https://deepinfra.com/dash/billing) 設定花費上限。**

### 自訂部署的適用場景

自訂部署最適合：
- 需要使用 DeepInfra 目錄中未提供的 **微調 checkpoint**
- 法規/安全要求需要 **資料隔離**
- 需要 **可預測的延遲**，不受共享競爭影響
- **持續高流量** 使 per-token 計費比 per-hour 更貴

低流量或突發性工作負載應使用共享 per-token API，成本更低。

## 參考資源

| 資源 | URL |
|------|-----|
| Private Models 概述 | https://docs.deepinfra.com/private-models/overview |
| Custom LLMs 完整指南 | https://docs.deepinfra.com/private-models/custom-llms |
| LoRA Adapters | https://docs.deepinfra.com/private-models/lora |
| API Reference | https://docs.deepinfra.com/api-reference/introduction |
| Pricing | https://deepinfra.com/pricing |
| GPU Instances | https://deepinfra.com/gpu-instances |
| GitHub Cookbooks | https://github.com/deepinfra/cookbooks |

## 參考文獻

[^private-overview]: DeepInfra. (n.d.). Private Models Overview. Retrieved 2026-10-01, from https://docs.deepinfra.com/private-models/overview
[^custom-llms]: DeepInfra. (n.d.). Custom LLMs — Deploy Fine-Tuned Custom LLMs. Retrieved 2026-10-01, from https://docs.deepinfra.com/private-models/custom-llms
[^pricing]: DeepInfra. (n.d.). DeepInfra Pricing. Retrieved 2026-10-01, from https://deepinfra.com/pricing