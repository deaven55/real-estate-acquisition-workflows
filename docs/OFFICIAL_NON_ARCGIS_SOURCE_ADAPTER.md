# Official Non-ArcGIS Source Adapter

## WF-022

`workflows/22-official-non-arcgis-source-adapter.json` is a provider-neutral n8n adapter for official public-record sources that are not ArcGIS FeatureServer/MapServer endpoints.

### Supported source types

- `rest_json`
- `json_api`
- `csv_download`
- `open_data`
- `official_download`

The adapter:

1. accepts only sources supplied by the jurisdiction-source resolution layer;
2. refuses sources marked `commercial_live_query_allowed: false`;
3. fetches the approved official URL;
4. parses common JSON response shapes and simple CSV downloads;
5. normalizes common parcel/property fields;
6. preserves the original source URL, source type, raw attributes, and observed timestamp;
7. returns limitations instead of inventing missing fields.

It does **not** scrape arbitrary commercial sites and it does not treat an HTML search page as a machine-readable source. HTML-only sources remain available to the research provider for controlled evidence gathering.

## WF-020 integration

WF-020 now calls WF-021 for ArcGIS sources and WF-022 for approved non-ArcGIS sources before the research-provider step. The research prompt receives both structured GIS records and structured non-ArcGIS records.

This makes the acquisition search path source-type aware:

`state/county -> curated official sources -> controlled discovery -> source-type adapters -> evidence-aware candidate research -> InsulaCRM`

## Commercial restrictions

A jurisdiction source may explicitly prohibit commercial live querying. Those sources are returned in `restricted_sources` and are not requested by WF-022. The system should use an authorized downloaded/hosted copy when the jurisdiction permits that model.

## Production validation

The workflow JSON is structurally validated in repository tooling. Actual n8n execution still requires the user's configured `N8N_INTERNAL_BASE_URL` and provider/source access. No production execution is claimed by this repository change.
