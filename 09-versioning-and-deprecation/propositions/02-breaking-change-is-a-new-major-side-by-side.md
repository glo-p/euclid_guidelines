# Prop. IX.2 - A breaking change is a new major version published side by side

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A breaking change ([Def. I.14](../../01-foundations/definitions.md#Def.%20I.14%20-%20Breaking%20Change)) MUST be published as a new major version ([Def. IX.2](../definitions.md#Def.%20IX.2%20-%20Major%20Version)) of the contract, using the medium-specific form of [Prop. II.7](../../02-api-guidelines/propositions/07-versioning.md) (URI path segment), [Prop. III.2](../../03-events/propositions/02-naming.md) (event type suffix) or [Prop. IV.1](../../04-shared-schemas/propositions/01-catalogue-and-distribution.md) (`$id` path). The previous major MUST continue to be served unchanged, in the same environments, until it reaches Sunset ([Def. IX.4](../definitions.md#Def.%20IX.4%20-%20Lifecycle%20Stage)). The new major MUST NOT become Active while two majors of the same contract are already Active or Deprecated ([Post. IX.1](../postulates.md#Post.%20IX.1%20-%20Two%20Majors%20at%20Most)).

## Given

* [Def. I.14](../../01-foundations/definitions.md#Def.%20I.14%20-%20Breaking%20Change), [Def. IX.2](../definitions.md#Def.%20IX.2%20-%20Major%20Version), [Def. IX.4](../definitions.md#Def.%20IX.4%20-%20Lifecycle%20Stage)
* [Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution), [Post. IX.1](../postulates.md#Post.%20IX.1%20-%20Two%20Majors%20at%20Most)
* [CN 1](../../01-foundations/common-notions.md#CN%201%20-%20Substitutability), [CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed)
* [Prop. II.7](../../02-api-guidelines/propositions/07-versioning.md), [Prop. III.2](../../03-events/propositions/02-naming.md), [Prop. IV.1](../../04-shared-schemas/propositions/01-catalogue-and-distribution.md)

## Demonstration

A breaking change, by [Def. I.14](../../01-foundations/definitions.md#Def.%20I.14%20-%20Breaking%20Change), can make a conforming consumer of the current major fail, and by [CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed) the current major is owed; so the change cannot be applied to it. The only remaining place for the change is a version that no existing consumer has bound to, which by [Def. IX.2](../definitions.md#Def.%20IX.2%20-%20Major%20Version) is a new major. Since consumers move at their own cadence ([Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution)), the old major must keep being served beside the new one; [CN 1](../../01-foundations/common-notions.md#CN%201%20-%20Substitutability) guarantees that consumers of the old major cannot observe the existence of the new. [Post. IX.1](../postulates.md#Post.%20IX.1%20-%20Two%20Majors%20at%20Most) bounds the number of simultaneously served majors, so the new major waits until the oldest one has left. Books II, III and IV already fix how a major is spelt in each medium, so this proposition only requires that the spelling be used. ∎ Q.E.D.

## Corollaries

* **Cor. IX.2.1** - Publishing major N+1 does not by itself deprecate major N. Deprecation is a separate, dated act ([Prop. IX.3](03-deprecation-is-declared-in-the-contract.md)).
* **Cor. IX.2.2** - Major N+1 starts in the Proposed stage and becomes Active only when its migration guide exists ([Prop. IX.7](07-every-major-ships-with-a-migration-guide.md)) and [Post. IX.1](../postulates.md#Post.%20IX.1%20-%20Two%20Majors%20at%20Most) is satisfied.

## Construction

* API: API Gateway stage or route prefix `/v{N+1}` deployed beside `/v{N}`; both served by the same .NET service using ASP.NET Core API versioning (`Asp.Versioning.Http`) with URL segment versioning, or by two deployments behind one API Gateway.
* Events: new event type `company.orders.order-placed.v2` registered in the EventBridge Schema Registry beside `.v1`; producer emits both for the overlap period from one outbox row ([Prop. III.5](../../03-events/propositions/05-transactional-outbox.md)).
* Shared schemas: new `$id` path `/schemas/v2/money.json` in the catalogue; NuGet and npm packages carry both namespaces (`Company.Contracts.Shared.V1`, `.V2`).
* Catalogue entry for the new major created in stage `Proposed` ([Prop. IX.8](08-lifecycle-stage-is-machine-readable.md)).
* Architecture test: `Company.Architecture.Tests` rule asserting that no contract has more than two majors in Active or Deprecated in the catalogue.

## Conformance

CI check `contract-diff` ([Prop. IX.1](01-only-compatible-changes-within-a-major.md)) fails the build when a breaking change is found under an existing major; catalogue lint ([Prop. IX.8](08-lifecycle-stage-is-machine-readable.md)) fails when a third major is marked Active.

## Scholium

Dual-emitting events for the overlap period is the cost of [Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution). Where the producer can generate the `.v1` payload from the `.v2` payload by a pure function, that function is the adapter of [Prop. IX.7](07-every-major-ships-with-a-migration-guide.md) and should live in the producer, not in every consumer.
