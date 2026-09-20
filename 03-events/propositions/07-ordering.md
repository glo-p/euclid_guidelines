# Prop. III.7 - Order is carried by the event, not by the transport

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Consumers MUST NOT assume that events arrive in the order they were published, even for one `subject`. Publishers MUST set `subject` (Def. III.15) and `sequence` (Def. III.16) on every event. A consumer that needs order MUST use `sequence` to detect and handle out-of-order delivery by one of: (a) ignoring an event whose `sequence` is not greater than the last one processed for that `subject` (last-writer-wins on state-transfer events), (b) buffering until the gap closes with a bounded wait, or (c) fetching current state from the publisher's API. SQS FIFO queues MAY be used only under a recorded exception (Def. I.20).

## Given

Def. III.15, Def. III.16, Post. III.4, Post. I.4, CN 3, CN 6, Prop. III.1, III.6.

## Demonstration

The transport does not order (Post. III.4), and even a FIFO transport cannot order across a retry from the DLQ or a replay (Prop. III.8, III.13). Therefore any order the consumer needs must be expressible from the message alone, which is what `subject` and `sequence` provide, and by CN 3 they must be in the contract (Prop. III.1). The consumer pays for tolerance (CN 6) by handling the reorder; the publisher pays for stability by never emitting a `sequence` that goes backwards for a `subject`. The three strategies cover, respectively, state-transfer events, notification events that must be processed in order, and consumers that can tolerate a call back. ∎ Q.E.D.

## Corollaries

* **Cor. III.7.1** - `sequence` is the aggregate's version after the change and is therefore also the value of `ResourceMetadata.version` (Prop. IV.7) for the same aggregate; a consumer can compare an event to an API response.
* **Cor. III.7.2** - Events about different subjects have no defined relative order; a workflow that needs "A then B" across aggregates is a process (saga) that reacts to both and is itself an aggregate with a `subject`.

## Construction

`Company.Platform.Messaging.Ordering` provides `LastSequenceStore` (per consumer, per subject) and a pipeline behaviour implementing strategy (a); strategies (b) and (c) are documented recipes in the template.

## Conformance

Publisher unit test: for a series of changes to one aggregate, emitted `sequence` values are strictly increasing. Consumer contract test (Prop. III.12): delivering events for one subject in reverse order yields the same final state as in order.
