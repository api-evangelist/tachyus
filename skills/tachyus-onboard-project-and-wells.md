---
name: Onboard a project and its wells
description: Create a Tachapps project, add wells to it, and verify them.
api: openapi/tachyus-tachapps-openapi.yml
operations: [createProject, createWell, listWells, getWell]
---

# Onboard a project and its wells

Use the Tachapps API to stand up a new project and register its wells.

## Auth
- Send `Authorization: Bearer tach_live_...` on every request (see `authentication/tachyus-authentication.yml`).
- Creating projects/wells requires the `write:projects` and `write:wells` scopes; a token missing a scope returns `403 FORBIDDEN`.
- All requests are HTTPS with `Content-Type: application/json`.

## Steps
1. **Create the project** — `createProject` (`POST /projects`) with required `name`, `description`, `timezone`, `unitSystem`. Capture the returned `id` as `projectId`.
2. **Add each well** — `createWell` (`POST /projects/{projectId}/wells`) with `name`, `apiNumber`, `type` (`producer|injector|observer|swd`), `status` (`active|inactive|shut_in`), and optional location/`perfInterval` fields. Capture each returned well `id`.
3. **Verify** — `listWells` (`GET /projects/{projectId}/wells`) and page with `limit`/`cursor` until `nextCursor` is null; or `getWell` (`GET /projects/{projectId}/wells/{wellId}`) for a single well.

## Conventions & errors
- Cursor pagination: read `data[]`, follow `nextCursor`, stop when null (`conventions/tachyus-conventions.yml`).
- On `429 RATE_LIMITED`, back off for the `Retry-After` seconds and retry.
- Errors arrive as `{ "error": { "code", "message", "requestId" } }` — log `requestId` for support (`errors/tachyus-error-codes.yml`).
- No idempotency-key header exists, so avoid blind retries of `createWell` on network timeouts; re-list to check whether the well already exists before recreating.
