# Prop. VIII.1 - Everything is infrastructure as code

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every AWS resource in every environment ([Def. VIII.4](../definitions.md#Def.%20VIII.4%20-%20Environment%20%28refines%20Def.%20I.21%29)) MUST be declared as infrastructure as code ([Def. VIII.6](../definitions.md#Def.%20VIII.6%20-%20Infrastructure%20as%20Code)) and applied only by a pipeline ([Def. VIII.8](../definitions.md#Def.%20VIII.8%20-%20Pipeline)). Changes made through the console, CLI or SDK outside a sandbox account are prohibited, MUST be detected by drift detection, and MUST be reverted or codified within one working day.

## Given

* [Def. VIII.4](../definitions.md#Def.%20VIII.4%20-%20Environment%20%28refines%20Def.%20I.21%29), [Def. VIII.6](../definitions.md#Def.%20VIII.6%20-%20Infrastructure%20as%20Code), [Def. VIII.8](../definitions.md#Def.%20VIII.8%20-%20Pipeline), [Def. VIII.9](../definitions.md#Def.%20VIII.9%20-%20Artefact)
* [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification), [Post. VIII.3](../postulates.md#Post.%20VIII.3%20-%20Terraform), [Post. VIII.4](../postulates.md#Post.%20VIII.4%20-%20One%20Pipeline%20System)
* [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts)

## Demonstration

By [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts) the architecture is what is recorded; a resource created by hand is recorded nowhere and is therefore, by [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), not part of the platform even though it runs. The only way for every environment to be a faithful copy ([Def. VIII.4](../definitions.md#Def.%20VIII.4%20-%20Environment%20%28refines%20Def.%20I.21%29)) is for each to be produced from the same source by the same tool ([Post. VIII.3](../postulates.md#Post.%20VIII.3%20-%20Terraform)) through the same pipeline ([Post. VIII.4](../postulates.md#Post.%20VIII.4%20-%20One%20Pipeline%20System)). [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification) tells us a hand change will happen regardless of the rule, so detection must be mechanical: the tool's plan against the live estate reveals the difference. Sandbox accounts are excluded because they are, by construction, not an environment. ∎ Q.E.D.

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
