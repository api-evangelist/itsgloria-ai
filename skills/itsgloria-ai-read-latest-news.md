---
generated: '2026-09-19'
method: generated
name: Read the latest curated news
description: Obtain a token, discover feed categories, page through the curated crypto news feed and fetch one item's full record.
api: openapi/itsgloria-ai-openapi.yml
operations: [getPublicToken, getAvailableFeedCategories, getNews, getNewsById]
source: >-
  operationIds verified in openapi/itsgloria-ai-openapi.yml; parameters and defaults from
  https://docs.itsgloria.ai/gloria-data-platform/api-integration/.
---

# Read the latest curated news

Base URL `https://ai-hub.cryptobriefing.com` (the docs' published base; the spec's `servers[]` host does not resolve).

## Auth
- Anonymous, read-only: `getPublicToken` (`GET /auth/public-token`) returns `{"access_token": {"token": "<jwt>", ...}}`. The public token has no `allowed_feed_categories`, so category-scoped requests may answer `403`; use an API token or the SIWE flow (see `itsgloria-ai-wallet-auth-and-api-tokens.md`) for scoped access.
- Send the JWT as `Authorization: Bearer <jwt>` or as `?token=<jwt>`. See `authentication/itsgloria-ai-authentication.yml`.

## Steps
1. **Discover categories** — `getAvailableFeedCategories` (`GET /available-feed-categories`, no auth). Use the `code` values (`crypto`, `bitcoin`, `defi`, `macro`, ...).
2. **List news** — `getNews` (`GET /news`) with `feed_categories` (comma-separated codes), optional `keyword`, `from_date`/`to_date` (`YYYY-MM-DD`), `page` (default 1) and `limit` (default 20). The response is a bare JSON array of `NewsItem`; there is no `has_more` — stop when a page comes back short.
3. **Fetch one item** — `getNewsById` (`GET /news/{id}`) with the UUID `id` from step 2 to re-read a single item (same fields: `signal`, `sentiment`, `sentiment_value`, `feed_categories`, `short_context`, `long_context`, `sources`, `tokens`, `tweet_url`, `narrative_id`).

## Errors
- `401` missing/invalid token, `403` token not permitted for a requested feed category, `404` no data for the parameters. Errors are `{"detail": "..."}` — see `errors/itsgloria-ai-problem-types.yml`.

## Notes
- Read-only: no idempotency or reversibility concerns (`conventions/itsgloria-ai-conventions.yml`).
- `GET /news` is cached at the edge for ~30 s (`cache-control: public, s-maxage=30`).
- The same headlines are also purchasable without any token over x402 at `https://api.itsgloria.ai/news` ($0.03, USDC on Base) — see `x402/itsgloria-ai-x402.yml`.
