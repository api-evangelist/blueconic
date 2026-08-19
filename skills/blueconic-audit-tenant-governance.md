---
name: blueconic-audit-tenant-governance
description: Take a read-only inventory of a BlueConic tenant — audit events, users, roles, connections, listeners, dialogues, objectives — to answer who changed what, who has access, and what is running. Safe, read-only.
api: openapi/blueconic-audit-events-api-openapi.yml
operations:
  - getAuditEvents
  - getAllUsers
  - getAllRoles
  - getAllConnections
  - getConnectionRuns
  - getAllListeners
  - getAllDialogues
  - getAllObjectives
  - getDialogueStatistics
generated: '2026-08-13'
method: generated
source: openapi/blueconic-audit-events-api-openapi.yml + openapi/blueconic-users-api-openapi.yml + openapi/blueconic-connections-api-openapi.yml
---

# Audit a BlueConic tenant

Every operation here is a `GET`. Nothing in this skill mutates the tenant. Scopes required:
`read:audit-events`, `read:users`, `read:roles`, `read:connections`, `read:listeners`,
`read:dialogues`, `read:objectives`. Authenticate first — see
`blueconic-authenticate-rest-api`.

## Who changed what

`getAuditEvents` — `GET /auditEvents`. Bound the window with `fromDate` / `toDate` and page
with `startIndex` + `count`. Ask for the narrowest window that answers the question; the audit
log is the largest collection in the tenant.

## Who has access

1. `getAllUsers` — `GET /users` — every BlueConic user. `getOneUser` for one.
2. `getAllRoles` — `GET /roles` — the permission sets. `getOneRole` for the grants on one role.
3. Cross-reference: a role carrying "Applications" or "Authorize applications" is what lets a
   person mint API credentials. Call those out explicitly in any access review.

## What is running

- `getAllConnections` — `GET /connections` — the integrations moving data in and out, including
  Webhook connections. `getOneConnection` for configuration, `getConnectionRuns` —
  `GET /connections/{connection}/runs` — for execution history and failures.
- `getAllListeners` — `GET /listeners` — browser-side collection.
- `getAllDialogues` — `GET /dialogues` — the customer-facing experiences;
  `getDialogueStatistics` — `GET /reporting/dialogues` — for their performance.
- `getAllObjectives` — `GET /objectives` — the consent objectives, which is where a privacy
  review starts.
- `getAllPlugins`, `getAllModels`, `getAllNotebooks`, `getAllLifecycles`,
  `getAllContentStores`, `getAllRollups` complete the inventory.

## Rules

- Read-only means read-only. If the user asks you to fix something you found, stop and hand
  back to `blueconic-bulk-upsert-profiles` or to a human — do not improvise a write.
- Report findings ordered by severity, each with the operation and the ids you read them from.
- Audit output frequently contains user identities and profile identifiers. Treat it as PII:
  summarise, do not dump.
- `503` on any of these is back-pressure, not a failure. Back off and resume.
