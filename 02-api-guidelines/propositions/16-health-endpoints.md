# Prop. II.16 - Every service exposes health and readiness

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Every service MUST expose, outside the versioned base path and outside the edge, `GET /health/live` (process is up; no dependencies checked) and `GET /health/ready` (service can serve traffic; critical dependencies checked with short timeouts). Both return `200` when healthy and `503` otherwise, with a body conforming to the shared `Health` schema (Prop. VI.5). They MUST NOT require authentication and MUST NOT be routed through the edge.

## Given

Post. I.8, Post. II.4, CN 7, Prop. VI.5.

## Demonstration

The orchestrator that gives a service its stable name (Post. I.8) must know whether to send traffic, and by CN 7 that is only possible if the service says so. Liveness and readiness answer different questions (restart me / do not route to me), so they are separate. The orchestrator is inside the network and has no token, hence no authentication and no edge (Post. II.4 names this exception explicitly). ∎ Q.E.D.

## Corollaries

* **Cor. II.16.1** - Readiness checks are cheap and bounded (≤ 2 s total); a slow readiness check is itself an outage.
* **Cor. II.16.2** - Readiness does not check non-critical dependencies; their failure is degraded operation, reported through metrics (Prop. VI.3), not through readiness.

## Construction

`Microsoft.Extensions.Diagnostics.HealthChecks` with tags `live` and `ready`; ECS/ALB target group health check on `/health/ready`; Lambda-hosted services expose the same shape through a dedicated function URL used only by synthetic checks.

## Conformance

Spectral: health paths present and tagged `health`, no `security`. Template test.
