<div align="center">

# Real Estate Acquisition. Wholesaling Seller-Side Automation (n8n)

### 7 production n8n workflows. 341 nodes. The operations backbone of a real estate wholesaling business, from raw property data to a booked seller conversation.

[![n8n](https://img.shields.io/badge/n8n-workflows-FF6D5A?style=for-the-badge&logo=n8n&logoColor=white)](https://n8n.io)
[![Nodes](https://img.shields.io/badge/total_nodes-341-6366F1?style=for-the-badge)](#)
[![Workflows](https://img.shields.io/badge/workflows-7-10B981?style=for-the-badge)](#)
[![Status](https://img.shields.io/badge/status-production-success?style=for-the-badge)](#)

</div>

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
Close CRM lead creation (dedup, DNC, TCPA guard) + first outbound text
      │
      ├──▶  03-seller-followup-engine (multi-day drip, rate-limited)
      │
      └──▶  01-seller-response-handler (20+ intent classifier on inbound replies)
                                    │
                                    ▼  06-lead-cleanup-sms-dnc (continuous hygiene)
```

---

## The Workflows

### 01. Seller Response Handler — 136 nodes
`workflows/01-seller-response-handler.json`

The largest workflow in the stack. Classifies every inbound seller SMS reply into 20+ distinct intents (motivated, price question, not selling, wrong number, deceased owner, already sold, lien on chain, opt-out, and more) and routes each to its own response and follow-up path.

**What it does:** dedup check, reply logging to Convex, intent classification, hot-lead Slack alert, DNC registration on opt-out, follow-up scheduling. Handles the messy reality that sellers never reply in clean categories.

**Patterns:** multi-branch intent routing, Convex-backed dedup and state, Slack hot-lead alerting, opt-out compliance.

### 02. Ghost Rider CRM Import (Lead Processor) — 40 nodes
`workflows/02-ghost-rider-crm-import.json`

Weekly job that takes skip-traced leads and imports them into Close CRM, then fires the first outbound text, all with layered deduplication and TCPA compliance.

**What it does:** fetch pending leads, DNC check, address-level dedup, Close-CRM duplicate check, create lead, TCPA send-window guard (queues for later if outside hours), send Text 1, log SMS cost, enqueue for follow-up, aggregate daily stats.

**Patterns:** three-layer dedup (DNC + address + CRM), TCPA time-window enforcement, per-lead rate limiting with Wait, cost tracking to a central ledger, sticky-note documented canvas.

### 03. Seller Follow-Up Engine (Enhanced) — 44 nodes
`workflows/03-seller-followup-engine.json`

The multi-day drip. Pulls due follow-ups from a Convex queue, verifies the lead still exists and is in a sendable status, builds the message, sends via Close, and schedules the next touch, with stalled-conversation detection and stale-lead cleanup.

**What it does:** fetch due items, rate-limit to 1 per second, verify lead in Close, check status exclusions, build message, send SMS, calculate next follow-up, handle phase transitions, detect stalled conversations, delete stale leads from the queue, track phone-health stats and cost.

**Patterns:** durable queue + cron (no in-workflow sleeping), 1-per-second rate limiter, phase-based cadence, stale-record cleanup, API-error handling, phone deliverability stats.

### 04. Tracerfy Results Router — 35 nodes
`workflows/04-skiptrace-results-router.json`

A webhook receiver that catches skip-trace results and routes them by source product (Ghost Rider, VoiceSDR, DishCraft, agent network), because one skip-trace vendor feeds multiple products.

**What it does:** receive Tracerfy webhook, identify the source queue, download results, split into phone / email-only / phoneless buckets, stage each back to Convex, complete the job, log return stats and pipeline progress.

**Patterns:** single webhook multiplexed across products, result bucketing by data quality, idempotent job completion, pipeline-stage logging.

### 05. BatchData Weekly Staging (Convex) — 28 nodes
`workflows/05-batchdata-weekly-staging.json`

The top of the funnel. Every Sunday it scrapes distressed-property records from BatchData across configurable tiers, dedupes against Convex, and uploads new leads to Tracerfy for skip tracing.

**What it does:** load tier configs, loop each tier, build the BatchData request with pagination state, call the API, filter to new leads only, batch-upload to Tracerfy (only if 5+ leads), store the job, advance pagination, wait between tiers, and post a Slack report with cost.

**Patterns:** tier-based configurable scraping, stateful pagination across runs, dedup-before-spend, minimum-batch threshold, inter-tier throttling, cost reporting.

### 06. Lead Cleanup - SMS Errors & DNC — 49 nodes
`workflows/06-lead-cleanup-sms-dnc.json`

Continuous list hygiene on three schedules. Removes leads whose texts bounced (bad numbers) and leads who opted out, keeping the CRM clean and compliant.

**What it does:** three triggers (manual blitz, 10am/4pm SMS-error sweep, 9pm DNC sweep), paginated fetch of leads, inspect recent SMS activities for delivery errors, delete error and DNC leads, clean up the Convex queue, and post per-run Slack reports.

**Patterns:** multi-trigger single workflow, paginated batch processing, delivery-error detection, compliance-driven deletion, Convex queue cleanup, per-segment reporting.

### 07. Ghost Rider to Tracerfy Skip Trace — 9 nodes
`workflows/07-ghost-rider-skiptrace-dispatch.json`

The dispatcher that turns raw scanned properties into skip-trace jobs. Small but does the heavy filtering: strips corporate, government, and bank-owned parcels, standardizes addresses, and handles absentee-owner mailing-address swaps before spending a cent on skip tracing.

**What it does:** fetch pending leads from Convex, apply corporate/government/bank owner filters, standardize address formatting, swap to mailing address for absentee owners, upload the clean batch to Tracerfy, mark sent, push cost to the central ledger.

**Patterns:** filter-before-spend, regex owner classification, USPS-style address standardization, absentee detection.

---

## Tech Stack

| Layer | Tool |
|-------|------|
| Orchestration | n8n |
| State + backend | Convex (HTTP actions) |
| CRM | Close CRM |
| Property data | BatchData API |
| Skip tracing | Tracerfy |
| Alerts + reports | Slack |
| Compliance | TCPA send-window guards, DNC lists, opt-out handling |

---

## Setup (to actually run these)

1. Import a workflow JSON into your n8n instance (Workflows → Import from File).
2. Replace the placeholders:
   - `YOUR_CONVEX_DEPLOYMENT` with your Convex deployment name / URL
   - `YOUR_API_TOKEN` / `YOUR_TRACERFY_TOKEN` with your Tracerfy token
   - `YOUR_CLOSE_API_KEY` with your Close CRM key
   - `YOUR_SLACK_WEBHOOK` / `YOUR_SLACK_TOKEN` with your Slack credentials
3. Add the matching n8n credentials (credentials were stripped for publishing).
4. Point the Convex HTTP endpoints at your own backend (schemas not included here).

---

## About

Built by **Rafiul Islam**. AI Voice Agent Systems Architect and Full-Stack Automation Engineer.

- Portfolio: [rafiul-dev.vercel.app](https://rafiul-dev.vercel.app)
- These 7 are a slice of 200+ production workflows across 15+ products. Full architecture walkthroughs available on request.

---

## License

Shared for portfolio and reference. The Real Estate Acquisition product, brand, and backend (Convex schemas, CRM configuration, business logic) are proprietary.
