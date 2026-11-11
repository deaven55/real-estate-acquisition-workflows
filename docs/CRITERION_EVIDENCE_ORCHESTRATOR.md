# Criterion Evidence Adapter Orchestrator

## WF-026

Webhook: `wholestack-criterion-evidence-orchestrator-v1`

WF-026 is the criterion-specific routing layer used by WF-020.

### Routing

- `tax_delinquent` → WF-024 Official County Tax Evidence Adapter
- `probate` / `pre_foreclosure` → WF-025 Official Court Evidence Adapter
- `fixer_upper` / `distressed` → WF-023 generic criterion evidence router

The orchestrator preserves the original jurisdiction, criteria, sources, and request ID and returns a unified evidence record set.

### Source policy

Official state/county sources are preferred. Commercially restricted live sources are not silently queried.

North Carolina public-record guidance identifies county offices as custodians for county information, Register of Deeds for real-estate records, and the Judicial Branch/Clerk for court records. citeturn700359search2turn700359search0turn126712search0

### Safety

The system does not infer distress from missing data. Evidence signals are not seller-contact triggers. They feed the downstream InsulaCRM evidence, underwriting, ranking, and compliance layers.
