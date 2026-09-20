---
generated: '2026-09-19'
method: generated
name: Get a category recap and a ticker summary
description: Pull the AI-generated 12h/24h recap for a feed category and a 24-hour bullet summary for a token ticker.
api: openapi/itsgloria-ai-openapi.yml
operations: [getAvailableFeedCategories, getRecaps, getTickerSummary]
source: >-
  operationIds verified in openapi/itsgloria-ai-openapi.yml; timeframe table from
  https://docs.itsgloria.ai/gloria-data-platform/api-integration/#recaps-api.
---

# Get a category recap and a ticker summary

## Auth
- Any JWT (`Authorization: Bearer` or `?token=`) whose feed permissions include the category — `authentication/itsgloria-ai-authentication.yml`.

## Steps
1. **Pick the category and its window** — `getAvailableFeedCategories` returns each category's `recap_timeframe`: `12h` for `ai`, `ai_agents`, `crypto`, `defi`, `machine_learning`, `macro`, `tech`, `virtuals`; `24h` for `base`, `bitcoin`, `dats`, `ethereum`, `hyperliquid`, `on_chain_whale`, `perps`, `ripple`, `rwa`, `solana`; `token_listings` has no recap.
2. **Fetch the recap** — `getRecaps` (`GET /recaps`) with `feed_category` and `timeframe` (`12h` or `24h`). Response: `{feed_category, timeframe, recap, created_at}`; `404` when no recap exists for that pair.
3. **Summarise a ticker** — `getTickerSummary` (`GET /news-ticker-summary`) with `ticker` (symbol or name, e.g. `SOL`, `LayerZero`). Response `{summary}` is a bullet list combining Gloria news with web search; it can `500` transiently because of the web-search dependency — retry once after a delay.

## Errors
- `401`, `403`, `404`, `500` per `errors/itsgloria-ai-problem-types.yml`.

## Notes
- Both reads are also sold per request over x402 (`/recaps` $0.10, `/news-ticker-summary` $0.031) and surfaced as the MCP tools `get_news_recap` (free) and `get_ticker_summary` (x402) — see `mcp/itsgloria-ai-tool-crosswalk.yml`.
