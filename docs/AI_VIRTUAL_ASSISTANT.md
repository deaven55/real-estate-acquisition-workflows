# AI Virtual Wholesaling Assistant

## Purpose

WF-060 adds the command layer for the wholesaling operating system. It accepts a natural-language command or a structured intent, normalizes the request, identifies the intended operating capability, and routes the request to the appropriate controlled workflow.

The assistant is designed as a command center over the existing acquisition, underwriting, seller, buyer, transaction, CRM, control-plane, approval, audit, and recovery workflows.

## Supported assistant capabilities

- Portfolio and pipeline snapshots
- Lead lookup and follow-up queues
- Deal lookup and underwriting requests
- Buyer matching
- Production health and SLA review
- Approval-state review
- CRM activity routing
- Seller/buyer outreach requests
- Offer lifecycle requests
- Contract/transaction preparation
- Closing coordination

## Safety boundary

WF-060 is read-only by default.

Actions that can create external side effects are marked approval_required, including:

- send_offer
- contact_seller
- contact_buyer
- prepare_contract
- schedule_closing

Those actions must pass through WF-052 Human Approval & Action Authorization before execution. WF-060 does not itself send messages, execute contracts, schedule closings, make legal determinations, or claim funding/title verification.

## Routing contract

Request body may contain a command, optional structured intent, and canonical lead/property/deal identifiers.

The response includes:

- interpreted_intent
- intent_confidence
- execution_mode
- approval_required
- canonical identifiers
- missing context
- suggested workflow route
- dispatch readiness
- next action
- policy boundary

## Example commands

- "Show me my hottest deals."
- "Who needs seller follow-up today?"
- "Analyze this property and calculate MAO."
- "Find buyers for this contract."
- "Show me the current pipeline."
- "Are there any production blockers?"
- "What approvals are waiting?"
- "Send the offer to the seller." -> approval required
- "Contact the buyer." -> approval required
- "Prepare the assignment contract." -> approval required

## Architecture

User command
→ WF-060 AI Virtual Wholesaling Assistant
→ intent + identity + safety classification
→ existing workflow route
→ controlled execution
→ execution receipt / audit evidence
→ InsulaCRM and operational control plane

WF-060 is intentionally a command/routing contract, not an LLM vendor lock-in. A future LLM adapter can supply structured intent and tool arguments while WF-060 remains the policy boundary.

## Production requirements

Before production activation:

1. Configure the n8n webhook.
2. Connect the assistant to the chosen LLM/provider or UI command surface.
3. Provide canonical lead/property/deal IDs when available.
4. Configure InsulaCRM environment variables and API isolation.
5. Route action requests through WF-052.
6. Persist dispatch and reconciliation evidence through WF-055/WF-056.
7. Persist the assistant request/result in the audit timeline.
8. Run replay, duplicate, approval-expiry, and failure-recovery tests.

## Current status

WF-060 is implemented in the acquisition workflow repository on feature/insula-crm-integration. It establishes the AI assistant command contract and safety boundary. It does not claim that the conversational UI, LLM provider, or production deployment has already been connected.

## Dispatch and execution boundary

WF-061 adds the tool-dispatch boundary between assistant intent and workflow execution. It creates a deterministic request envelope containing request_id, idempotency_key, intent, workflow_route, canonical identity, authorization state, and receipt requirements.

The assistant therefore has two distinct stages:

1. WF-060 interprets the user's command and identifies the correct capability.
2. WF-061 validates the execution contract and permits dispatch only when the request is structurally valid and any required human approval is explicitly approved.

WF-061 does not execute external side effects itself. It produces the controlled execution envelope for the mapped workflow and requires the result to be recorded through the dispatch/receipt and reconciliation layers.

This makes the assistant auditable and replay-safe rather than allowing an LLM to directly call seller, buyer, contract, or closing actions.

## Operating context snapshot

WF-062 creates a normalized context package for the assistant from current pipeline, leads, deals, operating queue, exceptions, approvals, and identity-quality data supplied by the operating system.

It produces a compact decision context containing:

- hot/urgent deals
- seller follow-up candidates
- urgent operating queue items
- blocking exceptions
- pending human approvals
- stale active records
- canonical identity coverage
- the highest-priority supplied operating action
- explicit data gaps

This gives the assistant a structured answer source for commands such as “What should I work on right now?” without allowing the assistant to invent current deal state.
