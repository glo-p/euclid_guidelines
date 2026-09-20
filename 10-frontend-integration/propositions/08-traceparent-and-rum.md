# Prop. XI.8 - Front-ends originate traceparent and report client telemetry

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A front-end MUST originate a W3C `traceparent` header for every request it makes and MUST propagate the same trace-id across all requests belonging to one user action (Prop. VI.2). A front-end SHOULD report client-side telemetry (page and view timings, errors, failed requests) through a real user monitoring (RUM) agent that carries the same trace-id.

## Given

* Def. I.22, Def. XI.1
* Post. XI.1, Post. XI.2
* CN 7
* Prop. II.10, Prop. VI.1, Prop. VI.2, Prop. XI.1

## Demonstration

Correlation (Def. I.22) associates every log line and span with the originating business activity, and the originating activity of almost every request is a user action in a front-end. If the front-end does not start the trace, the edge starts it, and the user action that caused it is invisible; by CN 7 what cannot be observed cannot be operated. Prop. VI.2 requires every hop to propagate `traceparent` and Prop. II.10 declares it a standard request header, so the front-end is simply the first hop. A client-side error that reaches no telemetry is, by CN 7, absent, so RUM is the front-end's share of Prop. VI.1. The agent is untrusted (Post. XI.2), so its data is evidence, not authority. ∎ Q.E.D.

## Corollaries

* **Cor. XI.8.1** - A retry (Prop. XI.12) reuses the trace-id and takes a new parent-id, so that retries are visible as siblings of the original attempt.
* **Cor. XI.8.2** - Telemetry MUST NOT carry access tokens, cookies or fields classified above `Internal` (Def. V.9); the RUM agent's request and response capture is configured accordingly.

## Construction

* Trace origination: OpenTelemetry JS (`@opentelemetry/sdk-trace-web` with `@opentelemetry/instrumentation-fetch`) or the RUM agent's own tracer; injected into the generated client's `fetch` (Prop. XI.1) so that no call escapes.
* Header: `traceparent: 00-<trace-id>-<parent-id>-01`; `tracestate` passed through unmodified.
* RUM: Amazon CloudWatch RUM (`aws-rum-web`) as the default agent, with the X-Ray trace header disabled in favour of W3C `traceparent`; Datadog RUM is a recorded alternative where a team already runs it, with `allowedTracingUrls` set to the API domains.
* Propagation across the edge: `traceparent` in the CORS allowed headers (Prop. XI.7) and passed through API Gateway to the service (Prop. VI.2).
* Sampling: head-based at the front-end, rate per environment from configuration; the sampled flag in `traceparent` respected downstream.

## Conformance

Contract test in every front-end repository: an intercepted request through the generated client carries a well-formed `traceparent`; two requests within one user action share a trace-id. Telemetry check: CloudWatch RUM or Datadog app monitor exists for each front-end per environment and is configured with tracing to the API domains.

## Scholium

RUM is a SHOULD because a server-rendered front-end with little script gains less from it; trace origination is a MUST because it costs one header and buys end-to-end visibility. Privacy configuration of the RUM agent (masking, consent) is governed by Book VII and the company's privacy policy, not by this proposition.
