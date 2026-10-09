---
generated: '2026-10-09'
method: generated
name: seca-provision-kubernetes-cluster
description: Create a managed Kubernetes cluster and a node pool in a workspace, then tear them down.
api: openapi/eu-sovereign-cloud-api-extensions-kubernetes-v1beta1-openapi.yml
operations:
- createOrUpdateCluster
- getCluster
- createOrUpdateNode-Pool
- getNode-Pool
- listNode-Pools
- deleteNode-Pool
- deleteCluster
source: Generated from openapi/eu-sovereign-cloud-api-extensions-kubernetes-v1beta1-openapi.yml; every operationId verified
  verbatim in that file.
---

# seca-provision-kubernetes-cluster

Create a managed Kubernetes cluster and a node pool in a workspace, then tear them down.

## Steps
1. `createOrUpdateCluster` (KubernetesClusterSpec requires `skuRef`; optional `podCidr`, `serviceCidr`, `restrictKubernetesApi`).
2. `getCluster` until active.
3. `createOrUpdateNode-Pool` (KubernetesNodePoolSpec requires `instances` and `nodeTemplate`), then `getNode-Pool` until active.
4. `listNode-Pools` to inventory pools.
5. Teardown: `deleteNode-Pool` for each pool, then `deleteCluster`. This is a v1beta1 extension, so not every provider implements it.

## Cross-cutting rules (from the spec)
- Auth: send `Authorization: Bearer <token>` (the specs' `bearerAuth` scheme); the token and the base URL come from the implementing cloud provider, there is no central SECA host.
- Writes are `PUT` create-or-update: a 200 or 201 both mean success. Send `If-Unmodified-Since` (the spec's `ifUnmodifiedSince` parameter) to avoid overwriting a concurrent change; a 412 means the precondition failed, so re-read and retry.
- Resources are asynchronous: poll the matching get operation until `status.state` (ResourceState) is `active`; `error` is terminal. The other states are pending, creating, updating and deleting.
- Lists paginate with `limit` and `skipToken`, and can filter with `labels` (labelSelector).
- Errors: 400/401/403/404/409/412/422/500 are documented on write operations.
