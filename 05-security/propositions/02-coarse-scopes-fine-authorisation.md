# Prop. V.2 - Coarse scopes at the edge, fine-grained authorisation in the service

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

API Gateway MUST enforce only scopes ([Def. V.4](../definitions.md#Def.%20V.4%20-%20Scope)) declared per operation in the OpenAPI contract. Every decision about whether a principal may perform an action on a particular resource MUST be made inside the owning service using permissions ([Def. V.5](../definitions.md#Def.%20V.5%20-%20Permission)) derived from the token's claims and the service's own data. Authorisation MUST NOT be decided on role names ([Def. V.6](../definitions.md#Def.%20V.6%20-%20Role)).

## Given

[Def. V.4](../definitions.md#Def.%20V.4%20-%20Scope), [Def. V.5](../definitions.md#Def.%20V.5%20-%20Permission), [Def. V.6](../definitions.md#Def.%20V.6%20-%20Role), [Def. I.5](../../01-foundations/definitions.md#Def.%20I.5%20-%20Bounded%20Context), [Post. I.3](../../01-foundations/postulates.md#Post.%20I.3%20-%20Autonomy), [Post. V.2](../postulates.md#Post.%20V.2%20-%20Single%20Ingress), [Prop. V.1](01-every-api-behind-the-edge.md), [Prop. II.1](../../02-api-guidelines/propositions/01-contract-first.md).

## Demonstration

The edge ([Post. V.2](../postulates.md#Post.%20V.2%20-%20Single%20Ingress)) knows the contract but not the domain: it can compare a scope in the token with a scope on the operation, and nothing finer, because the meaning of "may cancel this order" lives in the bounded context ([Def. I.5](../../01-foundations/definitions.md#Def.%20I.5%20-%20Bounded%20Context)) that owns orders. By [Post. I.3](../../01-foundations/postulates.md#Post.%20I.3%20-%20Autonomy) that context belongs to one team and no other component may embed its rules. So the edge decides scope, the service decides permission. Roles are administrative bundles ([Def. V.6](../definitions.md#Def.%20V.6%20-%20Role)) whose composition changes without a contract change; a decision on a role name would break silently when the bundle changes, whereas a decision on a permission is stable. ∎ Q.E.D.

## Corollaries

* **Cor. V.2.1** - Every operation in an OpenAPI document declares its required scopes under `security`; an operation with none is a linter failure ([Prop. II.1](../../02-api-guidelines/propositions/01-contract-first.md)).
* **Cor. V.2.2** - A missing scope yields `403` with Problem Details type `insufficient-scope` ([Prop. II.4](../../02-api-guidelines/propositions/04-problem-details.md)); a missing permission yields `403` with a domain-specific type, and `404` where the resource's existence is itself confidential.

## Construction

* Scopes declared in the OpenAPI `securitySchemes` and per-operation `security` blocks; API Gateway JWT authorizer `AuthorizationScopes` generated from the contract.
* ASP.NET Core authorisation policies (`AddAuthorization`, `IAuthorizationHandler`) evaluating permissions against the resource, with the resource loaded from the service's own store.
* Permission resolution from token claims plus a per-tenant role-to-permission table owned by the service, or by the identity provider's claims mapping where roles are global.
* Shared package `Company.Security.Authorization` providing the claims principal abstraction and policy helpers.

## Conformance

OpenAPI linter: every operation has a non-empty `security` requirement. Architecture test: no reference to role name strings in authorisation handlers; every controller action or endpoint carries an authorisation policy.

## Scholium

Two lists, not one. The edge list is short and changes with the contract; the service list is long and changes with the domain. Merging them into a central policy engine has been considered and is deferred; see the open questions.
