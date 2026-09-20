# Prop. X.3 - Every service publishes a conformance manifest

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every service repository (Post. X.3) MUST contain a conformance manifest (Def. X.5) `conformance.json` that lists every MUST proposition of Books II to XI applicable to the service with a status of `conformant`, `exception ADR-nnnn` or `non-conformant`. CI MUST validate the manifest against its JSON Schema on every pull request and MUST publish it to the central conformance index on every merge to the default branch. A `conformant` entry MUST name the machine check that produced it.

## Given

* Def. I.1, Def. X.4, Def. X.5
* Post. I.6, Post. X.3
* CN 7, CN 8
* Prop. I.2, Prop. I.3, Prop. X.2

## Demonstration

Prop. I.2 binds every service to every MUST, and Prop. I.3 gives each MUST a machine check; the manifest is where the results of those checks for one service are collected. By CN 7 a conformance nobody can observe is absent, and by CN 8 the state of the estate must be recorded to be architecture; a central index of manifests is the observation. Post. X.3 guarantees a repository and a CI pipeline per service, so there is one place to keep the file and one process to publish it. By Post. I.6 the file must be validated and published by CI, not maintained by hand, or it will drift within a year. ∎ Q.E.D.

## Corollaries

* **Cor. X.3.1** - The index is the sole source for "which services conform to Prop. B.n"; a report produced any other way is not authoritative.
* **Cor. X.3.2** - When a new MUST proposition is accepted, every manifest lacking an entry for it is `non-conformant` by default until updated.

## Construction

* JSON Schema `https://schemas.company.example/v1/conformance-manifest.json` in the catalogue: `{service, team, generatedAt, propositions: [{id, level, status, evidence, adr}]}`.
* Generator `dotnet tool` `Company.Conformance` that reads the repository's check results (Spectral, contract tests, architecture tests, AWS Config findings exported by the pipeline) and writes the manifest; teams edit only `exception` and `non-conformant` rows.
* CI step `conformance-validate` on pull requests; step `conformance-publish` on merge that `PUT`s the manifest to the index (S3 bucket `company-conformance-index` with object key `<service>/latest.json` plus a DynamoDB summary table; store is an open question).
* Index read API `GET /conformance/services/{service}` and `GET /conformance/propositions/{id}` on the platform catalogue API.
* Nightly job re-validating every manifest against the current proposition list and exception register (Prop. X.2), emitting `company.platform.conformance-changed.v1`.

## Conformance

CI check `conformance-validate` required in every service repository; the index itself lists services with no manifest published in the last 30 days.

## Scholium

The manifest is a report, not a promise. A team that writes `conformant` without a check is not conformant; it is wrong, and Prop. I.3 is the reason the evidence field is mandatory.
