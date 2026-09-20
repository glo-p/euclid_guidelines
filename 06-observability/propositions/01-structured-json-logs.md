# Prop. VI.1 - Structured JSON logs with mandatory fields

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every service MUST emit logs as structured JSON (Def. VI.2), one object per record, and every record MUST carry the fields `timestamp` (RFC 3339, UTC), `level`, `traceId`, `spanId`, `service`, `version`, `environment`, `message`, and `tenantId` whenever the record is attributable to a tenant (Def. I.23). Additional fields MUST use OpenTelemetry semantic convention names where one exists.

## Given

Def. VI.1, Def. VI.2, Def. VI.6, Def. I.21, Def. I.23, Post. VI.1, Post. VI.2, CN 7.

## Demonstration

By CN 7 a behaviour that cannot be observed is operationally absent, and a log line that a machine cannot filter is, at the volume of a platform, not observable. Structure (Def. VI.2) is therefore required, not preferred. The mandatory fields are exactly the coordinates needed to locate a record: which service and version (Def. I.1), in which environment (Def. I.21), for which tenant (Def. I.23), and in which trace and span (Def. VI.6). OTel semantic conventions (Post. VI.1) give the additional fields names the backend (Post. VI.2) already understands. ∎ Q.E.D.

## Corollaries

* **Cor. VI.1.1** - A record without `traceId` outside process start-up and shutdown is a defect (Def. VI.6).
* **Cor. VI.1.2** - `level` uses exactly `Trace`, `Debug`, `Information`, `Warning`, `Error`, `Critical`; the backend maps them to its own scale.

## Construction

* `Microsoft.Extensions.Logging.ILogger` as the only logging API in service code.
* Serilog with `Serilog.Formatting.Compact` (or the OTel log bridge) writing JSON to stdout; enrichers for `service`, `version`, `environment` from `OTEL_RESOURCE_ATTRIBUTES` and for `traceId`/`spanId` from `System.Diagnostics.Activity`.
* Tenant enricher fed by `ITenantContext` (Prop. V.7).
* Container logs shipped by the AWS Distro for OpenTelemetry (ADOT) collector sidecar or the Datadog Lambda extension; CloudWatch Logs subscription as the AWS-side path (Post. VI.2).
* Shared package `Company.Observability.Logging` wiring the above (Prop. VI.9).

## Conformance

Architecture test: no direct use of `Console.WriteLine` or `Serilog.Log` static logger. Log schema test in the service template: a sample record validates against the `log-record.schema.json` published in the catalogue. Datadog log pipeline monitor on records missing `traceId`.

## Scholium

`message` is a rendered string; the message template and its named properties are also emitted so that grouping works. Field names are camelCase to match Book II JSON conventions (Prop. II.11).
