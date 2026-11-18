# Source Adapter Dispatch Planner

WF-029 is the runtime dispatch layer between jurisdiction-source resolution and the concrete evidence adapters.

## Contract

Webhook: `wholestack-source-adapter-dispatch-v1`

Input:
- normalized state/county/location context
- criteria
- `official_sources` from WF-028

Output:
- `adapter_plan.arcgis` → WF-021
- `adapter_plan.non_arcgis` → WF-022
- `adapter_plan.criterion_evidence` → WF-026
- `adapter_plan.restricted` → sources explicitly blocked from commercial live querying
- `adapter_plan.unsupported` → unknown source types that require review

## Dispatch rules

- `state_parcel` / `county_gis` and ArcGIS FeatureServer/MapServer URLs → WF-021.
- `county_property`, `county_tax`, `county_land`, `county_open_data`, REST/JSON, CSV/download and approved open-data types → WF-022.
- court/probate, foreclosure, tax-delinquency, permit/code and related criterion-specific source types → WF-026.
- `commercial_live_query_allowed: false` → restricted; never dispatched.
- Unknown source types are surfaced instead of guessed.

WF-020 now passes each adapter only the source set assigned to it by WF-029. This makes the canonical jurisdiction registry materially control adapter selection rather than merely documenting which sources exist.

## Design goal

The source registry remains provider-neutral. A jurisdiction can therefore add a new official source without changing the candidate-search business logic; only the registry/source type mapping needs to be updated when a new adapter family is required.

## Validation

WF-020 and WF-029 were structurally validated after integration:
- JSON parses successfully.
- All connection targets resolve.
- All Code-node JavaScript passes syntax validation.
- No n8n live execution is claimed.
