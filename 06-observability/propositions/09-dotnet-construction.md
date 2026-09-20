# Prop. VI.9 - .NET construction of the observability stack

| | |
|---|---|
| **Level** | SHOULD |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A .NET service SHOULD obtain the whole of Prop. VI.1 to Prop. VI.7 by referencing the package `Company.Observability` and calling one registration method, which configures the OpenTelemetry .NET SDK, Serilog JSON logging through `ILogger`, health endpoints and redaction, exporting over OTLP to the AWS Distro for OpenTelemetry collector and from there to Datadog. A service SHOULD NOT reference a vendor observability SDK directly.

## Given

Post. I.2, Post. VI.1, Post. VI.2, CN 1, CN 5, Prop. VI.1, Prop. VI.2, Prop. VI.3, Prop. VI.5, Prop. VI.7, Prop. II.17.

## Demonstration

The propositions of this book fix one set of signals with one set of names; by CN 5 a second implementation of the same wiring in each service is a cost without benefit. Post. I.2 fixes the language, so a single package can serve every service, in the manner of Prop. II.17. Emitting through OTel (Post. VI.1) and exporting through the collector rather than a vendor SDK means that if Post. VI.2 changes, only the collector configuration changes and, by CN 1, no service can tell. What was to be done is therefore one package and one collector configuration. ∎ Q.E.F.

## Corollaries

* **Cor. VI.9.1** - The package version is pinned by the service template and bumped by Renovate; a service more than one major behind is a finding in Prop. V.8's scan.
* **Cor. VI.9.2** - Lambda services use the same package with the Datadog Lambda extension or ADOT Lambda layer as the collector.

## Construction

* NuGet `Company.Observability` (meta-package) depending on: `OpenTelemetry`, `OpenTelemetry.Extensions.Hosting`, `OpenTelemetry.Instrumentation.AspNetCore`, `OpenTelemetry.Instrumentation.Http`, `OpenTelemetry.Instrumentation.AWS`, `OpenTelemetry.Instrumentation.Runtime`, `OpenTelemetry.Exporter.OpenTelemetryProtocol`, `Npgsql.OpenTelemetry` or the relevant data instrumentation.
* Logging: `Serilog.AspNetCore`, `Serilog.Formatting.Compact`, `Serilog.Enrichers.Span`, plus the redaction destructuring policy (Prop. VI.7) and tenant enricher (Prop. V.7); `ILogger` remains the only API used by service code.
* Health: `Microsoft.Extensions.Diagnostics.HealthChecks` with the standard response writer (Prop. VI.5).
* Metrics: shared `Meter` and histogram bucket boundaries (Prop. VI.3).
* Resource attributes from `OTEL_RESOURCE_ATTRIBUTES` and `OTEL_SERVICE_NAME` set by the ECS task definition or Lambda configuration.
* Collector: AWS Distro for OpenTelemetry (ADOT) collector as an ECS sidecar or Lambda layer, with the `datadog` exporter (traces, metrics, logs) and the `awsxray` propagator mapping for AWS-native headers; collector configuration in the `company-adot-collector` Terraform module.
* Datadog AWS integration for CloudWatch metrics and CloudTrail (Post. VI.2).
* Registration: `services.AddCompanyObservability(builder.Configuration)` and `app.UseCompanyObservability()` in the service template.

## Conformance

Architecture test: `Company.Observability` referenced and no direct reference to `Datadog.Trace`, `Serilog.Sinks.Datadog.Logs` or `AWSXRayRecorder`; service template smoke test that a request produces a trace, a log record with `traceId`, and a `http.server.request.duration` sample at the collector.

## Scholium

The Datadog .NET tracer (`dd-trace`) offers automatic instrumentation that the OTel path lacks for some libraries. It was considered and rejected as the primary path because it ties every service to Post. VI.2; it remains available as a recorded exception for a service with a specific gap.
