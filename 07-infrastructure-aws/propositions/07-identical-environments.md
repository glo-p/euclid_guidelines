# Prop. VIII.7 - Environments differ only in configuration; one artefact is promoted

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every environment (Def. I.21, Def. VIII.4) MUST be produced from the same infrastructure source and MUST run the same artefact (Def. VIII.9), identified by digest, that passed the previous environment. Only configuration values MAY differ between environments, and those values MUST be held outside the artefact.

## Given

* Def. I.21, Def. VIII.4, Def. VIII.8, Def. VIII.9
* Post. I.5, Post. I.6
* Prop. VIII.1, Prop. VIII.8
* CN 1

## Demonstration

Def. I.21 defines environments as identical except configuration; Prop. VIII.1 makes the infrastructure reproducible from source, so identity of infrastructure is achievable. For the code, a test in `test` says nothing about `prod` unless what runs in `prod` is the same bytes; by CN 1 two builds from the same commit are not guaranteed interchangeable (different base images, different dependency resolution), so the artefact is built once and its digest is carried forward. Configuration must then live outside the artefact or the artefact could not be reused. Post. I.6 requires the digest equality to be checked by the pipeline (Def. VIII.8), not asserted. ∎ Q.E.D.

## Corollaries

* **Cor. VIII.7.1** - A "hotfix" is a new artefact that passes every environment in order; there is no path that skips one.
* **Cor. VIII.7.2** - Configuration is data with a schema; it is validated at start-up and a missing value fails the deployment, not the first request.

## Construction

* Container images in Amazon ECR (shared-services account) referenced by digest in the ECS task definition; Lambda deployment packages by S3 object version and SHA-256 hash; immutable image tags enabled on ECR repositories.
* Configuration in AWS Systems Manager Parameter Store (non-secret) and AWS Secrets Manager (secret, Prop. V.4) under `/<env>/<service>/...`; loaded via `Microsoft.Extensions.Configuration` providers (`Amazon.Extensions.Configuration.SystemsManager`).
* Terraform workspaces or per-environment `tfvars` files; no environment-conditional resources in modules.
* Pipeline promotion step records the digest in a release manifest and refuses a deployment whose digest differs from the manifest.

## Conformance

Pipeline policy `artefact-digest-match` comparing the deployed task definition or function code SHA against the release manifest; AWS Config rule `ecr-private-tag-immutability-enabled`; architecture test that no `#if` or environment-name branch exists in service code.

## Scholium

"It worked in test" is only evidence when test and prod ran the same thing. This proposition is what makes that sentence meaningful.
