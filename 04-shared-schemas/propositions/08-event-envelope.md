# Prop. IV.8 - Messages are an `EventEnvelope`

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Every event and command on the bus or a queue MUST validate against [`event-envelope.json`](../schemas/shared/v1/event-envelope.json), which is the shared schema form of Prop. III.1: CloudEvents 1.0 core attributes with `subject`, `dataschema` and `time` made required, plus the extensions `traceparent` (required), `tracestate`, `causationid`, `sequence` (required), `tenantid` and `dataclassification`. `data` MUST additionally validate against the schema at `dataschema`. The envelope schema MUST NOT gain required attributes within v1; a new required attribute is a new major of the envelope and therefore of every event type.

## Given

Def. I.11, Def. III.3, CN 4, CN 5, CN 6, Prop. III.1, III.4.

## Demonstration

Prop. III.1 fixes the envelope; expressing it as a catalogue schema is what makes it checkable at both ends (Post. III.7) and generated into `CloudEvent<T>` for every service (CN 5). Because every event type embeds the envelope, a breaking change to the envelope is, by CN 4, a breaking change to every contract on the bus; the cost is company-wide and so the constraint is stated explicitly. Optional extensions can be added freely because consumers ignore what they do not know (CN 6). ∎ Q.E.D.

## Corollaries

* **Cor. IV.8.1** - Validators MUST allow additional properties on the envelope (CloudEvents extensions may appear from tooling, e.g. `replay-name`).
* **Cor. IV.8.2** - On EventBridge, `detail` is the whole envelope; `detail-type` and `source` duplicate `type` and `source` (Prop. III.1).

## Construction

C#: `CloudEvent<T>` record in `Company.Contracts.Shared` (thin wrapper over `CloudNative.CloudEvents.CloudEvent` with typed `Data` and strongly typed accessors for the company extensions). TypeScript: `interface CloudEvent<T>`.

## Conformance

Catalogue CI validates examples; every consumer pipeline validates on receive (Prop. III.1 conformance).
