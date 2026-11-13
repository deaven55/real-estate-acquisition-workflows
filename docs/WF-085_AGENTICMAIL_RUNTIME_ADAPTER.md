# WF-085 — AgenticMail Provider Runtime Adapter

## Purpose

WF-085 is the provider-runtime boundary for autonomous business email/SMS. It receives an already-authorized communication envelope from WF-082/WF-084, validates the request, resolves AgenticMail runtime configuration, sends through the configured AgenticMail API endpoint, and returns a normalized provider receipt.

AgenticMail documents a REST API under `/api/agenticmail`, including `POST /mail/send` for email. SMS/phone access is provider-backed and should remain configurable rather than hardcoding an unverified SMS route into the acquisition OS. citeturn884773search0turn884773search1

## Runtime configuration

Configure these values in the n8n runtime/environment or an equivalent secret manager. Do not commit secrets.

- `AGENTICMAIL_API_URL` — AgenticMail base URL.
- `AGENTICMAIL_API_KEY` — scoped AgenticMail agent key.
- `AGENTICMAIL_EMAIL_SEND_PATH` — optional; defaults to `/api/agenticmail/mail/send`.
- `AGENTICMAIL_SMS_SEND_PATH` — required only when SMS is enabled; intentionally provider/runtime configurable.

WF-085 never accepts an API key from the webhook body.

## Dispatch contract

Required:

- `channel`: `email` or `sms`
- `action`: `send` or `reply`
- `request_id`
- `idempotency_key`
- `recipient`
- `message`
- `authorization.authorized=true` (or the equivalent approved autonomous authorization envelope)

Email send additionally requires `subject`.

The adapter hard-blocks DNC/suppressed recipients and invalid authorization envelopes.

## Receipt contract

WF-085 returns:

- `status`: `sent` or `transport_error`
- `provider`: `agenticmail`
- `channel`
- `action`
- `request_id`
- `idempotency_key`
- `thread_id` when supplied
- `provider_status_code`
- `provider_response`
- `receipt_at`

The upstream workflow remains responsible for CRM activity/audit persistence and retry/reconciliation policy.

## Deployment sequence

1. Provision the business email and SMS identity/provider.
2. Run AgenticMail and obtain the scoped agent API key.
3. Set the runtime variables above in n8n.
4. Import/activate WF-085.
5. Point WF-082's approved outbound dispatch to WF-085.
6. Point the provider's inbound webhook/event path to WF-083.
7. Run a controlled email test and a controlled SMS test.
8. Confirm provider receipt, CRM activity, idempotency behavior, DNC suppression, and thread continuity before enabling autonomous production traffic.

AgenticMail supports external email through relay/domain gateway modes and documents SMS integrations including Google Voice and 46elks; the OS therefore keeps the communication boundary provider-agnostic while using AgenticMail as the current integration target. citeturn884773search0
