# Book III - Events and Asynchronous Communication

Book III governs everything a service says to other services without waiting for an answer: what an event is, what it looks like on the wire, how it is named, published, delivered, consumed, retried, archived and replayed, and how all of that is built on Amazon EventBridge and SQS from .NET. It is one of the three *detailed* books.

## Contents

| File | Contents |
|---|---|
| [`definitions.md`](definitions.md) | [`Def. III.1`](definitions.md#Def.%20III.1%20-%20Domain%20Event) - [`Def. III.22`](definitions.md#Def.%20III.22%20-%20Command%20Queue): integration event, domain event, envelope, bus, rule, queue, DLQ, outbox, inbox, at-least-once, ordering key, … |
| [`postulates.md`](postulates.md) | [`Post. III.1`](postulates.md#Post.%20III.1%20-%20The%20Bus) - [`Post. III.7`](postulates.md#Post.%20III.7%20-%20Schema%20Validation%20Is%20the%20Check): EventBridge is the bus, SQS the buffer, delivery is at-least-once, order is not guaranteed, payloads are bounded, … |

### Propositions

| # | Title | Level | File |
|---|---|---|---|
| III.1 | Every message is a CloudEvent | MUST | [`01-envelope.md`](propositions/01-envelope.md) |
| III.2 | Event types are named `<context>.<aggregate>.<fact>.v<major>` | MUST | [`02-naming.md`](propositions/02-naming.md) |
| III.3 | Only integration events cross a service boundary | MUST | [`03-integration-vs-domain-events.md`](propositions/03-integration-vs-domain-events.md) |
| III.4 | Every event type has a registered schema | MUST | [`04-schemas-and-registry.md`](propositions/04-schemas-and-registry.md) |
| III.5 | Events are published through a transactional outbox | MUST | [`05-transactional-outbox.md`](propositions/05-transactional-outbox.md) |
| III.6 | Consumers are idempotent by event id | MUST | [`06-idempotent-consumers.md`](propositions/06-idempotent-consumers.md) |
| III.7 | Order is carried by the event, not by the transport | MUST | [`07-ordering.md`](propositions/07-ordering.md) |
| III.8 | Failures retry with backoff and land in a dead-letter queue | MUST | [`08-retries-and-dead-letters.md`](propositions/08-retries-and-dead-letters.md) |
| III.9 | One bus per environment; consumers own their queues | MUST | [`09-topology.md`](propositions/09-topology.md) |
| III.10 | Payloads carry enough state and no more | SHOULD | [`10-payload-design.md`](propositions/10-payload-design.md) |
| III.11 | Commands travel point-to-point over SQS | MUST | [`11-commands.md`](propositions/11-commands.md) |
| III.12 | Producers and consumers are held to the schema by tests | MUST | [`12-consumer-contracts-and-testing.md`](propositions/12-consumer-contracts-and-testing.md) |
| III.13 | The bus is archived and replayable | MUST | [`13-archive-and-replay.md`](propositions/13-archive-and-replay.md) |
| III.14 | The .NET construction of a conforming producer and consumer | Q.E.F. | [`14-dotnet-construction.md`](propositions/14-dotnet-construction.md) |

## Reading order

III.1 to III.4 define what an event *is* for us. III.5 to III.8 are the four rules that make asynchronous communication correct under [Post. I.4](../01-foundations/postulates.md#Post.%20I.4%20-%20Unreliable%20Network) (unreliable network); a team that skips any one of them will lose or duplicate business facts. III.9 is the wiring. III.10 to III.13 are design and operations. III.14 is the template.

## Open questions

* Whether to run a second, FIFO-capable path (SNS FIFO + SQS FIFO) for the rare aggregate that needs strict per-key ordering, or to insist on III.7 everywhere. Current text: III.7 everywhere, FIFO by recorded exception.
* Whether the EventBridge Schema Registry is the source of truth for schemas, or a mirror of the catalogue repository. Current text: the repository is the source and CI pushes to the registry (III.4).
* Whether MassTransit is adopted as the .NET messaging abstraction or the platform package wraps the AWS SDK directly. Current text: platform package wraps the SDK; MassTransit is a MAY (III.14).
