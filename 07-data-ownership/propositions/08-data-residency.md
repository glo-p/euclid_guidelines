# Prop. VII.8 - A tenant's data lives in the tenant's home region

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every tenant ([Def. I.23](../../01-foundations/definitions.md#Def.%20I.23%20-%20Tenant)) MUST be homed in exactly one AWS region. Data attributable to that tenant, in the system of record, every read model, the reporting store, the event archive and backups, MUST be stored and processed only in that region or in a region the tenant's residency policy ([Def. VII.10](../definitions.md#Def.%20VII.10%20-%20Data%20Residency)) explicitly permits. Cross-region replication of tenant data MUST be a recorded exception ([Def. I.20](../../01-foundations/definitions.md#Def.%20I.20%20-%20Exception)).

## Given

* [Def. I.20](../../01-foundations/definitions.md#Def.%20I.20%20-%20Exception), [Def. I.23](../../01-foundations/definitions.md#Def.%20I.23%20-%20Tenant), [Def. VII.9](../definitions.md#Def.%20VII.9%20-%20Retention%20Period), [Def. VII.10](../definitions.md#Def.%20VII.10%20-%20Data%20Residency)
* [Post. I.1](../../01-foundations/postulates.md#Post.%20I.1%20-%20Cloud), [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification)
* [Prop. III.9](../../03-events/propositions/09-topology.md), [Prop. V.7](../../05-security/propositions/07-tenant-isolation-at-data-access.md), [Prop. VII.2](02-system-of-record-in-catalogue.md), [Prop. VII.5](05-retention-and-deletion.md), [Prop. VIII.2](../../08-infrastructure-aws/propositions/02-account-topology-and-central-bus.md)
* [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated)

## Demonstration

By [Def. I.23](../../01-foundations/definitions.md#Def.%20I.23%20-%20Tenant) every resource, message and log line is attributable to a tenant, so the question "where may this byte live" has an answer for every byte. Residency ([Def. VII.10](../definitions.md#Def.%20VII.10%20-%20Data%20Residency)) is a property of the tenant, and [Prop. V.7](../../05-security/propositions/07-tenant-isolation-at-data-access.md) already isolates data access by tenant, so the home region is the natural key on which isolation is extended from "which rows" to "which region". Copies are enumerable by [Prop. VII.2](02-system-of-record-in-catalogue.md) and governed by [Prop. VII.5](05-retention-and-deletion.md), so the constraint can be applied to each. Because the bus topology ([Prop. III.9](../../03-events/propositions/09-topology.md)) and account topology ([Prop. VIII.2](../../08-infrastructure-aws/propositions/02-account-topology-and-central-bus.md)) are per region, events do not cross regions unless routed to, which by [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification) must be blocked by policy rather than by care. ∎ Q.E.D.

## Corollaries

* **Cor. VII.8.1** - The tenant's home region is master data ([Def. VII.8](../definitions.md#Def.%20VII.8%20-%20Master%20Data)) owned by the tenant service and carried in the event envelope ([Prop. IV.8](../../04-shared-schemas/propositions/08-event-envelope.md)) so consumers can route.
* **Cor. VII.8.2** - Backups obey residency; AWS Backup vaults are regional and are not copied cross-region for tenants without a permitting policy.

## Construction

* Tenant registry service as system of record for `tenant.homeRegion`; value distributed by integration event and cached as a read model at the edge.
* One platform stack per region ([Prop. VIII.2](../../08-infrastructure-aws/propositions/02-account-topology-and-central-bus.md), [Prop. VIII.7](../../08-infrastructure-aws/propositions/07-identical-environments.md)); API Gateway custom domain with Route 53 latency or geolocation routing pinned per tenant ([Prop. VIII.5](../../08-infrastructure-aws/propositions/05-api-gateway-single-ingress.md)).
* AWS Organizations SCP denying resource creation outside approved regions; AWS Config conformance pack for region restriction.
* EventBridge rules filter on `tenant.homeRegion` and are not permitted to target cross-region buses without an exception ADR.
* S3 buckets with `LocationConstraint`; DynamoDB global tables and Aurora Global Database prohibited by SCP except in the exception's account.

## Conformance

SCP `deny-unapproved-regions` in the organisation root; AWS Config rule `eventbridge-cross-region-target-prohibited` (custom); pipeline policy check that Terraform plans contain no provider alias for an unapproved region.

## Scholium

Multi-region residency is expensive to retrofit. Deciding the tenant's home region at onboarding, and carrying it in every message, is the cheapest point at which to be correct.
