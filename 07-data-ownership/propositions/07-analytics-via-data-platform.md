# Prop. VII.7 - Analytics through a governed data platform, never production replicas

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Analytical and regulatory queries MUST run against the governed data platform, a reporting store ([Def. VII.6](../definitions.md#Def.%20VII.6%20-%20Reporting%20Store)) fed by integration events or change data capture from each system of record. No analytical workload MAY query a service database or a replica ([Def. VII.5](../definitions.md#Def.%20VII.5%20-%20Replica)). The default platform is Amazon S3 with AWS Glue Data Catalog and Amazon Athena.

## Given

* [Def. VII.5](../definitions.md#Def.%20VII.5%20-%20Replica), [Def. VII.6](../definitions.md#Def.%20VII.6%20-%20Reporting%20Store)
* [Post. I.1](../../01-foundations/postulates.md#Post.%20I.1%20-%20Cloud), [Post. I.3](../../01-foundations/postulates.md#Post.%20I.3%20-%20Autonomy), [Post. VII.2](../postulates.md#Post.%20VII.2%20-%20Private%20Database), [Post. VII.3](../postulates.md#Post.%20VII.3%20-%20Reporting%20Is%20Fed%2C%20Not%20Read)
* [Prop. III.3](../../03-events/propositions/03-integration-vs-domain-events.md), [Prop. V.5](../../05-security/propositions/05-data-classification-in-contracts.md), [Prop. VII.1](01-one-database-per-service.md), [Prop. VII.5](05-retention-and-deletion.md)
* [CN 1](../../01-foundations/common-notions.md#CN%201%20-%20Substitutability)

## Demonstration

[Post. VII.3](../postulates.md#Post.%20VII.3%20-%20Reporting%20Is%20Fed%2C%20Not%20Read) states that reporting is fed, not read, and [Prop. VII.1](01-one-database-per-service.md) makes a direct read of a database or replica impossible by IAM; therefore the only lawful input to analytics is an event stream or change data capture governed by the data owner. A replica ([Def. VII.5](../definitions.md#Def.%20VII.5%20-%20Replica)) is the same private state as the primary ([Post. VII.2](../postulates.md#Post.%20VII.2%20-%20Private%20Database)), so it is not an exception. Because analysts span every bounded context, their store is the one place where data from many owners meets, which is exactly the definition of a reporting store ([Def. VII.6](../definitions.md#Def.%20VII.6%20-%20Reporting%20Store)), and it must carry the classification of [Prop. V.5](../../05-security/propositions/05-data-classification-in-contracts.md) and the retention of [Prop. VII.5](05-retention-and-deletion.md). By [Post. I.1](../../01-foundations/postulates.md#Post.%20I.1%20-%20Cloud) the managed AWS services are preferred, which fixes the default platform. ∎ Q.E.D.

## Corollaries

* **Cor. VII.7.1** - A service's database schema may change at any time without notice to analysts; only the events and CDC contracts are owed ([CN 1](../../01-foundations/common-notions.md#CN%201%20-%20Substitutability), [CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed)).
* **Cor. VII.7.2** - The data platform is itself a consumer ([Def. I.4](../../01-foundations/definitions.md#Def.%20I.4%20-%20Consumer)) and is bound by the consumer rules of [Book III](../../03-events/README.md).

## Construction

* Landing: EventBridge rule per environment routing all integration events to Amazon Data Firehose, writing partitioned Parquet to an S3 "raw" bucket in the data platform account.
* CDC where events are insufficient: AWS DMS or Aurora zero-ETL to S3, configured by the data owner, tagged with classification.
* Catalogue: AWS Glue Data Catalog with Glue crawlers or explicit table definitions generated from the shared schemas ([Prop. IV.1](../../04-shared-schemas/propositions/01-catalogue-and-distribution.md)); AWS Lake Formation for column-level access by classification tag ([Prop. V.5](../../05-security/propositions/05-data-classification-in-contracts.md)).
* Query: Amazon Athena workgroups per team with per-query data-scanned limits; Glue jobs or dbt for curated layers.
* Lineage and retention: S3 lifecycle per classification; Lake Formation tags mirror `x-classification`.

> **Open question.** Whether Amazon Redshift (Serverless) is added as a curated warehouse
> layer above S3/Athena, and at what data volume or latency requirement that becomes the
> default.

## Conformance

AWS Config rule that no RDS or Aurora instance outside the data platform account has a security group admitting the data platform's VPC or Athena federated connectors; Lake Formation audit showing every table carries a classification tag.

## Scholium

Read replicas look free. They are not: every query against one is an undocumented contract on a schema the owning team believes it may change.
