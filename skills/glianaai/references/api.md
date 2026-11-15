# GlianaAI API reference (short)

Base URL `https://api.glianalabs.com`. Everything here is live; nothing in this file is a price or
a model list, on purpose: read those from the endpoints.

## Free endpoints

| Call | Returns |
|---|---|
| `GET /v1/consult?intent=...` | Ranked candidates (`kind`, `id`, `endpoint`, `price`, `why`) for a goal |
| `GET /v1/models` | Every model on sale: `id`, `name`, `category`, `unit`, `unitPriceMicroUsd`, `releasedAt` |
| `GET /v1/catalog` | Same, plus per-1M-token rates (`inMicroPerMTok`, `outMicroPerMTok`) and `contextTokens` for chat models |
| `GET /v1/schema?model=<id>` | `required` and `props` for one model |
| `GET /v1/price?model=<id>&...` | `costMicroUsd` for one exact request |
| `GET /v1/tools` | Every utility tool with `path`, `method`, `priceMicroUsd`, `example`, `guidance`, `category` |
| `GET /v1/keys/balance` | The key's balance (needs the key) |
| `POST /v1/media` | Upload a file (raw bytes, up to 40 MB), get a URL to pass as input |

## Paid endpoints

| Call | Body |
|---|---|
| `POST /v1/infer` | `{"model": "<id>", ...inputs}` — any model |
| `POST /v1/images/generations` | OpenAI images format |
| `POST /v1/audio/speech`, `/v1/audio/transcriptions` | OpenAI audio format |
| `POST /v1/chat/completions` | OpenAI chat format, `stream: true` supported |
| `POST /v1/messages` | Anthropic Messages format (key only), streaming and tools |
| `POST /v1/recipes/<name>` | Multi-model pipelines (e.g. text → image → video) at one price |
| `POST|GET /v1/tools/<name>` | Utility tools; free ones are `GET` and need no key |

## Response headers on paid calls with a key

- `x-gliana-charged-micro-usd` — what this call took
- `x-gliana-balance-micro-usd` — the balance after it

## Billing rules worth knowing

- Priced and charged before the run; a request that fails validation is rejected with no charge.
- A run that fails at the provider is refunded.
- Video bills per second at the resolution tier; unset duration or resolution bills the highest.
- Chat reserves estimated input + `max_tokens`, then keeps only what was used and refunds the rest
  (on a balance). Set `max_tokens` to bound one call.
