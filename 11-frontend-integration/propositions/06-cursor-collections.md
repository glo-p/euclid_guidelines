# Prop. XI.6 - Collection UIs use cursor semantics

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A front-end presenting a collection MUST navigate it with the cursor semantics of the `Page` schema ([Prop. II.5](../../02-api-guidelines/propositions/05-cursor-pagination.md), [Prop. IV.6](../../04-shared-schemas/propositions/06-page.md)): next, previous where the API supplies a previous cursor, and continuation for infinite scroll. A front-end MUST NOT compute, display or request page numbers or total counts unless the API's contract explicitly declares them.

## Given

* [Def. I.2](../../01-foundations/definitions.md#Def.%20I.2%20-%20Contract), [Def. I.13](../../01-foundations/definitions.md#Def.%20I.13%20-%20Shared%20Schema), [Def. XI.1](../definitions.md#Def.%20XI.1%20-%20Front-end)
* [Post. I.4](../../01-foundations/postulates.md#Post.%20I.4%20-%20Unreliable%20Network), [Post. XI.3](../postulates.md#Post.%20XI.3%20-%20Contracts%20Only)
* [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 6](../../01-foundations/common-notions.md#CN%206%20-%20The%20Producer%20Pays%20for%20Stability%3B%20the%20Consumer%20Pays%20for%20Tolerance)
* [Prop. II.5](../../02-api-guidelines/propositions/05-cursor-pagination.md), [Prop. IV.6](../../04-shared-schemas/propositions/06-page.md), [Prop. XI.2](02-shared-schema-types.md)

## Demonstration

By [Prop. II.5](../../02-api-guidelines/propositions/05-cursor-pagination.md) collections are paginated by opaque cursor, and by [Prop. IV.6](../../04-shared-schemas/propositions/06-page.md) the response carries `items` and `pageInfo` with `nextCursor` and `hasNext`. A page number or total count is not in that contract, so by [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit) it does not exist for the consumer; a front-end that derives one is relying on an assumption about the producer's ordering and stability which [Post. I.4](../../01-foundations/postulates.md#Post.%20I.4%20-%20Unreliable%20Network) and the opacity of the cursor do not warrant. Where a contract does declare a total (as an explicit optional field), it is in the contract and may be shown. The front-end therefore builds its navigation from exactly the fields the shared schema provides ([Prop. XI.2](02-shared-schema-types.md)). ∎ Q.E.D.

## Corollaries

* **Cor. XI.6.1** - "Jump to page N" controls are not built against cursor APIs. A product need for them is a contract change request to the API, not a client-side computation.
* **Cor. XI.6.2** - A cursor is stored and replayed verbatim; a front-end never parses, constructs or modifies one.

## Construction

* Types: `Page<T>` and `PageInfo` from `@company/contracts` ([Prop. XI.2](02-shared-schema-types.md)); the generated client ([Prop. XI.1](01-generated-clients.md)) returns them typed.
* Navigation state: the front-end keeps a stack of cursors it has visited to offer "previous" where the API supplies no `previousCursor`; the stack is per view and is discarded on filter or sort change ([Prop. II.6](../../02-api-guidelines/propositions/06-filtering-and-sorting.md)).
* Infinite scroll: request the next page with `cursor=nextCursor` while `hasNext` is true; render `items` append-only; de-duplicate by `id` ([Prop. IV.2](../../04-shared-schemas/propositions/02-identifier.md)) because a page may be delivered twice ([Post. I.4](../../01-foundations/postulates.md#Post.%20I.4%20-%20Unreliable%20Network)).
* Page size: `limit` as the API declares; the front-end passes what the contract allows and does not assume a default.
* Framework-neutral helper: a small pure `paginate` state reducer in `@company/contracts` may be provided; it is optional.

## Conformance

CI check in every front-end repository: a lint rule forbids query parameters named `page`, `pageNumber`, `offset` or `totalCount` in calls through generated clients unless the corresponding OpenAPI operation declares them; UI test asserts that a collection view renders with `hasNext=false` and no total.

## Scholium

Totals are expensive for producers and misleading for consumers on live data, which is why [Book II](../../02-api-guidelines/README.md) makes them opt-in. Front-ends that need a sense of scale can show "more than N" from the pages loaded so far, which is honest. Nothing here prevents a producer from declaring `totalCount` in its contract for a small, stable collection; then it is contract and may be shown.
