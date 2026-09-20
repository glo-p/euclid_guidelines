# Prop. VIII.10 - Cost allocation per service via tags and budgets with alerts

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

The cost of every service MUST be attributable from billing data by its `service`, `team` and `cost-centre` tags (Prop. VIII.6). Every team account MUST have an AWS Budget per environment, and every service SHOULD have one, with alerts at forecast thresholds delivered to the owning team. Unattributed spend MUST be reported and driven to zero.

## Given

* Def. I.17, Def. VIII.1, Def. VIII.13
* Post. I.3, Post. I.6, Post. VIII.2
* Prop. VIII.6
* CN 7

## Demonstration

Teams own their services end to end (Post. I.3, Def. I.17), and cost is part of what is owned; a cost that cannot be attributed to a team is owned by no one. Accounts (Post. VIII.2) attribute cost to a team; tags (Prop. VIII.6) attribute it to a service within the account, and both are already required, so attribution is a query rather than a new obligation. By CN 7 a cost with no alert is a cost nobody operates, and by Post. I.6 the alert must be mechanical: a budget with a threshold. Unattributed spend is the measure of tag non-compliance and is therefore reported as such. ∎ Q.E.D.

## Corollaries

* **Cor. VIII.10.1** - Shared platform costs (central bus, egress, landing zone) are allocated to the platform cost centre, not spread across teams by formula.
* **Cor. VIII.10.2** - A budget alert is a page-free notification; exceeding a budget is a review trigger, not an incident.

## Construction

* Cost allocation tags activated for `service`, `team`, `cost-centre`; AWS Cost Categories mapping accounts and tags to teams and cost centres.
* AWS Budgets per account per environment and per service (tag-filtered), with `FORECASTED` alerts at 80 and 100 per cent to an SNS topic subscribed by the team's channel; created by the Terraform module `budget`.
* AWS Cost and Usage Report (CUR 2.0) delivered to the data platform S3 bucket (Prop. VII.7) and queried with Athena for the unattributed-spend report.
* AWS Cost Anomaly Detection monitors per cost category.
* Standard dashboard panel (Prop. VI.8) showing cost per service alongside RED metrics.

## Conformance

Terraform module `budget` required by the account baseline; scheduled Athena query `unattributed-spend` publishing a CloudWatch metric with an alarm above a stated percentage of total spend.

## Scholium

Cost is a non-functional requirement that arrives monthly and in arrears. Budgets turn it into a signal that arrives in time to act.
