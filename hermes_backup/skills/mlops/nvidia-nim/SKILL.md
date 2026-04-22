---
name: nvidia-nim
description: Use NVIDIA NIM as a free LLM provider via OpenAI-compatible API. Works with any LangChain/OpenAI SDK project.
tags: [nvidia, nim, llm, free, openai-compatible, langchain]
---

# NVIDIA NIM — Free LLM Provider

NVIDIA NIM provides free access to 130+ models via an OpenAI-compatible API endpoint.

## API Configuration

- **Base URL:** `https://integrate.api.nvidia.com/v1`
- **Auth:** Bearer token (API key from https://build.nvidia.com)
- **Interface:** OpenAI-compatible (`/v1/chat/completions`, `/v1/models`)

## Verified Working Models (Free Tier)

| Model | Size | Best For |
|---|---|---|
| `deepseek-ai/deepseek-v3.2` | Large | General reasoning, complex analysis |
| `minimaxai/minimax-m2.7` | Large | Chinese + English, detailed reasoning |
| `meta/llama-3.1-70b-instruct` | 70B | General purpose, reliable |
| `microsoft/phi-4-mini-instruct` | Small | Fast responses, simple tasks |
| `google/gemma-2-2b-it` | 2B | Ultra-fast, lightweight |

**Note:** Some models like `writer/palmyra-fin-70b-32k` return 404 on the free tier despite appearing in the model list.

## Using with LangChain (OpenAI-compatible projects)

### .env Configuration

```bash
# NVIDIA NIM
OPENAI_API_KEY=nvapi-YOUR_KEY_HERE
OPENAI_API_BASE=https://integrate.api.nvidia.com/v1
```

Then use `--model provider-model-name` and the project's OpenAI client will route to NVIDIA NIM.

### Python Direct Usage

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://integrate.api.nvidia.com/v1",
    api_key="nvapi-YOUR_KEY"
)

response = client.chat.completions.create(
    model="deepseek-ai/deepseek-v3.2",
    messages=[{"role": "user", "content": "Hello"}],
    max_tokens=100
)
```

## Testing Model Availability

Before using a model, verify it works:

```python
import urllib.request, json

url = "https://integrate.api.nvidia.com/v1/chat/completions"
payload = json.dumps({
    "model": "model-name-here",
    "messages": [{"role": "user", "content": "Reply OK"}],
    "max_tokens": 5
}).encode()

req = urllib.request.Request(url, data=payload, headers={
    "Content-Type": "application/json",
    "Authorization": "Bearer nvapi-YOUR_KEY"
})
resp = urllib.request.urlopen(req, timeout=60)
print(json.loads(resp.read()))
```

## Listing All Available Models

```python
import urllib.request, json

url = "https://integrate.api.nvidia.com/v1/models"
req = urllib.request.Request(url, headers={
    "Authorization": "Bearer nvapi-YOUR_KEY"
})
resp = urllib.request.urlopen(req, timeout=15)
models = json.loads(resp.read())["data"]
for m in models:
    print(m["id"])
```

## Model Comparison (AI Hedge Fund, 19 analysts, AAPL)

| Model | Time | Success Rate | LLM Errors | Notes |
|---|---|---|---|---|
| `minimaxai/minimax-m2.7` | ~120s | 19/19 | 0 | Best reasoning quality, detailed analysis |
| `meta/llama-3.1-70b-instruct` | ~100s | 19/19 | 0 | Good quality, more concise |
| `deepseek-ai/deepseek-v3.2` | 300s+ | Timeout | N/A | Too slow for multi-agent (20+ sequential calls) |
| `writer/palmyra-fin-70b-32k` | N/A | 3/19 | 10 (404) | Not available on free tier |

**Recommendation for multi-agent systems:** Use `minimaxai/minimax-m2.7` (quality) or `meta/llama-3.1-70b-instruct` (speed). Avoid `deepseek-v3.2` for sequential multi-call workflows.

## Pitfalls

- **Slow first response:** Large models (70B+) may take 15-30s on first request (cold start)
- **Rate limiting:** Free tier has rate limits; 429 errors require waiting 60s+ before retry
- **404 on some models:** Not all listed models are actually available on free tier; always test first
- **Timeout:** Use `timeout=90` or higher for large model inference calls
- **Large model inference:** Multi-agent systems running 20+ LLM calls sequentially can take 10+ minutes with large models

## Adding Custom Models to Projects (e.g., AI Hedge Fund)

For projects that load models from a JSON config file (like `ai-hedge-fund`):

```python
import json

with open("src/llm/api_models.json") as f:
    models = json.load(f)

models.append({
    "display_name": "Your Model (NVIDIA NIM)",
    "model_name": "provider/model-name",
    "provider": "OpenAI"  # Uses OpenAI-compatible API
})

with open("src/llm/api_models.json", "w") as f:
    json.dump(models, f, indent=2)
```

Then run with `--model provider/model-name`.
