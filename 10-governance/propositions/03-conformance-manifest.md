# Prop. X.3 - Every service publishes a conformance manifest

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every service repository ([Post. X.3](../postulates.md#Post.%20X.3%20-%20Every%20Service%20Has%20a%20Repository)) MUST contain a conformance manifest ([Def. X.5](../definitions.md#Def.%20X.5%20-%20Conformance%20Manifest)) `conformance.json` that lists every MUST proposition of Books II to XI applicable to the service with a status of `conformant`, `exception ADR-nnnn` or `non-conformant`. CI MUST validate the manifest against its JSON Schema on every pull request and MUST publish it to the central conformance index on every merge to the default branch. A `conformant` entry MUST name the machine check that produced it.

## Given

* [Def. I.1](../../01-foundations/definitions.md#Def.%20I.1%20-%20Service), [Def. X.4](../definitions.md#Def.%20X.4%20-%20Exception), [Def. X.5](../definitions.md#Def.%20X.5%20-%20Conformance%20Manifest)
* [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification), [Post. X.3](../postulates.md#Post.%20X.3%20-%20Every%20Service%20Has%20a%20Repository)
* [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated), [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts)
* [Prop. I.2](../../01-foundations/method.md#Prop.%20I.2%20-%20Levels%20and%20their%20obligations), [Prop. I.3](../../01-foundations/method.md#Prop.%20I.3%20-%20Every%20MUST%20has%20a%20machine%20check), [Prop. X.2](02-exception-process.md)

## Demonstration

[Prop. I.2](../../01-foundations/method.md#Prop.%20I.2%20-%20Levels%20and%20their%20obligations) binds every service to every MUST, and [Prop. I.3](../../01-foundations/method.md#Prop.%20I.3%20-%20Every%20MUST%20has%20a%20machine%20check) gives each MUST a machine check; the manifest is where the results of those checks for one service are collected. By [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated) a conformance nobody can observe is absent, and by [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts) the state of the estate must be recorded to be architecture; a central index of manifests is the observation. [Post. X.3](../postulates.md#Post.%20X.3%20-%20Every%20Service%20Has%20a%20Repository) guarantees a repository and a CI pipeline per service, so there is one place to keep the file and one process to publish it. By [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification) the file must be validated and published by CI, not maintained by hand, or it will drift within a year. ∎ Q.E.D.

## Corollaries

* **Cor. X.3.1** - The index is the sole source for "which services conform to Prop. B.n"; a report produced any other way is not authoritative.
* **Cor. X.3.2** - When a new MUST proposition is accepted, every manifest lacking an entry for it is `non-conformant` by default until updated.

## Construction

* JSON Schema `https://schemas.company.example/v1/conformance-manifest.json` in the catalogue: `{service, team, generatedAt, propositions: [{id, level, status, evidence, adr}]}`.
* Generator `dotnet tool` `Company.Conformance` that reads the repository's check results (Spectral, contract tests, architecture tests, AWS Config findings exported by the pipeline) and writes the manifest; teams edit only `exception` and `non-conformant` rows.
* CI step `conformance-validate` on pull requests; step `conformance-publish` on merge that `PUT`s the manifest to the index (S3 bucket `company-conformance-index` with object key `<service>/latest.json` plus a DynamoDB summary table; store is an open question).
* Index read API `GET /conformance/services/{service}` and `GET /conformance/propositions/{id}` on the platform catalogue API.
* Nightly job re-validating every manifest against the current proposition list and exception register ([Prop. X.2](02-exception-process.md)), emitting `company.platform.conformance-changed.v1`.

## Conformance

CI check `conformance-validate` required in every service repository; the index itself lists services with no manifest published in the last 30 days.

## Scholium

The manifest is a report, not a promise. A team that writes `conformant` without a check is not conformant; it is wrong, and [Prop. I.3](../../01-foundations/method.md#Prop.%20I.3%20-%20Every%20MUST%20has%20a%20machine%20check) is the reason the evidence field is mandatory.
