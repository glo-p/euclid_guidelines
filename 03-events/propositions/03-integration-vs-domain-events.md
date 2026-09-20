# Prop. III.3 - Only integration events cross a service boundary

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

A service MUST NOT publish its domain events ([Def. III.1](../definitions.md#Def.%20III.1%20-%20Domain%20Event)) or persistence-level change records to the bus. What crosses the boundary MUST be an integration event ([Def. III.2](../definitions.md#Def.%20III.2%20-%20Integration%20Event)) with a schema designed for consumers, registered under [Prop. III.4](04-schemas-and-registry.md), and raised deliberately by the publisher. The mapping from domain event to integration event is code in the publisher and is the place where the publisher decides what it is willing to be held to.

## Given

[Def. I.2](../../01-foundations/definitions.md#Def.%20I.2%20-%20Contract), [Def. III.1](../definitions.md#Def.%20III.1%20-%20Domain%20Event), [Def. III.2](../definitions.md#Def.%20III.2%20-%20Integration%20Event), [Post. I.3](../../01-foundations/postulates.md#Post.%20I.3%20-%20Autonomy), [CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed), [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts).

## Demonstration

A domain event is shaped by the internal model and changes whenever the model does; an integration event is a contract ([Def. III.2](../definitions.md#Def.%20III.2%20-%20Integration%20Event)) and by [CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed) is owed once published. If domain events went straight to the bus, every refactoring of the model would be a breaking change, and the service would in practice lose the autonomy [Post. I.3](../../01-foundations/postulates.md#Post.%20I.3%20-%20Autonomy) guarantees. By [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit) a consumer may rely only on what is declared, so the declared thing must be designed, not leaked. [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts) says the architecture is the set of contracts; a service that leaks its internals has no boundary and therefore, architecturally, does not exist as a service. ∎ Q.E.D.

## Corollaries

* **Cor. III.3.1** - Change data capture from a service's database is permitted only as an *internal* input to that service's own outbox relay, never as a public feed.
* **Cor. III.3.2** - An integration event may aggregate several domain events (one `order.placed.v1` for many internal `LineAdded`s).
* **Cor. III.3.3** - Integration events use catalogue types (`Money`, `Identifier`, `Timestamp`) in their payloads ([CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape)), never internal entity classes.

## Construction

```
Domain/            OrderPlaced : IDomainEvent            (internal, in-process handler)
Contracts/Events/  OrderPlacedV1 : record (from generated schema types)   (public)
Application/       OrderPlacedIntegrationMapper : IDomainEventHandler<OrderPlaced>
                   -> writes OrderPlacedV1 to the outbox (Prop. III.5)
```

Architecture test: no type in `Domain` is referenced from `Contracts`; no type outside `Contracts` is serialised to the outbox.

## Conformance

The architecture test above, in the template. Catalogue CI rejects a schema whose `$id` is not under a registered context.
