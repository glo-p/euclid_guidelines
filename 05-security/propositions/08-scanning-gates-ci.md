# Prop. V.8 - Dependency, image and IaC scanning gate CI

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every service pipeline MUST run, and MUST fail on findings at or above the agreed severity from, three scans before any artefact is deployed: package dependencies (NuGet and npm), container images, and infrastructure as code. A finding MUST be fixed or recorded as a time-bound exception ([Def. I.20](../../01-foundations/definitions.md#Def.%20I.20%20-%20Exception)); it MUST NOT be suppressed locally.

## Given

[Def. V.10](../definitions.md#Def.%20V.10%20-%20Attack%20Surface), [Def. I.20](../../01-foundations/definitions.md#Def.%20I.20%20-%20Exception), [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification), [Post. V.4](../postulates.md#Post.%20V.4%20-%20Least%20Privilege), [Post. I.1](../../01-foundations/postulates.md#Post.%20I.1%20-%20Cloud), [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated).

## Demonstration

The attack surface ([Def. V.10](../definitions.md#Def.%20V.10%20-%20Attack%20Surface)) includes the dependencies a service executes and the image and infrastructure it runs on, none of which the owning team wrote. Those parts change beneath the team without a commit, so only a machine check at each build sees the current state ([Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification)). [Post. V.4](../postulates.md#Post.%20V.4%20-%20Least%20Privilege) is itself unverifiable without IaC scanning, since an over-broad IAM policy is invisible in a code review. A suppression that is not recorded is, by [Def. I.20](../../01-foundations/definitions.md#Def.%20I.20%20-%20Exception), a defect rather than an exception, and by [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated) an unobserved vulnerability is one the company does not know it has. ∎ Q.E.D.

## Corollaries

* **Cor. V.8.1** - Base images come only from the company registry, rebuilt on a fixed cadence.
* **Cor. V.8.2** - A dependency with no maintained version is a finding, not a pass.

## Construction

* Dependencies: `dotnet list package --vulnerable --include-transitive` and `npm audit` in CI, or Dependabot/Renovate with GitHub Advanced Security; SBOM via `CycloneDX`.
* Images: Amazon ECR enhanced scanning (Inspector) with scan-on-push; pipeline gate on the ECR findings API.
* IaC: `checkov` or `tfsec` for Terraform, `cdk-nag` for CDK; AWS IAM Access Analyzer policy validation for every policy document.
* Runtime: Amazon Inspector for Lambda and ECS; AWS Security Hub aggregating findings.
* Exceptions: `.security-exceptions.yaml` in the repository, each entry citing an ADR and an expiry date; the gate reads it and fails on expired entries.

## Conformance

CI policy: pipeline template requires the three scan stages and forbids skipping them; Security Hub control for ECR scan-on-push; scheduled job fails any repository whose exceptions file has an expired entry.

## Scholium

The severity threshold is an open question. The scaffold assumes `High` and above block, `Medium` warns, subject to the principals' decision.
