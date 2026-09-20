# Prop. II.6 - Filtering, sorting and field selection use one grammar

| | |
|---|---|
| **Level** | SHOULD |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Collection operations that support filtering SHOULD accept one query parameter per filterable field, named after the field, with the operator suffixes `.gte`, `.gt`, `.lte`, `.lt`, `.ne`, `.in` (comma-separated) and `.contains` where meaningful. Sorting SHOULD use a single `sort` parameter whose value is a comma-separated list of field names, each optionally prefixed with `-` for descending. Field selection, where offered, SHOULD use `fields` with a comma-separated list. Every filterable and sortable field MUST be enumerated in the contract; an unknown filter or sort field yields `400`.

## Given

Def. II.3, Def. II.7, Def. II.15, CN 3, CN 5, Prop. II.5.

## Demonstration

Filter and sort parameters are part of the interface and therefore of the contract (CN 3); enumerating them is what makes an unknown one an error rather than silently ignored input. One grammar across APIs is CN 5 applied to query strings. The grammar must compose with cursors (Prop. II.5, Cor. II.5.1), which requires that the sort be total; the producer therefore always appends `id` as the final tie-breaker even when the consumer did not ask for it. ∎ Q.E.D.

## Corollaries

* **Cor. II.6.1** - Free-text search is a separate parameter `q`, and its semantics (which fields, which matching) are stated in the operation's description.
* **Cor. II.6.2** - Filtering on a shared `Money` field filters on `amount` with the currency fixed by a separate `currency` filter; there is no cross-currency comparison.

## Construction

```
GET /v1/orders?status.in=placed,paid&createdAt.gte=2026-09-01T00:00:00Z&sort=-createdAt,customerId&limit=50
```

OpenAPI: declare each parameter explicitly (no `additionalProperties` style free-for-all). A `FilterBinder` helper in the project template maps the suffixes to `Expression<Func<T,bool>>` for EF Core.

## Conformance

Spectral: `sort-param-shape` (if `sort` exists it is a string with the documented pattern), `filter-params-documented` (every query param has a `description`).

## Scholium

Rejected alternatives: OData `$filter`, RSQL and GraphQL-style filter objects. They are more expressive and considerably harder to lint, to authorise field-by-field, and to index for. If a team needs them, that is a sign the operation is a search endpoint and should be designed as one (`POST /v1/orders/searches` with a body schema).
