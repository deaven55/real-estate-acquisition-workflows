# Official Court Probate / Foreclosure Evidence Adapter

## WF-025

Webhook: `wholestack-official-court-evidence-v1`

WF-025 handles official court/probate/foreclosure source inputs while preserving the distinction between public docket/index evidence and an official court record.

Accepted source types:
- `court_probate`
- `probate_record`
- `estate_record`
- `foreclosure`
- `court_record`
- `land_record`

### Signal rules

Supported criteria:
- `probate`
- `pre_foreclosure`
- `distressed`

Probate signals require probate/estate/decedent/executor/administrator evidence.

Pre-foreclosure signals require foreclosure/default/lis pendens/notice-of-sale/power-of-sale evidence.

Distressed signals may combine explicit probate or foreclosure evidence, but missing data is never treated as distress.

### Court-record status

The adapter preserves `public_docket_status` in structured facts. A court portal or index is not represented as a certified official court record.

North Carolina's Judicial Branch states that eCourts Portal is not the official court record; it provides public docket entries, while official copies are obtained from the appropriate Clerk of Court. citeturn126712search12turn126712search0

### Downstream policy

Court evidence is a lead signal only. It must remain traceable to its source and pass compliance/human review before seller outreach.
