---
name: glianaai-provider
description: "Become a GlianaAI model provider: serve a chat model GlianaAI sells (Claude, GPT, Gemini, Grok, DeepSeek, GLM, Kimi, MiniMax and others) from your own GPU or a hosted OpenAI-compatible endpoint, check it meets the rules, pick a price that wins calls, list it on the GlianaAI dashboard and get paid in USDC per token. Use when someone wants to sell inference, earn from idle GPUs, or list an endpoint on GlianaAI."
license: MIT
---

# GlianaAI provider

GlianaAI routes each chat call to the cheapest route that can serve it. A provider lists an
OpenAI-compatible endpoint for models GlianaAI already sells; when it is the cheapest healthy
route, it answers the call and earns its own price for the tokens. Guide for humans:
https://ai.glianalabs.com/docs/provider

## 1. Read the live list first

Never use a remembered model list or price. Everything moves:

```bash
curl -s https://api.glianalabs.com/v1/market | jq '.models[] | {id, name, max, to_beat, cheapest_seller}'
```

Per model, in dollars per 1M tokens (`in_per_mtok` / `out_per_mtok`):

- `max`: the most a provider may ask (GlianaAI's own route cost). Higher is refused.
- `to_beat`: ask under this on both sides to get calls; another route is tried first at your price. `null` means only `max` applies.
- `cheapest_seller`: the cheapest provider already listed, if any.

Every chat model GlianaAI sells is listable. A model that is not in the list cannot be listed yet: the
human asks for it with the "Model not in the list?" form on https://dashboard.glianalabs.com/provider,
and GlianaAI adds it to its catalog first.

## 2. Serve the model

Own GPU, with vLLM (SGLang works the same way). Generate the key into the environment and do not
print it:

```bash
export SELLER_KEY=$(openssl rand -hex 24)
vllm serve <org>/<model> --api-key "$SELLER_KEY" --port 8000 --disable-log-requests
```

Check the model card for the memory it needs before promising anything: the listable models are
large mixture-of-experts models and usually need several GPUs. A hosted OpenAI-compatible
inference service also works.

## 3. Put it on https

The endpoint must be `https` on a domain name the human controls, with no IP address, no port and no
credentials in the URL, for example `https://llm.example.com/v1`. With Caddy:

```
llm.example.com {
  reverse_proxy localhost:8000
}
```

Any tunnel or reverse proxy that gives an https address on their own domain is fine. Addresses on
`glianalabs.com`, `utus.dev` and `workers.dev` are refused.

## 4. Check it the way GlianaAI will

```bash
# Plain: must return choices[0].message (text, or reasoning only is fine for a thinking model)
curl -s https://llm.example.com/v1/chat/completions \
  -H "authorization: Bearer $SELLER_KEY" -H "content-type: application/json" \
  -d '{"model":"<org>/<model>","messages":[{"role":"user","content":"Reply with: ok"}],"max_tokens":256}' | jq '.choices[0].message'

# Streaming: must answer with content-type text/event-stream
curl -sN -D - -o /dev/null https://llm.example.com/v1/chat/completions \
  -H "authorization: Bearer $SELLER_KEY" -H "content-type: application/json" \
  -d '{"model":"<org>/<model>","messages":[{"role":"user","content":"hi"}],"max_tokens":64,"stream":true}' | grep -i content-type
```

Rules a call must meet: an answer within 120 seconds; text only (requests with images never reach
providers); `usage` in the reply is welcome but checked against GlianaAI's own count.

## 5. Pick a price

- Both sides at or under `max`, and under `to_beat` to get calls.
- Work out the floor from the hardware: GPU cost per hour ÷ (output tokens per second × 3600) × 1,000,000 gives the cost per 1M output tokens. Below that the provider loses money on every call.
- Show the human the numbers and let them choose.

## 6. List it (the human does this)

Listing needs the human's Google sign-in at https://dashboard.glianalabs.com/provider. Give them
each value in its own code block: endpoint name, base URL (ending in `/v1`), and per model the
GlianaAI model id, the upstream name (the name the server uses, e.g. the vLLM model path) and both
prices. They paste the key themselves; never type, print or paste it for them.

GlianaAI sends each model a short test (up to 256 tokens), then its team reviews the endpoint. Calls
start once it is approved.

## 7. After it is live

- Earnings, health and payouts are on the provider page. Pay is per token at the provider's own price; reasoning tokens count; fractions of a cent add up.
- Three failed calls in a row rest a model until "Test again" passes. A failed call is not paid.
- To change a price: delete the endpoint and list it again (earnings stay owed).
- From $5 owed, the human asks for a payout: USDC on Solana or Base, sent by GlianaAI's team.

## Customers' data

Each request carries a customer's prompt. Use it only to answer that request: do not store, log,
share or train on prompts or answers. Keep request logging off (`--disable-log-requests` on vLLM).
