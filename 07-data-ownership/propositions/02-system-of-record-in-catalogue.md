# Prop. VII.2 - The system of record is named in the catalogue

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

For every business concept, the catalogue ([Def. I.24](../../01-foundations/definitions.md#Def.%20I.24%20-%20Catalogue)) MUST name exactly one system of record ([Def. VII.1](../definitions.md#Def.%20VII.1%20-%20System%20of%20Record)) and its data owner ([Def. VII.2](../definitions.md#Def.%20VII.2%20-%20Data%20Owner)). Every other store holding that concept MUST be declared a read model ([Def. VII.3](../definitions.md#Def.%20VII.3%20-%20Read%20Model)) and MUST NOT be presented to any consumer as authoritative.

## Given

* [Def. I.6](../../01-foundations/definitions.md#Def.%20I.6%20-%20Aggregate), [Def. I.24](../../01-foundations/definitions.md#Def.%20I.24%20-%20Catalogue), [Def. VII.1](../definitions.md#Def.%20VII.1%20-%20System%20of%20Record), [Def. VII.2](../definitions.md#Def.%20VII.2%20-%20Data%20Owner), [Def. VII.3](../definitions.md#Def.%20VII.3%20-%20Read%20Model)
* [Post. VII.1](../postulates.md#Post.%20VII.1%20-%20One%20System%20of%20Record)
* [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape), [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts)

## Demonstration

[Post. VII.1](../postulates.md#Post.%20VII.1%20-%20One%20System%20of%20Record) gives each aggregate exactly one system of record, but a fact that is not written down does not exist for a consumer ([CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit)) and is not architecture ([CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts)). Therefore the assignment must be recorded, and the only place shared by every team is the catalogue ([Def. I.24](../../01-foundations/definitions.md#Def.%20I.24%20-%20Catalogue)). Once one store is authoritative, every other store is by definition a copy ([Def. VII.3](../definitions.md#Def.%20VII.3%20-%20Read%20Model)); leaving a copy's status implicit invites two "truths" for one concept, which [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape) identifies as cost without benefit. ∎ Q.E.D.

## Corollaries

* **Cor. VII.2.1** - A concept with no entry in the catalogue has no system of record and MUST NOT be written by any service until one is assigned.
* **Cor. VII.2.2** - Moving a concept's system of record between services is a breaking change to the concept and is performed under [Book IX](../../09-versioning-and-deprecation/README.md).

## Construction

* A `systems-of-record.yaml` file in the catalogue repository ([Prop. IV.1](../../04-shared-schemas/propositions/01-catalogue-and-distribution.md)) listing concept, owning service, owning team, schema reference and classification.
* A CI check in the catalogue that every shared schema ([Prop. IV.1](../../04-shared-schemas/propositions/01-catalogue-and-distribution.md)) for a business concept references an entry in that file.
* The service's own OpenAPI document ([Prop. II.1](../../02-api-guidelines/propositions/01-contract-first.md)) declares in `info.x-system-of-record` the concepts it is authoritative for; read-model endpoints declare `x-read-model-of` naming the source concept.

## Conformance

Catalogue CI job `sor-registry-check` fails if a concept appears twice, has no owner, or is referenced by an OpenAPI `x-system-of-record` extension in more than one service contract.

## Scholium

The catalogue entry is small. Its value is that the question "who owns customer address" has one answer that a machine can look up.
