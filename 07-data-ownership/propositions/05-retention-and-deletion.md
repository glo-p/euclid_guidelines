# Prop. VII.5 - Retention and deletion per data classification

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every field carrying a data classification tag ([Prop. V.5](../../05-security/propositions/05-data-classification-in-contracts.md)) MUST have a retention period ([Def. VII.9](../definitions.md#Def.%20VII.9%20-%20Retention%20Period)) fixed by the data owner ([Def. VII.2](../definitions.md#Def.%20VII.2%20-%20Data%20Owner)) and applied to the system of record, every read model, the reporting store, the event archive and backups. A right-to-erasure request MUST be propagated as an integration event so that every copy is erased or anonymised within the period the policy states.

## Given

* [Def. I.9](../../01-foundations/definitions.md#Def.%20I.9%20-%20Event), [Def. VII.2](../definitions.md#Def.%20VII.2%20-%20Data%20Owner), [Def. VII.3](../definitions.md#Def.%20VII.3%20-%20Read%20Model), [Def. VII.9](../definitions.md#Def.%20VII.9%20-%20Retention%20Period)
* [Post. I.3](../../01-foundations/postulates.md#Post.%20I.3%20-%20Autonomy), [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification)
* [Prop. III.3](../../03-events/propositions/03-integration-vs-domain-events.md), [Prop. III.13](../../03-events/propositions/13-archive-and-replay.md), [Prop. V.5](../../05-security/propositions/05-data-classification-in-contracts.md), [Prop. V.6](../../05-security/propositions/06-pii-minimisation-in-events.md)
* [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated)

## Demonstration

Classification ([Prop. V.5](../../05-security/propositions/05-data-classification-in-contracts.md)) tells us what a field is; retention ([Def. VII.9](../definitions.md#Def.%20VII.9%20-%20Retention%20Period)) tells us how long it may exist, and only the data owner ([Def. VII.2](../definitions.md#Def.%20VII.2%20-%20Data%20Owner)) can decide that. Copies of a concept exist in read models ([Def. VII.3](../definitions.md#Def.%20VII.3%20-%20Read%20Model)), the archive ([Prop. III.13](../../03-events/propositions/13-archive-and-replay.md)) and reporting, all held by other teams who cannot be reached except through contracts ([Post. I.3](../../01-foundations/postulates.md#Post.%20I.3%20-%20Autonomy)); therefore an erasure obligation must itself travel as an event ([Prop. III.3](../../03-events/propositions/03-integration-vs-domain-events.md)). Because events are immutable ([Def. I.9](../../01-foundations/definitions.md#Def.%20I.9%20-%20Event)), personal data in the archive must already be minimised ([Prop. V.6](../../05-security/propositions/06-pii-minimisation-in-events.md)) or be erasable by key destruction. By [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification) and [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated) a retention policy that is not executed and observed by a machine does not exist. ∎ Q.E.D.

## Corollaries

* **Cor. VII.5.1** - A read model MUST subscribe to the erasure event of every concept it copies; a projection that ignores it is non-conforming.
* **Cor. VII.5.2** - Personal data in an archived event is stored encrypted under a per-subject key, so that erasure is achieved by deleting the key (crypto-shredding).

## Construction

* Retention declared per field in the shared schema via a `x-retention` extension next to the `x-classification` tag of [Prop. V.5](../../05-security/propositions/05-data-classification-in-contracts.md).
* Erasure integration event `Subject.ErasureRequested` (naming per [Prop. III.2](../../03-events/propositions/02-naming.md)) in the catalogue ([Prop. IV.8](../../04-shared-schemas/propositions/08-event-envelope.md) envelope).
* S3 lifecycle rules and Object Lock on the reporting store and archive buckets; RDS and Aurora automated backup retention set from policy; DynamoDB TTL attribute for time-bounded records.
* Per-subject data keys in AWS KMS with envelope encryption; key deletion scheduled via `ScheduleKeyDeletion` on erasure.
* Retention jobs as EventBridge Scheduler rules invoking a Lambda in each service.

## Conformance

Catalogue CI job `retention-check` fails when a classified field has no retention period; AWS Config custom rule verifying lifecycle rules on tagged buckets; consumer contract test ([Prop. III.12](../../03-events/propositions/12-consumer-contracts-and-testing.md)) that each read model service subscribes to the erasure event for each concept it declares under `x-read-model-of`.

## Scholium

The hard part is not deleting from the primary; it is knowing where the copies are. [Prop. VII.2](02-system-of-record-in-catalogue.md) makes the copies enumerable, which is why this proposition can exist.
