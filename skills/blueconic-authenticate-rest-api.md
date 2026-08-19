---
name: blueconic-authenticate-rest-api
description: Obtain and use an OAuth 2.0 access token for the BlueConic REST API v2, on the correct tenant host, with the right scopes. Use before any other BlueConic skill.
api: openapi/blueconic-oauth-2-0-api-openapi.yml
operations:
  - getToken
  - startAuthorizationCodeFlow
  - revokeToken
generated: '2026-08-13'
method: generated
source: openapi/blueconic-oauth-2-0-api-openapi.yml + https://support.blueconic.com/en/articles/248009-using-the-blueconic-rest-api-v2
---

# Authenticate against the BlueConic REST API v2

## Before you start

- **You need the tenant host.** BlueConic is single-tenant: every customer runs on its own
  host, e.g. `https://acme.blueconic.net`. The API base is `https://<tenant>.blueconic.net/rest/v2`.
  There is no shared public API host — never guess one, ask for it.
- **You need a registered application.** A BlueConic user with the "Applications" permission
  creates it under **Settings > Access management > Applications**, which yields a client ID
  and client secret. The scopes granted there, combined with the application's configured
  "run as user" permissions, are the ceiling on everything you can do.

## Choose the flow

| Situation | Flow | Operation |
|---|---|---|
| Server-to-server / machine-to-machine job | client credentials | `getToken` |
| A human must consent, on their own BlueConic account | authorization code + PKCE | `startAuthorizationCodeFlow` then `getToken` |

## Client credentials

1. `getToken` — `POST /rest/v2/oauth/token` on the tenant host, with the client ID and
   secret. Read the returned access token and its expiry.
2. Send `Authorization: Bearer <token>` on every subsequent request.
3. Refresh before expiry. Do not cache a token across tenants — a token is tenant-scoped.

## Authorization code

1. `startAuthorizationCodeFlow` — `GET /rest/v2/oauth/authorize`. Generate a code verifier and
   code challenge first: **PKCE is required**, and the registered client must have
   "Send Proof Key for Code Exchange" enabled.
2. The user authenticates on the BlueConic redirect page and consents.
3. `getToken` — exchange the authorization code plus the verifier for an access token and a
   refresh token.
4. **Handle refresh-token rotation.** Every refresh returns a NEW refresh token as well as a
   new access token. Store the new one; the old one is spent.
5. `revokeToken` — `POST /rest/v2/oauth/revoke` when the user disconnects. After revocation a
   fresh authorization grant is required.

## Rules

- Never put credentials in a URL. `Authorization: Bearer <token>` in the header.
- Ask for the narrowest scope set that does the job. Scopes are `read:<resource>` /
  `write:<resource>` — the full list is in `scopes/blueconic-scopes.yml`.
- On `401` the token is bad or expired: re-authenticate, do not retry the same token.
- On `403` the token is valid but the application lacks the scope or the run-as user lacks the
  permission. That is a configuration fix in BlueConic, not something to retry around.
- On `503` the server is busy — back off and retry. It is declared on 63 of the 64 operations.

## What this does not give you

There is no API key, no test-mode key, and no idempotency key. See
`conventions/blueconic-conventions.yml`.
