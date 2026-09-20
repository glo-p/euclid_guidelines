# Prop. V.6 - PII minimisation in events

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

An event payload MUST carry only the identifiers ([Def. I.18](../../01-foundations/definitions.md#Def.%20I.18%20-%20Identifier)) and the minimum fields a consumer needs to react; personal data classified `Confidential` or `Restricted` ([Def. V.9](../definitions.md#Def.%20V.9%20-%20Data%20Classification)) MUST NOT be placed in an event where the consumer can obtain it by calling the producer's API with the identifier. A `Restricted` field MUST NOT appear in any event payload.

## Given

[Def. V.9](../definitions.md#Def.%20V.9%20-%20Data%20Classification), [Def. I.9](../../01-foundations/definitions.md#Def.%20I.9%20-%20Event), [Def. I.18](../../01-foundations/definitions.md#Def.%20I.18%20-%20Identifier), [Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution), [Prop. III.10](../../03-events/propositions/10-payload-design.md), [Prop. III.13](../../03-events/propositions/13-archive-and-replay.md), [Prop. V.5](05-data-classification-in-contracts.md).

## Demonstration

An event is immutable ([Def. I.9](../../01-foundations/definitions.md#Def.%20I.9%20-%20Event)), is retained and replayable ([Prop. III.13](../../03-events/propositions/13-archive-and-replay.md)), and by [Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution) reaches consumers whose number and identity the producer does not control. Personal data in an event is therefore copied to an unbounded set of stores with no means of later correction or erasure. An identifier carries no meaning ([Def. I.18](../../01-foundations/definitions.md#Def.%20I.18%20-%20Identifier)), so publishing it discloses nothing, and the consumer that needs more can ask the producer, which can then apply authorisation ([Prop. V.2](02-coarse-scopes-fine-authorisation.md)). [Prop. III.10](../../03-events/propositions/10-payload-design.md) already states the payload design; this proposition fixes the classification threshold using the tags of [Prop. V.5](05-data-classification-in-contracts.md). ∎ Q.E.D.

## Corollaries

* **Cor. V.6.1** - The tenant identifier in the envelope ([Prop. III.1](../../03-events/propositions/01-envelope.md)) is `Internal`, not personal data, and is always present.
* **Cor. V.6.2** - Where a consumer genuinely cannot call back (an outbound integration with no network path), the event is a recorded exception naming the fields carried and their retention.

## Construction

* Schema registry admission check ([Prop. III.4](../../03-events/propositions/04-schemas-and-registry.md)) rejecting `x-data-classification: Restricted` in event schemas and warning on `Confidential`.
* EventBridge archive retention set per bus with classification recorded in the archive's tags.
* `Company.Contracts.Shared` analyser flagging `[DataClassification(Restricted)]` members in types used as event payloads.

## Conformance

Schema registry admission check on every event schema version; Roslyn analyser `COMPANY-SEC-006` in the events package.

## Scholium

"Thin events" cost a call-back; "fat events" cost an erasure request the producer cannot fulfil. The second cost is unbounded, so the default is thin.
