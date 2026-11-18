# WF-096 Seller Follow-Up Action Engine

WF-096 turns the seller follow-up queue into a deterministic next-best-action decision.

## Decisions

- Seller reply -> analyze reply before response.
- Delivery failure -> reconcile through WF-089.
- DNC/opt-out -> suppress outbound.
- Successful outbound awaiting response -> wait or schedule follow-up.
- Missing identity -> block state mutation.

The engine does not invent seller/property facts and does not bypass communication authorization.

Webhook: `POST /webhook/wf-096-seller-follow-up-action-engine`
