# Prop. IX.2 - A breaking change is a new major version published side by side

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A breaking change (Def. I.14) MUST be published as a new major version (Def. IX.2) of the contract, using the medium-specific form of Prop. II.7 (URI path segment), Prop. III.2 (event type suffix) or Prop. IV.1 (`$id` path). The previous major MUST continue to be served unchanged, in the same environments, until it reaches Sunset (Def. IX.4). The new major MUST NOT become Active while two majors of the same contract are already Active or Deprecated (Post. IX.1).

## Given

* Def. I.14, Def. IX.2, Def. IX.4
* Post. I.5, Post. IX.1
* CN 1, CN 2
* Prop. II.7, Prop. III.2, Prop. IV.1

## Demonstration

A breaking change, by Def. I.14, can make a conforming consumer of the current major fail, and by CN 2 the current major is owed; so the change cannot be applied to it. The only remaining place for the change is a version that no existing consumer has bound to, which by Def. IX.2 is a new major. Since consumers move at their own cadence (Post. I.5), the old major must keep being served beside the new one; CN 1 guarantees that consumers of the old major cannot observe the existence of the new. Post. IX.1 bounds the number of simultaneously served majors, so the new major waits until the oldest one has left. Books II, III and IV already fix how a major is spelt in each medium, so this proposition only requires that the spelling be used. ∎ Q.E.D.

## Corollaries

* **Cor. IX.2.1** - Publishing major N+1 does not by itself deprecate major N. Deprecation is a separate, dated act (Prop. IX.3).
* **Cor. IX.2.2** - Major N+1 starts in the Proposed stage and becomes Active only when its migration guide exists (Prop. IX.7) and Post. IX.1 is satisfied.

## Construction

* API: API Gateway stage or route prefix `/v{N+1}` deployed beside `/v{N}`; both served by the same .NET service using ASP.NET Core API versioning (`Asp.Versioning.Http`) with URL segment versioning, or by two deployments behind one API Gateway.
* Events: new event type `company.orders.order-placed.v2` registered in the EventBridge Schema Registry beside `.v1`; producer emits both for the overlap period from one outbox row (Prop. III.5).
* Shared schemas: new `$id` path `/schemas/v2/money.json` in the catalogue; NuGet and npm packages carry both namespaces (`Company.Contracts.Shared.V1`, `.V2`).
* Catalogue entry for the new major created in stage `Proposed` (Prop. IX.8).
* Architecture test: `Company.Architecture.Tests` rule asserting that no contract has more than two majors in Active or Deprecated in the catalogue.

## Conformance

CI check `contract-diff` (Prop. IX.1) fails the build when a breaking change is found under an existing major; catalogue lint (Prop. IX.8) fails when a third major is marked Active.

## Scholium

Dual-emitting events for the overlap period is the cost of Post. I.5. Where the producer can generate the `.v1` payload from the `.v2` payload by a pure function, that function is the adapter of Prop. IX.7 and should live in the producer, not in every consumer.
