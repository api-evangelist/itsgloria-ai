---
generated: '2026-09-19'
method: generated
name: Stream news in real time over WebSocket
description: Open the authenticated WebSocket feed, subscribe to feed categories, handle data/ping/error frames and close cleanly.
api: openapi/itsgloria-ai-openapi.yml
operations: [getAvailableFeedCategories, websocketFeed]
source: >-
  websocketFeed verified in openapi/itsgloria-ai-openapi.yml; message vocabulary and lifecycle from
  https://docs.itsgloria.ai/gloria-data-platform/api-integration/#news-websocket-api, captured in
  asyncapi/itsgloria-ai-websocket-feed.yml.
---

# Stream news in real time over WebSocket

## Auth
- A JWT with feed permissions for every category you subscribe to, passed as `?token=<jwt>`. WebSocket streaming is listed as Enterprise-only on the pricing page.

## Steps
1. **Choose categories** — `getAvailableFeedCategories` for the `code` list.
2. **Connect** — `websocketFeed`: `wss://ai-hub.cryptobriefing.com/ws/feed?token=<jwt>` (the docs' host; the spec's `ai.gloriaterminal.com` does not resolve). Expect `{"type":"connected","message":"Connection established."}`.
3. **Subscribe** — send `{"type":"subscribe","feed_category":"crypto"}` per category; expect `{"type":"subscribed","feed_category":"crypto"}`.
4. **Consume** — each `{"type":"data","content":{...}}` carries a `NewsItem` identical to the REST `/news` shape.
5. **Keep alive** — the server sends `{"type":"ping","timestamp":n}` every 30 s; reply `{"type":"pong","timestamp":n}` or the connection closes after 5 minutes idle.
6. **Leave** — send `{"type":"unsubscribe","feed_category":"crypto"}` and close.

## Errors
- `{"type":"error","error":"Permission denied","details":"No permission for crypto feed"}` when the token lacks that category.
- Close code `1008` = missing/invalid token (re-authenticate); `1000` = normal closure (reconnect with backoff).
