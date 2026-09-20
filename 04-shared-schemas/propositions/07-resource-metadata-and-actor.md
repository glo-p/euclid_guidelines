# Prop. IV.7 - Resources carry `ResourceMetadata`; actions carry an `Actor`

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Every item representation MUST have a field `metadata` conforming to [`resource-metadata.json`](../schemas/shared/v1/resource-metadata.json): `createdAt`, `updatedAt`, `version` required; `createdBy`, `updatedBy`, `deletedAt` optional. `version` MUST equal the unquoted `ETag` (Prop. II.9) and, for aggregates that publish events, the latest `sequence` (Cor. III.7.1). Wherever a contract records who performed something (`createdBy`, an audit event, an approval) it MUST use [`actor.json`](../schemas/shared/v1/actor.json). A separate response-level "meta" object is NOT defined: request correlation travels in the `traceparent` header (Prop. II.10) and pagination state in `pageInfo` (Prop. IV.6).

## Given

Def. II.11, Def. I.22, CN 3, CN 5, CN 7, Prop. II.9, II.10, III.7, V.9.

## Demonstration

Every consumer that caches, sorts by recency, resolves conflicts or displays "last changed by" needs the same four facts about a resource; one shape is CN 5, and a fixed field name (`metadata`) lets generated clients and front-ends find it without per-resource knowledge (CN 3). Tying `version` to the ETag and to the event sequence makes three views of the same aggregate comparable, which is what lets a consumer decide whether an event is newer than what it fetched (CN 7, Prop. III.7). `Actor` exists because "who" appears in metadata, in audit events (Prop. V.9) and in domain records, and by CN 5 it has one shape. A response-level meta object would duplicate information already carried by headers and `pageInfo`, so it is deliberately absent. ∎ Q.E.D.

## Corollaries

* **Cor. IV.7.1** - `metadata` is read-only on input; a `PUT` or `PATCH` body that includes it is accepted and the field ignored (CN 6), except that `If-Match` still governs concurrency.
* **Cor. IV.7.2** - `Actor.displayName` is Internal-classified and is omitted from events (Prop. III.10) unless the event's purpose is display.
* **Cor. IV.7.3** - `deletedAt` appears only on administrative APIs that expose soft-deleted resources; public APIs return `404` for them (Prop. II.3).

## Construction

C#: `ResourceMetadata` record and `Actor` record in `Company.Contracts.Shared`; an EF Core interceptor in `Company.Platform.AspNetCore` populates `CreatedAt/By`, `UpdatedAt/By` and increments `Version` on `SaveChanges`, using the current principal (Book V) as the actor. TypeScript: `interface ResourceMetadata`, `interface Actor`.

## Conformance

Spectral `item-has-metadata` (Prop. II.10). Template test: `version` in body equals `ETag` header.
