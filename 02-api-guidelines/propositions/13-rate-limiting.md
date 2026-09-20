# Prop. II.13 - Rate limits are declared and signalled

| | |
|---|---|
| **Level** | SHOULD |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Every API SHOULD have a rate limit per client (Def. II.13) enforced at the edge and declared in the contract's `info.description`. When a limit is exceeded the response MUST be `429` with a Problem of type `…/rate-limited` and a `Retry-After` header. Responses SHOULD carry the IETF `RateLimit` and `RateLimit-Policy` headers so clients can pace themselves before hitting `429`.

## Given

Def. II.12, Post. I.4, Post. II.5, CN 3, CN 6, Prop. II.3, II.4.

## Demonstration

Consumers we do not deploy (Post. II.5) will retry under Post. I.4, sometimes in tight loops; without a limit one client can deny service to the rest. The limit is part of what the consumer may rely on and so belongs in the contract (CN 3). Signalling the remaining budget is the producer's half of CN 6: it lets a well-behaved consumer avoid the error rather than merely receive it. ∎ Q.E.D.

## Corollaries

* **Cor. II.13.1** - Load shedding by the service itself (as opposed to the edge) uses `503` with `Retry-After`, never `429`, so that clients can tell "you are too fast" from "we are unwell".

## Construction

* Edge: API Gateway usage plans (REST API) or throttling per route (HTTP API); the `429` body is rewritten to a Problem via a gateway response template.
* Service-level (for internal callers bypassing usage plans): `Microsoft.AspNetCore.RateLimiting` with a policy keyed by client id from the token; `OnRejected` writes the Problem and headers.
* Headers: `RateLimit: limit=1000, remaining=12, reset=30`, `RateLimit-Policy: 1000;w=60`.

## Conformance

Spectral: `429-declared` (every operation lists a `429` response with the shared Problem `$ref`). Runtime test in template for the `OnRejected` shape.
