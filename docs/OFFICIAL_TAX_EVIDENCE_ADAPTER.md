# Official County Tax Evidence Adapter

## WF-024

Webhook: `wholestack-official-tax-evidence-v1`

WF-024 handles official county tax/property sources for `tax_delinquent` evidence.

### Contract

Input accepts:
- state/county/city/ZIP
- optional parcel_id, owner_name, property_address
- criteria
- limit
- official `sources[]`

Accepted source types:
- `county_tax`
- `tax_delinquency`
- `county_property`

Sources marked `commercial_live_query_allowed: false` are skipped.

### Signal rules

A `tax_delinquent` signal is created only when the source exposes explicit delinquency evidence, such as:
- positive delinquent/past-due amount
- explicit delinquent/past-due/default/unpaid tax status

Missing tax data is never treated as delinquent.

Each record preserves:
- source URL
- source record ID
- observed timestamp
- owner/property identity when available
- tax status
- delinquent amount
- due date
- raw attributes

The adapter emits `confirmed` when explicit evidence supports the signal and `needs_review` otherwise.

### Downstream policy

Tax evidence is a lead signal, not permission to contact a seller. The candidate remains subject to InsulaCRM evidence reconciliation, jurisdiction compliance, and human review.
