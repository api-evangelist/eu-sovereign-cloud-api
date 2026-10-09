---
generated: '2026-10-09'
method: generated
name: seca-discover-regions
description: Discover the regions a SECA provider offers before placing resources.
api: openapi/eu-sovereign-cloud-api-foundation-region-v1-openapi.yml
operations:
- listRegions
- getRegion
source: Generated from openapi/eu-sovereign-cloud-api-foundation-region-v1-openapi.yml; every operationId verified verbatim
  in that file.
---

# seca-discover-regions

Discover the regions a SECA provider offers before placing resources.

## Steps
1. `listRegions` to enumerate the provider's regions.
2. `getRegion` for the details of the chosen region (including its zones and the providers it exposes). The provider's entry point can also be discovered via `getWellknown` in the wellknown extension (`/.wellknown/secapi`).

## Cross-cutting rules (from the spec)
- Auth: send `Authorization: Bearer <token>` (the specs' `bearerAuth` scheme); the token and the base URL come from the implementing cloud provider, there is no central SECA host.
- Writes are `PUT` create-or-update: a 200 or 201 both mean success. Send `If-Unmodified-Since` (the spec's `ifUnmodifiedSince` parameter) to avoid overwriting a concurrent change; a 412 means the precondition failed, so re-read and retry.
- Resources are asynchronous: poll the matching get operation until `status.state` (ResourceState) is `active`; `error` is terminal. The other states are pending, creating, updating and deleting.
- Lists paginate with `limit` and `skipToken`, and can filter with `labels` (labelSelector).
- Errors: 400/401/403/404/409/412/422/500 are documented on write operations.
