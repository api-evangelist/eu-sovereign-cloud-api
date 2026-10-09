---
generated: '2026-10-09'
method: generated
name: seca-build-a-workspace-network
description: 'Lay down a private network in a workspace: network, route table, internet gateway, subnet, security group and
  rules, a public IP and a NIC.'
api: openapi/eu-sovereign-cloud-api-foundation-network-v1-openapi.yml
operations:
- listSkus
- createOrUpdateNetwork
- getNetwork
- createOrUpdateRoute-Table
- createOrUpdateInternetGateway
- createOrUpdateSubnet
- createOrUpdateSecurityGroup
- createOrUpdateSecurityGroupRule
- createOrUpdatePublicIp
- createOrUpdateNic
- getNic
source: Generated from openapi/eu-sovereign-cloud-api-foundation-network-v1-openapi.yml; every operationId verified verbatim
  in that file.
---

# seca-build-a-workspace-network

Lay down a private network in a workspace: network, route table, internet gateway, subnet, security group and rules, a public IP and a NIC.

## Steps
1. `listSkus` to pick a network SKU (NetworkSpec requires `cidr` and `skuRef`).
2. `createOrUpdateNetwork`, then `getNetwork` until active.
3. `createOrUpdateRoute-Table` under the network, and `createOrUpdateInternetGateway` for egress.
4. `createOrUpdateSubnet` (SubnetSpec requires `cidr`, `zone`, `routeTableRef`).
5. `createOrUpdateSecurityGroup` and one `createOrUpdateSecurityGroupRule` per rule.
6. `createOrUpdatePublicIp` if the workload needs inbound internet access.
7. `createOrUpdateNic` (NicSpec requires `addresses` and `subnetRef`; attach `securityGroupRefs` and `publicIpRefs`), then `getNic` until active.

## Cross-cutting rules (from the spec)
- Auth: send `Authorization: Bearer <token>` (the specs' `bearerAuth` scheme); the token and the base URL come from the implementing cloud provider, there is no central SECA host.
- Writes are `PUT` create-or-update: a 200 or 201 both mean success. Send `If-Unmodified-Since` (the spec's `ifUnmodifiedSince` parameter) to avoid overwriting a concurrent change; a 412 means the precondition failed, so re-read and retry.
- Resources are asynchronous: poll the matching get operation until `status.state` (ResourceState) is `active`; `error` is terminal. The other states are pending, creating, updating and deleting.
- Lists paginate with `limit` and `skipToken`, and can filter with `labels` (labelSelector).
- Errors: 400/401/403/404/409/412/422/500 are documented on write operations.
