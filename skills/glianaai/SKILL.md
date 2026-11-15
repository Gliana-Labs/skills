---
name: glianaai
description: "GlianaAI (api.glianalabs.com): load this whenever the user mentions GlianaAI, Gliana or api.glianalabs.com, or a task needs an AI model or data tool the agent cannot run itself (images, video, speech, music, transcripts, another LLM, scraping, screenshots, market data, ID OCR, Indonesian reference data such as prayer times or region codes). Setup (API key, MCP or plain HTTP), finding the right model or tool from the live catalogue, quoting the price before spending, running it, returning the result. The glianaai-* skills hold the details per area."
license: MIT
---

# GlianaAI

One HTTP API for 150+ generative models and 70+ utility tools, paid per call. No subscription:
each call is priced up front and taken from a prepaid balance (an API key) or paid from a crypto
wallet. Discovery and pricing are free; running something costs money, so quote first.

Base URL: `https://api.glianalabs.com`

## Setup

Pick one. All three reach the same catalogue and the same prices.

1. **MCP (simplest in an agent).** Hosted, nothing to install, free discovery tools:
   `https://mcp.glianalabs.com/mcp`. For paid calls use the local server, which can spend:
   `npx -y gliana-ai-mcp` with `GLIANA_API_KEY` set (Claude Code plugin:
   `/plugin install gliana-ai --marketplace Gliana-Labs/gliana-mcp`).
2. **HTTP with an API key.** Send `Authorization: Bearer $GLIANA_API_KEY` on every paid call.
   Keys start with `gk_` and come from https://dashboard.glianalabs.com (Google sign-in; top up
   in rupiah by QRIS or bank transfer, or in USDC).
3. **HTTP with a wallet, no account.** Call without a key and the API answers `402 Payment
   Required` with a payment challenge (MPP on `/v1/*`; x402 on `/x402/<model>` and
   `/x402/tools/<name>`; USDC on Base or Solana). Only for clients that can sign payments.

Rules for the key:
- Read it from the environment (`GLIANA_API_KEY`). Never print it, echo it, write it into a file
  that gets committed, or paste it into a prompt or URL.
- If no key and no wallet is configured, stop and tell the user how to get one (point 2). Do not
  invent a key or retry without one.

## The loop

1. **Find it.** Never assume a model or tool id from memory: ids and prices change and are only
   current on the live endpoints. GlianaAI has utility tools as well as models: before
   concluding it cannot do something, check `GET /v1/tools` and `/v1/consult`, not only
   `/v1/models`.
   - `GET /v1/consult?intent=<what you want, percent-encoded>` ranks candidates (models, tools,
     recipes) for a plain-language goal. Treat it as a shortlist: check each candidate's category
     matches the task.
   - `GET /v1/models` lists every model with `id`, `name`, `category`, `unit`,
     `unitPriceMicroUsd`. Categories: `text-to-image`, `image-to-image`, `text-to-video`,
     `image-to-video`, `video-to-video`, `tts`, `stt`, `music`, `text`. It takes no filter
     parameters; filter it yourself, cheapest first:
     `curl -s https://api.glianalabs.com/v1/models | jq -c '[.models[] | select(.category=="text-to-image")] | sort_by(.unitPriceMicroUsd) | .[:5][] | {id, unitPriceMicroUsd}'`
   - `GET /v1/tools` lists every utility tool with `path`, `method`, `priceMicroUsd`, an
     `example` body and `guidance` on when to use it.
2. **Read the inputs.** `GET /v1/schema?model=<id>` returns `required` and `props` (types,
   enums, ranges, defaults). A tool's `example` in `/v1/tools` is its input shape.
3. **Quote it.** `GET /v1/price?model=<id>&<billing fields>` returns `costMicroUsd` for exactly
   that request. Billing fields are the ones that change the price: `duration`, `resolution`,
   `quality`, `text` (speech length), `music_length_ms`, `target`. Pass the values you will
   send: an unset `duration` or `resolution` is billed at the most expensive option.
4. **Ask before spending** when the call costs more than a few cents, when the user did not
   name the model, or when you will make many calls. State the model and the price.
5. **Run it.**
   - Any model: `POST /v1/infer` with `{"model": "<id>", ...inputs}`.
   - A tool: its `path` and `method` from `/v1/tools`.
   - Chat models also speak the OpenAI and Anthropic formats, and `/v1/embeddings` (bge-m3) gives vectors (see `glianaai-chat`).
6. **Return the result.** The response is `{"output": ..., "costMicroUsd": n}`. Media outputs
   are stable URLs on `api.glianalabs.com/v1/media/...`: give the user the link (or download it
   into the project if they asked for a file). Report what it cost.

Money is in micro-USD everywhere: 1,000,000 = $1.00.

## Files as inputs

Fields that take an image, audio or video take a public URL. For a local file, upload it first,
free: `POST /v1/media` with the raw bytes and the right `content-type` (png, jpeg, webp, gif,
mp3, wav, ogg, mp4, webm, quicktime) returns `{"url": ...}`; pass that url.

## Errors

| Status | Code | Meaning |
|---|---|---|
| 400 | `missing_input` | A required field is missing or invalid. Nothing was charged. |
| 401 | `invalid_key` | Unknown or revoked key. |
| 402 | `insufficient_balance` | The balance is lower than the price. Tell the user to top up. |
| 402 | `daily_cap_reached` | The key's daily spending cap (default $5) is used up; resets 00:00 UTC. |
| 402 | (no key) | A payment challenge: no key was sent. |
| 503 | `model_unavailable` | Temporarily off sale. Nothing was charged; pick another model. |
| 502 | `inference_failed` | The provider failed. The charge is refunded to the balance. |

Never retry a paid call in a loop. One retry after a 502 is reasonable; after that, report it.

Check the balance from code: `GET /v1/keys/balance` with the key.

More: [references/api.md](references/api.md) · docs https://ai.glianalabs.com/docs ·
machine-readable https://api.glianalabs.com/openapi.json
