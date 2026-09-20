# Prop. II.2 - Resources are plural nouns in kebab-case paths

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Paths MUST be composed of the base path `/v{n}`, followed by segments that alternate between plural kebab-case nouns naming collections and identifiers naming items: `/v1/purchase-orders/{purchaseOrderId}/lines/{lineId}`. Paths MUST NOT contain verbs, file extensions, query-like segments or trailing slashes. Where an action cannot be expressed as a state change on a resource it MUST be modelled as a sub-resource noun (`/v1/purchase-orders/{id}/approvals`) and created with `POST`. Path parameters MUST be named `{<singularNoun>Id}` in camelCase.

## Given

Def. I.7, Def. II.3, Def. II.4, Def. II.18, Post. II.1, CN 1, CN 5.

## Demonstration

A resource is a thing with a stable identifier (Def. I.7); HTTP methods already supply the verbs (Post. II.1), so a verb in the path either duplicates the method or contradicts it. Collections and items alternate by definition (Def. II.3, II.4), which fixes the segment pattern. One casing and one number (plural) make every path predictable to a consumer who has seen any other path, which is the practical form of CN 5 applied to naming. Actions that are not state changes still produce a record of having been requested; that record is a resource (Def. II.18), so it is named as a noun and created, which keeps the vocabulary closed. ∎ Q.E.D.

## Corollaries

* **Cor. II.2.1** - The only allowed characters in a static path segment are `a-z`, `0-9` and `-`.
* **Cor. II.2.2** - Nesting is at most two levels deep (`/a/{aId}/b/{bId}`). Deeper relationships are expressed by filtering (`/v1/lines?purchaseOrderId=…`), because an item that can be addressed without its parent is not a sub-resource.
* **Cor. II.2.3** - Singleton resources (`/v1/tenants/{tenantId}/settings`) are the only permitted singular nouns; they are documented as such in the contract.

## Construction

Spectral rules in `@company/spectral-ruleset`:

| Rule | Checks |
|---|---|
| `paths-kebab-case` | static segments match `^[a-z0-9]+(-[a-z0-9]+)*$` |
| `paths-no-trailing-slash` | |
| `paths-no-verbs` | segment not in a deny-list of common verbs (`get`, `create`, `update`, `delete`, `do`, `run`, `execute`, `process`) |
| `path-params-camel-id` | `{…}` matches `^[a-z][A-Za-z0-9]*Id$` |
| `paths-version-prefix` | first segment matches `^v[0-9]+$` |
| `paths-max-depth` | at most 5 segments after the version |

## Conformance

The Spectral rules above.

## Scholium

Examples:

| Intent | Path |
|---|---|
| List orders | `GET /v1/orders` |
| Read one | `GET /v1/orders/{orderId}` |
| Create | `POST /v1/orders` |
| Cancel an order (an action) | `POST /v1/orders/{orderId}/cancellations` |
| Order lines | `GET /v1/orders/{orderId}/lines` |
| Search across tenants (admin) | `GET /v1/orders?tenantId=…` |

Rejected alternative: `POST /v1/orders/{id}/cancel`. It reads well but introduces an open-ended vocabulary of verbs that every consumer has to learn per API. The sub-resource form also gives the cancellation an identifier, a timestamp and a body, which the audit requirements of Book V need anyway.
