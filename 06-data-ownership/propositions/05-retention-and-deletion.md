# Prop. VII.5 - Retention and deletion per data classification

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every field carrying a data classification tag (Prop. V.5) MUST have a retention period (Def. VII.9) fixed by the data owner (Def. VII.2) and applied to the system of record, every read model, the reporting store, the event archive and backups. A right-to-erasure request MUST be propagated as an integration event so that every copy is erased or anonymised within the period the policy states.

## Given

* Def. I.9, Def. VII.2, Def. VII.3, Def. VII.9
* Post. I.3, Post. I.6
* Prop. III.3, Prop. III.13, Prop. V.5, Prop. V.6
* CN 7

## Demonstration

Classification (Prop. V.5) tells us what a field is; retention (Def. VII.9) tells us how long it may exist, and only the data owner (Def. VII.2) can decide that. Copies of a concept exist in read models (Def. VII.3), the archive (Prop. III.13) and reporting, all held by other teams who cannot be reached except through contracts (Post. I.3); therefore an erasure obligation must itself travel as an event (Prop. III.3). Because events are immutable (Def. I.9), personal data in the archive must already be minimised (Prop. V.6) or be erasable by key destruction. By Post. I.6 and CN 7 a retention policy that is not executed and observed by a machine does not exist. ∎ Q.E.D.

## Corollaries

* **Cor. VII.5.1** - A read model MUST subscribe to the erasure event of every concept it copies; a projection that ignores it is non-conforming.
* **Cor. VII.5.2** - Personal data in an archived event is stored encrypted under a per-subject key, so that erasure is achieved by deleting the key (crypto-shredding).

## Construction

* Retention declared per field in the shared schema via a `x-retention` extension next to the `x-classification` tag of Prop. V.5.
* Erasure integration event `Subject.ErasureRequested` (naming per Prop. III.2) in the catalogue (Prop. IV.8 envelope).
* S3 lifecycle rules and Object Lock on the reporting store and archive buckets; RDS and Aurora automated backup retention set from policy; DynamoDB TTL attribute for time-bounded records.
* Per-subject data keys in AWS KMS with envelope encryption; key deletion scheduled via `ScheduleKeyDeletion` on erasure.
* Retention jobs as EventBridge Scheduler rules invoking a Lambda in each service.

## Conformance

Catalogue CI job `retention-check` fails when a classified field has no retention period; AWS Config custom rule verifying lifecycle rules on tagged buckets; consumer contract test (Prop. III.12) that each read model service subscribes to the erasure event for each concept it declares under `x-read-model-of`.

## Scholium

The hard part is not deleting from the primary; it is knowing where the copies are. Prop. VII.2 makes the copies enumerable, which is why this proposition can exist.
