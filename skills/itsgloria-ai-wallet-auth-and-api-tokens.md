---
generated: '2026-09-19'
method: generated
name: Authenticate with a wallet and manage API tokens
description: Complete the SIWE (EIP-4361) login to get a JWT, then mint, list and revoke long-lived category-scoped API tokens.
api: openapi/itsgloria-ai-openapi.yml
operations: [getNonce, verifySignature, createApiToken, listApiTokens, revokeApiToken]
source: >-
  operationIds verified in openapi/itsgloria-ai-openapi.yml; flow from the spec's info.description
  and https://docs.itsgloria.ai/gloria-data-platform/api-integration/#wallet-authentication.
---

# Authenticate with a wallet and manage API tokens

## Steps
1. **Get a nonce** — `getNonce` (`GET /auth/nonce`) returns `{"nonce": "..."}`.
2. **Sign** — build an EIP-4361 (Sign-In with Ethereum) message containing the nonce and sign it with the wallet's private key. The wallet itself is the credential; never send the private key anywhere.
3. **Verify** — `verifySignature` (`POST /auth/verify`) with `{"message": "<SIWE message>", "signature": "<hex>"}` returns `{"access_token": "...", "is_admin": false}`. `401` means the signature or nonce did not verify — restart from step 1.
4. **Mint a token** — `createApiToken` (`POST /user/api-tokens`, Bearer JWT) with `{"name": "...", "allowed_feed_categories": ["crypto", "macro"]}` returns an `ApiToken` (`token`, `issued_at`, `expires_at`). The token is scoped to those categories for its whole life; requests for other categories answer `403`.
5. **Audit** — `listApiTokens` (`GET /user/api-tokens`).
6. **Revoke** — `revokeApiToken` (`DELETE /user/api-tokens`); `400` = token not found or not yours.

## Idempotency and reversibility
- No idempotency key exists (`conventions/itsgloria-ai-conventions.yml`, `idempotency.coverage: none`): a retried step 4 mints a second token. List before retrying.
- Revocation is the reversal of minting; no window is stated (`reversibility.grade: documented`).

## Notes
- Tokens can also be created in the browser at `https://www.itsgloria.ai/api-keys-new`.
- The hosted MCP server needs no token for its free tools; the stdio package (`gloria-mcp` on PyPI) needs one in `GLORIA_API_TOKEN`.
