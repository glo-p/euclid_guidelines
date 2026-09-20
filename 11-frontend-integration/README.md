# Book XI - Front-end Integration

Book XI states how front-ends of any technology consume the APIs of [Book II](../02-api-guidelines/README.md) and the schemas of [Book IV](../04-shared-schemas/README.md). It is framework-neutral by postulate ([Post. XI.1](postulates.md#Post.%20XI.1%20-%20Mixed%20Technology)): nothing here depends on a particular front-end framework, and nothing here governs how a front-end looks ([Def. XI.12](definitions.md#Def.%20XI.12%20-%20Design%20System)). Its subject is the boundary between an untrusted client and the platform's contracts.

Depth: **Scaffold**. Every proposition is `Draft`.

## Definitions

[`definitions.md`](definitions.md)

| Item | Summary |
|---|---|
| [Def. XI.1](definitions.md#Def.%20XI.1%20-%20Front-end) | Front-end: software on a device we do not control that presents the platform to a human; a consumer and nothing more. |
| [Def. XI.2](definitions.md#Def.%20XI.2%20-%20Browser%20Client) | Browser client: a front-end executed by a web browser under its security model. |
| [Def. XI.3](definitions.md#Def.%20XI.3%20-%20Native%20Client) | Native client: a front-end installed as an application outside a browser. |
| [Def. XI.4](definitions.md#Def.%20XI.4%20-%20Backend-for-Frontend%20%28BFF%29) | Backend-for-frontend: a service serving exactly one front-end; aggregates and shapes, owns no business data. |
| [Def. XI.5](definitions.md#Def.%20XI.5%20-%20Public%20API%20versus%20BFF%20API) | Public API versus BFF API: both [Book II](../02-api-guidelines/README.md) contracts; they differ only in audience. |
| [Def. XI.6](definitions.md#Def.%20XI.6%20-%20Generated%20Client) | Generated client: a library produced mechanically from a contract, versioned with it. |
| [Def. XI.7](definitions.md#Def.%20XI.7%20-%20Session) | Session: the period a front-end acts for one principal; held by the front-end or its BFF, never a public API. |
| [Def. XI.8](definitions.md#Def.%20XI.8%20-%20Token%20Storage) | Token storage: memory, httpOnly cookie, OS secure store, or web storage; only the first three are acceptable. |
| [Def. XI.9](definitions.md#Def.%20XI.9%20-%20Origin) | Origin: scheme, host and port a browser client was loaded from; one per front-end per environment. |
| [Def. XI.10](definitions.md#Def.%20XI.10%20-%20CORS%20Allow-list) | CORS allow-list: the per-environment set of origins an API accepts browser requests from; configuration, not contract. |
| [Def. XI.11](definitions.md#Def.%20XI.11%20-%20Real-time%20Channel) | Real-time channel: a push path carrying notifications derived from events; never the system of record. |
| [Def. XI.12](definitions.md#Def.%20XI.12%20-%20Design%20System) | Design system: named only to declare it out of scope. |

## Postulates

[`postulates.md`](postulates.md)

| Item | Summary |
|---|---|
| [Post. XI.1](postulates.md#Post.%20XI.1%20-%20Mixed%20Technology) | Front-ends are of mixed technology ([Post. I.2](../01-foundations/postulates.md#Post.%20I.2%20-%20Language)); this book is framework-neutral. |
| [Post. XI.2](postulates.md#Post.%20XI.2%20-%20Untrusted%20Clients) | Browsers and native apps are untrusted clients; a front-end enforces nothing. |
| [Post. XI.3](postulates.md#Post.%20XI.3%20-%20Contracts%20Only) | Front-ends consume only [Book II](../02-api-guidelines/README.md) contracts and [Book IV](../04-shared-schemas/README.md) schemas; never databases or internal endpoints. |
| [Post. XI.4](postulates.md#Post.%20XI.4%20-%20Common%20Language) | TypeScript is the common language across front-end technologies. *(Open question: confirm.)* |

## Propositions

| Item | Level | File | Summary |
|---|---|---|---|
| [Prop. XI.1](propositions/01-generated-clients.md) | MUST | [`01-generated-clients.md`](propositions/01-generated-clients.md) | APIs are consumed only through clients generated from the OpenAPI document, published as one npm package per API carrying the API's major version. |
| [Prop. XI.2](propositions/02-shared-schema-types.md) | MUST | [`02-shared-schema-types.md`](propositions/02-shared-schema-types.md) | Shared schema types come only from `@company/contracts`, so Money, Page, ProblemDetails and Identifier behave identically everywhere. |
| [Prop. XI.3](propositions/03-backend-for-frontend.md) | MAY | [`03-backend-for-frontend.md`](propositions/03-backend-for-frontend.md) | A BFF is permitted, one per front-end, owned by its team, a [Book II](../02-api-guidelines/README.md) service that aggregates and shapes but never owns business data. |
| [Prop. XI.4](propositions/04-browser-authentication.md) | MUST | [`04-browser-authentication.md`](propositions/04-browser-authentication.md) | OIDC authorisation code with PKCE; tokens in memory or in a BFF-issued httpOnly cookie; never in web storage. |
| [Prop. XI.5](propositions/05-problem-details-user-messages.md) | MUST | [`05-problem-details-user-messages.md`](propositions/05-problem-details-user-messages.md) | User-facing error messages are derived from Problem Details `type` URIs through a shared, versioned map in `@company/contracts`. |
| [Prop. XI.6](propositions/06-cursor-collections.md) | MUST | [`06-cursor-collections.md`](propositions/06-cursor-collections.md) | Collection UIs use cursor semantics; no page numbers or totals unless the contract declares them. |
| [Prop. XI.7](propositions/07-cors-allow-list.md) | MUST | [`07-cors-allow-list.md`](propositions/07-cors-allow-list.md) | CORS allow-list of exact origins per environment at the edge; no wildcard outside `dev`. |
| [Prop. XI.8](propositions/08-traceparent-and-rum.md) | MUST | [`08-traceparent-and-rum.md`](propositions/08-traceparent-and-rum.md) | Front-ends originate `traceparent` on every request and report client telemetry through RUM. |
| [Prop. XI.9](propositions/09-real-time-updates.md) | SHOULD | [`09-real-time-updates.md`](propositions/09-real-time-updates.md) | Real-time updates flow from [Book III](../03-events/README.md) events over WebSocket or server-sent events, carrying hints only, with polling as fallback. |
| [Prop. XI.10](propositions/10-client-side-formatting.md) | MUST | [`10-client-side-formatting.md`](propositions/10-client-side-formatting.md) | Money, dates and identifiers are formatted client-side from canonical shapes; APIs never return display strings. |
| [Prop. XI.11](propositions/11-deprecation-and-sunset-headers.md) | MUST | [`11-deprecation-and-sunset-headers.md`](propositions/11-deprecation-and-sunset-headers.md) | `Deprecation` and `Sunset` headers are recorded as telemetry warnings so migrations are visible. |
| [Prop. XI.12](propositions/12-client-side-idempotency-keys.md) | MUST | [`12-client-side-idempotency-keys.md`](propositions/12-client-side-idempotency-keys.md) | An `Idempotency-Key` is generated client-side per submit and reused across retries. |

## Reading order

Start with [Post. XI.2](postulates.md#Post.%20XI.2%20-%20Untrusted%20Clients) and [Post. XI.3](postulates.md#Post.%20XI.3%20-%20Contracts%20Only); most of the book follows from "the client is untrusted and may only see contracts". Then [Prop. XI.1](propositions/01-generated-clients.md) and [Prop. XI.2](propositions/02-shared-schema-types.md), which establish the two packages every front-end depends on; the remaining propositions attach behaviour to those packages.

## Open questions

1. **[Post. XI.4](postulates.md#Post.%20XI.4%20-%20Common%20Language), TypeScript as common language.** Confirm that every front-end technology in the estate, including server-rendered pages with script islands and any native platform, can consume a TypeScript npm package. If one cannot, decide whether the generated client and `@company/contracts` are also published in that platform's language, or whether that front-end is served solely through a BFF.
2. **[Prop. XI.9](propositions/09-real-time-updates.md), real-time transport.** Pick one company-wide: API Gateway WebSocket (needs an exception ADR against [Post. I.9](../01-foundations/postulates.md#Post.%20I.9%20-%20Transport)) or server-sent events (fits [Post. I.9](../01-foundations/postulates.md#Post.%20I.9%20-%20Transport), needs response streaming at the edge). Authors recommend server-sent events unless a bidirectional case is found.
3. **[Prop. XI.1](propositions/01-generated-clients.md), generator choice.** Pin one TypeScript OpenAPI generator company-wide and record it in the [Book IV](../04-shared-schemas/README.md) catalogue tooling.
4. **[Prop. XI.8](propositions/08-traceparent-and-rum.md), RUM vendor.** CloudWatch RUM is the default by [Post. I.1](../01-foundations/postulates.md#Post.%20I.1%20-%20Cloud); Datadog RUM is in use by some teams. Decide whether both remain sanctioned or one is chosen.
5. **[Prop. XI.4](propositions/04-browser-authentication.md), session store for the cookie pattern.** ElastiCache or DynamoDB with TTL; a shared BFF session module would settle this once.
6. **[Prop. XI.5](propositions/05-problem-details-user-messages.md), message keys versus prose.** Confirm that the shared map carries localisation keys and that each front-end resolves them locally, or decide that a default English message accompanies each key.
7. **Server-rendered front-ends.** Confirm that a .NET server-rendered front-end calling APIs from its own process is bound by this book as a front-end (the position taken here) and not by [Book II](../02-api-guidelines/README.md) as a service.
