---
generated: '2026-10-09'
method: generated
name: seca-provision-block-storage-and-image
description: Create a block storage volume and register an image from it for booting instances.
api: openapi/eu-sovereign-cloud-api-foundation-storage-v1-openapi.yml
operations:
- listSkus
- listImages
- createOrUpdateBlock-Storage
- getBlock-Storage
- createOrUpdateImage
- getImage
source: Generated from openapi/eu-sovereign-cloud-api-foundation-storage-v1-openapi.yml; every operationId verified verbatim
  in that file.
---

# seca-provision-block-storage-and-image

Create a block storage volume and register an image from it for booting instances.

## Steps
1. `listSkus` to choose a storage SKU, and `listImages` to see images already available in the tenant.
2. `createOrUpdateBlock-Storage` (BlockStorageSpec requires `skuRef` and `sizeGB`; optional `sourceImageRef` clones from an image).
3. `getBlock-Storage` until active.
4. To publish a custom image: `createOrUpdateImage` (ImageSpec requires `blockStorageRef` and `cpuArchitecture`), then `getImage` until active.

## Cross-cutting rules (from the spec)
- Auth: send `Authorization: Bearer <token>` (the specs' `bearerAuth` scheme); the token and the base URL come from the implementing cloud provider, there is no central SECA host.
- Writes are `PUT` create-or-update: a 200 or 201 both mean success. Send `If-Unmodified-Since` (the spec's `ifUnmodifiedSince` parameter) to avoid overwriting a concurrent change; a 412 means the precondition failed, so re-read and retry.
- Resources are asynchronous: poll the matching get operation until `status.state` (ResourceState) is `active`; `error` is terminal. The other states are pending, creating, updating and deleting.
- Lists paginate with `limit` and `skipToken`, and can filter with `labels` (labelSelector).
- Errors: 400/401/403/404/409/412/422/500 are documented on write operations.
