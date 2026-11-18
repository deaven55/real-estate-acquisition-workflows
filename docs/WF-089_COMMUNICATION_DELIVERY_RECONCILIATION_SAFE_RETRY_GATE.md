# WF-089 Communication Delivery Reconciliation & Safe Retry Gate

WF-089 handles failed outbound communication receipts without creating duplicate sends or bypassing DNC.

## Retry policy

A retry is allowed only when:

- the prior status is a known retryable transport outcome
- request ID is present
- the original idempotency key is present
- channel and recipient are present
- DNC suppression is false
- the retry count has not reached `COMMUNICATION_MAX_RETRIES` (default 2)

## Safety rules

- Reuses the original idempotency key.
- Never retries DNC/suppressed recipients.
- Never treats a failed delivery as sent.
- Never retries after the configured limit.
- Provider receipts remain authoritative.
- Failed retries remain failures and are escalated rather than recursively generating new keys.

## Runtime

WF-089 calls WF-082, preserving the existing authorization/dispatch controls. Provider credentials remain outside GitHub.

Runtime configuration:

`COMMUNICATION_RUNTIME_BASE_URL`

Optional:

`COMMUNICATION_MAX_RETRIES`

## Operational sequence

WF-085 provider receipt
→ WF-089 reconciliation
→ safe retry through WF-082 when permitted
→ WF-085 provider receipt
→ persist/reconcile outcome

WF-089 is inactive by default and should be wired to the operational failure/reconciliation path after controlled testing.
