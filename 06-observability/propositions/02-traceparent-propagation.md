# Prop. VI.2 - `traceparent` is propagated across HTTP and events

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every service MUST accept a W3C `traceparent` (and `tracestate`) header on inbound HTTP requests, MUST send it on every outbound HTTP request, and MUST carry the same trace context in the CloudEvents envelope of every message it publishes and read it from every message it consumes. A service that receives no trace context MUST start a new trace; it MUST NOT discard one it received.

## Given

Def. I.22, Def. VI.4, Def. VI.5, Def. VI.6, Post. I.4, Post. VI.1, Prop. II.10, Prop. III.1.

## Demonstration

Correlation (Def. VI.6) is defined by the propagated trace-id; a trace (Def. VI.4) exists only where every hop carries it. Post. I.4 says cross-process calls fail in ways the caller may not see, so the only way to reconstruct what happened is the trace, and a single hop that drops the header severs everything downstream. HTTP carriage is already fixed by Prop. II.10 and message carriage by the envelope of Prop. III.1; this proposition requires that both are honoured in both directions, which OTel propagators (Post. VI.1) do by default. ∎ Q.E.D.

## Corollaries

* **Cor. VI.2.1** - A consumed message produces a new span whose parent is the producer's span, or a span link where the consumer batches, so that a business activity is one trace across the bus.
* **Cor. VI.2.2** - The trace-id is the value returned to clients in error responses (Prop. II.4) and recorded in audit events (Prop. V.9).

## Construction

* `OpenTelemetry.Instrumentation.AspNetCore` and `OpenTelemetry.Instrumentation.Http` with the W3C `TraceContextPropagator` (default).
* CloudEvents extension attributes `traceparent` and `tracestate` per the CloudEvents Distributed Tracing extension, set by the outbox publisher and read by the SQS consumer host (Prop. III.14).
* `OpenTelemetry.Instrumentation.AWS` for SDK calls; EventBridge and SQS message attributes carrying the same values for AWS-native tooling.
* API Gateway access logs including `$context.requestId` and the forwarded `traceparent`.

## Conformance

Contract test (Prop. III.12): published events carry `traceparent`. Integration test in the service template: an inbound `traceparent` appears unchanged in the outbound call and the log records. Datadog monitor on the ratio of root spans to total spans per service.

## Scholium

AWS X-Ray headers are not the company standard; where an AWS service emits only `X-Amzn-Trace-Id`, the collector's propagator maps it. See open questions.
