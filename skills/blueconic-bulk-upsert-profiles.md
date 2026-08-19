---
name: blueconic-bulk-upsert-profiles
description: Create, update and delete BlueConic customer profiles in bulk through PUT /profiles, handling the 1000-entry cap, the per-entry result array, and the 429 backoff contract. Use for imports, syncs and enrichment writes.
api: openapi/blueconic-profiles-api-openapi.yml
operations:
  - createUpdateDeleteProfiles
  - getAllProfileOrGroupProperties
  - createUpdateProfileOrGroupProperty
  - getOneProfile
  - createUpdateDeleteGroups
generated: '2026-08-13'
method: generated
source: openapi/blueconic-profiles-api-openapi.yml + openapi/blueconic-properties-api-openapi.yml
---

# Bulk create, update and delete BlueConic profiles

Scopes required: `write:profiles` (plus `read:profile-properties`, and
`write:profile-properties` only if you must define a new property first). Authenticate first
— see `blueconic-authenticate-rest-api`.

## Steps

1. **Resolve the target properties.** `getAllProfileOrGroupProperties` — `GET /profileProperties`.
   Map every incoming field to a real BlueConic property id before you write anything.
2. **Create a missing property, deliberately.** `createUpdateProfileOrGroupProperty` —
   `PUT /profileProperties/{propertyId}`. Do this only when the user has asked for it: a new
   property changes the tenant's data model for everyone. Note that properties created by a
   plugin cannot be created or updated this way (the API returns `400`).
3. **Write in batches.** `createUpdateDeleteProfiles` — `PUT /profiles`. One call expresses
   create, update AND delete.
   - **Hard cap: 1000 entries per request.** Above it the API returns `413` — and the body
     still contains the `BulkResultBean` array for the entries that *were* processed. Read it;
     do not assume the whole batch failed.
   - Always inspect the per-entry result array on a `200` too. A `200` is not proof that every
     entry succeeded.
4. **Groups follow the same shape.** `createUpdateDeleteGroups` — `PUT /groups` — with
   `write:groups`, capped and typed identically.
5. **Verify a sample.** `getOneProfile` — `GET /profiles/{profileId}` — on a few written ids.

## Concurrency and retries — read this before writing a retry loop

- **There is no idempotency key.** BlueConic documents none and the specification declares
  none. A blind retry of a bulk write is a real re-application of that write. Make your
  batches naturally idempotent (set absolute values rather than increments) and key each
  batch to a stable identifier so a resend is safe.
- `429` means too many concurrent requests. BlueConic's own guidance: "implement a request
  queue with an exponential backoff algorithm." Serialise your writes; do not fan out.
- `503` means the server is busy. Back off and retry the same batch.
- `413` means you exceeded the cap — split, do not retry unchanged.
- There are no `X-RateLimit-*` or `Retry-After` headers to read. Your backoff has to be
  self-driven.

## Rules

- This writes real customer PII into a production CDP. Confirm the tenant host before the
  first call, and never point an import at a host you were not given.
- Deletes inside a bulk PUT are irreversible. Confirm the count with the user first.
- The MCP server does **not** expose delete operations; if you are working through MCP, deletes
  are simply not available. See `mcp/blueconic-mcp.yml`.
