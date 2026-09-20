# Prop. II.12 - Authentication at the edge, authorisation in the service

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Every operation except the health endpoints (Prop. II.16) MUST require an OAuth 2.0 bearer access token issued by the company identity provider (Post. V.1), validated at the edge (Def. II.12) for signature, issuer, audience, expiry and required scopes, and validated again in the service. The contract MUST declare a `securitySchemes` entry of type `oauth2`/`openIdConnect` and the scopes each operation requires. Fine-grained decisions (ownership, tenant, state-dependent permission) MUST be made in the service (Prop. V.2). Failures are `401` for missing/invalid tokens and `403` for insufficient permission (Prop. II.3), except that cross-tenant access is `404` (Cor. II.3.1).

## Given

Def. II.12, Def. II.13, Post. II.4, Post. II.5, CN 3, Prop. V.1, V.2, V.7.

## Demonstration

There is one ingress (Post. II.4), so validating there means no API can be reached unauthenticated by omission. The edge, however, knows only the token and the route; it cannot know whether *this* principal may act on *this* resource in *this* state, which is why fine-grained decisions belong in the service (Prop. V.2). Validating twice costs microseconds and removes the edge from the trust boundary of the service, which matters when services call each other inside the VPC. Declaring scopes in the contract is CN 3: a consumer cannot request what it cannot see. ∎ Q.E.D.

## Corollaries

* **Cor. II.12.1** - Service-to-service calls carry a client-credentials token (Prop. V.3); there are no "internal, unauthenticated" endpoints.
* **Cor. II.12.2** - The principal's tenant is read from the token; a request body or query field naming a tenant is an *input* to an admin API, never the source of authority (Prop. V.7).

## Construction

* Edge: API Gateway JWT authoriser (HTTP API) or Lambda authoriser (REST API) against the IdP's JWKS; scopes per route.
* Service: `AddAuthentication().AddJwtBearer(o => { o.Authority = …; o.Audience = …; })` and `AddAuthorization(o => o.AddPolicy("orders:write", p => p.RequireClaim("scope", "orders:write")))`; resource-level checks via `IAuthorizationHandler` with the resource as the `resource` argument.
* OpenAPI:

```yaml
components:
  securitySchemes:
    oauth:
      type: openIdConnect
      openIdConnectUrl: https://id.company.com/.well-known/openid-configuration
security: [{ oauth: [] }]
paths:
  /v1/orders:
    post:
      security: [{ oauth: [orders:write] }]
```

## Conformance

Spectral: `operation-has-security` (every operation has non-empty `security` except those tagged `health`), `security-scheme-oidc`. Template test: request without token yields `401` Problem.
