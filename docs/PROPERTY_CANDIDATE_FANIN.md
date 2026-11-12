# WF-030 — Property Candidate Fan-In & Evidence Aggregator

`workflows/30-property-candidate-fanin.json` is the deterministic fan-in stage between the official-source adapters and the research/AI candidate parser in WF-020.

## Purpose

WF-020 can receive overlapping records from:

- official ArcGIS parcel/GIS sources (WF-021)
- official REST/JSON/CSV/open-data sources (WF-022)
- specialized criterion evidence adapters (WF-026 and its downstream adapters)

WF-030 combines those streams into one canonical candidate list before the research provider is asked to produce seller leads.

## Behavior

1. Combines GIS, non-ArcGIS, and criterion evidence records.
2. Deduplicates by parcel ID/source identity first, then owner + property address, then address/source identity.
3. Preserves multiple source-evidence records for the same property rather than discarding corroborating sources.
4. Carries normalized property identity/value/tax/structure fields forward when available.
5. Aggregates criterion signals and counts explicitly supported requested criteria.
6. Produces a deterministic candidate priority score for research ordering.
7. Labels evidence state as `criterion_supported`, `partial_support`, or `identity_only`.
8. Keeps missing evidence missing; the score is not an appraisal, equity determination, or distress determination.
9. Preserves source URL/record ID, source type, observation time, confidence, and raw attributes where supplied.

## WF-020 integration

WF-020 now sends the merged adapter output to WF-030 and then supplies `candidate_records` to the research-provider prompt.

This keeps the AI research step downstream of deterministic source collection and deduplication instead of asking the model to decide which duplicate government records represent the same property.

## Limits

WF-030 is a normalization/ranking layer, not a source-of-truth property database. Canonical property identity and evidence persistence remain in InsulaCRM after candidate ingestion.

Runtime execution still requires an n8n environment with the internal webhook base URL and configured source-provider credentials.
