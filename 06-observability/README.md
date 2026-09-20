# Book VI - Observability

Book VI covers logs, metrics, traces, correlation, health, SLOs, alerting and dashboards. It rests on Book I, cites the header rules of Book II, the envelope of Book III and the classification rules of Book V, and is cited by Book V (audit) and Book VIII (infrastructure).

**Depth:** Scaffold. Every proposition is `Draft`. The statements are settled enough to build against; the constructions name building blocks rather than give code.

## Definitions

| Item | Summary |
|---|---|
| `Def. VI.1` Log | Timestamped record of a discrete occurrence, retained for a bounded period. |
| `Def. VI.2` Structured Log | One JSON object per record with stable field names; the message is one field. |
| `Def. VI.3` Metric | Named numeric measurement over time with low-cardinality attributes. |
| `Def. VI.4` Trace | Record of one activity across services, composed of spans sharing a trace-id. |
| `Def. VI.5` Span | Named, timed unit of work within a trace, with parent, attributes and status. |
| `Def. VI.6` Correlation | Refines Def. I.22: trace-id joins every record; a record without it is absent. |
| `Def. VI.7` SLI | Ratio of good to total events for one aspect of a contract over a window. |
| `Def. VI.8` SLO | Target for an SLI over a window, declared by the owning team per contract. |
| `Def. VI.9` Error Budget | Complement of an SLO: the bad events allowed before the objective is missed. |
| `Def. VI.10` Health | A service's own machine-readable assessment of its ability to work. |
| `Def. VI.11` Readiness | Can process requests now; not ready means no traffic, not restart. |
| `Def. VI.12` Liveness | Process is running and recoverable; never depends on dependencies. |
| `Def. VI.13` Alert | Machine-raised notification requiring human action now; otherwise noise. |
| `Def. VI.14` Runbook | Procedure linked from an alert: meaning, confirmation, first action. |
| `Def. VI.15` Dashboard | Persistent shared view answering "healthy, and if not, where" in under a minute. |

See [`definitions.md`](definitions.md).

## Postulates

| Item | Summary |
|---|---|
| `Post. VI.1` OpenTelemetry | OTel API, SDK, semantic conventions and OTLP are the instrumentation standard. |
| `Post. VI.2` Backend | Datadog is the backend; CloudWatch is an AWS-side source forwarded into it. To be confirmed. |
| `Post. VI.3` Owned On-call | Every service is on-call owned by its owning team (Def. I.17). |

See [`postulates.md`](postulates.md).

## Propositions

| Item | Level | Summary |
|---|---|---|
| [`Prop. VI.1`](propositions/01-structured-json-logs.md) Structured JSON logs with mandatory fields | MUST | One JSON object per record with timestamp, level, traceId, spanId, service, version, environment, tenantId, message. |
| [`Prop. VI.2`](propositions/02-traceparent-propagation.md) `traceparent` is propagated across HTTP and events | MUST | Accept, forward and carry W3C trace context on every request and every message. |
| [`Prop. VI.3`](propositions/03-red-metrics.md) RED metrics per operation and per consumer | MUST | Rate, errors, duration by operationId and event type under standard OTel names. |
| [`Prop. VI.4`](propositions/04-slos-per-contract.md) Every service declares SLOs for each published contract | SHOULD | Availability, latency and freshness SLOs in `slo.yaml` with an error budget policy. |
| [`Prop. VI.5`](propositions/05-health-and-readiness-endpoints.md) Health, readiness and liveness endpoints with a standard shape | MUST | `/health/live` and `/health/ready` per Prop. II.16 with a standard JSON body. |
| [`Prop. VI.6`](propositions/06-alerts-on-symptoms-with-runbooks.md) Alerts page on symptoms, not causes, and every alert links a runbook | SHOULD | Page on burn rate, lag or readiness; every alert carries a runbook URL. |
| [`Prop. VI.7`](propositions/07-no-pii-or-secrets-in-logs.md) No PII or secrets in logs | MUST | Redact in the service, driven by classification tags, before emission. |
| [`Prop. VI.8`](propositions/08-dashboard-per-service.md) One dashboard per service, generated from a standard template | SHOULD | Primary dashboard generated from contracts and `slo.yaml`; never hand-edited. |
| [`Prop. VI.9`](propositions/09-dotnet-construction.md) .NET construction of the observability stack | SHOULD | `Company.Observability` package: OTel SDK, Serilog JSON, ADOT collector, Datadog exporter. Q.E.F. |

## Open questions

Decisions the principal engineers still need to make before any proposition here moves from `Draft` to `Accepted`:

1. **Backend confirmation** (Post. VI.2). Confirm Datadog as the single backend and CloudWatch as source only, or record a dual-destination model. Cost of log ingestion volume is the deciding input.
2. **Collector topology** (Prop. VI.9). ADOT sidecar per task versus a central collector service per account; Datadog Agent versus ADOT for ECS.
3. **Log retention per classification** (Prop. VI.7). Retention days for `Internal` logs in Datadog and in CloudWatch, and whether any log group needs longer retention than the default.
4. **Sampling** (Prop. VI.2). Head-based versus tail-based trace sampling, the default rate, and whether errors are always kept.
5. **Tenant claim name** (Prop. VI.1). Shared with Book V open question 3; `tenantId` in logs must match the claim.
6. **X-Ray interoperability** (Prop. VI.2). Whether API Gateway and Lambda `X-Amzn-Trace-Id` is mapped in the collector or ignored.
7. **Health body schema** (Prop. VI.5). Resolved for health: Prop. IV.11 (`health.json`). Still open: whether the structured log record shape (Prop. VI.1) also becomes a Book IV schema.
8. **Burn-rate thresholds and windows** (Prop. VI.6). Proposed 2 % and 5 % per hour over 1 h / 6 h windows; needs the principals' agreement.
9. **Paging tool** (Prop. VI.6). Datadog On-Call versus PagerDuty.
10. **Promotion of SHOULDs** (Prop. VI.4, VI.6, VI.8). Conditions under which SLOs, symptom-based alerting and the generated dashboard become MUST.
