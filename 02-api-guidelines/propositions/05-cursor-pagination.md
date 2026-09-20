# Prop. II.5 - Collections are paginated by opaque cursor

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Every operation that returns a collection MUST return the shared `Page` envelope ([Prop. IV.6](../../04-shared-schemas/propositions/06-page.md)): `{ "items": [...], "pageInfo": { "nextCursor": ..., "hasNext": ... } }`. It MUST accept the query parameters `cursor` ([Def. II.7](../definitions.md#Def.%20II.7%20-%20Cursor)) and `limit` (integer, default and maximum declared in the contract, maximum ≤ 200). The collection's ordering MUST be total and declared. Offset/page-number pagination and `totalCount` MUST NOT be exposed unless the contract marks the operation `x-pagination: offset` with a recorded exception ([Def. I.20](../../01-foundations/definitions.md#Def.%20I.20%20-%20Exception)).

## Given

[Def. II.3](../definitions.md#Def.%20II.3%20-%20Collection), [Def. II.6](../definitions.md#Def.%20II.6%20-%20Page), [Def. II.7](../definitions.md#Def.%20II.7%20-%20Cursor), [Def. II.14](../definitions.md#Def.%20II.14%20-%20Envelope), [Post. I.4](../../01-foundations/postulates.md#Post.%20I.4%20-%20Unreliable%20Network), [Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution), [Post. II.5](../postulates.md#Post.%20II.5%20-%20Consumers%20We%20Do%20Not%20Deploy), [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape), [Prop. IV.6](../../04-shared-schemas/propositions/06-page.md).

## Demonstration

A collection has a declared total ordering ([Def. II.3](../definitions.md#Def.%20II.3%20-%20Collection)); a page is a contiguous subset of it ([Def. II.6](../definitions.md#Def.%20II.6%20-%20Page)). A position in a total ordering can be named by the key of its last element, which is what a cursor is ([Def. II.7](../definitions.md#Def.%20II.7%20-%20Cursor)). An offset, by contrast, names a position only relative to a snapshot that no longer exists after concurrent writes, so under [Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution) two adjacent offset pages may overlap or skip items. A cursor is opaque ([Def. II.7](../definitions.md#Def.%20II.7%20-%20Cursor)) so that the producer may change the ordering key, index or storage without a breaking change, which [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit) permits only if the consumer never relied on the cursor's content. One envelope for all collections follows from [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape) and [Def. II.14](../definitions.md#Def.%20II.14%20-%20Envelope). `totalCount` is excluded by default because it costs a full scan on every page and its value is stale by the time it is read ([Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution)). ∎ Q.E.D.

## Corollaries

* **Cor. II.5.1** - A cursor is bound to the operation and its filter/sort parameters; presenting it with different parameters yields `400` with type `…/invalid-cursor`.
* **Cor. II.5.2** - `hasNext: false` with `nextCursor: null` terminates iteration; consumers loop on `hasNext`, never on `items.length < limit`.
* **Cor. II.5.3** - Previous-page navigation is optional: `pageInfo.previousCursor` MAY be provided; if absent the consumer keeps its own history of cursors.
* **Cor. II.5.4** - An empty collection is `200` with `items: []`, never `404`.

## Construction

Keyset pagination over `(sortKey, id)`:

```csharp
// cursor = base64url(JSON { "k": <last sort key>, "id": "<last ULID>", "v": 1 })
var (afterKey, afterId) = Cursor.Decode(request.Cursor);       // null on first page
var rows = await db.Orders
    .Where(o => o.TenantId == tenant)
    .Where(o => afterKey == null || o.CreatedAt > afterKey
             || (o.CreatedAt == afterKey && o.Id.CompareTo(afterId) > 0))
    .OrderBy(o => o.CreatedAt).ThenBy(o => o.Id)
    .Take(limit + 1)                                            // fetch one extra
    .ToListAsync(ct);
var hasNext = rows.Count > limit;
var items = rows.Take(limit).Select(ToDto).ToList();
return new Page<OrderDto>(items, new PageInfo(
    NextCursor: hasNext ? Cursor.Encode(items[^1].CreatedAt, items[^1].Id) : null,
    HasNext: hasNext));
```

`Cursor` (encode/decode, signed with an HMAC so tampering is detectable) ships in `Company.Contracts.Shared` ([Prop. IV.1](../../04-shared-schemas/propositions/01-catalogue-and-distribution.md)). The `Page<T>` and `PageInfo` types come from the same package. Because identifiers are ULIDs ([Prop. IV.2](../../04-shared-schemas/propositions/02-identifier.md)), `(createdAt, id)` is always a total order.

## Conformance

Spectral: `collection-returns-page` (array responses are wrapped in the shared `Page` `$ref`), `collection-has-cursor-limit` (both params declared), `no-offset-params` (no `page`, `offset`, `skip` query params unless `x-pagination: offset`).

## Scholium

Reporting and back-office screens sometimes genuinely need "jump to page 37 of 120". That is what the recorded exception is for; it is expected to be rare and to sit behind an administrative API, not a public one.
