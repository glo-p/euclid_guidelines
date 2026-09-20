# Prop. VII.7 - Analytics through a governed data platform, never production replicas

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Analytical and regulatory queries MUST run against the governed data platform, a reporting store (Def. VII.6) fed by integration events or change data capture from each system of record. No analytical workload MAY query a service database or a replica (Def. VII.5). The default platform is Amazon S3 with AWS Glue Data Catalog and Amazon Athena.

## Given

* Def. VII.5, Def. VII.6
* Post. I.1, Post. I.3, Post. VII.2, Post. VII.3
* Prop. III.3, Prop. V.5, Prop. VII.1, Prop. VII.5
* CN 1

## Demonstration

Post. VII.3 states that reporting is fed, not read, and Prop. VII.1 makes a direct read of a database or replica impossible by IAM; therefore the only lawful input to analytics is an event stream or change data capture governed by the data owner. A replica (Def. VII.5) is the same private state as the primary (Post. VII.2), so it is not an exception. Because analysts span every bounded context, their store is the one place where data from many owners meets, which is exactly the definition of a reporting store (Def. VII.6), and it must carry the classification of Prop. V.5 and the retention of Prop. VII.5. By Post. I.1 the managed AWS services are preferred, which fixes the default platform. ∎ Q.E.D.

## Corollaries

* **Cor. VII.7.1** - A service's database schema may change at any time without notice to analysts; only the events and CDC contracts are owed (CN 1, CN 2).
* **Cor. VII.7.2** - The data platform is itself a consumer (Def. I.4) and is bound by the consumer rules of Book III.

## Construction

* Landing: EventBridge rule per environment routing all integration events to Amazon Data Firehose, writing partitioned Parquet to an S3 "raw" bucket in the data platform account.
* CDC where events are insufficient: AWS DMS or Aurora zero-ETL to S3, configured by the data owner, tagged with classification.
* Catalogue: AWS Glue Data Catalog with Glue crawlers or explicit table definitions generated from the shared schemas (Prop. IV.1); AWS Lake Formation for column-level access by classification tag (Prop. V.5).
* Query: Amazon Athena workgroups per team with per-query data-scanned limits; Glue jobs or dbt for curated layers.
* Lineage and retention: S3 lifecycle per classification; Lake Formation tags mirror `x-classification`.

> **Open question.** Whether Amazon Redshift (Serverless) is added as a curated warehouse
> layer above S3/Athena, and at what data volume or latency requirement that becomes the
> default.

## Conformance

AWS Config rule that no RDS or Aurora instance outside the data platform account has a security group admitting the data platform's VPC or Athena federated connectors; Lake Formation audit showing every table carries a classification tag.

## Scholium

Read replicas look free. They are not: every query against one is an undocumented contract on a schema the owning team believes it may change.
