# Prop. IX.1 - Only compatible changes within a major version

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A change to an Active version of a contract MUST be a compatible change (Def. I.15) and MUST be published as a minor change (Def. IX.3) of the same major version (Def. IX.2). Every pull request that alters a contract artefact MUST pass a contract diff in CI that classifies the change against Def. I.14 and Def. I.15 and fails the build on any breaking part.

## Given

* Def. I.14, Def. I.15, Def. IX.2, Def. IX.3, Def. IX.4
* Post. I.5, Post. I.6
* CN 2, CN 4

## Demonstration

An Active version is owed (CN 2, Def. IX.4), so any change to it that can make a conforming consumer fail is a defect in the producer. By Def. I.14 and Def. I.15 the set of changes that cannot do so is exhaustively enumerated, and by CN 4 a change is safe if and only if every part of it is in that set. Because consumers lag producers indefinitely (Post. I.5), the producer cannot rely on consumers adapting; the change itself must be safe. By Post. I.6 the classification must be done by a machine on every change, so the contract diff runs in CI and blocks the merge. ∎ Q.E.D.

## Corollaries

* **Cor. IX.1.1** - Removing a field, path, event type or enum value from an Active version is never permitted, even if traffic evidence (Def. IX.9) shows the field is unused; removal happens only through a new major (Prop. IX.2).
* **Cor. IX.1.2** - A minor change is published without a deprecation notice and without consumer notification; consumers must tolerate it (CN 6).

## Construction

* OpenAPI: `oasdiff breaking` (or `openapi-diff`) as a required GitHub Actions / CodeBuild step comparing the PR document to the last published version of the same major.
* JSON Schema (events, shared schemas): `json-schema-diff` run against the previous published `$id`; enum openness read from the `x-open-enum` marker of Prop. IV.9.
* Spectral ruleset `company-contract-diff` that classifies the diff report into Def. I.14 / Def. I.15 categories and emits the category in the PR check summary.
* Contract artefacts tagged in git as `<contract>/v<major>.<minor>` on merge; the diff baseline is the highest tag of the same major.
* .NET: `Company.Contracts.Diff` CLI packaged as a `dotnet tool` so teams can run the same classifier locally.

## Conformance

CI check `contract-diff` is a required status check on every repository that publishes a contract; the central conformance index (Prop. X.3) records its presence.

## Scholium

The diff tool is the sole arbiter of "compatible". If a team believes a change is compatible and the tool disagrees, the correct move is an ADR proposing a refinement of Def. I.15, not a bypass. The exhaustiveness of Def. I.14 and Def. I.15 is what makes the tool possible.
