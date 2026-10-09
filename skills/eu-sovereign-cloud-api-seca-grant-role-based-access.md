---
generated: '2026-10-09'
method: generated
name: seca-grant-role-based-access
description: Define a role with permissions and assign it to subjects at chosen scopes.
api: openapi/eu-sovereign-cloud-api-foundation-authorization-v1-openapi.yml
operations:
- listRoles
- createOrUpdateRole
- getRole
- createOrUpdateRoleAssignment
- getRoleAssignment
- listRoleAssignments
- deleteRoleAssignment
source: Generated from openapi/eu-sovereign-cloud-api-foundation-authorization-v1-openapi.yml; every operationId verified
  verbatim in that file.
---

# seca-grant-role-based-access

Define a role with permissions and assign it to subjects at chosen scopes.

## Steps
1. `listRoles` to reuse an existing role where possible.
2. `createOrUpdateRole` (RoleSpec requires `permissions`), then `getRole`.
3. `createOrUpdateRoleAssignment` (RoleAssignmentSpec requires `subs`, `scopes`, `roles`), then `getRoleAssignment` until active.
4. `listRoleAssignments` to audit; `deleteRoleAssignment` to revoke.

## Cross-cutting rules (from the spec)
- Auth: send `Authorization: Bearer <token>` (the specs' `bearerAuth` scheme); the token and the base URL come from the implementing cloud provider, there is no central SECA host.
- Writes are `PUT` create-or-update: a 200 or 201 both mean success. Send `If-Unmodified-Since` (the spec's `ifUnmodifiedSince` parameter) to avoid overwriting a concurrent change; a 412 means the precondition failed, so re-read and retry.
- Resources are asynchronous: poll the matching get operation until `status.state` (ResourceState) is `active`; `error` is terminal. The other states are pending, creating, updating and deleting.
- Lists paginate with `limit` and `skipToken`, and can filter with `labels` (labelSelector).
- Errors: 400/401/403/404/409/412/422/500 are documented on write operations.
