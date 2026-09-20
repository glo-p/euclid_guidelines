# Prop. III.11 - Commands travel point-to-point over SQS

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

A command ([Def. I.10](../../01-foundations/definitions.md#Def.%20I.10%20-%20Command)) MUST be sent to a command queue ([Def. III.22](../definitions.md#Def.%20III.22%20-%20Command%20Queue)) owned by the executing service, never to the bus. It MUST use the CloudEvents envelope ([Prop. III.1](01-envelope.md)) with an imperative `type` ([Prop. III.2](02-naming.md)) and a registered schema ([Prop. III.4](04-schemas-and-registry.md)). Senders MUST be granted `SendMessage` explicitly. The outcome of a command MUST be published as an event (accepted/rejected/completed), never returned to the sender through a reply queue. Commands MUST carry an `Idempotency-Key`-equivalent: their `id`, deduplicated by the executor exactly as events are ([Prop. III.6](06-idempotent-consumers.md)).

## Given

[Def. I.9](../../01-foundations/definitions.md#Def.%20I.9%20-%20Event), [Def. I.10](../../01-foundations/definitions.md#Def.%20I.10%20-%20Command), [Def. III.22](../definitions.md#Def.%20III.22%20-%20Command%20Queue), [Post. III.6](../postulates.md#Post.%20III.6%20-%20Facts%20Are%20Permanent), [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts), [Prop. III.1](01-envelope.md), [III.2](02-naming.md), [III.4](04-schemas-and-registry.md), [III.6](06-idempotent-consumers.md).

## Demonstration

An event is a fact and cannot be rejected ([Post. III.6](../postulates.md#Post.%20III.6%20-%20Facts%20Are%20Permanent)); a command is a request and can be. Publishing a request on the bus, where any number of subscribers may react, makes it ambiguous who is responsible for executing it and whether it was executed at all, which [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts) identifies as an undocumented architecture. Point-to-point delivery to a queue owned by exactly one executor makes responsibility explicit ([CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit)). The outcome is a fact about the executor's aggregate and so is an event like any other, available to the sender and to anyone else. Reusing envelope, naming, schema and inbox rules follows from [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape) and keeps commands from being a second messaging system. ∎ Q.E.D.

## Corollaries

* **Cor. III.11.1** - A command queue is the executor's public interface as much as its API; its command types are listed in the executor's contract set ([Prop. IX.4](../../09-versioning-and-deprecation/propositions/04-producers-know-their-consumers.md)).
* **Cor. III.11.2** - Prefer an API call ([Book II](../../02-api-guidelines/README.md)) to a command when the sender needs the answer synchronously; prefer a command when the sender must not wait or the work is long ([Prop. II.15](../../02-api-guidelines/propositions/15-long-running-operations.md)).
* **Cor. III.11.3** - Scheduled work (EventBridge Scheduler) is delivered as a command to the service's own command queue, so that schedules, retries and dead-lettering are uniform.

## Construction

Terraform `company/sqs-command-queue` (queue + DLQ + sender allow-list). Sender code uses `ICommandSender.SendAsync(cloudEvent)` from `Company.Platform.Messaging`; the executor's consumer pipeline is identical to the event pipeline.

## Conformance

AWS Config: no EventBridge rule matches a `detail-type` whose last verb segment is imperative (checked against the catalogue's command list). Catalogue CI: command schemas carry `x-kind: command` and an `x-executor`.
