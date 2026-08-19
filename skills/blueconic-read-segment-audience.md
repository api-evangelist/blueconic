---
name: blueconic-read-segment-audience
description: Find a BlueConic segment and page through the customer profiles in it, returning only the profile properties you need. Use for audience export, audience analysis, or answering "who is in this segment".
api: openapi/blueconic-segments-api-openapi.yml
operations:
  - getAllSegments
  - getProfilesInSegment
  - searchProfiles
  - getOneProfile
  - getAllProfileOrGroupProperties
generated: '2026-08-13'
method: generated
source: openapi/blueconic-segments-api-openapi.yml + openapi/blueconic-profiles-api-openapi.yml
---

# Read a BlueConic segment's audience

Scopes required: `read:segments`, `read:profiles`, and `read:profile-properties` if you need
the property dictionary. Authenticate first — see `blueconic-authenticate-rest-api`.

## Steps

1. **Learn the vocabulary.** `getAllProfileOrGroupProperties` — `GET /profileProperties` —
   returns every profile property defined in the tenant with its id. BlueConic property ids
   are tenant-specific; never assume `email` or `firstname` exists. Use
   `getOneProfileOrGroupProperty` for a single property's definition.
2. **Find the segment.** `getAllSegments` — `GET /segments` — lists segments with their ids.
   Match on the name the user gave you; do not construct a segment id.
3. **Page the members.** `getProfilesInSegment` — `GET /segments/{segment}/profiles`.
   - `properties` — restrict the payload to the property ids you actually need. Do this. A
     BlueConic profile can be very wide.
   - `startIndex` + `count` — offset pagination. Advance `startIndex` by `count` until a short
     page comes back.
   - `cursor` — where the operation exposes it, prefer it over deep offsets for large segments.
4. **Or query directly.** `searchProfiles` — `GET /profiles` — takes `filterType` /
   `filterValue` / `refinement` when you need a query rather than a saved segment.
5. **Drill into one person.** `getOneProfile` — `GET /profiles/{profileId}`. Use `expand` to
   pull referenced objects inline.

## Rules

- **This is PII.** Only request the properties the task needs, never the whole profile "just
  in case", and never write profile values into logs, filenames or intermediate artifacts.
- Respect `maxHitsAllowed` where the operation declares it, rather than paging past it.
- `503` means the server is busy: back off exponentially and resume from the same
  `startIndex`. All these operations are safe to retry — they are reads.
- There is no total-count field in the envelope. Stop when a page returns fewer than `count`
  items.
- Related history: `getProfileEvents` (`GET /profileEvents/{profileId}`) for a profile's event
  stream, `getTimelineEventTypes` for the event-type dictionary.
