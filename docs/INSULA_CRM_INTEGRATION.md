# InsulaCRM Integration Contract

This branch prepares the acquisition workflows to use InsulaCRM as the canonical seller/property CRM while retaining the existing Convex staging and external messaging/skip-trace services where they are still operationally useful.

## Canonical ownership

| Data | System of record |
|---|---|
| Seller/lead identity | InsulaCRM |
| Property identity | InsulaCRM |
| DNC / do-not-contact state | InsulaCRM |
| Seller activities | InsulaCRM |
| Deal lifecycle | InsulaCRM |
| Raw acquisition staging / job state | Convex |
| Skip-trace execution | Tracerfy |
| Property source data | BatchData |
| SMS transport | Existing provider during migration |

## InsulaCRM API

Base URL is supplied through the n8n environment variable `INSULA_CRM_BASE_URL`.

Authentication uses the `X-API-Key` header. The API key must be stored in n8n credentials/environment configuration and never committed to this repository.

Endpoints used by acquisition:

- `POST /api/v1/leads` — create/upsert seller lead
- `GET /api/v1/leads/{id}` — retrieve seller lead
- `PUT /api/v1/leads/{id}` — update seller lead
- `POST /api/v1/properties` — create/upsert property
- `PUT /api/v1/properties/{id}` — update property
- `POST /api/v1/activities` — record seller/contact activity

## External identity

Every acquisition lead sent to InsulaCRM should include:

- `external_id`: stable source identifier, preferably the source parcel ID
- `external_source`: stable source name, for example `ghost_rider` or `batchdata_weekly`

The pair is tenant-scoped and idempotent in InsulaCRM.

## Required lead mapping

The acquisition workflow should normalize its existing lead object into:

```json
{
  "external_id": "<parcelId>",
  "external_source": "ghost_rider",
  "first_name": "<firstName>",
  "last_name": "<lastName>",
  "phone": "<phone>",
  "email": "<email-or-null>",
  "lead_type": "seller",
  "source": "batchdata",
  "status": "new",
  "motivation": "<primaryDistress>",
  "notes": "<source metadata>",
  "property_id": "<insula-property-id-if-known>"
}
```

Property creation should use the normalized address fields already produced by the workflows:

```json
{
  "external_id": "<parcelId>",
  "external_source": "ghost_rider",
  "address": "<street>",
  "city": "<city>",
  "state": "<state>",
  "zip": "<zip>",
  "county": "<county>"
}
```

## Idempotency

For webhook/event-style writes, n8n should send an `Idempotency-Key` header. A deterministic key is preferred:

```
<external_source>:<event_type>:<external_id>:<version>
```

Retries must be safe. Do not create a second seller/property record merely because an upstream HTTP request timed out.

## Canonical property-search handoff

WF-020 now routes official GIS, non-ArcGIS, and criterion evidence through WF-031 (Universal Property Record Normalizer) before WF-030 (Property Candidate Fan-In). WF-030 emits canonical candidate identity and evidence metadata including `candidate_key`, `normalized_identity_key`, `evidence_count`, `evidence_source_types`, `matched_criteria`, `evidence_status`, and normalization warnings. WF-020 preserves these fields when creating/upserting the InsulaCRM lead and deal and when persisting candidate evidence, so CRM records remain traceable to the canonical acquisition evidence layer.

The reusable `00-insula-crm-lead-upsert-adapter.json` also preserves the same canonical metadata in lead notes when supplied by an upstream workflow.

## Migration rules

1. Do not delete Convex staging immediately.
2. Do not remove TCPA/DNC guards.
3. Do not move API keys into workflow JSON.
4. Do not use Close CRM as the canonical seller database after the migration cutover.
5. Keep SMS delivery independent from CRM ownership so the messaging provider can be replaced later.
6. Record outbound and inbound communication as InsulaCRM activities.
7. Preserve the existing source parcel ID on every lead/property record.

## Acquisition workflow migration order

1. `02-ghost-rider-crm-import.json` — **implemented**: seller lead/property ingestion.
2. `04-skiptrace-results-router.json` — **implemented**: Tracerfy mobile/email/phoneless enrichment backfill.
3. `03-seller-followup-engine.json` — **implemented**: outbound follow-up activity sync.
4. `01-seller-response-handler.json` — **implemented**: inbound intent, DNC, and opt-out activity sync.
5. `06-lead-cleanup-sms-dnc.json` — **implemented**: DNC state is synchronized before legacy Close deletion.
6. `05-batchdata-weekly-staging.json` and `07-ghost-rider-skiptrace-dispatch.json` — staging/dispatch remain intentionally unchanged; Insula identity is established when records become CRM-eligible.

## Cutover criteria

The acquisition stack is ready to retire Close CRM as a CRM dependency when:

- lead/property upsert is idempotent;
- DNC checks are enforced by InsulaCRM;
- inbound/outbound activities are persisted in InsulaCRM;
- follow-up queue items contain stable Insula lead IDs;
- duplicate checks use Insula external identity plus normalized phone/address;
- a replayed n8n execution does not create duplicate CRM records;
- tenant/API-key isolation is verified in a non-production test.
## Qualification-to-CRM handoff

WF-032 (`32-distressed-lead-qualification.json`) now hands only `qualified_for_review` candidates to `POST /api/v1/leads`. The handoff splits owner identity into CRM name fields, carries property identity and qualification evidence in notes, uses `external_id` + `external_source` for stable identity, and sends a deterministic `Idempotency-Key`. Candidates that fail the evidence-based qualification gate remain in the qualification audit output and are not written as CRM leads.

This establishes the next canonical boundary: acquisition qualification -> InsulaCRM seller/property record -> underwriting/deal pipeline. The next integration slice is to make the qualification handoff consume the canonical WF-031/WF-030 identity fields end-to-end and then connect the resulting CRM lead/deal IDs into WF-033/WF-034 without re-keying or losing provenance.



## Offer-to-disposition CRM activity handoff

WF-035 now records offer-draft preparation as an InsulaCRM activity when a canonical Insula lead ID is present. The activity is idempotent and explicitly records that the offer is draft-only and not authorized for sending.

WF-036 now records seller offer lifecycle decisions (sent, countered, accepted, rejected, expired, withdrawn, contract-pending, or executed) as InsulaCRM activities while preserving the existing lifecycle and disposition handoff behavior.

WF-037 now records contract/disposition readiness evaluation as an InsulaCRM activity, including readiness state, contract/strategy readiness, buyer-matching readiness, next action, and missing fields. These activity writes are best-effort and do not block the acquisition workflow if CRM activity logging fails.
