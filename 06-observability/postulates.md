# Book VI - Postulates

Postulates of [Book VI](README.md) are facts about the observability tooling and operating model of **our** estate. Every proposition of [Book VI](README.md) rests on at least one of them.

---

## Post. VI.1 - OpenTelemetry

OpenTelemetry (OTel) is the instrumentation standard: its API, SDK, semantic conventions and OTLP wire protocol. Services emit logs, metrics and traces through OTel and never through a vendor SDK directly.

## Post. VI.2 - Backend

Datadog is the observability backend in which logs, metrics, traces, SLOs, alerts and dashboards live. Amazon CloudWatch is an AWS-side source (managed service metrics, Lambda and ECS runtime logs, CloudTrail) that is forwarded into Datadog; it is not a second destination for service telemetry. *(To be confirmed; see the [Book VI](README.md) README open questions.)*

## Post. VI.3 - Owned On-call

Every service is on-call owned by its owning team ([Def. I.17](../01-foundations/definitions.md#Def.%20I.17%20-%20Team)). The team that builds a service is paged for it, defines its SLOs, and maintains its runbooks and dashboard. There is no separate operations team that owns alerts.
