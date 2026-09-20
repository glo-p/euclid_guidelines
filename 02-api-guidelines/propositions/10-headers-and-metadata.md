# Prop. II.10 - Standard headers and response metadata

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Every request MUST carry `traceparent` (W3C Trace Context); the edge MUST mint one if absent and services MUST propagate it to every outbound call and message. Every response MUST carry `traceparent` echoing the trace-id, and `Content-Type`. Item and collection representations MUST include a `metadata` field conforming to `ResourceMetadata` ([Prop. IV.7](../../04-shared-schemas/propositions/07-resource-metadata-and-actor.md)). Custom headers MUST NOT be prefixed `X-`. The full set of standard headers is:

| Header | Direction | Required | Defined by |
|---|---|---|---|
| `traceparent`, `tracestate` | both | MUST | W3C Trace Context; [Prop. VI.2](../../06-observability/propositions/02-traceparent-propagation.md) |
| `Authorization: Bearer` | request | MUST (except health) | [Prop. II.12](12-authentication-and-authorisation.md) |
| `Idempotency-Key` | request | MUST on side-effecting `POST` | [Prop. II.8](08-idempotency.md) |
| `If-Match`, `If-None-Match` | request | SHOULD | [Prop. II.9](09-optimistic-concurrency.md), [II.14](14-caching.md) |
| `ETag` | response | SHOULD | [Prop. II.9](09-optimistic-concurrency.md) |
| `Location` | response | MUST on `201`/`202` | [Prop. II.3](03-methods-and-status-codes.md), [II.15](15-long-running-operations.md) |
| `Deprecation`, `Sunset`, `Link rel=deprecation` | response | MUST when deprecated | [Prop. II.7](07-versioning.md) |
| `RateLimit`, `RateLimit-Policy`, `Retry-After` | response | SHOULD; MUST on `429`/`503` | [Prop. II.13](13-rate-limiting.md) |
| `Cache-Control` | response | MUST on `GET` | [Prop. II.14](14-caching.md) |
| `Accept-Language` | request | MAY | consumed by front-ends, ignored by APIs ([Cor. II.4.3](04-problem-details.md#Corollaries)) |

## Given

[Def. I.22](../../01-foundations/definitions.md#Def.%20I.22%20-%20Correlation), [Post. I.4](../../01-foundations/postulates.md#Post.%20I.4%20-%20Unreliable%20Network), [Post. II.1](../postulates.md#Post.%20II.1%20-%20HTTP%20Is%20Authoritative), [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape), [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated), [Prop. IV.7](../../04-shared-schemas/propositions/07-resource-metadata-and-actor.md), [Prop. VI.2](../../06-observability/propositions/02-traceparent-propagation.md).

## Demonstration

Correlation is by a propagated identifier ([Def. I.22](../../01-foundations/definitions.md#Def.%20I.22%20-%20Correlation)) and without it a request is operationally invisible ([CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated)); W3C Trace Context is the format every tracing tool and AWS X-Ray already understand, so choosing anything else would violate [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape) across the tool chain. The response echo lets a client that never received a body ([Post. I.4](../../01-foundations/postulates.md#Post.%20I.4%20-%20Unreliable%20Network)) still quote a trace to support. Resource metadata (created/updated/version) is needed by every consumer that caches, sorts or updates, so it has one shape ([CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape)) and one place. `X-` prefixes were deprecated by RFC 6648 and [Post. II.1](../postulates.md#Post.%20II.1%20-%20HTTP%20Is%20Authoritative) defers to HTTP. The table exists so that a new API has nothing to decide. ∎ Q.E.D.

## Corollaries

* **Cor. II.10.1** - A separate "request id" header is not introduced; the trace-id is the request id. Logs, Problems (`traceId`) and support tickets all quote the same value.
* **Cor. II.10.2** - `tenantId` is not a header. It is derived from the token ([Prop. V.7](../../05-security/propositions/07-tenant-isolation-at-data-access.md)) and, for administrative APIs, is a path or query parameter.

## Construction

* ASP.NET Core: `builder.Services.AddOpenTelemetry().WithTracing(...)`; the default `HttpClient` instrumentation propagates `traceparent` automatically. Response echo via a small middleware: `ctx.Response.Headers["traceparent"] = Activity.Current.Id`.
* Event publishing: copy `Activity.Current.Id` into the CloudEvents `traceparent` extension ([Prop. III.1](../../03-events/propositions/01-envelope.md)).
* Every DTO for an item derives its `metadata` from `ResourceMetadata` in `Company.Contracts.Shared`.

## Conformance

Spectral `no-x-headers`, `item-has-metadata` (item schemas have a `metadata` property `$ref`-ing the shared schema). Template test asserts `traceparent` on every response.
