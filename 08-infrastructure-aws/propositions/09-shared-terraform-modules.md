# Prop. VIII.9 - Shared Terraform modules in a private registry with semantic versions

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every building block that more than one team needs (queue with DLQ, Fargate service, Lambda consumer, database, EventBridge subscription, tags) MUST be provided as a module ([Def. VIII.7](../definitions.md#Def.%20VIII.7%20-%20Module)) published to the private module registry with a semantic version. Services MUST consume modules by exact version and MUST NOT copy module source into their repositories.

## Given

* [Def. I.2](../../01-foundations/definitions.md#Def.%20I.2%20-%20Contract), [Def. I.14](../../01-foundations/definitions.md#Def.%20I.14%20-%20Breaking%20Change), [Def. I.15](../../01-foundations/definitions.md#Def.%20I.15%20-%20Compatible%20Change), [Def. VIII.7](../definitions.md#Def.%20VIII.7%20-%20Module)
* [Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution), [Post. I.7](../../01-foundations/postulates.md#Post.%20I.7%20-%20Contract%20First), [Post. VIII.3](../postulates.md#Post.%20VIII.3%20-%20Terraform)
* [Prop. VIII.1](01-everything-is-iac.md), [Prop. VIII.3](03-compute-choice.md)
* [CN 4](../../01-foundations/common-notions.md#CN%204%20-%20Compatibility%20Is%20Compositional), [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape)

## Demonstration

A module's inputs and outputs are a contract ([Def. I.2](../../01-foundations/definitions.md#Def.%20I.2%20-%20Contract)) between the platform and every service that uses it, so the compatibility vocabulary of [Def. I.14](../../01-foundations/definitions.md#Def.%20I.14%20-%20Breaking%20Change) and [Def. I.15](../../01-foundations/definitions.md#Def.%20I.15%20-%20Compatible%20Change) applies and by [CN 4](../../01-foundations/common-notions.md#CN%204%20-%20Compatibility%20Is%20Compositional) a module change is breaking if any part is; semantic versioning is the standard encoding of exactly that distinction. Consumers lag producers ([Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution)), so a service must be able to stay on a known version, which requires a registry rather than a moving source reference. Copying the source creates a second shape for one concept ([CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape)) that no longer receives fixes. [Prop. VIII.3](03-compute-choice.md) relies on sanctioned modules to identify the compute tier, so the modules must be identifiable, which the registry provides. ∎ Q.E.D.

## Corollaries

* **Cor. VIII.9.1** - A module major version bump follows the deprecation process of [Book IX](../../09-versioning-and-deprecation/README.md); the previous major is supported for the stated sunset period.
* **Cor. VIII.9.2** - A module ships its own tests and a `CHANGELOG.md`; a version without both is not published.

## Construction

* Registry: Terraform Cloud private registry, or S3-backed registry in the shared-services account served through a Terraform-registry-protocol endpoint (open question).
* Module repository per module under a `terraform-aws-*` naming convention, owned by the platform team, released by tag `vMAJOR.MINOR.PATCH` through the same pipeline system ([Post. VIII.4](../postulates.md#Post.%20VIII.4%20-%20One%20Pipeline%20System)).
* Module tests with `terraform test` and, where behaviour matters, Terratest in an ephemeral sandbox account.
* Initial module set: `tags`, `fargate-service`, `lambda-sqs-consumer`, `schedule`, `eventbridge-subscription`, `sqs-queue`, `aurora-cluster`, `dynamodb-table`, `api-gateway-service`, `private-dns-name`.
* Consumption pinned: `source = "app.terraform.io/company/fargate-service/aws"`, `version = "= 2.3.1"`; Renovate or Dependabot opens upgrade pull requests.

## Conformance

Pipeline policy on the Terraform plan JSON rejecting `module` blocks whose `source` is a git or local path outside the service repository's own `modules/` for service-specific glue, and rejecting version constraints other than exact pins; registry CI job `module-semver` that compares input and output variables against the previous release and refuses a minor or patch tag for a breaking change.

## Scholium

The module registry is [Book IV](../../04-shared-schemas/README.md) for infrastructure: one shape per concept, versioned, distributed. The reasoning is the same.
