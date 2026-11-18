# WF-088 Controlled Communication Outbound Test Gate

WF-088 provides a deliberately constrained outbound test path for the business communication runtime.

## Purpose

It allows a controlled email/SMS test only when all of these are true:

- `test_mode=true`
- recipient is explicitly present
- recipient is present in `COMMUNICATION_TEST_RECIPIENT_ALLOWLIST`
- DNC is false
- `request_id` is present
- `COMMUNICATION_RUNTIME_BASE_URL` is configured

WF-088 then invokes WF-082. WF-082 remains the authorization and dispatch authority and ultimately routes through WF-085.

## Safety properties

- Arbitrary production recipients are rejected.
- DNC suppression remains authoritative.
- Provider credentials are never accepted in the webhook payload.
- Test requests receive deterministic idempotency keys.
- A send is not considered successful without a downstream provider receipt.
- The workflow is inactive by default.
- Production traffic should not use WF-088 as its normal outbound path.

## Runtime configuration

Set the allowlist as a runtime secret/configuration value:

`COMMUNICATION_TEST_RECIPIENT_ALLOWLIST`

Example shape:

`test-recipient@example.com,+15551234567`

Do not commit real recipient data or provider secrets to GitHub.

## Validation sequence

1. Configure the communication runtime.
2. Configure one controlled test recipient in the runtime allowlist.
3. Run WF-086 readiness checks.
4. Run WF-087 dry-run contract scenarios.
5. Run WF-088 with `test_mode=true`.
6. Verify provider receipt, CRM activity, thread continuity and idempotency.
7. Only then move toward production autonomous communication.

WF-088 does not provision a provider, phone number, email identity, or sender domain.
