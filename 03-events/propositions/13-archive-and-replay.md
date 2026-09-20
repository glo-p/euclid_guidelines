# Prop. III.13 - The bus is archived and replayable

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Every bus MUST have an EventBridge archive of all events with retention of at least 90 days in non-production and 400 days in production, plus a long-term copy to S3 (via a Firehose target) retained per data classification ([Prop. VII.5](../../07-data-ownership/propositions/05-retention-and-deletion.md)). Replay ([Def. III.21](../definitions.md#Def.%20III.21%20-%20Replay)) MUST be initiated only by the subscriber that owns the target rule, MUST be scoped by event type and time window, and MUST deliver to the subscriber's queue through its existing rule. Consumers MUST tolerate replay by virtue of [Prop. III.6](06-idempotent-consumers.md) and [III.7](07-ordering.md) and MUST NOT rely on `time` being recent.

## Given

[Def. III.21](../definitions.md#Def.%20III.21%20-%20Replay), [Post. III.6](../postulates.md#Post.%20III.6%20-%20Facts%20Are%20Permanent), [CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed), [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated), [Prop. III.6](06-idempotent-consumers.md), [III.7](07-ordering.md), [III.9](09-topology.md), [VII.3](../../07-data-ownership/propositions/03-read-models-from-events.md), [VII.5](../../07-data-ownership/propositions/05-retention-and-deletion.md).

## Demonstration

Facts are permanent ([Post. III.6](../postulates.md#Post.%20III.6%20-%20Facts%20Are%20Permanent)) and owed ([CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed)); a consumer that was down, buggy or not yet written when a fact was published still needs it, and read models are rebuildable only if the history exists ([Prop. VII.3](../../07-data-ownership/propositions/03-read-models-from-events.md)). The archive is that history; by [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated) an unarchived event is one the platform cannot show ever happened. Replay through the subscriber's own rule keeps the topology honest ([Prop. III.9](09-topology.md)) and keeps other subscribers from receiving unwanted duplicates. The consumer's idempotence and order tolerance are what make replay safe rather than dangerous. ∎ Q.E.D.

## Corollaries

* **Cor. III.13.1** - Replayed events carry the original `id`, `time` and `sequence`; EventBridge marks them with `replay-name`, which the platform pipeline surfaces as a boolean `isReplay` to handlers.
* **Cor. III.13.2** - A consumer that must not re-execute side effects on replay checks `isReplay` (for example, do not resend emails) and documents this in its repository.
* **Cor. III.13.3** - The S3 copy is the input to the analytics platform ([Prop. VII.7](../../07-data-ownership/propositions/07-analytics-via-data-platform.md)); analytics never reads queues.

## Construction

`company/eventbridge-bus` module creates the archive and a Firehose delivery stream to `s3://company-events-<env>/<context>/<type>/yyyy/mm/dd/` in JSON Lines, partitioned by `detail-type`. Replay runbook: `aws events start-replay --event-source-arn <bus> --destination Arn=<rule> --event-start-time … --event-end-time …`, permitted by IAM only to the rule owner's deployment role.

## Conformance

AWS Config: every custom bus has an archive with the minimum retention and a Firehose rule. Consumer contract test ([Prop. III.12](12-consumer-contracts-and-testing.md)) includes the `isReplay` case.
