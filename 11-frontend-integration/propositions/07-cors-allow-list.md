# Prop. XI.7 - CORS allow-list per environment at the edge

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every API reachable by a browser client MUST declare a CORS allow-list (Def. XI.10) of exact origins per environment (Def. I.21), enforced at the edge (Post. V.2). A wildcard origin (`*`) or a reflected `Origin` header MUST NOT be configured in any environment other than `dev`. Credentials (cookies) MUST be allowed only for origins on the list.

## Given

* Def. I.21, Def. XI.2, Def. XI.9, Def. XI.10
* Post. I.6, Post. V.2, Post. XI.2
* CN 3
* Prop. V.1, Prop. XI.4

## Demonstration

A browser client is confined by its origin (Def. XI.9), and CORS is the browser's mechanism for deciding which origins may read an API's responses. Because clients are untrusted (Post. XI.2), the API cannot rely on a client to be well-behaved; the allow-list is the API's own statement of who may call it from a browser, and by CN 3 an origin not on the list may not. Each environment has its own origins (Def. I.21), so the list is per environment, and because the edge is the single ingress (Post. V.2) the list is enforced there once rather than in every service. A wildcard with credentials is rejected by browsers and a wildcard without credentials exposes every response to any page; neither is acceptable outside a developer's own machine. ∎ Q.E.D.

## Corollaries

* **Cor. XI.7.1** - The allow-list is configuration, not contract; changing it is not a contract change and does not pass through Book IX.
* **Cor. XI.7.2** - Native clients (Def. XI.3) are not subject to CORS and are not on the list; their access is governed by Prop. V.1 alone.

## Construction

* Edge: Amazon API Gateway CORS configuration (HTTP API `CorsConfiguration` or REST API `OPTIONS` integration) with `AllowOrigins` set per environment from the infrastructure module's variables; `AllowCredentials=true` only where a BFF cookie session exists (Prop. XI.4).
* Allowed headers: `Authorization`, `Content-Type`, `traceparent`, `tracestate`, `Idempotency-Key`, `If-Match`; exposed headers: `ETag`, `RateLimit-*`, `Deprecation`, `Sunset`, `Location` (Prop. II.10) so that generated clients can read them.
* Origin source: each front-end's environment origins are recorded in its infrastructure module and referenced by the API's module; no hand-typed origins.
* `dev` exception: `AllowOrigins=["*"]` without credentials is permitted for local development only.

## Conformance

AWS Config custom rule or policy-as-code (OPA or `cfn-guard`) on every API Gateway resource: `AllowOrigins` contains no `*` and no origin outside the recorded set in any environment except `dev`; `AllowCredentials=true` is never combined with `*`.

## Scholium

CORS is not authentication and not authorisation; those are Prop. V.1 and Prop. V.2. It is a browser courtesy that limits which pages can read responses. It is still worth getting right, because a wildcard turns every authenticated user's browser into a proxy for any page they visit. Services that also configure CORS in their own middleware are not wrong, but the edge configuration is the one that counts.
