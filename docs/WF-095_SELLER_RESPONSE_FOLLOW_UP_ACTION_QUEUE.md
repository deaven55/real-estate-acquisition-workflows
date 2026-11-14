# WF-095 Seller Response to Follow-Up Action Queue

## Purpose

WF-095 converts communication outcomes into a deterministic seller follow-up queue item.

## Routing

- Seller reply/inbound response -> WF-078 conversation response intelligence.
- Delivery failure -> WF-089 delivery reconciliation.
- DNC/opt-out -> suppress outbound.
- Successful outbound -> await seller response.
- Unclassified event -> record only.

## Safety

- Requires lead or deal identity.
- Requires thread continuity.
- Requires request or audit identity.
- DNC suppresses autonomous outbound.
- Seller responses are analyzed before generating a new outbound message.
- Delivery failures do not create a second send directly from the feedback event.
- Queue identity is deterministic for idempotent downstream handling.

## Webhook

`POST /webhook/wf-095-seller-response-follow-up-queue`

WF-094 now exposes WF-095 as the follow-up route for seller-response outcomes.
