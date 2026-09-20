# Prop. VII.1 - One database per service

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every service (Def. I.1) MUST own a database that no other service can reach. Cross-service access to a database, its replicas, backups or snapshots MUST be prevented by network placement and by IAM, not by convention. A service MUST NOT hold credentials for another service's store.

## Given

* Def. I.1, Def. I.2, Def. VII.5
* Post. I.3, Post. I.6, Post. VII.2
* CN 1, CN 8

## Demonstration

By Post. I.3 teams interact only through contracts, and by Post. VII.2 a database is not a contract; therefore a second service reading a database is relying on something outside every contract, which by CN 1 destroys substitutability of the owning service. By Post. I.6 a rule enforced only by convention is broken within a year, so the prohibition must be enforced by a mechanism that fails closed: network reachability and IAM. Replicas and backups (Def. VII.5) are copies of the same private state and fall under the same rule. By CN 8 what is not in a contract is not architecture, so the database schema is never part of the architecture between teams. ∎ Q.E.D.

## Corollaries

* **Cor. VII.1.1** - A shared "reporting user" or "read-only login" on a service database is a cross-service access and is prohibited; reporting is fed per Prop. VII.7.
* **Cor. VII.1.2** - Two services owned by the same team are still two services; they do not share a database.

## Construction

* One RDS, Aurora cluster or DynamoDB table set per service, in the service's own AWS account (Prop. VIII.2) and private subnets (Prop. VIII.4).
* Security group on the database that admits only the service's compute security group.
* IAM database authentication (RDS) or IAM resource policies (DynamoDB) scoped to the service's task or execution role; no shared master credentials in application config.
* Credentials, where unavoidable, in AWS Secrets Manager under a path owned by the service (Prop. V.4), with a resource policy denying other principals.
* Aurora snapshots and backups: no cross-account sharing except to the backup vault account under AWS Backup.

## Conformance

AWS Config managed rules `rds-instance-public-access-check`, `rds-snapshots-public-prohibited` and a custom Config rule asserting that each database security group admits exactly one compute security group; IAM Access Analyzer findings for cross-account database access are treated as failures in the account's compliance dashboard.

## Scholium

The temptation is always the same: a report is needed by Friday and the data is "right there". The cost is paid later, when the owning team cannot change a column because an unknown consumer reads it. The rule is enforced by IAM precisely so that Friday cannot win.
