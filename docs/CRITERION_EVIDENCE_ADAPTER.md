# Criterion Evidence Adapter Router

## WF-023

`workflows/23-criterion-evidence-adapter-router.json` adds a criterion-aware evidence stage for distressed-property searches.

Supported criteria:

- tax delinquent
- probate
- pre-foreclosure
- fixer-upper
- distressed

The router accepts only official sources supplied by jurisdiction resolution and skips sources explicitly marked as commercially restricted for live querying.

### Evidence-first behavior

A criterion signal is only an evidence candidate. The adapter preserves:

- source URL
- source record ID
- observed timestamp
- owner/property identity where available
- structured source facts
- raw source attributes
- signal type
- limitations
- missing fields

It does not convert missing data into a negative or positive distress determination.

For court/probate sources, the system distinguishes public docket access from an official court record. North Carolina Judicial Branch documentation states that eCourts Portal provides public access to docket/case information but is not itself the official court record; official copies must be obtained from the appropriate Clerk of Court. citeturn555256search12turn555256search15

### WF-020 integration

WF-020 now runs:

1. official ArcGIS adapter (WF-021)
2. official non-ArcGIS adapter (WF-022)
3. criterion evidence adapter (WF-023)
4. evidence-aware candidate research
5. InsulaCRM ingestion
6. underwriting/compliance pipeline

This creates a direct path from a user's requested criterion to the type of public record that can actually substantiate it.

### Important limitation

WF-023 currently provides a provider-neutral normalization/router layer. It does not pretend that every county exposes tax, probate, foreclosure, or code-enforcement data through the same machine-readable interface. Jurisdiction-specific source discovery and adapter contracts remain the mechanism for adding those sources safely.
