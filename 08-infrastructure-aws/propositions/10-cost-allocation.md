# Prop. VIII.10 - Cost allocation per service via tags and budgets with alerts

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

The cost of every service MUST be attributable from billing data by its `service`, `team` and `cost-centre` tags ([Prop. VIII.6](06-mandatory-tags.md)). Every team account MUST have an AWS Budget per environment, and every service SHOULD have one, with alerts at forecast thresholds delivered to the owning team. Unattributed spend MUST be reported and driven to zero.

## Given

* [Def. I.17](../../01-foundations/definitions.md#Def.%20I.17%20-%20Team), [Def. VIII.1](../definitions.md#Def.%20VIII.1%20-%20AWS%20Account), [Def. VIII.13](../definitions.md#Def.%20VIII.13%20-%20Tag)
* [Post. I.3](../../01-foundations/postulates.md#Post.%20I.3%20-%20Autonomy), [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification), [Post. VIII.2](../postulates.md#Post.%20VIII.2%20-%20Account%20per%20Team%20per%20Environment)
* [Prop. VIII.6](06-mandatory-tags.md)
* [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated)

## Demonstration

Teams own their services end to end ([Post. I.3](../../01-foundations/postulates.md#Post.%20I.3%20-%20Autonomy), [Def. I.17](../../01-foundations/definitions.md#Def.%20I.17%20-%20Team)), and cost is part of what is owned; a cost that cannot be attributed to a team is owned by no one. Accounts ([Post. VIII.2](../postulates.md#Post.%20VIII.2%20-%20Account%20per%20Team%20per%20Environment)) attribute cost to a team; tags ([Prop. VIII.6](06-mandatory-tags.md)) attribute it to a service within the account, and both are already required, so attribution is a query rather than a new obligation. By [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated) a cost with no alert is a cost nobody operates, and by [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification) the alert must be mechanical: a budget with a threshold. Unattributed spend is the measure of tag non-compliance and is therefore reported as such. ∎ Q.E.D.

## Corollaries

* **Cor. VIII.10.1** - Shared platform costs (central bus, egress, landing zone) are allocated to the platform cost centre, not spread across teams by formula.
* **Cor. VIII.10.2** - A budget alert is a page-free notification; exceeding a budget is a review trigger, not an incident.

## Construction

* Cost allocation tags activated for `service`, `team`, `cost-centre`; AWS Cost Categories mapping accounts and tags to teams and cost centres.
* AWS Budgets per account per environment and per service (tag-filtered), with `FORECASTED` alerts at 80 and 100 per cent to an SNS topic subscribed by the team's channel; created by the Terraform module `budget`.
* AWS Cost and Usage Report (CUR 2.0) delivered to the data platform S3 bucket ([Prop. VII.7](../../07-data-ownership/propositions/07-analytics-via-data-platform.md)) and queried with Athena for the unattributed-spend report.
* AWS Cost Anomaly Detection monitors per cost category.
* Standard dashboard panel ([Prop. VI.8](../../06-observability/propositions/08-dashboard-per-service.md)) showing cost per service alongside RED metrics.

## Conformance

Terraform module `budget` required by the account baseline; scheduled Athena query `unattributed-spend` publishing a CloudWatch metric with an alarm above a stated percentage of total spend.

## Scholium

Cost is a non-functional requirement that arrives monthly and in arrears. Budgets turn it into a signal that arrives in time to act.
