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

## Session continuity

WF-063 provides the assistant with a normalized session context so multi-turn commands can carry forward the active lead, property, deal, buyer, session goal, unresolved operating items, approval state, and operating-context references.

The continuity boundary is explicit: references such as “this deal,” “that property,” “continue,” or “follow up” must resolve against session context before dispatch. If the referenced identity is missing, the assistant must resolve it rather than guess. Explicit identity changes take precedence and must be recorded for audit.

WF-063 does not execute side effects. It hands normalized context to WF-060 for intent interpretation and WF-061 for routing/authorization, while WF-062 supplies the current operating snapshot.

## Unified response composition

WF-064 is the presentation/decision layer for the assistant. It combines the command result, operating context, session continuity, blockers, missing context, pending approvals, prioritized actions, and authorization state into one deterministic assistant response envelope.

The response explicitly distinguishes:

- ready: request can continue through the controlled dispatch path
- blocked: operating exceptions prevent safe continuation
- needs_context: required identity or data is missing
- approval_required: a risky action lacks explicit human approval
- error: the supplied result did not complete safely

WF-064 is compose-only and cannot create external side effects. This keeps natural-language presentation separate from execution authorization.

## End-to-end assistant orchestration

WF-065 is the orchestration gateway for the assistant lifecycle. It establishes the session ID, request ID, deterministic idempotency key, intent/approval state, and explicit workflow handoff sequence across WF-063, WF-060, WF-062, WF-061, and WF-064.

The gateway is orchestration-only. It does not execute seller/buyer/contract/closing side effects. Controlled actions still require WF-061 authorization and the applicable human approval state. Every request carries audit identifiers so the assistant lifecycle can be reconstructed and replayed safely.

## Tool/data adapter contract

WF-066 defines the controlled adapter boundary between the assistant and the wholesaling operating system. It maps supported intents to the existing workflow/CRM routes and standardizes request_id, session_id, canonical identity, deterministic idempotency, read-only versus side-effect capability, approval state, and receipt requirements.

Supported adapter routes include portfolio analytics, CRM lead lookup/activity, operating queue, deal/control-plane lookup, underwriting, buyer matching, pipeline status, production readiness, approval status, offer lifecycle, buyer disposition, transaction tracking, and closing coordination.

WF-066 is a contract gate rather than a direct external executor. It prevents missing identity/contract fields from reaching downstream tools and preserves the read-only-by-default and human-approval boundaries.

## CRM and operating-data retrieval adapter

WF-067 defines the standardized data-retrieval contract used by the assistant before live execution is wired. It covers InsulaCRM lead/property/activity reads and updates plus adapters for deals, buyers, operating queue, approvals, pipeline, portfolio analytics, and production readiness.

The adapter preserves request/session identity, source-of-truth, provenance, retrieved-at metadata, record counts, and no-inference behavior. InsulaCRM reads use the known API contract and base URL; non-CRM resources route to their established workflow gates. WF-067 does not claim live execution merely because a route is defined.

Read operations remain the default. Any side-effect-capable operation is approval-controlled and must not execute without explicit human approval.

## Live InsulaCRM read adapter

WF-068 is the first live-capable assistant data adapter. It accepts a lead read request, validates the runtime CRM base URL/API key and request/session/lead identity, then performs a read-only GET against the known InsulaCRM endpoint when the runtime contract is configured. It never performs a write and does not infer missing CRM data.

The workflow is intentionally gated: a defined endpoint is not treated as live until the required runtime environment values are present. The response preserves request/session identity, source-of-truth, provenance, and no-inference behavior.

## Deal viability over rigid discount rules

WF-069 establishes the underwriting principle that the 70% rule is a secondary heuristic, not the primary acquisition decision. The primary question is whether the deal can work economically after realistic value, repairs, closing/holding costs, target profit, buyer demand, and exit strategy are considered.

The adapter supports property-type-aware logic and explicitly distinguishes land/vacant-property analysis from residential ARV analysis. A deal may remain viable above or below a 70% reference point when the underlying economics support it. It must not reject a deal solely because it misses 70%.

## Deal analysis explanation

WF-070 adds an explanation layer after economic underwriting. It identifies why a deal works or does not work, highlights strengths and risks, calculates the offer-to-MAO spread and implied margin when inputs permit, and proposes practical levers such as price/terms adjustment, repair verification, comp validation, and buyer-demand validation.

The assistant should explain the economics behind a recommendation rather than treating the 70% heuristic as a pass/fail rule. Missing inputs remain explicit and recommendations are not presented as verified legal, title, or funding conclusions.

## Comparable sales and market evidence

WF-071 adds an evidence-first adapter for comparable sales and market evidence. It preserves raw comp fields and provenance, separates verified from unverified evidence, summarizes comparable sale prices, and uses property-type-aware methodology. Vacant land uses land-sales/price-per-acre analysis rather than residential ARV assumptions.

WF-071 does not claim a guaranteed market value. Evidence quality and missing data remain explicit inputs to downstream underwriting and deal viability decisions.
