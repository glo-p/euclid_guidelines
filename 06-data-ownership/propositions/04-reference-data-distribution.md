# Prop. VII.4 - Reference data is distributed as a versioned dataset

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Reference data (Def. VII.7) MUST be published from the catalogue (Def. I.24) as a versioned dataset alongside the shared schema (Def. I.13) that describes it, in the packages of Prop. IV.1. Services MUST NOT fetch reference data from another service at run time and MUST NOT maintain private copies with divergent values.

## Given

* Def. I.13, Def. I.24, Def. VII.7, Def. VII.8
* Post. I.5, Post. I.7
* Prop. IV.1, Prop. IV.9
* CN 5

## Demonstration

Reference data has no system of record (Def. VII.7, contrast Def. VII.8), so no service can be its producer, and a run-time call for it would be a call to no contract. By CN 5 two copies of the same value set with different shapes or contents are a cost; the catalogue already distributes shared schemas to every team (Prop. IV.1), so the dataset travels the same road as its schema. Because consumers lag producers indefinitely (Post. I.5), the dataset is versioned and the schema declares its enumerations open (Prop. IV.9) so that an older consumer tolerates a newer value. ∎ Q.E.D.

## Corollaries

* **Cor. VII.4.1** - A change to reference data is a new dataset version, never an edit in place; the version is recorded in the event or response that used it where the value set affects interpretation.

## Construction

* Datasets as JSON files in the catalogue repository under `reference-data/<name>/`, each with a JSON Schema 2020-12 document and a `version` field.
* Packaged into `Company.Contracts.Shared` (NuGet) as embedded resources with typed accessors, and into `@company/contracts` (npm) as JSON modules.
* Also published to an S3 bucket in the platform account with a versioned key, for non-.NET consumers and for the data platform (Prop. VII.7).
* Catalogue CI validates each dataset against its schema and diffs it against the previous version to reject removals without a deprecation entry (Book IX).

## Conformance

Catalogue CI job `reference-data-validate`; architecture test in service repositories that no type under `*.ReferenceData` is backed by a database table or an HTTP client.

## Scholium

Currencies and countries change a few times a year. The cost of a run-time dependency for something that changes that slowly is never repaid.
