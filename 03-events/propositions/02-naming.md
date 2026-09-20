# Prop. III.2 - Event types are named `<context>.<aggregate>.<fact>.v<major>`

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

The `type` attribute MUST match `^com\.company\.[a-z][a-z0-9-]*\.[a-z][a-z0-9-]*\.[a-z][a-z0-9-]*\.v[1-9][0-9]*$`, read as `com.company.<bounded-context>.<aggregate>.<fact>.v<major>`, where `<fact>` is a past-tense verb phrase for events (`placed`, `payment-captured`, `address-changed`) and an imperative verb phrase for commands (`capture-payment`). The `<bounded-context>` segment MUST be the context's registered name ([Book VII](../../07-data-ownership/README.md)). The `source` attribute MUST be `/<bounded-context>/<service-name>`.

## Given

[Def. I.5](../../01-foundations/definitions.md#Def.%20I.5%20-%20Bounded%20Context), [Def. I.9](../../01-foundations/definitions.md#Def.%20I.9%20-%20Event), [Def. I.10](../../01-foundations/definitions.md#Def.%20I.10%20-%20Command), [Def. III.5](../definitions.md#Def.%20III.5%20-%20Event%20Type), [Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution), [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape), [Prop. II.7](../../02-api-guidelines/propositions/07-versioning.md), [Prop. IX.2](../../09-versioning-and-deprecation/propositions/02-breaking-change-is-a-new-major-side-by-side.md).

## Demonstration

An event type is a contract ([Def. III.5](../definitions.md#Def.%20III.5%20-%20Event%20Type)) and by [Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution) several versions of it will coexist; the major version therefore has to be part of the name, so that a consumer subscribes to exactly the shape it understands, just as the URI carries the major in [Book II](../../02-api-guidelines/README.md) ([Prop. II.7](../../02-api-guidelines/propositions/07-versioning.md), [Prop. IX.2](../../09-versioning-and-deprecation/propositions/02-breaking-change-is-a-new-major-side-by-side.md)). Events are facts in the past tense ([Def. I.9](../../01-foundations/definitions.md#Def.%20I.9%20-%20Event)) and commands are imperatives ([Def. I.10](../../01-foundations/definitions.md#Def.%20I.10%20-%20Command)); the grammar of the name tells a reader which it is without opening the schema ([CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit)). The context and aggregate segments scope the name so that two contexts may each have an `order` without collision ([Def. I.5](../../01-foundations/definitions.md#Def.%20I.5%20-%20Bounded%20Context)). One grammar is [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape). ∎ Q.E.D.

## Corollaries

* **Cor. III.2.1** - Rules may subscribe by prefix (`com.company.orders.order.`) to receive every fact about an aggregate; the trailing major still selects the shape.
* **Cor. III.2.2** - A new major of an event type is a new event type; the old one continues to be published until sunset ([Book IX](../../09-versioning-and-deprecation/README.md)).
* **Cor. III.2.3** - Renaming a fact is a new event type plus deprecation of the old one; there is no in-place rename.

## Construction

Registered in the catalogue as `schemas/orders/v1/order-placed.json` with `"$id": "https://schemas.company.com/orders/v1/order-placed.json"` and `"x-event-type": "com.company.orders.order.placed.v1"`.

## Conformance

Catalogue CI validates every `x-event-type` against the regex and checks the context segment against the context register. The bus-level envelope rule ([Prop. III.1](01-envelope.md) conformance) rejects non-matching `detail-type`.

## Scholium

Examples: `com.company.orders.order.placed.v1`, `com.company.billing.invoice.issued.v2`, `com.company.identity.user.email-verified.v1`, command: `com.company.billing.invoice.issue.v1` (sent to the billing command queue, [Prop. III.11](11-commands.md)).
