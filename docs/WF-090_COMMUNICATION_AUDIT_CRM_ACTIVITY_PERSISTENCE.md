# WF-090 Communication Audit & CRM Activity Persistence

WF-090 is the persistence boundary for business communication audit events.

It normalizes inbound and outbound events and persists them as InsulaCRM activities.

## Captured audit fields

- direction
- channel
- event type
- delivery status
- request ID
- deterministic audit/idempotency key
- provider
- sender/recipient
- thread/conversation ID
- lead/property/deal IDs when available
- DNC suppression state
- provider receipt

## CRM boundary

WF-090 writes to:

`POST /api/v1/activities`

using:

- `INSULA_CRM_BASE_URL`
- `INSULA_CRM_API_KEY`

The API key is read only from runtime environment and is never accepted from the webhook payload or committed to GitHub.

## Failure behavior

If InsulaCRM persistence fails, WF-090 returns a failure result while preserving the normalized audit ID and provider receipt so the event can be reconciled later.

## Intended chain

Communication runtime/decision workflows
→ WF-090
→ InsulaCRM activity timeline
→ later reconciliation/case reconstruction.

WF-090 is inactive by default until the communication runtime and CRM environment are configured and tested.
