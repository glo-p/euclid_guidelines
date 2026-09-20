# Prop. III.10 - Payloads carry enough state and no more

| | |
|---|---|
| **Level** | SHOULD |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

An event payload SHOULD be a state-transfer event (Def. III.19) containing the fields a typical consumer needs to act without calling the publisher, expressed with shared schemas. It SHOULD NOT contain the whole aggregate by reflex, and MUST NOT contain fields classified Restricted (Prop. V.5) or PII beyond what the fact itself is about (Prop. V.6). Payloads over 200 KB MUST use a claim check (Def. III.20) to an S3 object in the publisher's account with a pre-signed or cross-account-readable URI. Payloads MUST carry the identifier of the aggregate and its `sequence` even when a claim check is used.

## Given

Def. III.18 to III.20, Post. I.4, Post. III.5, Post. III.6, CN 2, CN 3, CN 5, Prop. V.5, V.6.

## Demonstration

A notification event forces every consumer to call back, which multiplies synchronous coupling under Post. I.4 and defeats the purpose of publishing asynchronously; a state-transfer event removes the call. But every field in a payload is a contract (CN 3) and is owed (CN 2), so a payload that dumps the whole aggregate promises far more than the publisher intends to keep. The middle is: what a consumer needs, no more. Restricted data and incidental PII would be copied into every subscriber's queue, archive and logs, which is why they are excluded rather than merely discouraged. The size limit follows from Post. III.5 with headroom for envelope growth. Shared schemas follow from CN 5. ∎ Q.E.D.

## Corollaries

* **Cor. III.10.1** - Deciding "what a consumer needs" is done with the first two consumers, not in the abstract; a field no consumer uses is removed before the schema reaches `active` (Book IX).
* **Cor. III.10.2** - Consumers that need more than the payload call the publisher's API with the `subject` and compare `sequence` to `metadata.version` (Cor. III.7.1).
* **Cor. III.10.3** - Claim-check objects are immutable, keyed by event `id`, and retained for the archive window (Prop. III.13).

## Construction

Schema review checklist (in the catalogue PR template): audience, fields per consumer, classification per field (`x-data-classification`), size estimate, PII justification.

## Conformance

Catalogue CI: every property has `x-data-classification` or inherits the schema-level default; `restricted` is rejected in event schemas. Publisher test: serialised envelope ≤ 200 KB with the largest example.
