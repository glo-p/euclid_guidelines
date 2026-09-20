# Prop. VI.7 - No PII or secrets in logs

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A log record (Def. VI.1) MUST NOT contain a secret (Def. V.8) or the value of any field classified `Restricted`, and MUST NOT contain the value of a field classified `Confidential` (Def. V.9) except in a redacted or hashed form. Redaction MUST be performed in the service before emission, driven by the classification tags of Prop. V.5, not by the backend after ingestion.

## Given

Def. VI.1, Def. V.8, Def. V.9, Post. VI.2, Post. I.6, Prop. V.4, Prop. V.5, Prop. VI.1.

## Demonstration

Logs are copied to the backend (Post. VI.2), retained, indexed and readable by many engineers; a secret in a log is disclosed to all of them, which is the harm Prop. V.4 exists to prevent, and a `Restricted` value in a log breaches the duty Def. V.9 attaches to that level. Redaction after ingestion is too late because the value has already left the account boundary. The classification tags (Prop. V.5) give the service a machine-readable list of what to redact, so the check is automatic (Post. I.6) rather than a matter of each developer's judgement at each log call. Structured logs (Prop. VI.1) make field-level redaction possible; free text does not. ∎ Q.E.D.

## Corollaries

* **Cor. VI.7.1** - Request and response bodies are never logged whole; the `traceId` (Prop. VI.2) is the handle for retrieving what happened.
* **Cor. VI.7.2** - Identifiers (Def. I.18) are not PII and are logged freely; they are the intended join key between logs and audit events (Prop. V.9).

## Construction

* Serilog destructuring policy in `Company.Observability.Logging` reading `[DataClassification]` attributes (Prop. V.5) and replacing values with `[REDACTED]` or a salted SHA-256 prefix.
* Roslyn analyser flagging log calls that pass a `Restricted` member or an `HttpRequest` body.
* Datadog Sensitive Data Scanner as a detective control (Post. VI.2), alerting on matches; not the primary mechanism.
* CloudWatch Logs data protection policies on the AWS-side log groups as defence in depth.

## Conformance

Roslyn analyser `COMPANY-OBS-007` in the logging package; unit test in the service template logging a `Restricted`-tagged object and asserting redaction; Datadog Sensitive Data Scanner monitor with zero tolerance.

## Scholium

Audit events (Prop. V.9) are the place for full identity; that is why they are not logs. A team that finds itself needing PII in a log needs an audit event instead.
