---
name: glianaai-crypto
description: "GlianaAI crypto: pay-per-call crypto market data and on-chain safety checks through GlianaAI (api.glianalabs.com) — spot token prices, DEX liquidity, DeFi TVL, gas in gwei and dollars, Hyperliquid perps and funding, fiat exchange rates, read-only JSON-RPC, swap and bridge quotes, an UNSIGNED swap transaction to sign yourself, what a contract can do to its users, and what a transaction's calldata really calls. Non-custodial: nothing here holds keys or moves funds. Paid per call with USDC (x402 on Base or Solana) or a GlianaAI key."
license: MIT
---

# GlianaAI crypto tools

Each tool is one `POST https://api.glianalabs.com/v1/tools/<name>` with a small JSON body, priced
per call (most cost a fraction of a cent). Prices, current inputs and an example body:
`GET https://api.glianalabs.com/v1/tools` (categories `market` and `security`).

## Paying

- **A GlianaAI key:** `Authorization: Bearer $GLIANA_API_KEY` (from https://dashboard.glianalabs.com).
- **A wallet, no account:** call without a key and pay the `402` challenge. For x402 clients the
  same tools are at `POST https://api.glianalabs.com/x402/tools/<name>` (USDC on Base or Solana).
- Never print or log a key, and never put a private key anywhere near these calls: none of
  them needs one.

## Market data

| Tool | Body | Use it to |
|---|---|---|
| `token-price` | `{"symbols": "BTC,ETH,SOL", "vs": "usd"}` | Value a holding or check a price before acting (cheap enough to poll) |
| `dex-pairs` | `{"query": "WIF"}` | See whether a token is actually tradeable on-chain, where, and how deep |
| `defi-tvl` | `{"protocol": "aave"}` or `{"chain": "Base"}` | Size a protocol or chain, from DefiLlama |
| `gas-price` | `{"chain": "base"}` | Decide whether to transact now; gwei and dollars |
| `perp-markets` | `{}` | Hyperliquid perps: price, 24h change, funding, open interest |
| `market-brief` | `{"chain": "base", "symbols": "BTC,ETH,SOL"}` | Prices, gas and DeFi TVL in one call |
| `exchange-rate` | `{"from": "USD", "to": "IDR,EUR", "amount": 100}` | Fiat conversion |
| `rpc` | `{"chain": "base", "method": "eth_blockNumber", "params": []}` | Read-only chain state without a node |

## Swaps (quote, build, check)

| Tool | Body | Returns |
|---|---|---|
| `swap-quote` | `{"chain": "base", "from": "USDC", "to": "ETH", "amount": "10"}` (+ `to_chain` to bridge) | Best routes across DEX aggregators and bridges |
| `swap-build` | `{"chain": "base", "from": "USDC", "to": "ETH", "amount": "10", "address": "0x<the user's wallet>"}` | An UNSIGNED transaction for that wallet to sign and send |

`swap-build` never signs or sends anything. Before the user signs what it returns:

1. `calldata-decode` with the transaction's `data` to confirm what it calls.
2. `contract-scan` on any token or contract you have not seen before.
3. Show the user the amounts, the route and the minimum received, and let them sign in their own
   wallet. Never ask for a seed phrase or private key.

## Safety checks before signing

| Tool | Body | Answers |
|---|---|---|
| `contract-scan` | `{"address": "0x...", "chain": "ethereum"}` | What a deployed contract can do to its users (mint, pause, blacklist, upgrade, fees), read from bytecode |
| `calldata-decode` | `{"calldata": "0x..."}` | Which function a transaction calls and with what arguments |

Treat a scan as evidence for the user, not a verdict: report what it found and let them decide.

## Not financial advice

Return data and its source and timestamp. Do not recommend buying, selling or sizing a position.
