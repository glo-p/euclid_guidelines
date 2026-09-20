# Prop. VI.9 - .NET construction of the observability stack

| | |
|---|---|
| **Level** | SHOULD |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A .NET service SHOULD obtain the whole of [Prop. VI.1](01-structured-json-logs.md) to [Prop. VI.7](07-no-pii-or-secrets-in-logs.md) by referencing the package `Company.Observability` and calling one registration method, which configures the OpenTelemetry .NET SDK, Serilog JSON logging through `ILogger`, health endpoints and redaction, exporting over OTLP to the AWS Distro for OpenTelemetry collector and from there to Datadog. A service SHOULD NOT reference a vendor observability SDK directly.

## Given

[Post. I.2](../../01-foundations/postulates.md#Post.%20I.2%20-%20Language), [Post. VI.1](../postulates.md#Post.%20VI.1%20-%20OpenTelemetry), [Post. VI.2](../postulates.md#Post.%20VI.2%20-%20Backend), [CN 1](../../01-foundations/common-notions.md#CN%201%20-%20Substitutability), [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape), [Prop. VI.1](01-structured-json-logs.md), [Prop. VI.2](02-traceparent-propagation.md), [Prop. VI.3](03-red-metrics.md), [Prop. VI.5](05-health-and-readiness-endpoints.md), [Prop. VI.7](07-no-pii-or-secrets-in-logs.md), [Prop. II.17](../../02-api-guidelines/propositions/17-dotnet-construction.md).

## Demonstration

The propositions of this book fix one set of signals with one set of names; by [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape) a second implementation of the same wiring in each service is a cost without benefit. [Post. I.2](../../01-foundations/postulates.md#Post.%20I.2%20-%20Language) fixes the language, so a single package can serve every service, in the manner of [Prop. II.17](../../02-api-guidelines/propositions/17-dotnet-construction.md). Emitting through OTel ([Post. VI.1](../postulates.md#Post.%20VI.1%20-%20OpenTelemetry)) and exporting through the collector rather than a vendor SDK means that if [Post. VI.2](../postulates.md#Post.%20VI.2%20-%20Backend) changes, only the collector configuration changes and, by [CN 1](../../01-foundations/common-notions.md#CN%201%20-%20Substitutability), no service can tell. What was to be done is therefore one package and one collector configuration. ∎ Q.E.F.

## Corollaries

* **Cor. VI.9.1** - The package version is pinned by the service template and bumped by Renovate; a service more than one major behind is a finding in [Prop. V.8](../../05-security/propositions/08-scanning-gates-ci.md)'s scan.
* **Cor. VI.9.2** - Lambda services use the same package with the Datadog Lambda extension or ADOT Lambda layer as the collector.

## Construction

* NuGet `Company.Observability` (meta-package) depending on: `OpenTelemetry`, `OpenTelemetry.Extensions.Hosting`, `OpenTelemetry.Instrumentation.AspNetCore`, `OpenTelemetry.Instrumentation.Http`, `OpenTelemetry.Instrumentation.AWS`, `OpenTelemetry.Instrumentation.Runtime`, `OpenTelemetry.Exporter.OpenTelemetryProtocol`, `Npgsql.OpenTelemetry` or the relevant data instrumentation.
* Logging: `Serilog.AspNetCore`, `Serilog.Formatting.Compact`, `Serilog.Enrichers.Span`, plus the redaction destructuring policy ([Prop. VI.7](07-no-pii-or-secrets-in-logs.md)) and tenant enricher ([Prop. V.7](../../05-security/propositions/07-tenant-isolation-at-data-access.md)); `ILogger` remains the only API used by service code.
* Health: `Microsoft.Extensions.Diagnostics.HealthChecks` with the standard response writer ([Prop. VI.5](05-health-and-readiness-endpoints.md)).
* Metrics: shared `Meter` and histogram bucket boundaries ([Prop. VI.3](03-red-metrics.md)).
* Resource attributes from `OTEL_RESOURCE_ATTRIBUTES` and `OTEL_SERVICE_NAME` set by the ECS task definition or Lambda configuration.
* Collector: AWS Distro for OpenTelemetry (ADOT) collector as an ECS sidecar or Lambda layer, with the `datadog` exporter (traces, metrics, logs) and the `awsxray` propagator mapping for AWS-native headers; collector configuration in the `company-adot-collector` Terraform module.
* Datadog AWS integration for CloudWatch metrics and CloudTrail ([Post. VI.2](../postulates.md#Post.%20VI.2%20-%20Backend)).
* Registration: `services.AddCompanyObservability(builder.Configuration)` and `app.UseCompanyObservability()` in the service template.

## Conformance

Architecture test: `Company.Observability` referenced and no direct reference to `Datadog.Trace`, `Serilog.Sinks.Datadog.Logs` or `AWSXRayRecorder`; service template smoke test that a request produces a trace, a log record with `traceId`, and a `http.server.request.duration` sample at the collector.

## Scholium

The Datadog .NET tracer (`dd-trace`) offers automatic instrumentation that the OTel path lacks for some libraries. It was considered and rejected as the primary path because it ties every service to [Post. VI.2](../postulates.md#Post.%20VI.2%20-%20Backend); it remains available as a recorded exception for a service with a specific gap.
