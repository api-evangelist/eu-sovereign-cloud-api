---
generated: '2026-10-09'
method: generated
name: seca-manage-instance-lifecycle
description: Launch a virtual machine instance, then stop, start, restart and delete it.
api: openapi/eu-sovereign-cloud-api-foundation-compute-v1-openapi.yml
operations:
- listSkus
- createOrUpdateInstance
- getInstance
- StopInstance
- StartInstance
- RestartInstance
- listInstances
- deleteInstance
source: Generated from openapi/eu-sovereign-cloud-api-foundation-compute-v1-openapi.yml; every operationId verified verbatim
  in that file.
---

# seca-manage-instance-lifecycle

Launch a virtual machine instance, then stop, start, restart and delete it.

## Steps
1. `listSkus` to choose an instance SKU.
2. `createOrUpdateInstance` (InstanceSpec requires `skuRef`, `zone`, `bootVolume`; reference a NIC via `primaryNicRef` and a security group via `securityGroupRef` from the network skill, and a boot volume from the storage skill).
3. `getInstance` until `status.state` is `active`.
4. `StopInstance`, `StartInstance` and `RestartInstance` are POST actions and are NOT idempotent: check state with `getInstance` before repeating one.
5. `listInstances` to inventory the workspace; `deleteInstance` to remove.

## Cross-cutting rules (from the spec)
- Auth: send `Authorization: Bearer <token>` (the specs' `bearerAuth` scheme); the token and the base URL come from the implementing cloud provider, there is no central SECA host.
- Writes are `PUT` create-or-update: a 200 or 201 both mean success. Send `If-Unmodified-Since` (the spec's `ifUnmodifiedSince` parameter) to avoid overwriting a concurrent change; a 412 means the precondition failed, so re-read and retry.
- Resources are asynchronous: poll the matching get operation until `status.state` (ResourceState) is `active`; `error` is terminal. The other states are pending, creating, updating and deleting.
- Lists paginate with `limit` and `skipToken`, and can filter with `labels` (labelSelector).
- Errors: 400/401/403/404/409/412/422/500 are documented on write operations.
