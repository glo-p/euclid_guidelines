# Prop. VI.3 - RED metrics per operation and per consumer

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every service MUST emit, for each API operation and for each event consumer, the three RED metrics with the standard names: request rate, error rate and duration. Operations MUST be identified by the OpenAPI `operationId` and consumers by the event type (Prop. III.2), never by raw path or queue URL. Attributes MUST be low-cardinality: `service`, `version`, `environment`, `operation` or `event_type`, `outcome`.

## Given

Def. VI.3, Def. VI.7, Def. I.2, Post. VI.1, Post. VI.2, CN 7, Prop. II.1, Prop. III.2.

## Demonstration

By CN 7 a contract whose traffic is not measured cannot be operated, and the SLIs of Def. VI.7 are ratios that require exactly a count, an error count and a duration distribution. Naming by `operationId` and event type ties the metric to the contract (Def. I.2, Prop. II.1, Prop. III.2), which is stable, rather than to a path or queue URL, which is an implementation detail and unbounded in cardinality. Standard names across services let one dashboard template (Prop. VI.8) and one SLO definition (Prop. VI.4) apply to every service. OTel (Post. VI.1) provides the instruments and the names. ∎ Q.E.D.

## Corollaries

* **Cor. VI.3.1** - Standard names: `http.server.request.duration` (histogram, seconds) with `http.response.status_code`, and `messaging.process.duration` (histogram) with `messaging.destination.name` and `outcome`; rate and error rate are derived from the histogram count. No service invents a parallel set.
* **Cor. VI.3.2** - Consumer lag (`messaging.sqs.approximate_age_of_oldest_message`) is a fourth mandatory metric for every consumer queue.

## Construction

* `OpenTelemetry.Instrumentation.AspNetCore` HTTP server metrics with `operationId` enrichment from endpoint metadata.
* `System.Diagnostics.Metrics.Meter` per service with histograms for consumer processing in the SQS consumer host (Prop. III.14).
* CloudWatch `AWS/SQS` and `AWS/ApiGateway` namespaces forwarded via the Datadog AWS integration (Post. VI.2).
* Histogram buckets fixed in `Company.Observability.Metrics` so that percentiles are comparable across services.

## Conformance

Architecture test: every endpoint has an `operationId`; every consumer registration uses the shared host. Datadog monitor: a service with a deployed version but no `http.server.request.duration` series within 15 minutes.

## Scholium

USE metrics (utilisation, saturation, errors) for compute are supplied by the platform (ECS, Lambda, RDS) and are not each team's job to emit; they are each team's job to watch.
