# Prop. V.1 - Every API is behind the edge with JWT validation

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every synchronous API MUST be exposed only through API Gateway, and API Gateway MUST validate the access token (Def. V.3) on every request before it reaches the service: signature against the identity provider's published keys, issuer, audience, expiry and not-before. A service MUST NOT accept a request that did not pass through the edge.

## Given

Def. V.2, Def. V.3, Post. V.1, Post. V.2, Post. V.5, Prop. II.12, CN 3.

## Demonstration

By Post. V.5 nothing a client sends may be trusted except the identity provider's signature on a token, and by Post. V.1 there is exactly one identity provider whose signature counts. Validation therefore has a single, uniform definition, and by Post. V.2 there is a single place through which every request passes. Doing the uniform thing in the single place is Prop. II.12, and doing it anywhere else would either duplicate it or leave a path that skips it, which CN 3 says a consumer may then rely on. Therefore the edge validates, and the service refuses whatever did not come from the edge. ∎ Q.E.D.

## Corollaries

* **Cor. V.1.1** - A service's own listener (ALB, Lambda function URL, container port) is reachable only from API Gateway, enforced by network policy or IAM authorisation, never by the service checking a shared header.
* **Cor. V.1.2** - The service still reads the validated token's claims (Prop. V.2); the edge decides whether the token is genuine, not what it may do.

## Construction

* API Gateway (HTTP API) JWT authorizer configured with the identity provider's issuer and audience, or REST API with a Lambda authorizer where claims must be transformed.
* Private integrations via VPC Link to an internal ALB; security group ingress restricted to the VPC Link ENIs.
* Lambda: function URL disabled; invocation permitted only to the API Gateway execution role.
* `Microsoft.AspNetCore.Authentication.JwtBearer` in the service as defence in depth, configured with the same issuer and audience (Prop. II.17).
* Terraform module `company-api-gateway` with the authorizer as a required input.

## Conformance

AWS Config rule: every API Gateway stage has an authorizer on every route; every ALB or Lambda fronting a service has no public ingress. OpenAPI linter: every operation carries a `security` requirement (Prop. II.12).

## Scholium

Public, unauthenticated endpoints (a status page, a webhook receiver from a third party) are still behind API Gateway; they are recorded as exceptions (Def. I.20) naming the compensating control, typically a signature check specific to that third party.
