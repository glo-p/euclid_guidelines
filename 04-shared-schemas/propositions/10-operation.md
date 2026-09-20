# Prop. IV.10 - Long-running work is an `Operation`

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

The body of a `202 Accepted` response and of `GET /v{n}/operations/{operationId}` (Prop. II.15) MUST conform to [`operation.json`](../schemas/shared/v1/operation.json): `id`, `status` (closed: `pending`, `running`, `succeeded`, `failed`, `cancelled`), `createdAt`, `updatedAt`; `completedAt` and `result` when succeeded; `completedAt` and `error` (a Problem) when failed; optional `progress`. APIs MAY extend `result` with declared fields.

## Given

Def. II.17, CN 3, CN 5, Prop. II.15, IV.5.

## Demonstration

Prop. II.15 requires an operation resource; a generic client (a front-end progress component, a CLI) can poll any operation only if all operations have one shape (CN 5). The status set is closed because it is a state machine the *client* drives decisions from and because a new state would change client control flow, which is the "proven closed" case of Prop. IV.9. `result` is left open because outcomes are operation-specific and are declared per API (CN 3). ∎ Q.E.D.

## Corollaries

* **Cor. IV.10.1** - `status` transitions are `pending → running → {succeeded, failed, cancelled}`; a terminal state never changes.
* **Cor. IV.10.2** - Cancellation, where supported, is `POST /v{n}/operations/{id}/cancellations` (Prop. II.2) and may be refused with `409`.

## Construction

C# `Operation` record and `OperationStatus` enum in `Company.Contracts.Shared`; the platform package provides the `operations` endpoint group and storage abstraction.

## Conformance

Spectral `202-has-location-and-operation-body` (Prop. II.15). Catalogue CI validates examples and the conditional `required` rules.
