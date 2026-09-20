# Prop. III.11 - Commands travel point-to-point over SQS

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

A command (Def. I.10) MUST be sent to a command queue (Def. III.22) owned by the executing service, never to the bus. It MUST use the CloudEvents envelope (Prop. III.1) with an imperative `type` (Prop. III.2) and a registered schema (Prop. III.4). Senders MUST be granted `SendMessage` explicitly. The outcome of a command MUST be published as an event (accepted/rejected/completed), never returned to the sender through a reply queue. Commands MUST carry an `Idempotency-Key`-equivalent: their `id`, deduplicated by the executor exactly as events are (Prop. III.6).

## Given

Def. I.9, Def. I.10, Def. III.22, Post. III.6, CN 3, CN 8, Prop. III.1, III.2, III.4, III.6.

## Demonstration

An event is a fact and cannot be rejected (Post. III.6); a command is a request and can be. Publishing a request on the bus, where any number of subscribers may react, makes it ambiguous who is responsible for executing it and whether it was executed at all, which CN 8 identifies as an undocumented architecture. Point-to-point delivery to a queue owned by exactly one executor makes responsibility explicit (CN 3). The outcome is a fact about the executor's aggregate and so is an event like any other, available to the sender and to anyone else. Reusing envelope, naming, schema and inbox rules follows from CN 5 and keeps commands from being a second messaging system. ∎ Q.E.D.

## Corollaries

* **Cor. III.11.1** - A command queue is the executor's public interface as much as its API; its command types are listed in the executor's contract set (Prop. IX.4).
* **Cor. III.11.2** - Prefer an API call (Book II) to a command when the sender needs the answer synchronously; prefer a command when the sender must not wait or the work is long (Prop. II.15).
* **Cor. III.11.3** - Scheduled work (EventBridge Scheduler) is delivered as a command to the service's own command queue, so that schedules, retries and dead-lettering are uniform.

## Construction

Terraform `company/sqs-command-queue` (queue + DLQ + sender allow-list). Sender code uses `ICommandSender.SendAsync(cloudEvent)` from `Company.Platform.Messaging`; the executor's consumer pipeline is identical to the event pipeline.

## Conformance

AWS Config: no EventBridge rule matches a `detail-type` whose last verb segment is imperative (checked against the catalogue's command list). Catalogue CI: command schemas carry `x-kind: command` and an `x-executor`.
