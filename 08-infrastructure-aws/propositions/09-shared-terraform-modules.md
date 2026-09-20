# Prop. VIII.9 - Shared Terraform modules in a private registry with semantic versions

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every building block that more than one team needs (queue with DLQ, Fargate service, Lambda consumer, database, EventBridge subscription, tags) MUST be provided as a module (Def. VIII.7) published to the private module registry with a semantic version. Services MUST consume modules by exact version and MUST NOT copy module source into their repositories.

## Given

* Def. I.2, Def. I.14, Def. I.15, Def. VIII.7
* Post. I.5, Post. I.7, Post. VIII.3
* Prop. VIII.1, Prop. VIII.3
* CN 4, CN 5

## Demonstration

A module's inputs and outputs are a contract (Def. I.2) between the platform and every service that uses it, so the compatibility vocabulary of Def. I.14 and Def. I.15 applies and by CN 4 a module change is breaking if any part is; semantic versioning is the standard encoding of exactly that distinction. Consumers lag producers (Post. I.5), so a service must be able to stay on a known version, which requires a registry rather than a moving source reference. Copying the source creates a second shape for one concept (CN 5) that no longer receives fixes. Prop. VIII.3 relies on sanctioned modules to identify the compute tier, so the modules must be identifiable, which the registry provides. ∎ Q.E.D.

## Corollaries

* **Cor. VIII.9.1** - A module major version bump follows the deprecation process of Book IX; the previous major is supported for the stated sunset period.
* **Cor. VIII.9.2** - A module ships its own tests and a `CHANGELOG.md`; a version without both is not published.

## Construction

* Registry: Terraform Cloud private registry, or S3-backed registry in the shared-services account served through a Terraform-registry-protocol endpoint (open question).
* Module repository per module under a `terraform-aws-*` naming convention, owned by the platform team, released by tag `vMAJOR.MINOR.PATCH` through the same pipeline system (Post. VIII.4).
* Module tests with `terraform test` and, where behaviour matters, Terratest in an ephemeral sandbox account.
* Initial module set: `tags`, `fargate-service`, `lambda-sqs-consumer`, `schedule`, `eventbridge-subscription`, `sqs-queue`, `aurora-cluster`, `dynamodb-table`, `api-gateway-service`, `private-dns-name`.
* Consumption pinned: `source = "app.terraform.io/company/fargate-service/aws"`, `version = "= 2.3.1"`; Renovate or Dependabot opens upgrade pull requests.

## Conformance

Pipeline policy on the Terraform plan JSON rejecting `module` blocks whose `source` is a git or local path outside the service repository's own `modules/` for service-specific glue, and rejecting version constraints other than exact pins; registry CI job `module-semver` that compares input and output variables against the previous release and refuses a minor or patch tag for a breaking change.

## Scholium

The module registry is Book IV for infrastructure: one shape per concept, versioned, distributed. The reasoning is the same.
