# Book III - Definitions

These refine the Book I definitions for asynchronous communication.

---

**Def. III.1 - Domain Event.** A *domain event* is an event (Def. I.9) raised inside an aggregate (Def. I.6) and handled within the same service and, usually, the same transaction. It is an implementation detail of the service.

**Def. III.2 - Integration Event.** An *integration event* is an event published across a service boundary for consumption by other services. It is a contract (Def. I.2). Only integration events are governed by this Book; where this Book says "event" it means integration event.

**Def. III.3 - Envelope.** The *envelope* is the set of context attributes that accompany every event independently of its payload: identity, source, type, time, subject, correlation and schema reference. Our envelope is CloudEvents 1.0 (Prop. III.1) and its shared schema is `EventEnvelope` (Prop. IV.8).

**Def. III.4 - Payload.** The *payload* (`data`) is the event-type-specific body, conforming to that event type's registered schema (Prop. III.4).

**Def. III.5 - Event Type.** The *event type* is the string in the envelope's `type` attribute. It names a kind of fact and its major version (Prop. III.2). An event type is the unit of contract in this Book, as an operation (Def. II.2) is in Book II.

**Def. III.6 - Publisher.** The *publisher* is the service (Def. I.3) that owns the aggregate the event is about and writes it to the bus. An event type has exactly one publisher.

**Def. III.7 - Subscriber.** A *subscriber* (or *consumer*) is a service that has a rule (Def. III.9) delivering events of one or more types to a queue it owns.

**Def. III.8 - Bus.** The *bus* is the Amazon EventBridge custom event bus, one per environment (Def. I.21), to which every publisher writes and from which every rule reads.

**Def. III.9 - Rule.** A *rule* is an EventBridge rule owned by a subscriber that selects events by pattern (at minimum `detail-type`) and targets the subscriber's queue.

**Def. III.10 - Queue.** A *queue* is an Amazon SQS standard queue owned by exactly one subscriber, holding events until the subscriber processes them. A queue has exactly one consuming application.

**Def. III.11 - Dead-Letter Queue.** A *dead-letter queue* (DLQ) is the SQS queue to which a message is moved after the maximum number of failed receives on its source queue. Every queue has one.

**Def. III.12 - At-Least-Once.** *At-least-once delivery* means every published event is delivered to every matching rule's queue one or more times. It is the delivery guarantee of the bus and the queue.

**Def. III.13 - Outbox.** The *outbox* is a table in the publisher's database into which events are written in the same transaction as the state change they describe, and from which a relay publishes them to the bus.

**Def. III.14 - Inbox.** The *inbox* is a table (or key set) in the consumer's store recording the ids of events already processed, consulted before processing to achieve idempotency (Def. I.16).

**Def. III.15 - Ordering Key.** The *ordering key* is the identifier of the aggregate an event is about (`subject` in the envelope). Events sharing an ordering key have a meaningful order; events with different keys do not.

**Def. III.16 - Sequence.** The *sequence* is a monotonically increasing integer carried in the envelope (`sequence` extension) that orders events with the same ordering key. It is typically the aggregate's version after the change.

**Def. III.17 - Correlation and Causation.** The *correlation id* (`traceparent` trace-id, Def. I.22) names the business activity an event belongs to; the *causation id* (`causationid` extension) is the `id` of the message that directly caused this one.

**Def. III.18 - Notification Event.** A *notification event* carries the identity of what changed and little else; consumers fetch state through the publisher's API.

**Def. III.19 - State-Transfer Event.** A *state-transfer event* carries the state a consumer needs so that no call back to the publisher is required (event-carried state transfer).

**Def. III.20 - Claim Check.** A *claim check* is a reference (an S3 object URI) placed in the payload in lieu of content too large for the transport.

**Def. III.21 - Replay.** *Replay* is the re-delivery of archived events to a rule, by the platform, for rebuilding a consumer's state or recovering from a fault.

**Def. III.22 - Command Queue.** A *command queue* is an SQS queue owned by the service that executes the commands, to which authorised senders write commands (Def. I.10) directly, bypassing the bus.
