# Prop. VII.8 - A tenant's data lives in the tenant's home region

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every tenant (Def. I.23) MUST be homed in exactly one AWS region. Data attributable to that tenant, in the system of record, every read model, the reporting store, the event archive and backups, MUST be stored and processed only in that region or in a region the tenant's residency policy (Def. VII.10) explicitly permits. Cross-region replication of tenant data MUST be a recorded exception (Def. I.20).

## Given

* Def. I.20, Def. I.23, Def. VII.9, Def. VII.10
* Post. I.1, Post. I.6
* Prop. III.9, Prop. V.7, Prop. VII.2, Prop. VII.5, Prop. VIII.2
* CN 7

## Demonstration

By Def. I.23 every resource, message and log line is attributable to a tenant, so the question "where may this byte live" has an answer for every byte. Residency (Def. VII.10) is a property of the tenant, and Prop. V.7 already isolates data access by tenant, so the home region is the natural key on which isolation is extended from "which rows" to "which region". Copies are enumerable by Prop. VII.2 and governed by Prop. VII.5, so the constraint can be applied to each. Because the bus topology (Prop. III.9) and account topology (Prop. VIII.2) are per region, events do not cross regions unless routed to, which by Post. I.6 must be blocked by policy rather than by care. ∎ Q.E.D.

## Corollaries

* **Cor. VII.8.1** - The tenant's home region is master data (Def. VII.8) owned by the tenant service and carried in the event envelope (Prop. IV.8) so consumers can route.
* **Cor. VII.8.2** - Backups obey residency; AWS Backup vaults are regional and are not copied cross-region for tenants without a permitting policy.

## Construction

* Tenant registry service as system of record for `tenant.homeRegion`; value distributed by integration event and cached as a read model at the edge.
* One platform stack per region (Prop. VIII.2, Prop. VIII.7); API Gateway custom domain with Route 53 latency or geolocation routing pinned per tenant (Prop. VIII.5).
* AWS Organizations SCP denying resource creation outside approved regions; AWS Config conformance pack for region restriction.
* EventBridge rules filter on `tenant.homeRegion` and are not permitted to target cross-region buses without an exception ADR.
* S3 buckets with `LocationConstraint`; DynamoDB global tables and Aurora Global Database prohibited by SCP except in the exception's account.

## Conformance

SCP `deny-unapproved-regions` in the organisation root; AWS Config rule `eventbridge-cross-region-target-prohibited` (custom); pipeline policy check that Terraform plans contain no provider alias for an unapproved region.

## Scholium

Multi-region residency is expensive to retrofit. Deciding the tenant's home region at onboarding, and carrying it in every message, is the cheapest point at which to be correct.
