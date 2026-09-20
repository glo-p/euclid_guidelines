# Prop. XI.3 - A backend-for-frontend is permitted, one per front-end

| | |
|---|---|
| **Level** | MAY |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A front-end MAY be served by exactly one BFF (Def. XI.4), owned by the front-end's own team (Def. I.17). A BFF MUST be a Book II service in every respect (contract-first, Problem Details, versioning, edge authentication). A BFF MUST aggregate and shape only: it MUST NOT own business data, MUST NOT be the system of record for any resource, MUST NOT publish domain events, and MUST NOT be consumed by anything other than its front-end.

## Given

* Def. I.1, Def. I.5, Def. I.17, Def. XI.4, Def. XI.5
* Post. I.3, Post. XI.2, Post. XI.3
* CN 8
* Prop. II.1, Prop. II.4, Prop. II.7, Prop. II.12

## Demonstration

A front-end often needs a representation that no single public API provides, and by Post. XI.3 it may not assemble one by reaching past the contracts. A BFF supplies that representation from the front-end's side of the boundary; because it is a service (Def. I.1) it must have one owning team (Def. I.17), and the only team whose interests it serves is the front-end's. By Post. I.3 teams interact only through contracts, so a BFF that held business data would create a second owner for a bounded context (Def. I.5), a contradiction of Def. I.1; therefore it holds none and calls public APIs for everything of substance. By CN 8 the BFF's API is architecture only if it is a contract, so it is written under Book II like any other. ∎ Q.E.D.

## Corollaries

* **Cor. XI.3.1** - Two front-ends never share a BFF. If two front-ends need the same aggregation, that aggregation is a public API owned by a domain team.
* **Cor. XI.3.2** - A BFF's authorisation is a pass-through: it forwards the principal's token to public APIs and never widens what the principal may do (Prop. V.2).

## Construction

* Runtime: .NET on the current LTS, deployed as any Book II service (Prop. II.17); behind Amazon API Gateway (Post. V.2) with edge JWT validation (Prop. V.1).
* Contract: OpenAPI 3.1 in the front-end team's contract repository (Prop. II.1); generated client per Prop. XI.1.
* Outbound: generated .NET clients for the public APIs it calls; traceparent forwarded (Prop. VI.2); the caller's access token forwarded on behalf of the principal.
* Session (optional): the BFF terminates OIDC for the browser and holds the session in an `httpOnly` cookie (Prop. XI.4), with token state in Amazon ElastiCache or DynamoDB with a TTL; this is session state, not business data.
* State it may hold: cache entries with a TTL, session state. State it may not hold: anything a public API is the source of truth for.
* Naming: `<frontend>-bff`; API Gateway route restricted to the front-end's CORS allow-list (Prop. XI.7).

## Conformance

Architecture test in the BFF repository: no database other than a cache or session store is provisioned in its infrastructure module; no EventBridge `PutEvents` permission in its IAM role; API Gateway usage plan or authoriser restricts callers to the front-end's origins and client id. Catalogue check: no generated client for a BFF API is depended upon by any package other than its front-end.

## Scholium

A BFF is an option, not a default. Many front-ends are well served by the public APIs directly and gain nothing from an extra hop. The pattern earns its place when a front-end needs server-side session handling (Prop. XI.4), when several APIs must be composed for one screen, or when a native client needs a different shape from the browser. It is deliberately a MAY: teams choose, and the constraints above apply when they do.
