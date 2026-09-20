# Prop. V.5 - Confidential and Restricted fields are tagged in the contract

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every schema field in an OpenAPI document or JSON Schema whose classification ([Def. V.9](../definitions.md#Def.%20V.9%20-%20Data%20Classification)) is `Confidential` or `Restricted` MUST carry the extension `x-data-classification` with that value. A field without the extension is treated as `Confidential` by every tool and by the retention, logging and export rules that depend on classification.

## Given

[Def. V.9](../definitions.md#Def.%20V.9%20-%20Data%20Classification), [Def. I.2](../../01-foundations/definitions.md#Def.%20I.2%20-%20Contract), [Def. I.12](../../01-foundations/definitions.md#Def.%20I.12%20-%20Schema), [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts), [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification), [Prop. II.1](../../02-api-guidelines/propositions/01-contract-first.md), [Prop. III.4](../../03-events/propositions/04-schemas-and-registry.md).

## Demonstration

Classification is a property of data, and the contract ([Def. I.2](../../01-foundations/definitions.md#Def.%20I.2%20-%20Contract)) is the whole of what is known about the data crossing a boundary. By [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit) a classification not written in the contract does not exist for the consumer, and by [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts) it is not architecture. The contract is already the single machine-readable artefact ([Prop. II.1](../../02-api-guidelines/propositions/01-contract-first.md), [Prop. III.4](../../03-events/propositions/04-schemas-and-registry.md)), so the tag belongs there, where linters, generators and log redaction ([Prop. VI.7](../../06-observability/propositions/07-no-pii-or-secrets-in-logs.md)) can read it ([Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification)). Defaulting the untagged case to `Confidential` makes omission safe rather than permissive. ∎ Q.E.D.

## Corollaries

* **Cor. V.5.1** - A `Restricted` field MUST NOT appear in a query string, a path segment, or an event payload ([Prop. V.6](06-pii-minimisation-in-events.md)).
* **Cor. V.5.2** - Generated C# and TypeScript types carry the classification as an attribute or JSDoc tag so that redaction and serialisation policies can act on it.

## Construction

* OpenAPI 3.1 and JSON Schema 2020-12 extension keyword `x-data-classification` with enum `Public | Internal | Confidential | Restricted`, published in the catalogue ([Book IV](../../04-shared-schemas/README.md)) as a shared vocabulary.
* Spectral rule set `company-openapi` and `ajv` custom keyword validating the enum.
* Generator hooks (NSwag or Kiota; openapi-typescript) emitting `[DataClassification(Restricted)]` and `@classification` respectively.
* `Company.Contracts.Shared` exposing the `DataClassification` enum and attribute.

## Conformance

Spectral rule: every property whose name matches the PII and credential dictionaries carries the extension; every `x-data-classification` value is in the enum. Schema registry admission check for events.

## Scholium

Tagging `Public` and `Internal` is permitted and encouraged where a field would otherwise match a dictionary heuristic and be flagged. The dictionary is a heuristic; the tag is the truth.
