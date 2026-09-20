# Prop. IV.11 - Health is a `Health` report

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

The body of `/health/live` and `/health/ready` ([Prop. II.16](../../02-api-guidelines/propositions/16-health-endpoints.md), [VI.5](../../06-observability/propositions/05-health-and-readiness-endpoints.md)) MUST conform to [`health.json`](../schemas/shared/v1/health.json): `status` (closed: `healthy`, `degraded`, `unhealthy`), `service`, `version`, `checkedAt`, and `checks[]` of `{ name, status, critical, durationMs, description? }`. `unhealthy` MUST be served with `503`; `healthy` and `degraded` with `200`. A `description` MUST NOT contain hosts, connection strings or secrets.

## Given

[CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape), [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated), [Prop. II.16](../../02-api-guidelines/propositions/16-health-endpoints.md), [VI.5](../../06-observability/propositions/05-health-and-readiness-endpoints.md), [VI.7](../../06-observability/propositions/07-no-pii-or-secrets-in-logs.md).

## Demonstration

Orchestrators read the status code; humans and dashboards read the body. One body shape ([CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape)) lets a single dashboard template ([Prop. VI.8](../../06-observability/propositions/08-dashboard-per-service.md)) render every service, and lets `degraded` be surfaced ([CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated)) without failing readiness. The exclusion of connection details follows from [Prop. VI.7](../../06-observability/propositions/07-no-pii-or-secrets-in-logs.md) applied to a response that is, by design, unauthenticated. ∎ Q.E.D.

## Corollaries

* **Cor. IV.11.1** - `version` is the same string the service reports in its logs (`service.version`, [Prop. VI.1](../../06-observability/propositions/01-structured-json-logs.md)) and in `conformance.json` ([Prop. X.3](../../10-governance/propositions/03-conformance-manifest.md)).

## Construction

`Company.Platform.AspNetCore` maps `Microsoft.Extensions.Diagnostics.HealthChecks` results to this shape with a custom `ResponseWriter`.

## Conformance

Template test asserts the body validates against the schema.
