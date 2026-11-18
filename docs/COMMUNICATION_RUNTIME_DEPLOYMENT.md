# Communication Runtime Integration — WF-082 / WF-085 / WF-086 / WF-087

The communication stack now has a concrete executable provider-runtime boundary plus a non-secret readiness and contract-test layer.

## Runtime flow

WF-083 receives and normalizes inbound business SMS/email.

WF-084 determines whether the inbound conversation may receive an autonomous response.

WF-082 is the authorization/dispatch policy boundary. When autonomous dispatch is allowed, WF-082 calls WF-085 through the configured n8n runtime webhook URL and preserves the deterministic idempotency key and conversation thread.

WF-085 is the provider transport adapter. It accepts only an already-authorized envelope and resolves AgenticMail credentials from n8n runtime configuration.

WF-086 is a readiness check. It reports whether the runtime is configured for email and/or SMS without returning secret values.

WF-087 is a contract-only test layer. It validates response-mode policy without sending external traffic.

## Runtime configuration

Configure this in n8n runtime secrets/environment, never in GitHub:

- `COMMUNICATION_RUNTIME_BASE_URL` — base URL used by WF-082 to reach the WF-085 webhook.
- `AGENTICMAIL_API_URL`
- `AGENTICMAIL_API_KEY`
- `AGENTICMAIL_EMAIL_SEND_PATH` (default: `/api/agenticmail/mail/send`)
- `AGENTICMAIL_SMS_SEND_PATH` — required for SMS.

## Before production

1. Provision the business email and separate business SMS identity.
2. Configure the runtime values above.
3. Run WF-086 for email and SMS.
4. Run WF-087 contract scenarios.
5. Run one controlled email test and one controlled SMS test.
6. Verify provider receipts, CRM activity, DNC suppression, idempotency, and thread continuity.
7. Enable autonomous communication only after those checks pass.

## Failure behavior

WF-085 returns HTTP 502 when the provider transport itself fails.

WF-082 does not mark a message as sent unless WF-085 returns a successful provider receipt.

Failed transport attempts retain the same idempotency key so the retry/reconciliation layer can safely reason about the attempt.

## Important boundary

A green WF-086 result means the runtime is configured, not that the provider account, sender identity, domain authentication, SMS registration, deliverability, or compliance configuration has been independently verified.

No provider secret, private phone number, or API credential belongs in GitHub.
