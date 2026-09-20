# Prop. VI.5 - Health, readiness and liveness endpoints with a standard shape

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every service MUST expose `GET /health/live` and `GET /health/ready` as specified by [Prop. II.16](../../02-api-guidelines/propositions/16-health-endpoints.md), returning `200` or `503` with a JSON body of the standard shape: `status` (`pass` | `fail` | `warn`), `version`, `checks` (a map of named checks each with `status`, `observedValue` where numeric, and `time`). Liveness ([Def. VI.12](../definitions.md#Def.%20VI.12%20-%20Liveness)) MUST NOT depend on any external dependency; readiness ([Def. VI.11](../definitions.md#Def.%20VI.11%20-%20Readiness)) MUST reflect every dependency the service cannot operate without.

## Given

[Def. VI.10](../definitions.md#Def.%20VI.10%20-%20Health), [Def. VI.11](../definitions.md#Def.%20VI.11%20-%20Readiness), [Def. VI.12](../definitions.md#Def.%20VI.12%20-%20Liveness), [Post. I.1](../../01-foundations/postulates.md#Post.%20I.1%20-%20Cloud), [Post. I.4](../../01-foundations/postulates.md#Post.%20I.4%20-%20Unreliable%20Network), [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape), [Prop. II.16](../../02-api-guidelines/propositions/16-health-endpoints.md).

## Demonstration

The platform ([Post. I.1](../../01-foundations/postulates.md#Post.%20I.1%20-%20Cloud)) routes traffic and restarts instances based on what the service says about itself ([Def. VI.10](../definitions.md#Def.%20VI.10%20-%20Health)); the two decisions have different consequences, so they need different signals ([Def. VI.11](../definitions.md#Def.%20VI.11%20-%20Readiness), [Def. VI.12](../definitions.md#Def.%20VI.12%20-%20Liveness)). A liveness check that consults a dependency turns a dependency outage ([Post. I.4](../../01-foundations/postulates.md#Post.%20I.4%20-%20Unreliable%20Network)) into a restart storm, so liveness is internal only. One shape for every service is [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape) applied to the health body, and lets the platform, the dashboard template ([Prop. VI.8](08-dashboard-per-service.md)) and the edge ([Prop. II.16](../../02-api-guidelines/propositions/16-health-endpoints.md)) read every service the same way. ∎ Q.E.D.

## Corollaries

* **Cor. VI.5.1** - Health endpoints are unauthenticated at the edge but not exposed publicly; they are reachable by the load balancer, orchestrator and monitoring only.
* **Cor. VI.5.2** - `checks` names are stable and appear as attributes on a `service.health.check` metric so that readiness flaps are observable.

## Construction

* `Microsoft.Extensions.Diagnostics.HealthChecks` with tags `live` and `ready`; `AspNetCore.HealthChecks.*` packages for RDS, DynamoDB, SQS, Secrets Manager.
* Custom `IHealthCheckPublisher` and response writer in `Company.Observability.Health` emitting the standard body (shape proposed for [Book IV](../../04-shared-schemas/README.md) as a shared schema).
* ECS task definition and ALB target group health checks on `/health/ready`; ECS container `healthCheck` on `/health/live`.
* Datadog synthetic or Datadog Agent HTTP check on `/health/ready` per environment.

## Conformance

OpenAPI linter: both paths present with the standard response schema ([Prop. II.16](../../02-api-guidelines/propositions/16-health-endpoints.md)). Architecture test: no health check tagged `live` registers a dependency client. Service template integration test: `/health/live` returns `200` with dependencies stopped.

## Scholium

The body follows the IETF draft `health+json` shape without claiming the media type until it is standardised. `warn` is for degraded-but-serving states such as a stale cache.
