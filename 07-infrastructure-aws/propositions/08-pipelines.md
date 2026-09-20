# Prop. VIII.8 - Pipelines: trunk-based, build once, gated promotion

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every service MUST be delivered by a pipeline (Def. VIII.8) from the central system (Post. VIII.4) that builds one artefact (Def. VIII.9) from trunk and promotes it through environments in order. Promotion beyond `dev` MUST be gated by contract tests (Prop. III.12 and the API contract checks of Prop. II.1) and by security scanning (Prop. V.8). Long-lived branches MUST NOT be deployed to any environment above sandbox.

## Given

* Def. VIII.8, Def. VIII.9
* Post. I.5, Post. I.6, Post. I.7, Post. VIII.4
* Prop. II.1, Prop. III.12, Prop. V.8, Prop. VIII.7
* CN 2

## Demonstration

Post. I.7 lets contracts be verified before deployment, and CN 2 makes a broken contract a defect in the producer; the only point at which a machine can stop such a defect (Post. I.6) before it reaches consumers is the promotion gate, so the gate runs the contract checks. Prop. VIII.7 fixes one artefact per commit, which requires that the commit be on the line every environment sees, that is, trunk; a branch deployed to an environment would create an artefact that no later environment receives. Because consumers are deployed independently (Post. I.5), the gate includes the consumer-driven tests of Prop. III.12 rather than only the producer's own. Scanning (Prop. V.8) is a gate for the same reason: it is the last machine before the estate. ∎ Q.E.D.

## Corollaries

* **Cor. VIII.8.1** - Feature flags, not branches, hold incomplete work; a flag's default is off and its removal date is recorded.
* **Cor. VIII.8.2** - A pipeline stage that is skipped by a human is an exception (Def. I.20) and is logged as such.

## Construction

* Central system: one of AWS CodePipeline with CodeBuild, or GitHub Actions with OIDC federation into AWS (open question); templates per compute tier (Post. VIII.4) published as reusable workflows.
* Stages: build and unit test (`dotnet build`, `dotnet test`); contract lint (Spectral ruleset of Prop. II.1, JSON Schema validation of Prop. III.4); package (ECR push, Lambda zip); scan (Prop. V.8: Amazon Inspector, dependency and secret scanning); deploy `dev`; consumer contract tests (Prop. III.12); promote `test`; smoke and SLO check (Prop. VI.4); promote `prod` with progressive delivery (ECS blue/green via CodeDeploy, Lambda aliases with weighted routing).
* Deployment roles per environment assumed via OIDC; no long-lived pipeline credentials.
* Release manifest stored as an artefact with digest, commit and gate results.

## Conformance

Pipeline template check that gate stages are present and `required`; branch protection on trunk requiring the pipeline status; CloudTrail query for deployments by principals other than the pipeline role.

## Scholium

Trunk-based delivery is uncomfortable for a week and then invisible. The discomfort is the cost of never merging a three-month branch.
