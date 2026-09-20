# Prop. III.13 - The bus is archived and replayable

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Every bus MUST have an EventBridge archive of all events with retention of at least 90 days in non-production and 400 days in production, plus a long-term copy to S3 (via a Firehose target) retained per data classification (Prop. VII.5). Replay (Def. III.21) MUST be initiated only by the subscriber that owns the target rule, MUST be scoped by event type and time window, and MUST deliver to the subscriber's queue through its existing rule. Consumers MUST tolerate replay by virtue of Prop. III.6 and III.7 and MUST NOT rely on `time` being recent.

## Given

Def. III.21, Post. III.6, CN 2, CN 7, Prop. III.6, III.7, III.9, VII.3, VII.5.

## Demonstration

Facts are permanent (Post. III.6) and owed (CN 2); a consumer that was down, buggy or not yet written when a fact was published still needs it, and read models are rebuildable only if the history exists (Prop. VII.3). The archive is that history; by CN 7 an unarchived event is one the platform cannot show ever happened. Replay through the subscriber's own rule keeps the topology honest (Prop. III.9) and keeps other subscribers from receiving unwanted duplicates. The consumer's idempotence and order tolerance are what make replay safe rather than dangerous. ∎ Q.E.D.

## Corollaries

* **Cor. III.13.1** - Replayed events carry the original `id`, `time` and `sequence`; EventBridge marks them with `replay-name`, which the platform pipeline surfaces as a boolean `isReplay` to handlers.
* **Cor. III.13.2** - A consumer that must not re-execute side effects on replay checks `isReplay` (for example, do not resend emails) and documents this in its repository.
* **Cor. III.13.3** - The S3 copy is the input to the analytics platform (Prop. VII.7); analytics never reads queues.

## Construction

`company/eventbridge-bus` module creates the archive and a Firehose delivery stream to `s3://company-events-<env>/<context>/<type>/yyyy/mm/dd/` in JSON Lines, partitioned by `detail-type`. Replay runbook: `aws events start-replay --event-source-arn <bus> --destination Arn=<rule> --event-start-time … --event-end-time …`, permitted by IAM only to the rule owner's deployment role.

## Conformance

AWS Config: every custom bus has an archive with the minimum retention and a Firehose rule. Consumer contract test (Prop. III.12) includes the `isReplay` case.
