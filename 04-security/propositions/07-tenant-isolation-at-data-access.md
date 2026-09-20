# Prop. V.7 - Tenant isolation is enforced at the data access layer

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A multi-tenant service MUST take the tenant identifier (Def. I.23) exclusively from the validated access token (Def. V.3) or, for a message, from the envelope written by a producer that did so. A service MUST NOT accept a tenant identifier from a request body, query string, path or header. Every query and every write against tenant-scoped data MUST be constrained by that tenant identifier in the data access layer, not in individual handlers.

## Given

Def. I.23, Def. V.3, Def. V.7, Post. V.5, Post. I.6, Prop. V.1, Prop. III.1.

## Demonstration

Post. V.5 says any value a client sends may be forged; the tenant claim inside a token is the one value the client cannot forge, because it is signed by the identity provider and checked at the edge (Prop. V.1). Tenant isolation (Def. V.7) must hold regardless of the request constructed, so it cannot depend on each handler remembering a filter, which Post. I.6 says will be forgotten within a year. The single place every read and write passes is the data access layer, so the constraint lives there. Events carry the tenant in the envelope (Prop. III.1) and a consumer treats that as the token equivalent. ∎ Q.E.D.

## Corollaries

* **Cor. V.7.1** - A request whose token carries no tenant against a tenant-scoped resource is rejected with `403`, never defaulted.
* **Cor. V.7.2** - Cross-tenant operations (platform administration, reporting) are distinct operations with their own scope, and are audited (Prop. V.9).

## Construction

* `ITenantContext` populated from the `tid` (or agreed) claim by ASP.NET Core middleware and from the CloudEvents envelope by the SQS consumer host (Prop. III.14).
* EF Core global query filters keyed on `ITenantContext`, plus `SaveChanges` interceptor stamping the tenant on inserts and rejecting mismatches.
* DynamoDB: tenant identifier as partition key prefix; IAM condition `dynamodb:LeadingKeys` where per-tenant credentials are used.
* PostgreSQL row-level security with `SET app.tenant_id` per connection as defence in depth.
* Shared package `Company.MultiTenancy` providing the middleware, filters and interceptor.

## Conformance

Architecture test: no request DTO, route or query parameter named `tenantId` or equivalent. Integration test in the service template: a request for tenant A against a resource of tenant B returns `404`/`403`. Roslyn analyser: every `DbContext` has the tenant filter registered.

## Scholium

Single-tenant services are not exempt from the token rule; they simply have one tenant. The filter costs nothing and prevents the day the service becomes multi-tenant from being a security incident.
