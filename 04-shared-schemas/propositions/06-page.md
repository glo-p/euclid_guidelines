# Prop. IV.6 - Collections are a `Page` with `PageInfo`

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Every collection representation ([Prop. II.5](../../02-api-guidelines/propositions/05-cursor-pagination.md)) MUST conform to [`page.json`](../schemas/shared/v1/page.json), specialised by `allOf` with a constrained `items` schema, and its `pageInfo` MUST conform to [`page-info.json`](../schemas/shared/v1/page-info.json). `pageInfo.hasNext` and `pageInfo.nextCursor` are required; `previousCursor` and `limit` are optional. `items` is never null. No other top-level members are added to a `Page` except `metadata` where the collection itself has lifecycle metadata.

## Given

[Def. II.6](../../02-api-guidelines/definitions.md#Def.%20II.6%20-%20Page), [Def. II.7](../../02-api-guidelines/definitions.md#Def.%20II.7%20-%20Cursor), [Def. II.14](../../02-api-guidelines/definitions.md#Def.%20II.14%20-%20Envelope), [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape), [Prop. II.5](../../02-api-guidelines/propositions/05-cursor-pagination.md).

## Demonstration

[Prop. II.5](../../02-api-guidelines/propositions/05-cursor-pagination.md) establishes cursor pagination; this fixes the one envelope ([CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape), [Def. II.14](../../02-api-guidelines/definitions.md#Def.%20II.14%20-%20Envelope)) so that a generic client can iterate any collection with one loop. The two required members are exactly what that loop needs; everything else is optional so that producers pay only for what they offer, and by [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit) offer it explicitly. Keeping the top level closed leaves the shape predictable for generated clients. ∎ Q.E.D.

## Corollaries

* **Cor. IV.6.1** - Cursors are opaque strings matching `^[A-Za-z0-9_-]{1,512}$`; the producer signs them ([Prop. II.5](../../02-api-guidelines/propositions/05-cursor-pagination.md) construction) so a tampered cursor is `400`.
* **Cor. IV.6.2** - Event payloads never contain a `Page`; an event carries a bounded list or a claim check ([Prop. III.10](../../03-events/propositions/10-payload-design.md)).

## Construction

OpenAPI specialisation:

```yaml
OrderPage:
  allOf:
    - $ref: "https://schemas.company.com/shared/v1/page.json"
    - type: object
      properties:
        items:
          type: array
          items: { $ref: "#/components/schemas/Order" }
```

C#: `Page<T>(IReadOnlyList<T> Items, PageInfo PageInfo)`; TypeScript: `interface Page<T> { items: T[]; pageInfo: PageInfo }` with an `async function* iterate(fetch)` helper.

## Conformance

Spectral `collection-returns-page` ([Prop. II.5](../../02-api-guidelines/propositions/05-cursor-pagination.md)). Catalogue CI validates examples.
