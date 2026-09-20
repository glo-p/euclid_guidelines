# Prop. X.4 - Architecture review is triggered by events

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

An architecture review (Def. X.6) MUST be held before any of the following is built or merged: a new service (Def. I.1); a new contract (Def. I.2); a new external dependency (a SaaS, a third-party API, or an AWS service not yet used by the company); a new data store; or a new major version (Def. IX.2) of an existing contract. Architecture reviews MUST NOT be scheduled by calendar, and no other event obliges one. The review outcome MUST be recorded in the service repository or, where it changes a guideline, as an ADR.

## Given

* Def. I.1, Def. I.2, Def. IX.2, Def. X.6
* Post. I.3, Post. X.3
* CN 2, CN 8
* Prop. IX.2, Prop. IX.8

## Demonstration

By CN 8 the architecture is exactly the contracts and the topology connecting them, so architecture changes only when a contract, a node in the topology (a service, a data store, an external dependency) or a major version (Prop. IX.2) is added. Those are the events listed; anything else is implementation, which by Post. I.3 belongs to the team alone. A calendar review either arrives when nothing has changed, wasting the team's autonomy, or after the change is built and owed (CN 2), when it is too late to alter. By Post. X.3 every change passes through a repository, so the trigger can be detected there and the outcome recorded there. ∎ Q.E.D.

## Corollaries

* **Cor. X.4.1** - A new major version is reviewed at the Proposed stage (Def. IX.4) and cannot become Active without the review outcome linked from its catalogue entry (Prop. IX.8).
* **Cor. X.4.2** - A review that finds a guideline would be violated ends either with the design changed or with a proposal (Def. X.2); it never ends with silent deviation.

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
