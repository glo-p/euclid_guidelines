# Prop. IX.8 - The lifecycle stage of every contract is machine-readable

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

The catalogue (Def. I.24) MUST record, for every version (Def. IX.1) of every contract in the company, its lifecycle stage (Def. IX.4), the dates of each transition, its sunset date where Deprecated, its successor where one exists, and its migration guide link, in a machine-readable form that can be queried by contract name and major. Every transition MUST be a pull request to the catalogue, and the catalogue MUST reject any transition that is not forwards in the order of Def. IX.4.

## Given

* Def. I.24, Def. IX.1, Def. IX.4, Def. IX.5, Def. IX.6, Def. IX.7
* Post. I.6, Post. IX.1
* CN 7, CN 8

## Demonstration

By CN 8 the architecture is the set of contracts and their topology, and the stage of a contract version is part of what a consumer must know to bind safely (Def. IX.4); so the stage is architecture and must be recorded, not remembered. By CN 7 a stage that no tool can read is operationally absent, and by Post. I.6 the checks of Props. IX.2, IX.5 and IX.7 will not be enforced unless a machine can read the stage they test. The catalogue already exists and is unique (Def. I.24), so it is the natural single place. Post. IX.1 is a constraint over all versions of one contract, which can only be checked where all of them are listed together. ∎ Q.E.D.

## Corollaries

* **Cor. IX.8.1** - The `x-lifecycle` block in each contract artefact (Prop. IX.3) is a copy of the catalogue record; the catalogue is authoritative and CI fails on disagreement.
* **Cor. IX.8.2** - A version absent from the catalogue is in no stage and is not owed; a service serving such a version is non-conformant.

## Construction

* Catalogue file `contracts/<name>/lifecycle.json` validated by JSON Schema `https://schemas.company.example/v1/contract-lifecycle.json` with fields `name`, `kind: api|event|schema`, `majors[]: {major, stage, proposedOn, activeOn, deprecatedOn, sunsetOn, retiredOn, successor, migrationGuide, notice}`.
* Catalogue CI: JSON Schema validation, forward-only transition check against the previous commit, Post. IX.1 check (count of `active|deprecated` majors at most 2).
* Publication: catalogue CI pushes the merged records to an S3 bucket behind CloudFront and to a DynamoDB table; read API `GET /contracts/{name}/lifecycle` on the platform catalogue API used by the notifier (Prop. IX.3), the advancer (Prop. IX.5) and the conformance index (Prop. X.3).
* Emits `company.platform.contract-lifecycle-changed.v1` on every merged transition.
* Backstage (or equivalent) plugin reading the same records for humans.

## Conformance

Catalogue CI schema validation and transition check; conformance manifest (Prop. X.3) entry for every service asserting each served contract version has a catalogue record in stage `active` or `deprecated`.

## Scholium

The catalogue holds the fact; the artefact carries a copy so that a consumer with only the artefact still sees it. Two sources of truth would be a defect, so CI enforces that the copy equals the record.
