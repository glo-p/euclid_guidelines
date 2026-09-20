# Prop. V.5 - Confidential and Restricted fields are tagged in the contract

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every schema field in an OpenAPI document or JSON Schema whose classification (Def. V.9) is `Confidential` or `Restricted` MUST carry the extension `x-data-classification` with that value. A field without the extension is treated as `Confidential` by every tool and by the retention, logging and export rules that depend on classification.

## Given

Def. V.9, Def. I.2, Def. I.12, CN 3, CN 8, Post. I.6, Prop. II.1, Prop. III.4.

## Demonstration

Classification is a property of data, and the contract (Def. I.2) is the whole of what is known about the data crossing a boundary. By CN 3 a classification not written in the contract does not exist for the consumer, and by CN 8 it is not architecture. The contract is already the single machine-readable artefact (Prop. II.1, Prop. III.4), so the tag belongs there, where linters, generators and log redaction (Prop. VI.7) can read it (Post. I.6). Defaulting the untagged case to `Confidential` makes omission safe rather than permissive. ∎ Q.E.D.

## Corollaries

* **Cor. V.5.1** - A `Restricted` field MUST NOT appear in a query string, a path segment, or an event payload (Prop. V.6).
* **Cor. V.5.2** - Generated C# and TypeScript types carry the classification as an attribute or JSDoc tag so that redaction and serialisation policies can act on it.

## Construction

* OpenAPI 3.1 and JSON Schema 2020-12 extension keyword `x-data-classification` with enum `Public | Internal | Confidential | Restricted`, published in the catalogue (Book IV) as a shared vocabulary.
* Spectral rule set `company-openapi` and `ajv` custom keyword validating the enum.
* Generator hooks (NSwag or Kiota; openapi-typescript) emitting `[DataClassification(Restricted)]` and `@classification` respectively.
* `Company.Contracts.Shared` exposing the `DataClassification` enum and attribute.

## Conformance

Spectral rule: every property whose name matches the PII and credential dictionaries carries the extension; every `x-data-classification` value is in the enum. Schema registry admission check for events.

## Scholium

Tagging `Public` and `Internal` is permitted and encouraged where a field would otherwise match a dictionary heuristic and be flagged. The dictionary is a heuristic; the tag is the truth.
