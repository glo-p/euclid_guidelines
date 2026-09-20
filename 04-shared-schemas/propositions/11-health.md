# Prop. IV.11 - Health is a `Health` report

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

The body of `/health/live` and `/health/ready` (Prop. II.16, VI.5) MUST conform to [`health.json`](../schemas/shared/v1/health.json): `status` (closed: `healthy`, `degraded`, `unhealthy`), `service`, `version`, `checkedAt`, and `checks[]` of `{ name, status, critical, durationMs, description? }`. `unhealthy` MUST be served with `503`; `healthy` and `degraded` with `200`. A `description` MUST NOT contain hosts, connection strings or secrets.

## Given

CN 5, CN 7, Prop. II.16, VI.5, VI.7.

## Demonstration

Orchestrators read the status code; humans and dashboards read the body. One body shape (CN 5) lets a single dashboard template (Prop. VI.8) render every service, and lets `degraded` be surfaced (CN 7) without failing readiness. The exclusion of connection details follows from Prop. VI.7 applied to a response that is, by design, unauthenticated. ∎ Q.E.D.

## Corollaries

* **Cor. IV.11.1** - `version` is the same string the service reports in its logs (`service.version`, Prop. VI.1) and in `conformance.json` (Prop. X.3).

## Construction

`Company.Platform.AspNetCore` maps `Microsoft.Extensions.Diagnostics.HealthChecks` results to this shape with a custom `ResponseWriter`.

## Conformance

Template test asserts the body validates against the schema.
