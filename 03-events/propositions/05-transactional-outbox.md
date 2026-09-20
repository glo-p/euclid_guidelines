# Prop. III.5 - Events are published through a transactional outbox

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

A publisher MUST write each integration event to an outbox table (Def. III.13) in the same database transaction that commits the state change the event describes, and MUST NOT call the bus from inside a request or transaction. A relay MUST read the outbox and publish to the bus, marking rows published only after the bus acknowledges. The event `id` MUST be assigned at outbox-write time. Events MUST NOT be published for state that was rolled back.

## Given

Def. I.9, Def. III.13, Post. I.4, Post. III.3, Post. III.6, CN 2, CN 7, Prop. III.1, Prop. III.6.

## Demonstration

An event is a record of a committed fact (Def. I.9, Post. III.6). If the bus is called before commit, the network (Post. I.4) may deliver an event for a fact that is then rolled back: a lie. If the bus is called after commit but outside the transaction, the process may die between commit and publish: a fact that is never announced, which by CN 7 is operationally a lost fact, and by CN 2 a broken promise to subscribers. The only construction that survives both failures is to make the announcement part of the same atomic write as the fact, and to publish from durable storage afterwards. Publishing from durable storage may repeat (Post. III.3), which is why the `id` is fixed at write time so that repeats are deduplicable (Prop. III.6). ∎ Q.E.D.

## Corollaries

* **Cor. III.5.1** - The outbox relay is part of the publishing service (same deployable, same team), not a shared platform component reading everyone's database (Post. I.3, Prop. VII.1).
* **Cor. III.5.2** - Outbox rows are retained for 7 days after publication for diagnosis, then purged.
* **Cor. III.5.3** - Publishing latency is a metric (Prop. VI.3): age of the oldest unpublished row. It is the readiness of the relay.
* **Cor. III.5.4** - Sequence (Def. III.16) is assigned at write time from the aggregate's version, so ordering survives relay retries.

## Construction

```sql
CREATE TABLE outbox (
  id            CHAR(26) PRIMARY KEY,         -- ULID, becomes CloudEvent id
  occurred_at   TIMESTAMPTZ NOT NULL,
  event_type    TEXT NOT NULL,
  subject       TEXT NOT NULL,
  sequence      BIGINT NOT NULL,
  envelope      JSONB NOT NULL,               -- the full CloudEvent
  published_at  TIMESTAMPTZ NULL,
  attempts      INT NOT NULL DEFAULT 0
);
CREATE INDEX outbox_unpublished ON outbox (occurred_at) WHERE published_at IS NULL;
```

.NET (EF Core): `IOutbox.Add(CloudEvent)` inside the `DbContext` transaction; a `BackgroundService` (or, for Lambda-hosted services, an EventBridge Scheduler-triggered function) polls with `FOR UPDATE SKIP LOCKED`, calls `PutEvents` in batches of ≤ 10, sets `published_at` per successful entry, increments `attempts` on failure with exponential backoff. Alternatively, for RDS PostgreSQL, a logical replication slot read by the same service (Cor. III.3.1) replaces polling. Provided by `Company.Platform.Messaging.Outbox`.

## Conformance

Architecture test: no reference to `AmazonEventBridgeClient` outside the `Outbox` relay namespace. Integration test in template: a transaction that throws after `IOutbox.Add` leaves no outbox row.
