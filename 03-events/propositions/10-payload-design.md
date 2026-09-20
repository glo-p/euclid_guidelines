# Prop. III.10 - Payloads carry enough state and no more

| | |
|---|---|
| **Level** | SHOULD |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

An event payload SHOULD be a state-transfer event ([Def. III.19](../definitions.md#Def.%20III.19%20-%20State-Transfer%20Event)) containing the fields a typical consumer needs to act without calling the publisher, expressed with shared schemas. It SHOULD NOT contain the whole aggregate by reflex, and MUST NOT contain fields classified Restricted ([Prop. V.5](../../05-security/propositions/05-data-classification-in-contracts.md)) or PII beyond what the fact itself is about ([Prop. V.6](../../05-security/propositions/06-pii-minimisation-in-events.md)). Payloads over 200 KB MUST use a claim check ([Def. III.20](../definitions.md#Def.%20III.20%20-%20Claim%20Check)) to an S3 object in the publisher's account with a pre-signed or cross-account-readable URI. Payloads MUST carry the identifier of the aggregate and its `sequence` even when a claim check is used.

## Given

[Def. III.18](../definitions.md#Def.%20III.18%20-%20Notification%20Event) to [III.20](../definitions.md#Def.%20III.20%20-%20Claim%20Check), [Post. I.4](../../01-foundations/postulates.md#Post.%20I.4%20-%20Unreliable%20Network), [Post. III.5](../postulates.md#Post.%20III.5%20-%20Bounded%20Payloads), [Post. III.6](../postulates.md#Post.%20III.6%20-%20Facts%20Are%20Permanent), [CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed), [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape), [Prop. V.5](../../05-security/propositions/05-data-classification-in-contracts.md), [V.6](../../05-security/propositions/06-pii-minimisation-in-events.md).

## Demonstration

A notification event forces every consumer to call back, which multiplies synchronous coupling under [Post. I.4](../../01-foundations/postulates.md#Post.%20I.4%20-%20Unreliable%20Network) and defeats the purpose of publishing asynchronously; a state-transfer event removes the call. But every field in a payload is a contract ([CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit)) and is owed ([CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed)), so a payload that dumps the whole aggregate promises far more than the publisher intends to keep. The middle is: what a consumer needs, no more. Restricted data and incidental PII would be copied into every subscriber's queue, archive and logs, which is why they are excluded rather than merely discouraged. The size limit follows from [Post. III.5](../postulates.md#Post.%20III.5%20-%20Bounded%20Payloads) with headroom for envelope growth. Shared schemas follow from [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape). ∎ Q.E.D.

## Corollaries

* **Cor. III.10.1** - Deciding "what a consumer needs" is done with the first two consumers, not in the abstract; a field no consumer uses is removed before the schema reaches `active` ([Book IX](../../09-versioning-and-deprecation/README.md)).
* **Cor. III.10.2** - Consumers that need more than the payload call the publisher's API with the `subject` and compare `sequence` to `metadata.version` ([Cor. III.7.1](07-ordering.md#Corollaries)).
* **Cor. III.10.3** - Claim-check objects are immutable, keyed by event `id`, and retained for the archive window ([Prop. III.13](13-archive-and-replay.md)).

## Construction

Schema review checklist (in the catalogue PR template): audience, fields per consumer, classification per field (`x-data-classification`), size estimate, PII justification.

## Conformance

Catalogue CI: every property has `x-data-classification` or inherits the schema-level default; `restricted` is rejected in event schemas. Publisher test: serialised envelope ≤ 200 KB with the largest example.
