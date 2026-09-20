# Prop. VIII.8 - Pipelines: trunk-based, build once, gated promotion

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every service MUST be delivered by a pipeline ([Def. VIII.8](../definitions.md#Def.%20VIII.8%20-%20Pipeline)) from the central system ([Post. VIII.4](../postulates.md#Post.%20VIII.4%20-%20One%20Pipeline%20System)) that builds one artefact ([Def. VIII.9](../definitions.md#Def.%20VIII.9%20-%20Artefact)) from trunk and promotes it through environments in order. Promotion beyond `dev` MUST be gated by contract tests ([Prop. III.12](../../03-events/propositions/12-consumer-contracts-and-testing.md) and the API contract checks of [Prop. II.1](../../02-api-guidelines/propositions/01-contract-first.md)) and by security scanning ([Prop. V.8](../../05-security/propositions/08-scanning-gates-ci.md)). Long-lived branches MUST NOT be deployed to any environment above sandbox.

## Given

* [Def. VIII.8](../definitions.md#Def.%20VIII.8%20-%20Pipeline), [Def. VIII.9](../definitions.md#Def.%20VIII.9%20-%20Artefact)
* [Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution), [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification), [Post. I.7](../../01-foundations/postulates.md#Post.%20I.7%20-%20Contract%20First), [Post. VIII.4](../postulates.md#Post.%20VIII.4%20-%20One%20Pipeline%20System)
* [Prop. II.1](../../02-api-guidelines/propositions/01-contract-first.md), [Prop. III.12](../../03-events/propositions/12-consumer-contracts-and-testing.md), [Prop. V.8](../../05-security/propositions/08-scanning-gates-ci.md), [Prop. VIII.7](07-identical-environments.md)
* [CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed)

## Demonstration

[Post. I.7](../../01-foundations/postulates.md#Post.%20I.7%20-%20Contract%20First) lets contracts be verified before deployment, and [CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed) makes a broken contract a defect in the producer; the only point at which a machine can stop such a defect ([Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification)) before it reaches consumers is the promotion gate, so the gate runs the contract checks. [Prop. VIII.7](07-identical-environments.md) fixes one artefact per commit, which requires that the commit be on the line every environment sees, that is, trunk; a branch deployed to an environment would create an artefact that no later environment receives. Because consumers are deployed independently ([Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution)), the gate includes the consumer-driven tests of [Prop. III.12](../../03-events/propositions/12-consumer-contracts-and-testing.md) rather than only the producer's own. Scanning ([Prop. V.8](../../05-security/propositions/08-scanning-gates-ci.md)) is a gate for the same reason: it is the last machine before the estate. ∎ Q.E.D.

## Corollaries

* **Cor. VIII.8.1** - Feature flags, not branches, hold incomplete work; a flag's default is off and its removal date is recorded.
* **Cor. VIII.8.2** - A pipeline stage that is skipped by a human is an exception ([Def. I.20](../../01-foundations/definitions.md#Def.%20I.20%20-%20Exception)) and is logged as such.

## Construction

* Central system: one of AWS CodePipeline with CodeBuild, or GitHub Actions with OIDC federation into AWS (open question); templates per compute tier ([Post. VIII.4](../postulates.md#Post.%20VIII.4%20-%20One%20Pipeline%20System)) published as reusable workflows.
* Stages: build and unit test (`dotnet build`, `dotnet test`); contract lint (Spectral ruleset of [Prop. II.1](../../02-api-guidelines/propositions/01-contract-first.md), JSON Schema validation of [Prop. III.4](../../03-events/propositions/04-schemas-and-registry.md)); package (ECR push, Lambda zip); scan ([Prop. V.8](../../05-security/propositions/08-scanning-gates-ci.md): Amazon Inspector, dependency and secret scanning); deploy `dev`; consumer contract tests ([Prop. III.12](../../03-events/propositions/12-consumer-contracts-and-testing.md)); promote `test`; smoke and SLO check ([Prop. VI.4](../../06-observability/propositions/04-slos-per-contract.md)); promote `prod` with progressive delivery (ECS blue/green via CodeDeploy, Lambda aliases with weighted routing).
* Deployment roles per environment assumed via OIDC; no long-lived pipeline credentials.
* Release manifest stored as an artefact with digest, commit and gate results.

## Conformance

Pipeline template check that gate stages are present and `required`; branch protection on trunk requiring the pipeline status; CloudTrail query for deployments by principals other than the pipeline role.

## Scholium

Trunk-based delivery is uncomfortable for a week and then invisible. The discomfort is the cost of never merging a three-month branch.
