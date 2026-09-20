# Prop. II.15 - Long-running work is an operation resource

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Any request whose processing may exceed 10 seconds, or that is executed asynchronously ([Def. II.17](../definitions.md#Def.%20II.17%20-%20Long-Running%20Operation)), MUST return `202 Accepted` with a `Location` header pointing at an operation resource `/v1/operations/{operationId}` and a body conforming to the shared `Operation` schema (`{ id, status, createdAt, updatedAt, result?, error? }` with `status` in `pending | running | succeeded | failed | cancelled`). The client polls the operation resource; when `succeeded`, `result` contains the created resource's URI or the outcome; when `failed`, `error` is a Problem ([Prop. II.4](04-problem-details.md)). Completion SHOULD also be published as an event ([Book III](../../03-events/README.md)) so that consumers need not poll.

## Given

[Def. II.17](../definitions.md#Def.%20II.17%20-%20Long-Running%20Operation), [Post. I.4](../../01-foundations/postulates.md#Post.%20I.4%20-%20Unreliable%20Network), [Post. II.1](../postulates.md#Post.%20II.1%20-%20HTTP%20Is%20Authoritative), [Post. II.4](../postulates.md#Post.%20II.4%20-%20One%20Ingress), [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape), [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated), [Prop. II.3](03-methods-and-status-codes.md), [II.4](04-problem-details.md), [Prop. III.1](../../03-events/propositions/01-envelope.md).

## Demonstration

The edge ([Post. II.4](../postulates.md#Post.%20II.4%20-%20One%20Ingress)) has a fixed request timeout (29 s on API Gateway), and under [Post. I.4](../../01-foundations/postulates.md#Post.%20I.4%20-%20Unreliable%20Network) any long connection may drop, so a synchronous response cannot be relied upon for long work. HTTP provides `202` for exactly this ([Post. II.1](../postulates.md#Post.%20II.1%20-%20HTTP%20Is%20Authoritative)). The client then needs something to ask about, and by [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit) that something must be in the contract: an addressable resource with a declared shape. One shape for all operations is [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape). The same fact should be observable by other services without polling ([CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated) read constructively), which is what the event is for. ∎ Q.E.D.

## Corollaries

* **Cor. II.15.1** - Operation resources are retained for at least 24 hours after completion and then return `410 Gone`.
* **Cor. II.15.2** - `GET /v1/operations/{id}` supports `Retry-After` hints while `pending`/`running`.
* **Cor. II.15.3** - Bulk requests (`POST /v1/orders:batch` style) are modelled as a long-running operation whose `result` is a list of per-item outcomes; they are the only place a `200` body may contain per-item failures, and each such failure is a Problem.

## Construction

* Accept the request, persist an `Operation` row and an outbox message ([Prop. III.5](../../03-events/propositions/05-transactional-outbox.md)), return `202`.
* A consumer (Lambda or hosted service) does the work, updates the row, and publishes `…operation.completed.v1`.
* `Operation` schema lives in the catalogue (`shared/v1/operation.json`).

## Conformance

Spectral: `202-has-location-and-operation-body`. Architecture test: no endpoint has a request timeout above 10 s.
