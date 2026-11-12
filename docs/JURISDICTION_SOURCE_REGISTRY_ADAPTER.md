# Jurisdiction Source Registry Adapter

## Purpose

WF-028 provides one canonical runtime resolver for the provider-neutral jurisdiction source registry at `config/jurisdiction-source-registry.json`.

WF-020 calls WF-028 instead of carrying a duplicated county/state registry inside the workflow. This keeps jurisdiction configuration in one versioned file and allows the search intake to accept arbitrary state/county input.

## Runtime contract

Webhook: `wholestack-jurisdiction-source-registry-v1`

Request fields:
- `state` — required two-letter state code after WF-020 normalization.
- `county` — optional county name.
- `city`, `zip`, `criteria`, `property_type`, `limit`, `search_notes` — optional context.
- `official_sources` — sources discovered by WF-020 when the jurisdiction is not fully covered by the curated registry.

Response includes:
- `official_sources`
- `commercial_restrictions`
- `curated_source_count`
- `discovered_source_count`
- `source_resolution`

## Registry precedence

1. Curated sources from `config/jurisdiction-source-registry.json`.
2. Official sources discovered by WF-020 for incomplete or unknown jurisdictions.
3. Duplicate URLs/source IDs are removed deterministically.
4. Sources marked `commercial_live_query_allowed: false` are surfaced as restrictions and are not silently queried by downstream adapters.

## Canonical registry location

`config/jurisdiction-source-registry.json`

The runtime URL defaults to the repository's `main` branch raw registry. Set `WHOLESTACK_REGISTRY_URL` in n8n when a deployment needs a different pinned registry location.

## Design rule

The registry is a source-selection layer, not a lead database. Adapter workflows remain responsible for querying, normalizing, and preserving provenance. Unknown jurisdictions must retain controlled official-source discovery rather than fabricating a source.

## Validation

WF-020 and WF-028 were structurally validated after this change:
- JSON parses successfully.
- All connection targets resolve to existing nodes.
- All Code nodes pass JavaScript syntax validation.
