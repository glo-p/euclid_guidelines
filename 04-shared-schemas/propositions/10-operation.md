# Prop. IV.10 - Long-running work is an `Operation`

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

The body of a `202 Accepted` response and of `GET /v{n}/operations/{operationId}` ([Prop. II.15](../../02-api-guidelines/propositions/15-long-running-operations.md)) MUST conform to [`operation.json`](../schemas/shared/v1/operation.json): `id`, `status` (closed: `pending`, `running`, `succeeded`, `failed`, `cancelled`), `createdAt`, `updatedAt`; `completedAt` and `result` when succeeded; `completedAt` and `error` (a Problem) when failed; optional `progress`. APIs MAY extend `result` with declared fields.

## Given

[Def. II.17](../../02-api-guidelines/definitions.md#Def.%20II.17%20-%20Long-Running%20Operation), [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape), [Prop. II.15](../../02-api-guidelines/propositions/15-long-running-operations.md), [IV.5](05-problem-details.md).

## Demonstration

[Prop. II.15](../../02-api-guidelines/propositions/15-long-running-operations.md) requires an operation resource; a generic client (a front-end progress component, a CLI) can poll any operation only if all operations have one shape ([CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape)). The status set is closed because it is a state machine the *client* drives decisions from and because a new state would change client control flow, which is the "proven closed" case of [Prop. IV.9](09-open-enumerations.md). `result` is left open because outcomes are operation-specific and are declared per API ([CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit)). ∎ Q.E.D.

## Corollaries

* **Cor. IV.10.1** - `status` transitions are `pending → running → {succeeded, failed, cancelled}`; a terminal state never changes.
* **Cor. IV.10.2** - Cancellation, where supported, is `POST /v{n}/operations/{id}/cancellations` ([Prop. II.2](../../02-api-guidelines/propositions/02-resource-naming.md)) and may be refused with `409`.

## Construction

C# `Operation` record and `OperationStatus` enum in `Company.Contracts.Shared`; the platform package provides the `operations` endpoint group and storage abstraction.

## Conformance

Spectral `202-has-location-and-operation-body` ([Prop. II.15](../../02-api-guidelines/propositions/15-long-running-operations.md)). Catalogue CI validates examples and the conditional `required` rules.
