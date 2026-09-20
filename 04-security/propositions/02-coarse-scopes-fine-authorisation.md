# Prop. V.2 - Coarse scopes at the edge, fine-grained authorisation in the service

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

API Gateway MUST enforce only scopes (Def. V.4) declared per operation in the OpenAPI contract. Every decision about whether a principal may perform an action on a particular resource MUST be made inside the owning service using permissions (Def. V.5) derived from the token's claims and the service's own data. Authorisation MUST NOT be decided on role names (Def. V.6).

## Given

Def. V.4, Def. V.5, Def. V.6, Def. I.5, Post. I.3, Post. V.2, Prop. V.1, Prop. II.1.

## Demonstration

The edge (Post. V.2) knows the contract but not the domain: it can compare a scope in the token with a scope on the operation, and nothing finer, because the meaning of "may cancel this order" lives in the bounded context (Def. I.5) that owns orders. By Post. I.3 that context belongs to one team and no other component may embed its rules. So the edge decides scope, the service decides permission. Roles are administrative bundles (Def. V.6) whose composition changes without a contract change; a decision on a role name would break silently when the bundle changes, whereas a decision on a permission is stable. ∎ Q.E.D.

## Corollaries

* **Cor. V.2.1** - Every operation in an OpenAPI document declares its required scopes under `security`; an operation with none is a linter failure (Prop. II.1).
* **Cor. V.2.2** - A missing scope yields `403` with Problem Details type `insufficient-scope` (Prop. II.4); a missing permission yields `403` with a domain-specific type, and `404` where the resource's existence is itself confidential.

## Construction

* Scopes declared in the OpenAPI `securitySchemes` and per-operation `security` blocks; API Gateway JWT authorizer `AuthorizationScopes` generated from the contract.
* ASP.NET Core authorisation policies (`AddAuthorization`, `IAuthorizationHandler`) evaluating permissions against the resource, with the resource loaded from the service's own store.
* Permission resolution from token claims plus a per-tenant role-to-permission table owned by the service, or by the identity provider's claims mapping where roles are global.
* Shared package `Company.Security.Authorization` providing the claims principal abstraction and policy helpers.

## Conformance

OpenAPI linter: every operation has a non-empty `security` requirement. Architecture test: no reference to role name strings in authorisation handlers; every controller action or endpoint carries an authorisation policy.

## Scholium

Two lists, not one. The edge list is short and changes with the contract; the service list is long and changes with the domain. Merging them into a central policy engine has been considered and is deferred; see the open questions.
