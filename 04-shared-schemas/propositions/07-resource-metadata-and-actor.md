# Prop. IV.7 - Resources carry `ResourceMetadata`; actions carry an `Actor`

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Every item representation MUST have a field `metadata` conforming to [`resource-metadata.json`](../schemas/shared/v1/resource-metadata.json): `createdAt`, `updatedAt`, `version` required; `createdBy`, `updatedBy`, `deletedAt` optional. `version` MUST equal the unquoted `ETag` ([Prop. II.9](../../02-api-guidelines/propositions/09-optimistic-concurrency.md)) and, for aggregates that publish events, the latest `sequence` ([Cor. III.7.1](../../03-events/propositions/07-ordering.md#Corollaries)). Wherever a contract records who performed something (`createdBy`, an audit event, an approval) it MUST use [`actor.json`](../schemas/shared/v1/actor.json). A separate response-level "meta" object is NOT defined: request correlation travels in the `traceparent` header ([Prop. II.10](../../02-api-guidelines/propositions/10-headers-and-metadata.md)) and pagination state in `pageInfo` ([Prop. IV.6](06-page.md)).

## Given

[Def. II.11](../../02-api-guidelines/definitions.md#Def.%20II.11%20-%20Entity%20Tag), [Def. I.22](../../01-foundations/definitions.md#Def.%20I.22%20-%20Correlation), [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape), [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated), [Prop. II.9](../../02-api-guidelines/propositions/09-optimistic-concurrency.md), [II.10](../../02-api-guidelines/propositions/10-headers-and-metadata.md), [III.7](../../03-events/propositions/07-ordering.md), [V.9](../../05-security/propositions/09-audit-events.md).

## Demonstration

Every consumer that caches, sorts by recency, resolves conflicts or displays "last changed by" needs the same four facts about a resource; one shape is [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape), and a fixed field name (`metadata`) lets generated clients and front-ends find it without per-resource knowledge ([CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit)). Tying `version` to the ETag and to the event sequence makes three views of the same aggregate comparable, which is what lets a consumer decide whether an event is newer than what it fetched ([CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated), [Prop. III.7](../../03-events/propositions/07-ordering.md)). `Actor` exists because "who" appears in metadata, in audit events ([Prop. V.9](../../05-security/propositions/09-audit-events.md)) and in domain records, and by [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape) it has one shape. A response-level meta object would duplicate information already carried by headers and `pageInfo`, so it is deliberately absent. ∎ Q.E.D.

## Corollaries

* **Cor. IV.7.1** - `metadata` is read-only on input; a `PUT` or `PATCH` body that includes it is accepted and the field ignored ([CN 6](../../01-foundations/common-notions.md#CN%206%20-%20The%20Producer%20Pays%20for%20Stability%3B%20the%20Consumer%20Pays%20for%20Tolerance)), except that `If-Match` still governs concurrency.
* **Cor. IV.7.2** - `Actor.displayName` is Internal-classified and is omitted from events ([Prop. III.10](../../03-events/propositions/10-payload-design.md)) unless the event's purpose is display.
* **Cor. IV.7.3** - `deletedAt` appears only on administrative APIs that expose soft-deleted resources; public APIs return `404` for them ([Prop. II.3](../../02-api-guidelines/propositions/03-methods-and-status-codes.md)).

## Construction

C#: `ResourceMetadata` record and `Actor` record in `Company.Contracts.Shared`; an EF Core interceptor in `Company.Platform.AspNetCore` populates `CreatedAt/By`, `UpdatedAt/By` and increments `Version` on `SaveChanges`, using the current principal ([Book V](../../05-security/README.md)) as the actor. TypeScript: `interface ResourceMetadata`, `interface Actor`.

## Conformance

Spectral `item-has-metadata` ([Prop. II.10](../../02-api-guidelines/propositions/10-headers-and-metadata.md)). Template test: `version` in body equals `ETag` header.
