---
name: Ingest and read well production data
description: Upsert daily production records for a well and read them back over a date range.
api: openapi/tachyus-tachapps-openapi.yml
operations: [ingestProductionData, getProductionData, deleteProductionData]
---

# Ingest and read well production data

Load production history for a well and query it back.

## Auth
- `Authorization: Bearer tach_live_...`. Writing needs `write:wells`; reading needs `read:production`.

## Steps
1. **Ingest records** — `ingestProductionData` (`POST /projects/{projectId}/wells/{wellId}/production`) with a `records[]` array of daily entries (`date` plus metrics: `oil_rate`, `water_rate`, `gas_rate`, `bhp`, `thp`, `choke_size`, `runtime`, `gor`, `watercut`). Records with an existing `date` are **upserted (replaced)**. The response reports `inserted`, `updated`, `total`. For historical loads over a few thousand records, prefer CSV import / bulk migration instead.
2. **Read back** — `getProductionData` (`GET /projects/{projectId}/wells/{wellId}/production`) with required `from`/`to` (ISO 8601, inclusive), optional `fields` (comma-separated) and `aggregation` (`daily` default or `monthly`).
3. **Correct mistakes** — `deleteProductionData` (`DELETE .../production`) with `from`/`to` removes records in that range; the response reports `deleted`.

## Conventions & errors
- Upsert-by-date makes ingest naturally idempotent per date — re-posting the same day's record is safe (see `conventions/tachyus-conventions.yml`).
- `429 RATE_LIMITED` → honor `Retry-After`. `401 UNAUTHORIZED` / `403 FORBIDDEN` → check token and scopes.
- Error envelope: `{ "error": { "code", "message", "requestId" } }`.
