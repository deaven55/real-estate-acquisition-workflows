# Communication Runtime Integration — WF-085 / WF-086

The communication stack now has a concrete provider-runtime boundary plus a non-secret readiness gate.

## Runtime flow

WF-084 determines whether an inbound conversation may receive an autonomous response.

WF-082 is the authorization/dispatch policy boundary.

WF-085 is the transport adapter. It accepts only an already-authorized envelope and resolves AgenticMail credentials from n8n runtime configuration.

WF-086 is a readiness check. It reports whether the runtime is configured for email and/or SMS without returning secret values.

## Before production

1. Provision the business email and separate business SMS identity.
2. Configure `AGENTICMAIL_API_URL` and `AGENTICMAIL_API_KEY` in n8n runtime secrets.
3. Configure the AgenticMail email path (default: `/api/agenticmail/mail/send`).
4. Configure the actual SMS send path supplied by the selected provider/runtime.
5. Run WF-086 for `email` and `sms`.
6. Run controlled test messages.
7. Verify provider receipts, CRM activity, DNC suppression, idempotency, and thread continuity.
8. Enable autonomous communication only after those checks pass.

## Important boundary

A green WF-086 result means the runtime is configured, not that the provider account, sender identity, domain authentication, SMS registration, deliverability, or compliance configuration has been independently verified.

No provider secret, private phone number, or API credential belongs in GitHub.
