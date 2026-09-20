# Prop. III.2 - Event types are named `<context>.<aggregate>.<fact>.v<major>`

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

The `type` attribute MUST match `^com\.company\.[a-z][a-z0-9-]*\.[a-z][a-z0-9-]*\.[a-z][a-z0-9-]*\.v[1-9][0-9]*$`, read as `com.company.<bounded-context>.<aggregate>.<fact>.v<major>`, where `<fact>` is a past-tense verb phrase for events (`placed`, `payment-captured`, `address-changed`) and an imperative verb phrase for commands (`capture-payment`). The `<bounded-context>` segment MUST be the context's registered name (Book VII). The `source` attribute MUST be `/<bounded-context>/<service-name>`.

## Given

Def. I.5, Def. I.9, Def. I.10, Def. III.5, Post. I.5, CN 3, CN 5, Prop. II.7, Prop. IX.2.

## Demonstration

An event type is a contract (Def. III.5) and by Post. I.5 several versions of it will coexist; the major version therefore has to be part of the name, so that a consumer subscribes to exactly the shape it understands, just as the URI carries the major in Book II (Prop. II.7, Prop. IX.2). Events are facts in the past tense (Def. I.9) and commands are imperatives (Def. I.10); the grammar of the name tells a reader which it is without opening the schema (CN 3). The context and aggregate segments scope the name so that two contexts may each have an `order` without collision (Def. I.5). One grammar is CN 5. ∎ Q.E.D.

## Corollaries

* **Cor. III.2.1** - Rules may subscribe by prefix (`com.company.orders.order.`) to receive every fact about an aggregate; the trailing major still selects the shape.
* **Cor. III.2.2** - A new major of an event type is a new event type; the old one continues to be published until sunset (Book IX).
* **Cor. III.2.3** - Renaming a fact is a new event type plus deprecation of the old one; there is no in-place rename.

## Construction

Registered in the catalogue as `schemas/orders/v1/order-placed.json` with `"$id": "https://schemas.company.com/orders/v1/order-placed.json"` and `"x-event-type": "com.company.orders.order.placed.v1"`.

## Conformance

Catalogue CI validates every `x-event-type` against the regex and checks the context segment against the context register. The bus-level envelope rule (Prop. III.1 conformance) rejects non-matching `detail-type`.

## Scholium

Examples: `com.company.orders.order.placed.v1`, `com.company.billing.invoice.issued.v2`, `com.company.identity.user.email-verified.v1`, command: `com.company.billing.invoice.issue.v1` (sent to the billing command queue, Prop. III.11).
