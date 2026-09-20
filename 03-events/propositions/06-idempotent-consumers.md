# Prop. III.6 - Consumers are idempotent by event id

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

A consumer MUST treat the CloudEvent `id` as the idempotency key of the message and MUST guarantee that processing the same `id` twice has the effect of processing it once (Def. I.16). It MUST record processed ids in an inbox (Def. III.14) that is written in the same transaction as the consumer's own state change, and MUST check the inbox before processing. Where the consumer's effect is naturally idempotent (an upsert keyed by `subject`), the inbox MAY be replaced by a documented argument in the consumer's repository.

## Given

Def. I.16, Def. III.14, Post. I.4, Post. III.3, CN 6, Prop. III.1, Prop. III.5.

## Demonstration

Delivery is at least once (Post. III.3) and the publisher's relay itself repeats (Prop. III.5); therefore duplicates are certain, not possible. The consumer is the only party that can know whether it has already acted (the publisher and the bus cannot see its state), so deduplication belongs to the consumer: this is the consumer's half of CN 6. The `id` is stable across repeats by construction (Cor. III.1.2), so it is the correct key. Recording the id outside the transaction reintroduces Post. I.4 between "acted" and "remembered acting", so the inbox write must be atomic with the effect. ∎ Q.E.D.

## Corollaries

* **Cor. III.6.1** - Inbox rows are retained for at least 7 days, longer than any redrive or replay window (Prop. III.8, III.13).
* **Cor. III.6.2** - A consumer that performs a non-transactional side effect (sends an email, calls a third party) records the intent in its own database first and executes from there, so that the side effect is itself covered by an idempotency key (Prop. II.8 applied outward).
* **Cor. III.6.3** - Deduplication by SQS FIFO `MessageDeduplicationId` is not a substitute: its window is 5 minutes and the bus is not FIFO.

## Construction

```sql
CREATE TABLE inbox (
  event_id     CHAR(26) PRIMARY KEY,
  consumer     TEXT NOT NULL,
  processed_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

```csharp
await using var tx = await db.Database.BeginTransactionAsync(ct);
if (!await inbox.TryClaimAsync(evt.Id, consumerName, ct)) return;   // INSERT ... ON CONFLICT DO NOTHING
await handler.HandleAsync(evt, ct);
await tx.CommitAsync(ct);
```

`Company.Platform.Messaging.Inbox` supplies the table, the EF Core convention and a pipeline behaviour that wraps every handler.

## Conformance

Architecture test: every `IEventHandler<T>` is invoked only through the inbox behaviour. Template integration test: same message delivered twice produces one effect.
