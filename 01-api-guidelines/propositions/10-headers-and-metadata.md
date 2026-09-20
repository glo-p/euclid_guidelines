# Prop. II.10 - Standard headers and response metadata

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Every request MUST carry `traceparent` (W3C Trace Context); the edge MUST mint one if absent and services MUST propagate it to every outbound call and message. Every response MUST carry `traceparent` echoing the trace-id, and `Content-Type`. Item and collection representations MUST include a `metadata` field conforming to `ResourceMetadata` (Prop. IV.7). Custom headers MUST NOT be prefixed `X-`. The full set of standard headers is:

| Header | Direction | Required | Defined by |
|---|---|---|---|
| `traceparent`, `tracestate` | both | MUST | W3C Trace Context; Prop. VI.2 |
| `Authorization: Bearer` | request | MUST (except health) | Prop. II.12 |
| `Idempotency-Key` | request | MUST on side-effecting `POST` | Prop. II.8 |
| `If-Match`, `If-None-Match` | request | SHOULD | Prop. II.9, II.14 |
| `ETag` | response | SHOULD | Prop. II.9 |
| `Location` | response | MUST on `201`/`202` | Prop. II.3, II.15 |
| `Deprecation`, `Sunset`, `Link rel=deprecation` | response | MUST when deprecated | Prop. II.7 |
| `RateLimit`, `RateLimit-Policy`, `Retry-After` | response | SHOULD; MUST on `429`/`503` | Prop. II.13 |
| `Cache-Control` | response | MUST on `GET` | Prop. II.14 |
| `Accept-Language` | request | MAY | consumed by front-ends, ignored by APIs (Cor. II.4.3) |

## Given

Def. I.22, Post. I.4, Post. II.1, CN 5, CN 7, Prop. IV.7, Prop. VI.2.

## Demonstration

Correlation is by a propagated identifier (Def. I.22) and without it a request is operationally invisible (CN 7); W3C Trace Context is the format every tracing tool and AWS X-Ray already understand, so choosing anything else would violate CN 5 across the tool chain. The response echo lets a client that never received a body (Post. I.4) still quote a trace to support. Resource metadata (created/updated/version) is needed by every consumer that caches, sorts or updates, so it has one shape (CN 5) and one place. `X-` prefixes were deprecated by RFC 6648 and Post. II.1 defers to HTTP. The table exists so that a new API has nothing to decide. ∎ Q.E.D.

## Corollaries

* **Cor. II.10.1** - A separate "request id" header is not introduced; the trace-id is the request id. Logs, Problems (`traceId`) and support tickets all quote the same value.
* **Cor. II.10.2** - `tenantId` is not a header. It is derived from the token (Prop. V.7) and, for administrative APIs, is a path or query parameter.

## Construction

* ASP.NET Core: `builder.Services.AddOpenTelemetry().WithTracing(...)`; the default `HttpClient` instrumentation propagates `traceparent` automatically. Response echo via a small middleware: `ctx.Response.Headers["traceparent"] = Activity.Current.Id`.
* Event publishing: copy `Activity.Current.Id` into the CloudEvents `traceparent` extension (Prop. III.1).
* Every DTO for an item derives its `metadata` from `ResourceMetadata` in `Company.Contracts.Shared`.

## Conformance

Spectral `no-x-headers`, `item-has-metadata` (item schemas have a `metadata` property `$ref`-ing the shared schema). Template test asserts `traceparent` on every response.
