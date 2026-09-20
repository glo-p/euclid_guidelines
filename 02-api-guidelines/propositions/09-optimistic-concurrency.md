# Prop. II.9 - Updates are conditional on an entity tag

| | |
|---|---|
| **Level** | SHOULD |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Every item resource that can be updated SHOULD return an `ETag` header ([Def. II.11](../definitions.md#Def.%20II.11%20-%20Entity%20Tag)) on `GET`, `PUT` and `PATCH` responses, and `PUT`/`PATCH`/`DELETE` SHOULD require `If-Match`. A mismatched `If-Match` MUST yield `412` with type `…/precondition-failed`. A missing `If-Match` on an operation that requires it MUST yield `428` with type `…/precondition-required`. The ETag value MUST equal the `version` field of the resource's `ResourceMetadata` ([Prop. IV.7](../../04-shared-schemas/propositions/07-resource-metadata-and-actor.md)), quoted.

## Given

[Def. II.11](../definitions.md#Def.%20II.11%20-%20Entity%20Tag), [Post. I.4](../../01-foundations/postulates.md#Post.%20I.4%20-%20Unreliable%20Network), [Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution), [Post. II.1](../postulates.md#Post.%20II.1%20-%20HTTP%20Is%20Authoritative), [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [Prop. IV.7](../../04-shared-schemas/propositions/07-resource-metadata-and-actor.md), [Prop. II.14](14-caching.md).

## Demonstration

Two clients may read the same resource and write back conflicting changes; under [Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution) this is certain to happen eventually. The last writer silently wins unless the producer can tell that the writer's view is stale, which requires the writer to say what it saw. HTTP already provides this (`ETag`/`If-Match`, [Post. II.1](../postulates.md#Post.%20II.1%20-%20HTTP%20Is%20Authoritative)), and it is the same validator that makes conditional reads cheap ([Prop. II.14](14-caching.md)), so one mechanism serves both. Tying the tag to the metadata `version` makes the tag visible in the body too, so a consumer that lost the header still has it ([CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit): it is in the contract either way). ∎ Q.E.D.

## Corollaries

* **Cor. II.9.1** - Sub-resources ([Def. II.18](../definitions.md#Def.%20II.18%20-%20Sub-resource)) carry their own ETag; updating a line does not change the order's ETag unless the order's representation changes.
* **Cor. II.9.2** - Weak ETags (`W/"…"`) are not used; the version is exact.

## Construction

```csharp
// EF Core: rowversion / xmin as concurrency token, surfaced as metadata.version
[Timestamp] public byte[] RowVersion { get; set; }
// Endpoint filter: compare If-Match to entity.Version before applying; on
// DbUpdateConcurrencyException -> 412 problem.
```

Response: `ETag: "00000000-0000-0000-0000-000000000042"` (or the base64 rowversion).

## Conformance

Spectral: `item-get-has-etag` (200 responses of item `GET` declare `ETag`), `update-declares-if-match`. Template test: stale `If-Match` returns `412`.

## Scholium

SHOULD rather than MUST because some resources are append-only or single-writer by construction, where the ceremony buys nothing. The deviation note in the repository should say which.
