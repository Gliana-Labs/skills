---
name: glianaai-web
description: "GlianaAI web and research: pay-per-call web and public-records tools through GlianaAI (api.glianalabs.com) — a page as clean markdown (JavaScript rendered), a screenshot, typed JSON pulled from a page by a schema you declare, web search, a YouTube video summarised without watching it, real books and journal articles to cite (free), and US company data from SEC filings (financials, filings, insider trades, 13F holdings, 8-K events, full-text search, answers with quotes)."
license: MIT
---

# GlianaAI web and research tools

One `POST https://api.glianalabs.com/v1/tools/<name>` per call (the free one is `GET`). Current
prices, inputs and example bodies: `GET https://api.glianalabs.com/v1/tools` (categories `web`,
`ai`, `filings`). Pay with `Authorization: Bearer $GLIANA_API_KEY` or a wallet (see the
`glianaai` skill).

## Pages

| Tool | Body | Returns |
|---|---|---|
| `scrape` | `{"url": "https://..."}` | The page as markdown, JavaScript included. Use when a plain fetch returns an empty shell. |
| `extract` | `{"url": "...", "schema": {"price": "the listed price in IDR", "in_stock": {"type": "boolean"}}}` | JSON matching YOUR schema: no guessing at field names |
| `screenshot` | `{"url": "...", "fullPage": true}` | A hosted PNG (width/height 320–1920) |
| `search` | `{"q": "...", "count": 5}` | Ranked results: title, url, snippet |

Prefer `extract` over `scrape` + parsing when you know which fields you want. Prefer `search`
only for things not in your training data; do not search what a free lookup answers (see
`glianaai-indonesia`).

## Video and citations

| Tool | Input | Returns |
|---|---|---|
| `youtube-summary` | `{"url": "<watch, youtu.be or shorts>", "style": "bullets"}` | A summary (bullets, paragraph or chapters), videos up to 60 minutes |
| `references` (free, `GET`) | `?q=rendang minangkabau&type=article&limit=3` | Real books and articles from OpenAlex and Crossref, with DOI |

Cite only works `references` returned. Never invent a citation.

## US companies, from SEC filings

| Tool | Body | Answers |
|---|---|---|
| `stock-brief` | `{"ticker": "AAPL"}` | Profile, latest reported financials and recent filings in one call |
| `stock-fundamentals` | `{"ticker": "AAPL", "concept": "Revenues"}` | Reported figures per fiscal period |
| `stock-filings` | `{"ticker": "AAPL", "form": "10-K"}` | Filings with links to the documents |
| `filing-digest` | `{"ticker": "AAPL", "form": "10-K", "question": "what does it say about tariffs?"}` | An answer with the passages it came from |
| `filing-search` | `{"query": "product recall", "form": "8-K"}` | Full-text search across filings since 2001 |
| `filing-events` | `{"ticker": "AAPL"}` | 8-K material events, item codes in words |
| `insider-trades` / `planned-insider-sales` | `{"ticker": "AAPL"}` | Form 4 trades / Form 144 notices |
| `ownership-changes` | `{"ticker": "AAPL"}` | 13D/13G holders above 5% |
| `institutional-holders` | `{"manager": "Berkshire Hathaway"}` | A fund's 13F holdings at a quarter end |

These are reported facts, not advice. Quote figures with their period and filing.
