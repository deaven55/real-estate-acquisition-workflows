# Criterion-Specific Property Lead Signals

WF-020 and WF-021 now distinguish lead criteria instead of treating every search as a generic property lookup.

## Signal policy

- **tax_delinquent**: requires explicit delinquent-tax evidence.
- **vacant**: requires an explicit structure indicator/count or property-use signal.
- **vacant_land**: prefers explicit land/vacant-land use plus structure evidence; acreage alone is not enough.
- **absentee**: compares explicit site and mailing addresses.
- **high_equity**: does not claim equity from assessed value alone; explicit debt evidence is required for a supported equity signal.
- **probate**: requires probate/estate/court evidence.
- **pre_foreclosure**: requires foreclosure/default/lis-pendens/court evidence.
- **fixer_upper**: requires condition, permit, code-enforcement, violation, repair, or comparable evidence.
- **distressed**: requires one or more explicit tax, foreclosure, permit, code, or court distress signals.

## Data carried downstream

Each GIS record can now carry:

- `criterion_signals`
- `criterion_evidence`
- `criterion_missing_fields`
- `criterion_match_count`
- `criterion_signal_version`

WF-020 preserves those fields in the candidate evidence written to InsulaCRM and returns them in the search result.

The workflow intentionally uses **needs_evidence** / **needs_source** states instead of guessing. This prevents a missing field from becoming a false distressed-property classification.

## Scope

This layer is provider-neutral. It operates on normalized GIS records and raw source attributes, so the same criterion contract can be reused as additional state/county adapters are added.

