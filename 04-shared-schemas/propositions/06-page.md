# Prop. IV.6 - Collections are a `Page` with `PageInfo`

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Every collection representation (Prop. II.5) MUST conform to [`page.json`](../schemas/shared/v1/page.json), specialised by `allOf` with a constrained `items` schema, and its `pageInfo` MUST conform to [`page-info.json`](../schemas/shared/v1/page-info.json). `pageInfo.hasNext` and `pageInfo.nextCursor` are required; `previousCursor` and `limit` are optional. `items` is never null. No other top-level members are added to a `Page` except `metadata` where the collection itself has lifecycle metadata.

## Given

Def. II.6, Def. II.7, Def. II.14, CN 3, CN 5, Prop. II.5.

## Demonstration

Prop. II.5 establishes cursor pagination; this fixes the one envelope (CN 5, Def. II.14) so that a generic client can iterate any collection with one loop. The two required members are exactly what that loop needs; everything else is optional so that producers pay only for what they offer, and by CN 3 offer it explicitly. Keeping the top level closed leaves the shape predictable for generated clients. ∎ Q.E.D.

## Corollaries

* **Cor. IV.6.1** - Cursors are opaque strings matching `^[A-Za-z0-9_-]{1,512}$`; the producer signs them (Prop. II.5 construction) so a tampered cursor is `400`.
* **Cor. IV.6.2** - Event payloads never contain a `Page`; an event carries a bounded list or a claim check (Prop. III.10).

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

Spectral `collection-returns-page` (Prop. II.5). Catalogue CI validates examples.
