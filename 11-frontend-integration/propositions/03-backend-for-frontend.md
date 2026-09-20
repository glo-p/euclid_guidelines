# Prop. XI.3 - A backend-for-frontend is permitted, one per front-end

| | |
|---|---|
| **Level** | MAY |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A front-end MAY be served by exactly one BFF ([Def. XI.4](../definitions.md#Def.%20XI.4%20-%20Backend-for-Frontend%20%28BFF%29)), owned by the front-end's own team ([Def. I.17](../../01-foundations/definitions.md#Def.%20I.17%20-%20Team)). A BFF MUST be a [Book II](../../02-api-guidelines/README.md) service in every respect (contract-first, Problem Details, versioning, edge authentication). A BFF MUST aggregate and shape only: it MUST NOT own business data, MUST NOT be the system of record for any resource, MUST NOT publish domain events, and MUST NOT be consumed by anything other than its front-end.

## Given

* [Def. I.1](../../01-foundations/definitions.md#Def.%20I.1%20-%20Service), [Def. I.5](../../01-foundations/definitions.md#Def.%20I.5%20-%20Bounded%20Context), [Def. I.17](../../01-foundations/definitions.md#Def.%20I.17%20-%20Team), [Def. XI.4](../definitions.md#Def.%20XI.4%20-%20Backend-for-Frontend%20%28BFF%29), [Def. XI.5](../definitions.md#Def.%20XI.5%20-%20Public%20API%20versus%20BFF%20API)
* [Post. I.3](../../01-foundations/postulates.md#Post.%20I.3%20-%20Autonomy), [Post. XI.2](../postulates.md#Post.%20XI.2%20-%20Untrusted%20Clients), [Post. XI.3](../postulates.md#Post.%20XI.3%20-%20Contracts%20Only)
* [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts)
* [Prop. II.1](../../02-api-guidelines/propositions/01-contract-first.md), [Prop. II.4](../../02-api-guidelines/propositions/04-problem-details.md), [Prop. II.7](../../02-api-guidelines/propositions/07-versioning.md), [Prop. II.12](../../02-api-guidelines/propositions/12-authentication-and-authorisation.md)

## Demonstration

A front-end often needs a representation that no single public API provides, and by [Post. XI.3](../postulates.md#Post.%20XI.3%20-%20Contracts%20Only) it may not assemble one by reaching past the contracts. A BFF supplies that representation from the front-end's side of the boundary; because it is a service ([Def. I.1](../../01-foundations/definitions.md#Def.%20I.1%20-%20Service)) it must have one owning team ([Def. I.17](../../01-foundations/definitions.md#Def.%20I.17%20-%20Team)), and the only team whose interests it serves is the front-end's. By [Post. I.3](../../01-foundations/postulates.md#Post.%20I.3%20-%20Autonomy) teams interact only through contracts, so a BFF that held business data would create a second owner for a bounded context ([Def. I.5](../../01-foundations/definitions.md#Def.%20I.5%20-%20Bounded%20Context)), a contradiction of [Def. I.1](../../01-foundations/definitions.md#Def.%20I.1%20-%20Service); therefore it holds none and calls public APIs for everything of substance. By [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts) the BFF's API is architecture only if it is a contract, so it is written under [Book II](../../02-api-guidelines/README.md) like any other. ∎ Q.E.D.

## Corollaries

* **Cor. XI.3.1** - Two front-ends never share a BFF. If two front-ends need the same aggregation, that aggregation is a public API owned by a domain team.
* **Cor. XI.3.2** - A BFF's authorisation is a pass-through: it forwards the principal's token to public APIs and never widens what the principal may do ([Prop. V.2](../../05-security/propositions/02-coarse-scopes-fine-authorisation.md)).

## Construction

* Runtime: .NET on the current LTS, deployed as any [Book II](../../02-api-guidelines/README.md) service ([Prop. II.17](../../02-api-guidelines/propositions/17-dotnet-construction.md)); behind Amazon API Gateway ([Post. V.2](../../05-security/postulates.md#Post.%20V.2%20-%20Single%20Ingress)) with edge JWT validation ([Prop. V.1](../../05-security/propositions/01-every-api-behind-the-edge.md)).
* Contract: OpenAPI 3.1 in the front-end team's contract repository ([Prop. II.1](../../02-api-guidelines/propositions/01-contract-first.md)); generated client per [Prop. XI.1](01-generated-clients.md).
* Outbound: generated .NET clients for the public APIs it calls; traceparent forwarded ([Prop. VI.2](../../06-observability/propositions/02-traceparent-propagation.md)); the caller's access token forwarded on behalf of the principal.
* Session (optional): the BFF terminates OIDC for the browser and holds the session in an `httpOnly` cookie ([Prop. XI.4](04-browser-authentication.md)), with token state in Amazon ElastiCache or DynamoDB with a TTL; this is session state, not business data.
* State it may hold: cache entries with a TTL, session state. State it may not hold: anything a public API is the source of truth for.
* Naming: `<frontend>-bff`; API Gateway route restricted to the front-end's CORS allow-list ([Prop. XI.7](07-cors-allow-list.md)).

## Conformance

Architecture test in the BFF repository: no database other than a cache or session store is provisioned in its infrastructure module; no EventBridge `PutEvents` permission in its IAM role; API Gateway usage plan or authoriser restricts callers to the front-end's origins and client id. Catalogue check: no generated client for a BFF API is depended upon by any package other than its front-end.

## Scholium

A BFF is an option, not a default. Many front-ends are well served by the public APIs directly and gain nothing from an extra hop. The pattern earns its place when a front-end needs server-side session handling ([Prop. XI.4](04-browser-authentication.md)), when several APIs must be composed for one screen, or when a native client needs a different shape from the browser. It is deliberately a MAY: teams choose, and the constraints above apply when they do.
