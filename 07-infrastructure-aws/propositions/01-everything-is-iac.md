# Prop. VIII.1 - Everything is infrastructure as code

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every AWS resource in every environment (Def. VIII.4) MUST be declared as infrastructure as code (Def. VIII.6) and applied only by a pipeline (Def. VIII.8). Changes made through the console, CLI or SDK outside a sandbox account are prohibited, MUST be detected by drift detection, and MUST be reverted or codified within one working day.

## Given

* Def. VIII.4, Def. VIII.6, Def. VIII.8, Def. VIII.9
* Post. I.6, Post. VIII.3, Post. VIII.4
* CN 3, CN 8

## Demonstration

By CN 8 the architecture is what is recorded; a resource created by hand is recorded nowhere and is therefore, by CN 3, not part of the platform even though it runs. The only way for every environment to be a faithful copy (Def. VIII.4) is for each to be produced from the same source by the same tool (Post. VIII.3) through the same pipeline (Post. VIII.4). Post. I.6 tells us a hand change will happen regardless of the rule, so detection must be mechanical: the tool's plan against the live estate reveals the difference. Sandbox accounts are excluded because they are, by construction, not an environment. ∎ Q.E.D.

## Corollaries

* **Cor. VIII.1.1** - Human IAM principals in non-sandbox accounts hold read-only permissions plus break-glass roles whose use raises an alert.
* **Cor. VIII.1.2** - A resource that Terraform does not know about is deleted, not adopted, unless an import is committed the same day.

## Construction

* Terraform with remote state in S3 and DynamoDB locking per account; state buckets in the shared-services account with cross-account roles.
* Terraform Cloud/Enterprise or scheduled `terraform plan -detailed-exitcode` in the pipeline as drift detection; non-zero exit opens a ticket and posts to the team's channel.
* AWS Config recorder in every account, with AWS CloudTrail organisation trail; a CloudWatch metric filter on `ConsoleLogin` plus write API calls by non-pipeline principals feeds an alarm.
* IAM Identity Center permission sets: `ReadOnly` by default; `BreakGlass` with session tags and a CloudTrail alert.
* Sandbox OU with a separate SCP allowing console writes and a nightly `aws-nuke` style clean-up.

## Conformance

Scheduled drift job `iac-drift` per account failing the account's compliance status on any plan diff; CloudTrail Lake query for write calls by human principals outside the sandbox OU, reported daily.

## Scholium

The console is an excellent viewer. It is a poor editor, because it leaves no record that the next person can read or the pipeline can reproduce.
