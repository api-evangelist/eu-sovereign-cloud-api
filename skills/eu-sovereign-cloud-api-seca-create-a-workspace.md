---
generated: '2026-10-09'
method: generated
name: seca-create-a-workspace
description: Create a SECA workspace in a tenant, confirm it is active, and clean it up.
api: openapi/eu-sovereign-cloud-api-foundation-workspace-v1-openapi.yml
operations:
- listWorkspaces
- createOrUpdateWorkspace
- getWorkspace
- deleteWorkspace
source: Generated from openapi/eu-sovereign-cloud-api-foundation-workspace-v1-openapi.yml; every operationId verified verbatim
  in that file.
---

# seca-create-a-workspace

Create a SECA workspace in a tenant, confirm it is active, and clean it up.

## Steps
1. `listWorkspaces` to check whether the workspace name is already taken in the tenant.
2. `createOrUpdateWorkspace` with the tenant and the new workspace name.
3. `getWorkspace` until `status.state` is `active`.
4. `deleteWorkspace` when the workspace is no longer needed. Deletion is not cascading: delete all child resources first, or the call returns 409.

## Cross-cutting rules (from the spec)
- Auth: send `Authorization: Bearer <token>` (the specs' `bearerAuth` scheme); the token and the base URL come from the implementing cloud provider, there is no central SECA host.
- Writes are `PUT` create-or-update: a 200 or 201 both mean success. Send `If-Unmodified-Since` (the spec's `ifUnmodifiedSince` parameter) to avoid overwriting a concurrent change; a 412 means the precondition failed, so re-read and retry.
- Resources are asynchronous: poll the matching get operation until `status.state` (ResourceState) is `active`; `error` is terminal. The other states are pending, creating, updating and deleting.
- Lists paginate with `limit` and `skipToken`, and can filter with `labels` (labelSelector).
- Errors: 400/401/403/404/409/412/422/500 are documented on write operations.
