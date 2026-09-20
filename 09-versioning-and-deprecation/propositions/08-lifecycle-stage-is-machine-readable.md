# Prop. IX.8 - The lifecycle stage of every contract is machine-readable

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

The catalogue ([Def. I.24](../../01-foundations/definitions.md#Def.%20I.24%20-%20Catalogue)) MUST record, for every version ([Def. IX.1](../definitions.md#Def.%20IX.1%20-%20Version)) of every contract in the company, its lifecycle stage ([Def. IX.4](../definitions.md#Def.%20IX.4%20-%20Lifecycle%20Stage)), the dates of each transition, its sunset date where Deprecated, its successor where one exists, and its migration guide link, in a machine-readable form that can be queried by contract name and major. Every transition MUST be a pull request to the catalogue, and the catalogue MUST reject any transition that is not forwards in the order of [Def. IX.4](../definitions.md#Def.%20IX.4%20-%20Lifecycle%20Stage).

## Given

* [Def. I.24](../../01-foundations/definitions.md#Def.%20I.24%20-%20Catalogue), [Def. IX.1](../definitions.md#Def.%20IX.1%20-%20Version), [Def. IX.4](../definitions.md#Def.%20IX.4%20-%20Lifecycle%20Stage), [Def. IX.5](../definitions.md#Def.%20IX.5%20-%20Deprecation%20Notice), [Def. IX.6](../definitions.md#Def.%20IX.6%20-%20Sunset%20Date), [Def. IX.7](../definitions.md#Def.%20IX.7%20-%20Migration%20Guide)
* [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification), [Post. IX.1](../postulates.md#Post.%20IX.1%20-%20Two%20Majors%20at%20Most)
* [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated), [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts)

## Demonstration

By [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts) the architecture is the set of contracts and their topology, and the stage of a contract version is part of what a consumer must know to bind safely ([Def. IX.4](../definitions.md#Def.%20IX.4%20-%20Lifecycle%20Stage)); so the stage is architecture and must be recorded, not remembered. By [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated) a stage that no tool can read is operationally absent, and by [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification) the checks of Props. IX.2, IX.5 and IX.7 will not be enforced unless a machine can read the stage they test. The catalogue already exists and is unique ([Def. I.24](../../01-foundations/definitions.md#Def.%20I.24%20-%20Catalogue)), so it is the natural single place. [Post. IX.1](../postulates.md#Post.%20IX.1%20-%20Two%20Majors%20at%20Most) is a constraint over all versions of one contract, which can only be checked where all of them are listed together. ∎ Q.E.D.

## Corollaries

* **Cor. IX.8.1** - The `x-lifecycle` block in each contract artefact ([Prop. IX.3](03-deprecation-is-declared-in-the-contract.md)) is a copy of the catalogue record; the catalogue is authoritative and CI fails on disagreement.
* **Cor. IX.8.2** - A version absent from the catalogue is in no stage and is not owed; a service serving such a version is non-conformant.

## Construction

* Catalogue file `contracts/<name>/lifecycle.json` validated by JSON Schema `https://schemas.company.example/v1/contract-lifecycle.json` with fields `name`, `kind: api|event|schema`, `majors[]: {major, stage, proposedOn, activeOn, deprecatedOn, sunsetOn, retiredOn, successor, migrationGuide, notice}`.
* Catalogue CI: JSON Schema validation, forward-only transition check against the previous commit, [Post. IX.1](../postulates.md#Post.%20IX.1%20-%20Two%20Majors%20at%20Most) check (count of `active|deprecated` majors at most 2).
* Publication: catalogue CI pushes the merged records to an S3 bucket behind CloudFront and to a DynamoDB table; read API `GET /contracts/{name}/lifecycle` on the platform catalogue API used by the notifier ([Prop. IX.3](03-deprecation-is-declared-in-the-contract.md)), the advancer ([Prop. IX.5](05-sunset-on-evidence-retirement-removes.md)) and the conformance index ([Prop. X.3](../../10-governance/propositions/03-conformance-manifest.md)).
* Emits `company.platform.contract-lifecycle-changed.v1` on every merged transition.
* Backstage (or equivalent) plugin reading the same records for humans.

## Conformance

Catalogue CI schema validation and transition check; conformance manifest ([Prop. X.3](../../10-governance/propositions/03-conformance-manifest.md)) entry for every service asserting each served contract version has a catalogue record in stage `active` or `deprecated`.

## Scholium

The catalogue holds the fact; the artefact carries a copy so that a consumer with only the artefact still sees it. Two sources of truth would be a defect, so CI enforces that the copy equals the record.
