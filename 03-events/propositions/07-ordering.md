# Prop. III.7 - Order is carried by the event, not by the transport

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Consumers MUST NOT assume that events arrive in the order they were published, even for one `subject`. Publishers MUST set `subject` ([Def. III.15](../definitions.md#Def.%20III.15%20-%20Ordering%20Key)) and `sequence` ([Def. III.16](../definitions.md#Def.%20III.16%20-%20Sequence)) on every event. A consumer that needs order MUST use `sequence` to detect and handle out-of-order delivery by one of: (a) ignoring an event whose `sequence` is not greater than the last one processed for that `subject` (last-writer-wins on state-transfer events), (b) buffering until the gap closes with a bounded wait, or (c) fetching current state from the publisher's API. SQS FIFO queues MAY be used only under a recorded exception ([Def. I.20](../../01-foundations/definitions.md#Def.%20I.20%20-%20Exception)).

## Given

[Def. III.15](../definitions.md#Def.%20III.15%20-%20Ordering%20Key), [Def. III.16](../definitions.md#Def.%20III.16%20-%20Sequence), [Post. III.4](../postulates.md#Post.%20III.4%20-%20No%20Transport%20Order), [Post. I.4](../../01-foundations/postulates.md#Post.%20I.4%20-%20Unreliable%20Network), [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 6](../../01-foundations/common-notions.md#CN%206%20-%20The%20Producer%20Pays%20for%20Stability%3B%20the%20Consumer%20Pays%20for%20Tolerance), [Prop. III.1](01-envelope.md), [III.6](06-idempotent-consumers.md).

## Demonstration

The transport does not order ([Post. III.4](../postulates.md#Post.%20III.4%20-%20No%20Transport%20Order)), and even a FIFO transport cannot order across a retry from the DLQ or a replay ([Prop. III.8](08-retries-and-dead-letters.md), [III.13](13-archive-and-replay.md)). Therefore any order the consumer needs must be expressible from the message alone, which is what `subject` and `sequence` provide, and by [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit) they must be in the contract ([Prop. III.1](01-envelope.md)). The consumer pays for tolerance ([CN 6](../../01-foundations/common-notions.md#CN%206%20-%20The%20Producer%20Pays%20for%20Stability%3B%20the%20Consumer%20Pays%20for%20Tolerance)) by handling the reorder; the publisher pays for stability by never emitting a `sequence` that goes backwards for a `subject`. The three strategies cover, respectively, state-transfer events, notification events that must be processed in order, and consumers that can tolerate a call back. ∎ Q.E.D.

## Corollaries

* **Cor. III.7.1** - `sequence` is the aggregate's version after the change and is therefore also the value of `ResourceMetadata.version` ([Prop. IV.7](../../04-shared-schemas/propositions/07-resource-metadata-and-actor.md)) for the same aggregate; a consumer can compare an event to an API response.
* **Cor. III.7.2** - Events about different subjects have no defined relative order; a workflow that needs "A then B" across aggregates is a process (saga) that reacts to both and is itself an aggregate with a `subject`.

## Construction

`Company.Platform.Messaging.Ordering` provides `LastSequenceStore` (per consumer, per subject) and a pipeline behaviour implementing strategy (a); strategies (b) and (c) are documented recipes in the template.

## Conformance

Publisher unit test: for a series of changes to one aggregate, emitted `sequence` values are strictly increasing. Consumer contract test ([Prop. III.12](12-consumer-contracts-and-testing.md)): delivering events for one subject in reverse order yields the same final state as in order.
