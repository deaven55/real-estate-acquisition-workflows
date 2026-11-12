# Real Estate Acquisition. Wholesaling Seller-Side Automation (n8n)

### 7 production n8n workflows. 356 nodes. The operations backbone of a real estate wholesaling business, from raw property data to a booked seller conversation.

[![n8n](https://img.shields.io/badge/n8n-workflows-FF6D5A?style=for-the-badge&logo=n8n&logoColor=white)](https://n8n.io)
[![Nodes](https://img.shields.io/badge/total_nodes-356-6366F1?style=for-the-badge)](#)
[![Workflows](https://img.shields.io/badge/workflows-7-10B981?style=for-the-badge)](#)
[![Status](https://img.shields.io/badge/status-production-success?style=for-the-badge)](#)

---

## What This Is

This repository contains 7 real production n8n workflows that run the operations of a real estate wholesaling business. Together they form a complete pipeline: pull distressed-property data, filter and skip-trace owners, import qualified leads into the CRM, send the first TCPA-compliant outbound text, run a multi-day follow-up sequence, classify every inbound reply into 20+ intents, and continuously clean the list.

Every workflow here is sanitized. API keys, tokens, CRM endpoints, Convex deployment URLs, phone numbers, and client identity have been replaced with placeholders. The logic, structure, node wiring, and code-node JavaScript are intact and importable.

> These are shared for portfolio and reference purposes. Import them, read the patterns, adapt the ideas. To run them you'll wire in your own Convex deployment, Close CRM, Tracerfy, BatchData, and Slack credentials (see Setup).

---

## The Pipeline

```
BatchData API (distressed property scrape, tier-based)
      │
      ▼  05-batchdata-weekly-staging
Convex staging (dedup, pagination, cost tracking)
      │
      ▼  07-ghost-rider-skiptrace-dispatch
Tracerfy skip trace (corporate/govt/bank filter, address standardization)
      │
      ▼  04-skiptrace-results-router (webhook)
Phone / email / phoneless lead staging back to Convex
      │
      ▼  02-ghost-rider-crm-import
Seller lead processing
      │
      ├──▶  00-insula-crm-lead-upsert-adapter
      │         │
      │         └──▶ InsulaCRM canonical seller/property record
      │
      ├──▶  03-seller-followup-engine (multi-day drip, rate-limited)
      └──▶  01-seller-response-handler (20+ intent classifier on inbound replies)
                                    │
                                    ▼
                         06-lead-cleanup-sms-dnc
```

---

## InsulaCRM Integration

InsulaCRM is the target canonical CRM for this acquisition stack. The integration is being introduced without immediately removing the existing staging and messaging services.

### Reusable adapter

`workflows/00-insula-crm-lead-upsert-adapter.json`

This workflow is designed to be called by the acquisition workflows after lead normalization. It:

- normalizes first/last name, phone, email, source, parcel ID, distress metadata, and property address;
- sends `POST /api/v1/leads` to InsulaCRM;
- authenticates with `X-API-Key`;
- sends a deterministic `Idempotency-Key`;
- returns the InsulaCRM `lead_id` and `property_id`;
- treats the source parcel ID as the stable external identity.

Configure these n8n environment variables:

- `INSULA_CRM_BASE_URL`
- `INSULA_CRM_API_KEY`

The API key must stay in n8n credentials/environment configuration and must not be committed to Git.

See `docs/INSULA_CRM_INTEGRATION.md` for the complete field mapping, migration order, and cutover criteria.

---

## The Workflows

### 01. Seller Response Handler — 139 nodes
`workflows/01-seller-response-handler.json`

The largest workflow in the stack. Classifies every inbound seller SMS reply into 20+ distinct intents (motivated, price question, not selling, wrong number, deceased owner, already sold, lien on chain, opt-out, and more) and routes each to its own response and follow-up path.

**What it does:** dedup check, reply logging to Convex, intent classification, hot-lead Slack alert, DNC registration on opt-out, follow-up scheduling. Handles the messy reality that sellers never reply in clean categories.

### 02. Ghost Rider CRM Import (Lead Processor) — 41 nodes
`workflows/02-ghost-rider-crm-import.json`

Weekly job that takes skip-traced leads through DNC, address deduplication, CRM duplicate protection, TCPA checks, first-message preparation, SMS delivery, and follow-up scheduling.

**Migration note:** this workflow is the first major cutover target. During migration, Close CRM remains the existing messaging dependency while InsulaCRM becomes the canonical seller/property system.

### 03. Seller Follow-Up Engine (Enhanced) — 46 nodes
`workflows/03-seller-followup-engine.json`

The multi-day drip. Pulls due follow-ups from a Convex queue, verifies the lead still exists and is in a sendable status, builds the message, sends SMS, and schedules the next touch.

### 04. Tracerfy Results Router — 36 nodes
`workflows/04-skiptrace-results-router.json`

A webhook receiver that catches skip-trace results and routes them by source product, then stages phone/email/phoneless results.

### 05. BatchData Weekly Staging — 28 nodes
`workflows/05-batchdata-weekly-staging.json`

The top of the funnel. Scrapes distressed-property records from BatchData across configurable tiers, dedupes against staging, and uploads new leads to Tracerfy for skip tracing.

### 06. Lead Cleanup - SMS Errors & DNC — 50 nodes
`workflows/06-lead-cleanup-sms-dnc.json`

Continuous list hygiene for bounced numbers and opt-outs.

### 07. Ghost Rider to Tracerfy Skip Trace — 9 nodes
`workflows/07-ghost-rider-skiptrace-dispatch.json`

Filters raw property leads before skip-trace spend, standardizes addresses, and submits clean batches to Tracerfy.

---

## Tech Stack

| Layer | Tool |
|-------|------|
| Orchestration | n8n |
| Canonical CRM | InsulaCRM |
| State + backend during migration | Convex |
| Messaging during migration | Existing provider |
| Property data | BatchData API |
| Skip tracing | Tracerfy |
| Alerts + reports | Slack |
| Compliance | TCPA send-window guards, DNC lists, opt-out handling |

---

## Setup

1. Import the required workflow JSON into n8n.
2. Configure the existing Convex, Tracerfy, BatchData, Slack, and messaging credentials as required by the workflows.
3. Configure:
   - `INSULA_CRM_BASE_URL`
   - `INSULA_CRM_API_KEY`
4. Import `00-insula-crm-lead-upsert-adapter.json`.
5. Call the adapter from the lead-processing workflow with the normalized lead object.
6. Store the returned `lead_id` / `property_id` in the staging state so later workflows can operate against InsulaCRM identity.
7. Do not remove existing DNC/TCPA controls during the migration.

---

## License

Shared for portfolio and reference. The original Real Estate Acquisition product, brand, and backend are proprietary.


### WF-020 — State/County Property Search Intake

- `workflows/20-property-search-intake.json` accepts a state plus optional county/city/ZIP and distress/property criteria.
- Researches candidate properties using the configured research provider with an official-public-record-first prompt.
- Requires owner identity and source attribution before ingesting a candidate.
- Creates/upserts the seller lead and acquisition deal in InsulaCRM using deterministic external IDs and idempotency keys.
- Sends each ingested deal into WF-019 for property intelligence → comps/ARV → offer decision → ranking → jurisdiction review.
- The search contract is state-agnostic; it is not hardcoded to Catawba County or North Carolina.
- The workflow does not itself send seller outreach. Compliance remains the control point before downstream automated activity.
- n8n execution and provider credentials must be validated before production activation.


### WF-020 integration update

Added criteria-driven source strategy and InsulaCRM candidate evidence persistence.

### WF-021 — Official ArcGIS Property Source Adapter

workflows/21-official-arcgis-source-adapter.json provides a reusable machine-readable source adapter for official ArcGIS FeatureServer/MapServer endpoints discovered by WF-020. It queries supported layers, normalizes common parcel fields (owner, situs/mailing address, parcel ID, assessed/land/building value, acreage, property type), and returns raw attributes alongside normalized evidence. WF-020 now invokes this adapter before its research-provider step so structured government GIS records are available to the candidate research prompt instead of relying exclusively on web search.

The adapter is provider-neutral and only acts on discovered FeatureServer/MapServer URLs; it does not assume that every county uses the same schema. Unsupported or non-machine-readable sources remain available to the research provider as official source URLs.

### WF-021 hardening — metadata-aware GIS extraction

WF-021 now reads ArcGIS layer metadata before querying. It uses advertised field names and data types to safely apply jurisdiction filters for state/county/city/ZIP and a numeric delinquent-tax filter when that field is actually present. It also uses the layer's advertised record limits, paginates supported layers, preserves deterministic ordering, and deduplicates parcel candidates across pages/sources.

The adapter remains provider-neutral: it does not assume every county uses the same schema, and it records which filters were actually applied plus limitations when a layer does not advertise pagination support.

### Jurisdiction source registry

`config/jurisdiction-source-registry.json` provides a provider-neutral registry for curated official sources. It is an accelerator, not a hardcoded geographic limit: unknown states/counties continue through controlled official-source discovery. The registry also records commercial-live-query restrictions where a government source publishes them.

For example, Mecklenburg County's GIS documentation states that commercial users should not rely on, integrate with, or continuously consume its live data services; the registry therefore marks those live services as non-permitted for commercial querying and points the system toward downloaded/hosted copies instead.

### WF-022 — Official Non-ArcGIS Property Source Adapter

`workflows/22-official-non-arcgis-source-adapter.json` provides a reusable adapter for official property/tax/open-data sources that are not ArcGIS FeatureServer/MapServer endpoints. It supports common REST/JSON and CSV/download responses, normalizes parcel/property identity and tax/value fields, preserves source provenance, and refuses sources explicitly marked as commercially restricted for live querying.

WF-020 now feeds both WF-021 (ArcGIS) and WF-022 (non-ArcGIS) structured records into the evidence-aware candidate research step. HTML-only official search pages remain discovery/evidence sources rather than being treated as machine-readable APIs.

See `docs/OFFICIAL_NON_ARCGIS_SOURCE_ADAPTER.md`.


### WF-028 — Canonical Jurisdiction Source Registry Adapter

`workflows/28-jurisdiction-source-registry-adapter.json` centralizes runtime jurisdiction-source resolution. WF-020 now calls WF-028 instead of carrying a duplicated state/county registry inside the intake workflow.

- Reads `config/jurisdiction-source-registry.json` at runtime.
- Accepts arbitrary state/county input rather than limiting searches to the currently curated North Carolina counties.
- Merges curated registry sources with official sources discovered by WF-020.
- Deduplicates source URLs/record IDs deterministically.
- Surfaces commercially restricted live sources instead of silently querying them.
- Supports a deployment-specific `WHOLESTACK_REGISTRY_URL` override; otherwise the adapter uses the repository's main-branch registry.
- Keeps unknown-jurisdiction controlled discovery as a fallback instead of fabricating sources.

See `docs/JURISDICTION_SOURCE_REGISTRY_ADAPTER.md`.


### WF-029 — Source Adapter Dispatch Planner

`workflows/29-source-adapter-dispatch-planner.json` turns the canonical jurisdiction source list into an explicit adapter execution plan. WF-020 now sends only the appropriate source family to each adapter:

- ArcGIS/state parcel/county GIS → WF-021
- approved REST/JSON/CSV/open-data/property/tax sources → WF-022
- tax delinquency/court/probate/foreclosure/permit/code evidence → WF-026
- commercially restricted live sources → blocked/review
- unknown source types → surfaced for review rather than guessed

See `docs/SOURCE_ADAPTER_DISPATCH_PLANNER.md`.


### WF-030 — Property Candidate Fan-In & Evidence Aggregator

`workflows/30-property-candidate-fanin.json` is the deterministic fan-in layer between the official source adapters and the AI candidate research step.

It:

- combines GIS, non-ArcGIS, and specialized criterion evidence;
- deduplicates overlapping records without discarding corroborating evidence;
- aggregates criterion signals and explicit supporting evidence;
- ranks candidate research priority deterministically;
- preserves source provenance and observation metadata;
- labels whether requested criteria are fully supported, partially supported, or only identity-supported.

WF-020 now routes its collected source records through WF-030 before the research provider sees them.

See `docs/PROPERTY_CANDIDATE_FANIN.md`.

### WF-032 — Distressed Lead Qualification Gate

### WF-033 — Underwriting to InsulaCRM Status Sync

### WF-035 — Offer Preparation & Seller Review Gate

`workflows/35-offer-preparation-seller-review-gate.json` converts an approved acquisition decision into a structured offer draft. It preserves the MAO and canonical property identity, accepts an explicit proposed offer amount or the calculated MAO, records seller/contact data when supplied, and returns `draft_ready` versus `needs_review`. It is intentionally draft-only: it does not send an offer, make a legal representation, or authorize seller outreach.

WF-020 now routes WF-034 → WF-035 → WF-033 so offer-readiness state is available to the CRM status boundary before the workflow returns the candidate result.

### WF-034 — Acquisition Decision & Offer Readiness Gate

`workflows/34-acquisition-decision-gate.json` converts verified underwriting inputs into an explicit acquisition decision. It calculates a maximum allowable offer only when required inputs are present, records missing fields instead of guessing, preserves candidate identity/evidence, and exposes `offer_ready` / `needs_data` / `not_approved` states for CRM and downstream disposition workflows.



`workflows/33-underwriting-crm-status-sync.json` is the CRM feedback boundary after underwriting. It maps underwriting outcomes to CRM acquisition stages and preserves the canonical candidate identity/evidence context. The actual InsulaCRM deal-update URL is intentionally environment-configurable through `INSULA_CRM_DEAL_UPDATE_URL_TEMPLATE`; when it is not configured, the adapter returns a safe skipped result rather than inventing an endpoint.



`workflows/32-distressed-lead-qualification.json` is the evidence gate between candidate fan-in and downstream property research. It requires explicit identity plus criterion evidence, marks candidates as `qualified_for_review`, and passes only review-ready candidates downstream while retaining the full qualification audit trail. It does not determine legal distress, equity, appraisal, seller intent, or outreach permission.

### WF-031 — Universal Property Record Normalizer

`workflows/31-universal-property-record-normalizer.json` adds the canonical normalization boundary for property records arriving from GIS, assessor/tax, open-data, court/probate, permit/code, and research-provider sources.

It:
- maps common field aliases into one canonical property schema;
- normalizes state, ZIP, text, and numeric values;
- preserves raw source attributes and provenance;
- creates deterministic normalized identity keys;
- removes exact identity duplicates without discarding the first canonical record;
- explicitly reports missing identity/provenance fields instead of inventing values.

WF-031 is intentionally source-provider-neutral. It is now wired directly before WF-030 so every evidence family enters candidate fan-in through the same canonical contract. WF-020 also preserves the resulting canonical identity, evidence metadata, and normalization warnings into InsulaCRM.
