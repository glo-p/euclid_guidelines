# Prop. III.3 - Only integration events cross a service boundary

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

A service MUST NOT publish its domain events (Def. III.1) or persistence-level change records to the bus. What crosses the boundary MUST be an integration event (Def. III.2) with a schema designed for consumers, registered under Prop. III.4, and raised deliberately by the publisher. The mapping from domain event to integration event is code in the publisher and is the place where the publisher decides what it is willing to be held to.

## Given

Def. I.2, Def. III.1, Def. III.2, Post. I.3, CN 2, CN 3, CN 8.

## Demonstration

A domain event is shaped by the internal model and changes whenever the model does; an integration event is a contract (Def. III.2) and by CN 2 is owed once published. If domain events went straight to the bus, every refactoring of the model would be a breaking change, and the service would in practice lose the autonomy Post. I.3 guarantees. By CN 3 a consumer may rely only on what is declared, so the declared thing must be designed, not leaked. CN 8 says the architecture is the set of contracts; a service that leaks its internals has no boundary and therefore, architecturally, does not exist as a service. ∎ Q.E.D.

## Corollaries

* **Cor. III.3.1** - Change data capture from a service's database is permitted only as an *internal* input to that service's own outbox relay, never as a public feed.
* **Cor. III.3.2** - An integration event may aggregate several domain events (one `order.placed.v1` for many internal `LineAdded`s).
* **Cor. III.3.3** - Integration events use catalogue types (`Money`, `Identifier`, `Timestamp`) in their payloads (CN 5), never internal entity classes.

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
