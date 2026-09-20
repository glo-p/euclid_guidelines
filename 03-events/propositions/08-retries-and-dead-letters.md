# Prop. III.8 - Failures retry with backoff and land in a dead-letter queue

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Every queue (Def. III.10) MUST have a dead-letter queue (Def. III.11) with `maxReceiveCount` between 3 and 10, a visibility timeout at least six times the handler's p99 duration, and a DLQ retention of 14 days. Every EventBridge rule target MUST have a retry policy and a DLQ for delivery failures. A consumer MUST distinguish *transient* failures (throw, let SQS retry) from *permanent* failures (schema invalid, business rule cannot ever apply) and MUST move permanent failures to the DLQ immediately with a reason. A non-empty DLQ MUST alert the owning team (Prop. VI.6) and MUST have a documented redrive procedure.

## Given

Def. III.10, Def. III.11, Post. I.4, Post. III.3, CN 2, CN 7, Prop. III.6, VI.6.

## Demonstration

Under Post. I.4 a consumer will sometimes fail for reasons that clear on their own; a retry is the correct response and SQS provides it through visibility timeout and receive count. A permanent failure retried forever blocks the queue for every other message and, by CN 7, hides the actual defect; therefore permanent failures leave the queue at once. A message in a DLQ is a published fact that has not been honoured (CN 2): it must be visible (alert) and recoverable (redrive), and retained long enough for a human to act. Retrying after a DLQ is safe only because consumers are idempotent (Prop. III.6). ∎ Q.E.D.

## Corollaries

* **Cor. III.8.1** - A consumer that catches all exceptions and acknowledges anyway is non-conforming: it converts every failure into silent data loss.
* **Cor. III.8.2** - Backoff between retries is exponential with jitter; on SQS this is achieved by changing the message's visibility on failure, not by sleeping in the handler.
* **Cor. III.8.3** - Redrive is from DLQ to the *source* queue (SQS redrive policy), never by re-publishing to the bus, so that other subscribers do not receive duplicates.

## Construction

Terraform module `company/sqs-consumer` (Prop. VIII.9) creates queue + DLQ + alarm
+ redrive allow policy + the EventBridge rule and target with `retry_policy { maximum_retry_attempts = 185, maximum_event_age_in_seconds = 86400 }` and `dead_letter_config`. `Company.Platform.Messaging` maps `PermanentFailureException` to an immediate move to the DLQ (send to DLQ with `reason` attribute, delete from source) and any other exception to a visibility change with backoff.

## Conformance

AWS Config rule `sqs-queue-has-dlq` and a custom rule for EventBridge targets; CloudWatch alarm `ApproximateNumberOfMessagesVisible > 0` on every DLQ, verified by the module's tests.
