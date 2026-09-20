# Prop. IX.1 - Only compatible changes within a major version

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A change to an Active version of a contract MUST be a compatible change ([Def. I.15](../../01-foundations/definitions.md#Def.%20I.15%20-%20Compatible%20Change)) and MUST be published as a minor change ([Def. IX.3](../definitions.md#Def.%20IX.3%20-%20Minor%20Change)) of the same major version ([Def. IX.2](../definitions.md#Def.%20IX.2%20-%20Major%20Version)). Every pull request that alters a contract artefact MUST pass a contract diff in CI that classifies the change against [Def. I.14](../../01-foundations/definitions.md#Def.%20I.14%20-%20Breaking%20Change) and [Def. I.15](../../01-foundations/definitions.md#Def.%20I.15%20-%20Compatible%20Change) and fails the build on any breaking part.

## Given

* [Def. I.14](../../01-foundations/definitions.md#Def.%20I.14%20-%20Breaking%20Change), [Def. I.15](../../01-foundations/definitions.md#Def.%20I.15%20-%20Compatible%20Change), [Def. IX.2](../definitions.md#Def.%20IX.2%20-%20Major%20Version), [Def. IX.3](../definitions.md#Def.%20IX.3%20-%20Minor%20Change), [Def. IX.4](../definitions.md#Def.%20IX.4%20-%20Lifecycle%20Stage)
* [Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution), [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification)
* [CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed), [CN 4](../../01-foundations/common-notions.md#CN%204%20-%20Compatibility%20Is%20Compositional)

## Demonstration

An Active version is owed ([CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed), [Def. IX.4](../definitions.md#Def.%20IX.4%20-%20Lifecycle%20Stage)), so any change to it that can make a conforming consumer fail is a defect in the producer. By [Def. I.14](../../01-foundations/definitions.md#Def.%20I.14%20-%20Breaking%20Change) and [Def. I.15](../../01-foundations/definitions.md#Def.%20I.15%20-%20Compatible%20Change) the set of changes that cannot do so is exhaustively enumerated, and by [CN 4](../../01-foundations/common-notions.md#CN%204%20-%20Compatibility%20Is%20Compositional) a change is safe if and only if every part of it is in that set. Because consumers lag producers indefinitely ([Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution)), the producer cannot rely on consumers adapting; the change itself must be safe. By [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification) the classification must be done by a machine on every change, so the contract diff runs in CI and blocks the merge. ∎ Q.E.D.

## Corollaries

* **Cor. IX.1.1** - Removing a field, path, event type or enum value from an Active version is never permitted, even if traffic evidence ([Def. IX.9](../definitions.md#Def.%20IX.9%20-%20Traffic%20Evidence)) shows the field is unused; removal happens only through a new major ([Prop. IX.2](02-breaking-change-is-a-new-major-side-by-side.md)).
* **Cor. IX.1.2** - A minor change is published without a deprecation notice and without consumer notification; consumers must tolerate it ([CN 6](../../01-foundations/common-notions.md#CN%206%20-%20The%20Producer%20Pays%20for%20Stability%3B%20the%20Consumer%20Pays%20for%20Tolerance)).

## Construction

* OpenAPI: `oasdiff breaking` (or `openapi-diff`) as a required GitHub Actions / CodeBuild step comparing the PR document to the last published version of the same major.
* JSON Schema (events, shared schemas): `json-schema-diff` run against the previous published `$id`; enum openness read from the `x-open-enum` marker of [Prop. IV.9](../../04-shared-schemas/propositions/09-open-enumerations.md).
* Spectral ruleset `company-contract-diff` that classifies the diff report into [Def. I.14](../../01-foundations/definitions.md#Def.%20I.14%20-%20Breaking%20Change) / [Def. I.15](../../01-foundations/definitions.md#Def.%20I.15%20-%20Compatible%20Change) categories and emits the category in the PR check summary.
* Contract artefacts tagged in git as `<contract>/v<major>.<minor>` on merge; the diff baseline is the highest tag of the same major.
* .NET: `Company.Contracts.Diff` CLI packaged as a `dotnet tool` so teams can run the same classifier locally.

## Conformance

CI check `contract-diff` is a required status check on every repository that publishes a contract; the central conformance index ([Prop. X.3](../../10-governance/propositions/03-conformance-manifest.md)) records its presence.

## Scholium

The diff tool is the sole arbiter of "compatible". If a team believes a change is compatible and the tool disagrees, the correct move is an ADR proposing a refinement of [Def. I.15](../../01-foundations/definitions.md#Def.%20I.15%20-%20Compatible%20Change), not a bypass. The exhaustiveness of [Def. I.14](../../01-foundations/definitions.md#Def.%20I.14%20-%20Breaking%20Change) and [Def. I.15](../../01-foundations/definitions.md#Def.%20I.15%20-%20Compatible%20Change) is what makes the tool possible.
