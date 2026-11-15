---
name: glianaai-chat
description: "GlianaAI chat: calling Claude, GPT, Gemini, Grok, Kimi, DeepSeek, Qwen and other chat models through GlianaAI (api.glianalabs.com) with the OpenAI or Anthropic SDK, streaming, tools, controlling cost with max_tokens, running Claude Code or Codex CLI on a GlianaAI key instead of an Anthropic or OpenAI account, pointing OpenAI-compatible apps (Open WebUI, Cline, Continue, n8n, LangChain) at it, and text embeddings for search and RAG. Use with the glianaai skill."
license: MIT
---

# GlianaAI chat

Two formats, same models, same balance:

| Format | Endpoint | Auth |
|---|---|---|
| OpenAI chat completions | `POST /v1/chat/completions` | `Authorization: Bearer gk_...` or a wallet (402) |
| Anthropic Messages | `POST /v1/messages` (+ `/v1/messages/count_tokens`) | key only: `x-api-key: gk_...` or `Authorization: Bearer gk_...` |
| OpenAI Responses | `POST /v1/responses` | key only: `Authorization: Bearer gk_...` |

All three serve every chat model on sale, not only the vendor's own. Ids are GlianaAI's
(`claude-sonnet-5.5`, `gpt-6.1-terra`, `gemini-4-carbon`, `kimi-k3.1-pro`, …), not the vendor's:
list them with `GET /v1/models` (`data` holds the chat models in OpenAI's shape) or `GET /v1/catalog`. Each row has
`inMicroPerMTok` / `outMicroPerMTok` (price per 1M input and output tokens, micro-USD) and
`contextTokens`.

## OpenAI SDK

```python
from openai import OpenAI
import os

client = OpenAI(base_url="https://api.glianalabs.com/v1", api_key=os.environ["GLIANA_API_KEY"])
r = client.chat.completions.create(
    model="kimi-k3.1-pro",
    messages=[{"role": "user", "content": "hello"}],
    max_tokens=512,
)
```

Streaming: `stream=True` returns standard OpenAI server-sent events; the last chunk carries
`usage` and `costMicroUsd`. A request with `tools` is answered whole, not streamed.

## Anthropic SDK

```python
from anthropic import Anthropic
import os

client = Anthropic(base_url="https://api.glianalabs.com", api_key=os.environ["GLIANA_API_KEY"])
msg = client.messages.create(model="claude-sonnet-5.5", max_tokens=1024,
                             messages=[{"role": "user", "content": "hello"}])
```

The base URL has no `/v1` for the Anthropic SDK. Streaming, custom tools and `count_tokens` work.
Server-side tools (Anthropic's web search, code execution) and document blocks are refused by
name rather than silently dropped.

## Claude Code on a GlianaAI key

```bash
export ANTHROPIC_BASE_URL=https://api.glianalabs.com
export ANTHROPIC_API_KEY=$GLIANA_API_KEY
export ANTHROPIC_MODEL=claude-sonnet-5.5
export ANTHROPIC_SMALL_FAST_MODEL=claude-haiku-5.5
claude
```

No Anthropic account is involved; the calls come off the GlianaAI balance. If Claude Code is
signed in to a Claude subscription, run `/logout` inside it first. A short task on
claude-sonnet-5.5 costs a few cents; the key's daily cap (default $5) still applies.

## Codex CLI on a GlianaAI key

```toml
# ~/.codex/config.toml
model = "claude-sonnet-5.5"
model_provider = "gliana"

[model_providers.gliana]
name = "GlianaAI"
base_url = "https://api.glianalabs.com/v1"
env_key = "GLIANA_API_KEY"
wire_api = "responses"
```

Codex speaks the Responses format, served by `/v1/responses` for every chat model. The gateway keeps no
conversation (`previous_response_id` is refused; Codex sends the whole history anyway), and Codex's
built-in web search is not available through it.

## Other OpenAI-compatible apps

Open WebUI, LibreChat, Cline, Continue, n8n, LiteLLM and the OpenAI SDKs take an "OpenAI-compatible"
provider: base URL `https://api.glianalabs.com/v1`, API key `gk_...`. Their model list loads from
`GET /v1/models`.

## Embeddings

`POST /v1/embeddings` in OpenAI's format, model `bge-m3` (1024 dimensions, multilingual, good on
Indonesian), $0.02 per million tokens with a $0.0001 minimum a call, up to 2048 texts a request.
It is the only embedding model: another model name is refused, so an index never mixes two models.

```python
vecs = client.embeddings.create(model="bge-m3", input=["harga tiket kereta", "train fare"])
```

LangChain's `OpenAIEmbeddings` sends token ids unless told not to; pass
`check_embedding_ctx_length=False` (and `model="bge-m3"`).

## Cost

- A call reserves estimated input + `max_tokens` up front (that is the price quoted by
  `/v1/price` and the 402). On a balance, only the tokens actually used are kept and the rest is
  refunded when the answer ends. Set `max_tokens` to what the task needs: it bounds a single
  call, and a large value needs that much balance free while the call runs.
- Reasoning models spend hidden reasoning tokens, billed as output. An answer can cost more
  output tokens than its visible text suggests.
- The response says what it cost: `costMicroUsd` in the body, `x-gliana-charged-micro-usd` in
  the headers.

## Picking a model

Cheapest that does the job: compare `inMicroPerMTok + outMicroPerMTok` in `/v1/catalog`. Use a
frontier model (Claude Opus/Sonnet, GPT-6.x, Gemini 4) for hard reasoning or long agentic work;
a small model (Haiku, Inkling 2 Small, Gemini Flash Lite) for classification, extraction and
short replies.
