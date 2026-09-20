# Prop. II.5 - Collections are paginated by opaque cursor

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Every operation that returns a collection MUST return the shared `Page` envelope (Prop. IV.6): `{ "items": [...], "pageInfo": { "nextCursor": ..., "hasNext": ... } }`. It MUST accept the query parameters `cursor` (Def. II.7) and `limit` (integer, default and maximum declared in the contract, maximum ≤ 200). The collection's ordering MUST be total and declared. Offset/page-number pagination and `totalCount` MUST NOT be exposed unless the contract marks the operation `x-pagination: offset` with a recorded exception (Def. I.20).

## Given

Def. II.3, Def. II.6, Def. II.7, Def. II.14, Post. I.4, Post. I.5, Post. II.5, CN 3, CN 5, Prop. IV.6.

## Demonstration

A collection has a declared total ordering (Def. II.3); a page is a contiguous subset of it (Def. II.6). A position in a total ordering can be named by the key of its last element, which is what a cursor is (Def. II.7). An offset, by contrast, names a position only relative to a snapshot that no longer exists after concurrent writes, so under Post. I.5 two adjacent offset pages may overlap or skip items. A cursor is opaque (Def. II.7) so that the producer may change the ordering key, index or storage without a breaking change, which CN 3 permits only if the consumer never relied on the cursor's content. One envelope for all collections follows from CN 5 and Def. II.14. `totalCount` is excluded by default because it costs a full scan on every page and its value is stale by the time it is read (Post. I.5). ∎ Q.E.D.

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

`Cursor` (encode/decode, signed with an HMAC so tampering is detectable) ships in `Company.Contracts.Shared` (Prop. IV.1). The `Page<T>` and `PageInfo` types come from the same package. Because identifiers are ULIDs (Prop. IV.2), `(createdAt, id)` is always a total order.

## Conformance

Spectral: `collection-returns-page` (array responses are wrapped in the shared `Page` `$ref`), `collection-has-cursor-limit` (both params declared), `no-offset-params` (no `page`, `offset`, `skip` query params unless `x-pagination: offset`).

## Scholium

Reporting and back-office screens sometimes genuinely need "jump to page 37 of 120". That is what the recorded exception is for; it is expected to be rare and to sit behind an administrative API, not a public one.
