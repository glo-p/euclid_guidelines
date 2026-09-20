# Prop. X.4 - Architecture review is triggered by events

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

An architecture review ([Def. X.6](../definitions.md#Def.%20X.6%20-%20Architecture%20Review)) MUST be held before any of the following is built or merged: a new service ([Def. I.1](../../01-foundations/definitions.md#Def.%20I.1%20-%20Service)); a new contract ([Def. I.2](../../01-foundations/definitions.md#Def.%20I.2%20-%20Contract)); a new external dependency (a SaaS, a third-party API, or an AWS service not yet used by the company); a new data store; or a new major version ([Def. IX.2](../../09-versioning-and-deprecation/definitions.md#Def.%20IX.2%20-%20Major%20Version)) of an existing contract. Architecture reviews MUST NOT be scheduled by calendar, and no other event obliges one. The review outcome MUST be recorded in the service repository or, where it changes a guideline, as an ADR.

## Given

* [Def. I.1](../../01-foundations/definitions.md#Def.%20I.1%20-%20Service), [Def. I.2](../../01-foundations/definitions.md#Def.%20I.2%20-%20Contract), [Def. IX.2](../../09-versioning-and-deprecation/definitions.md#Def.%20IX.2%20-%20Major%20Version), [Def. X.6](../definitions.md#Def.%20X.6%20-%20Architecture%20Review)
* [Post. I.3](../../01-foundations/postulates.md#Post.%20I.3%20-%20Autonomy), [Post. X.3](../postulates.md#Post.%20X.3%20-%20Every%20Service%20Has%20a%20Repository)
* [CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed), [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts)
* [Prop. IX.2](../../09-versioning-and-deprecation/propositions/02-breaking-change-is-a-new-major-side-by-side.md), [Prop. IX.8](../../09-versioning-and-deprecation/propositions/08-lifecycle-stage-is-machine-readable.md)

## Demonstration

By [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts) the architecture is exactly the contracts and the topology connecting them, so architecture changes only when a contract, a node in the topology (a service, a data store, an external dependency) or a major version ([Prop. IX.2](../../09-versioning-and-deprecation/propositions/02-breaking-change-is-a-new-major-side-by-side.md)) is added. Those are the events listed; anything else is implementation, which by [Post. I.3](../../01-foundations/postulates.md#Post.%20I.3%20-%20Autonomy) belongs to the team alone. A calendar review either arrives when nothing has changed, wasting the team's autonomy, or after the change is built and owed ([CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed)), when it is too late to alter. By [Post. X.3](../postulates.md#Post.%20X.3%20-%20Every%20Service%20Has%20a%20Repository) every change passes through a repository, so the trigger can be detected there and the outcome recorded there. ∎ Q.E.D.

## Corollaries

* **Cor. X.4.1** - A new major version is reviewed at the Proposed stage ([Def. IX.4](../../09-versioning-and-deprecation/definitions.md#Def.%20IX.4%20-%20Lifecycle%20Stage)) and cannot become Active without the review outcome linked from its catalogue entry ([Prop. IX.8](../../09-versioning-and-deprecation/propositions/08-lifecycle-stage-is-machine-readable.md)).
* **Cor. X.4.2** - A review that finds a guideline would be violated ends either with the design changed or with a proposal ([Def. X.2](../definitions.md#Def.%20X.2%20-%20Proposal%20%28RFC%29)); it never ends with silent deviation.

## Construction

* Repository template check: a new repository created from the service template opens a review issue automatically (`.github/ISSUE_TEMPLATE/architecture-review.md`).
* Catalogue CI: a new `contracts/<name>/` directory or a new entry in `majors[]` at stage `proposed` requires a `review` link before stage `active` is accepted.
* Terraform/CDK policy (OPA / `cdk-nag` custom rule): a new resource type not in the approved service list, or a new RDS, DynamoDB, OpenSearch, S3 data bucket, or third-party provider block, fails `plan` without a `review_id` tag.
* Review record: `docs/reviews/REVIEW-<date>-<topic>.md` in the service repository with attendees, guidelines checked, findings; linked from the conformance manifest.
* Trigger event `company.platform.architecture-review-requested.v1` raised by the above checks for the principal engineer and guild inboxes.

## Conformance

Catalogue lint (`review` link required for `active`); IaC policy check for new resource types and providers; repository template audit.

## Scholium

The list of triggering events is closed on purpose. If a team feels a review is needed for something else, it may ask for one; nothing forbids a voluntary review. What is forbidden is an obligatory one that the guidelines do not name.
