# Prop. VII.3 - Read models are built from integration events and are rebuildable

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A read model (Def. VII.3) MUST be populated only by a projection (Def. VII.4) over the integration events (Prop. III.3) of the system of record. A read model MUST be rebuildable from the event archive (Prop. III.13) without assistance from the owning service, and its owning team MUST demonstrate a rebuild at least once per environment.

## Given

* Def. I.9, Def. VII.3, Def. VII.4
* Post. I.3, Post. I.4, Post. VII.2
* Prop. III.3, Prop. III.10, Prop. III.13
* CN 6

## Demonstration

By Post. VII.2 the only sanctioned path out of a system of record is a contract, and the asynchronous contract is the integration event (Prop. III.3), so a read model can be fed by nothing else. Because the network is unreliable (Post. I.4), a read model will at some point be corrupted or fall behind; a store that can only be repaired by the owning team violates Post. I.3. The archive (Prop. III.13) holds every published event, and a projection (Def. VII.4) is deterministic, so replay from the archive reconstructs the read model. The payload rules of Prop. III.10 bound what a read model may contain. By CN 6 the projection ignores event types and fields it does not understand. ∎ Q.E.D.

## Corollaries

* **Cor. VII.3.1** - A read model contains no field that is absent from the events that feed it; enrichment from a synchronous call to the system of record is a design smell and requires a recorded reason.
* **Cor. VII.3.2** - A read model may be dropped and recreated at any time; nothing may depend on its identity, only on its contents.

## Construction

* SQS consumer queue per read model subscribed to the EventBridge bus (Prop. III.9).
* Projection as a .NET worker (`Microsoft.Extensions.Hosting` `BackgroundService`) or Lambda function using `Amazon.Lambda.SQSEvents`, idempotent per Prop. III.6.
* Read model store: DynamoDB table or a PostgreSQL schema private to the consuming service (Prop. VII.1).
* Rebuild path: EventBridge Archive replay (Prop. III.13) into a dedicated rebuild queue; a `rebuild` CLI verb in the service using `Amazon.EventBridge` `StartReplay`.
* Projection checkpoint (last event id, ULID) stored alongside the read model.

## Conformance

Architecture test in the service repository (`NetArchTest.Rules`) asserting that the projection assembly references no HTTP client to the source service; a scheduled pipeline job `readmodel-rebuild-drill` in the `test` environment that replays the archive and compares row counts against the live read model.

## Scholium

"Rebuildable" is the property that makes a read model safe to own. A store that cannot be rebuilt is a second system of record wearing a disguise.
