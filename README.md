# GlianaAI Agent Skills

Agent Skills that teach coding agents (Claude Code, Cursor, Codex, Copilot and others) to use
[GlianaAI](https://ai.glianalabs.com): 150+ generative models and 70+ data tools behind one API,
paid per call. No subscription; the free Indonesian reference lookups need no key at all.

Skills follow the [Agent Skills](https://agentskills.io) format.

## Install

```bash
# Everything
npx skills add Gliana-Labs/skills

# One skill
npx skills add Gliana-Labs/skills --skill glianaai
```

Then give the agent a key: create one at https://dashboard.glianalabs.com (Google sign-in, top up
in rupiah by QRIS or bank transfer, or in USDC) and set it in your shell:

```bash
export GLIANA_API_KEY=gk_...
```

Prefer MCP? The skills also work with the GlianaAI MCP server: `https://mcp.glianalabs.com/mcp`
(hosted, free discovery) or `npx -y gliana-ai-mcp` (local, can spend from your key).

## Skills

| Skill | Use it for |
|---|---|
| [glianaai](skills/glianaai/SKILL.md) | Setup and the core loop every other skill builds on: find a model or tool, read its inputs, quote the price, run it, return the result |
| [glianaai-media](skills/glianaai-media/SKILL.md) | Images, video, speech, music and transcripts: picking a model, the fields that drive the price, reference files, cheaper video recipes |
| [glianaai-chat](skills/glianaai-chat/SKILL.md) | Claude, GPT, Gemini, Kimi and more through the OpenAI or Anthropic SDK, streaming, cost control, and Claude Code on a GlianaAI key |
| [glianaai-indonesia](skills/glianaai-indonesia/SKILL.md) | Free official lookups (regions, prayer times, holidays, BMKG, NIK/NPWP checks, QRIS, terbilang) and paid address parsing, ID OCR and KYC |
| [glianaai-crypto](skills/glianaai-crypto/SKILL.md) | Token prices, DEX liquidity, DeFi TVL, gas, perps, swap quotes and unsigned swap transactions, contract scans and calldata decoding |
| [glianaai-web](skills/glianaai-web/SKILL.md) | Scrape, extract by schema, screenshot, search, YouTube summaries, real citations, and US company data from SEC filings |

Nothing in these skills hardcodes a model list or a price: they tell the agent to read the live
catalogue (`/v1/models`, `/v1/tools`, `/v1/price`), which is always current.

## Safety

- Discovery and pricing are free; every paid call is priced before it runs, and the skills tell
  the agent to state the price before spending more than a few cents.
- Keys are read from the environment and never printed.
- Crypto tools are non-custodial: nothing signs or sends a transaction for you.

## Links

- Docs: https://ai.glianalabs.com/docs · API reference: https://ai.glianalabs.com/reference
- OpenAPI: https://api.glianalabs.com/openapi.json
- MCP server: https://github.com/Gliana-Labs/gliana-mcp

MIT licensed. Made by PT Glianalabs Algoritma Desentralisasi, Indonesia.
