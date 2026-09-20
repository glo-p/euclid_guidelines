# Prop. XI.8 - Front-ends originate traceparent and report client telemetry

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A front-end MUST originate a W3C `traceparent` header for every request it makes and MUST propagate the same trace-id across all requests belonging to one user action ([Prop. VI.2](../../06-observability/propositions/02-traceparent-propagation.md)). A front-end SHOULD report client-side telemetry (page and view timings, errors, failed requests) through a real user monitoring (RUM) agent that carries the same trace-id.

## Given

* [Def. I.22](../../01-foundations/definitions.md#Def.%20I.22%20-%20Correlation), [Def. XI.1](../definitions.md#Def.%20XI.1%20-%20Front-end)
* [Post. XI.1](../postulates.md#Post.%20XI.1%20-%20Mixed%20Technology), [Post. XI.2](../postulates.md#Post.%20XI.2%20-%20Untrusted%20Clients)
* [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated)
* [Prop. II.10](../../02-api-guidelines/propositions/10-headers-and-metadata.md), [Prop. VI.1](../../06-observability/propositions/01-structured-json-logs.md), [Prop. VI.2](../../06-observability/propositions/02-traceparent-propagation.md), [Prop. XI.1](01-generated-clients.md)

## Demonstration

Correlation ([Def. I.22](../../01-foundations/definitions.md#Def.%20I.22%20-%20Correlation)) associates every log line and span with the originating business activity, and the originating activity of almost every request is a user action in a front-end. If the front-end does not start the trace, the edge starts it, and the user action that caused it is invisible; by [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated) what cannot be observed cannot be operated. [Prop. VI.2](../../06-observability/propositions/02-traceparent-propagation.md) requires every hop to propagate `traceparent` and [Prop. II.10](../../02-api-guidelines/propositions/10-headers-and-metadata.md) declares it a standard request header, so the front-end is simply the first hop. A client-side error that reaches no telemetry is, by [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated), absent, so RUM is the front-end's share of [Prop. VI.1](../../06-observability/propositions/01-structured-json-logs.md). The agent is untrusted ([Post. XI.2](../postulates.md#Post.%20XI.2%20-%20Untrusted%20Clients)), so its data is evidence, not authority. ∎ Q.E.D.

## Corollaries

* **Cor. XI.8.1** - A retry ([Prop. XI.12](12-client-side-idempotency-keys.md)) reuses the trace-id and takes a new parent-id, so that retries are visible as siblings of the original attempt.
* **Cor. XI.8.2** - Telemetry MUST NOT carry access tokens, cookies or fields classified above `Internal` ([Def. V.9](../../05-security/definitions.md#Def.%20V.9%20-%20Data%20Classification)); the RUM agent's request and response capture is configured accordingly.

## Construction

* Trace origination: OpenTelemetry JS (`@opentelemetry/sdk-trace-web` with `@opentelemetry/instrumentation-fetch`) or the RUM agent's own tracer; injected into the generated client's `fetch` ([Prop. XI.1](01-generated-clients.md)) so that no call escapes.
* Header: `traceparent: 00-<trace-id>-<parent-id>-01`; `tracestate` passed through unmodified.
* RUM: Amazon CloudWatch RUM (`aws-rum-web`) as the default agent, with the X-Ray trace header disabled in favour of W3C `traceparent`; Datadog RUM is a recorded alternative where a team already runs it, with `allowedTracingUrls` set to the API domains.
* Propagation across the edge: `traceparent` in the CORS allowed headers ([Prop. XI.7](07-cors-allow-list.md)) and passed through API Gateway to the service ([Prop. VI.2](../../06-observability/propositions/02-traceparent-propagation.md)).
* Sampling: head-based at the front-end, rate per environment from configuration; the sampled flag in `traceparent` respected downstream.

## Conformance

Contract test in every front-end repository: an intercepted request through the generated client carries a well-formed `traceparent`; two requests within one user action share a trace-id. Telemetry check: CloudWatch RUM or Datadog app monitor exists for each front-end per environment and is configured with tracing to the API domains.

## Scholium

RUM is a SHOULD because a server-rendered front-end with little script gains less from it; trace origination is a MUST because it costs one header and buys end-to-end visibility. Privacy configuration of the RUM agent (masking, consent) is governed by [Book VII](../../07-data-ownership/README.md) and the company's privacy policy, not by this proposition.
