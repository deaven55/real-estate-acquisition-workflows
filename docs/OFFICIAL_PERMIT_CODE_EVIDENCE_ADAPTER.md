# Official Permit / Code Enforcement Evidence Adapter

## WF-027

Webhook: `wholestack-official-permit-code-evidence-v1`

WF-027 handles official permit, inspection, code-enforcement, and property-record sources for `fixer_upper` and `distressed` evidence.

Accepted source types:
- `permit_code_enforcement`
- `permit_record`
- `code_enforcement`
- `property_record`

### Signal rules

A positive signal requires explicit source evidence such as:
- code violation
- condemnation
- unsafe/substandard condition
- demolition or stop-work action
- repair/inspection evidence
- permit/enforcement status that directly supports a property-condition signal

A missing permit or missing code record is never interpreted as proof that a property is distressed.

Each record preserves source URL, source record ID, observed timestamp, property identity, permit/violation fields, and raw attributes.

### NC source fit

North Carolina DEQ provides online public records, ArcGIS tools, permit databases, environmental application tracking, and a public-records request process. Its online resources include permit and compliance documents and mapping tools. citeturn532550search0turn532550search3turn532550search5

The adapter therefore supports both machine-readable official datasets and source pages that return structured records. HTML/document-only sources should remain traceable and may require human review rather than pretending the workflow extracted a complete structured record.

### Downstream policy

Permit/code evidence is a lead signal only. It feeds evidence reconciliation, ranking, and compliance gates; it does not automatically trigger seller outreach.
