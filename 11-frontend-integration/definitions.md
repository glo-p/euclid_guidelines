# Book XI - Definitions

These definitions fix the meaning of the front-end vocabulary used in [Book XI](README.md). They are not rules. Where a definition refines one in [Book I](../01-foundations/README.md) it says so. Nothing here assumes a particular front-end framework ([Post. I.2](../01-foundations/postulates.md#Post.%20I.2%20-%20Language)).

---

## Def. XI.1 - Front-end

A *front-end* is any software that runs on a device the company does not control and presents the platform to a human: a browser client ([Def. XI.2](definitions.md#Def.%20XI.2%20-%20Browser%20Client)) or a native client ([Def. XI.3](definitions.md#Def.%20XI.3%20-%20Native%20Client)). A front-end is a consumer ([Def. I.4](../01-foundations/definitions.md#Def.%20I.4%20-%20Consumer)) of contracts ([Def. I.2](../01-foundations/definitions.md#Def.%20I.2%20-%20Contract)) and nothing more; it owns no bounded context ([Def. I.5](../01-foundations/definitions.md#Def.%20I.5%20-%20Bounded%20Context)) and no business data.

## Def. XI.2 - Browser Client

A *browser client* is a front-end delivered as HTML, CSS and JavaScript and executed by a web browser under the browser's security model (origins, cookies, CORS). It may be a single-page application, a server-rendered page with islands of script, or a mixture; this book does not distinguish between them.

## Def. XI.3 - Native Client

A *native client* is a front-end installed on a device as an application (mobile, desktop, or embedded) and executed outside a browser. It holds credentials in the platform-provided secure store of its operating system and is not subject to CORS.

## Def. XI.4 - Backend-for-Frontend (BFF)

A *backend-for-frontend* is a service ([Def. I.1](../01-foundations/definitions.md#Def.%20I.1%20-%20Service)) that exists to serve exactly one front-end ([Def. XI.1](definitions.md#Def.%20XI.1%20-%20Front-end)). It aggregates and shapes the responses of public APIs ([Def. XI.5](definitions.md#Def.%20XI.5%20-%20Public%20API%20versus%20BFF%20API)) into representations ([Def. I.8](../01-foundations/definitions.md#Def.%20I.8%20-%20Representation)) fitted to that front-end, and may hold the front-end's session ([Def. XI.7](definitions.md#Def.%20XI.7%20-%20Session)). A BFF owns no business data and publishes no events of its own.

## Def. XI.5 - Public API versus BFF API

A *public API* is a contract published by a service for any consumer, inside or outside the company, under [Book II](../02-api-guidelines/README.md). A *BFF API* is the contract a BFF ([Def. XI.4](definitions.md#Def.%20XI.4%20-%20Backend-for-Frontend%20%28BFF%29)) publishes for its one front-end. Both are [Book II](../02-api-guidelines/README.md) contracts; they differ only in audience. A BFF API is not a substitute for a public API and no service other than its front-end may consume it.

## Def. XI.6 - Generated Client

A *generated client* is a library produced mechanically from a contract ([Def. I.2](../01-foundations/definitions.md#Def.%20I.2%20-%20Contract)) by a generator, without hand-written request or response code, and published as a package with the same identity and major version as the contract it was generated from.

## Def. XI.7 - Session

A *session* is the period during which a front-end acts on behalf of one authenticated principal ([Def. V.1](../05-security/definitions.md#Def.%20V.1%20-%20Principal)). It is bounded by the lifetime of the access token ([Def. V.3](../05-security/definitions.md#Def.%20V.3%20-%20Access%20Token)) or of the cookie that stands for it. A session lives in the front-end and, where one exists, in its BFF; it is never held by a public API.

## Def. XI.8 - Token Storage

*Token storage* is the place a front-end keeps an access token or refresh token between requests. The places that exist are: process memory, an `httpOnly` cookie set by a BFF, the operating system's secure store (native clients), and web storage (`localStorage`, `sessionStorage`, IndexedDB). Only the first three are acceptable; see [Prop. XI.4](propositions/04-browser-authentication.md).

## Def. XI.9 - Origin

An *origin* is the browser's unit of isolation: the tuple of scheme, host and port from which a browser client ([Def. XI.2](definitions.md#Def.%20XI.2%20-%20Browser%20Client)) was loaded. Each environment ([Def. I.21](../01-foundations/definitions.md#Def.%20I.21%20-%20Environment)) of each front-end has its own origin.

## Def. XI.10 - CORS Allow-list

A *CORS allow-list* is the explicit, per-environment set of origins ([Def. XI.9](definitions.md#Def.%20XI.9%20-%20Origin)) from which an API accepts cross-origin browser requests. It is configuration, not contract; it differs between environments while the contract does not ([Def. I.21](../01-foundations/definitions.md#Def.%20I.21%20-%20Environment)).

## Def. XI.11 - Real-time Channel

A *real-time channel* is a server-to-front-end path over which a front-end receives notifications without polling: a WebSocket connection or a server-sent events stream. A real-time channel carries notifications derived from events ([Def. I.9](../01-foundations/definitions.md#Def.%20I.9%20-%20Event)); it is never the system of record and never the only way to learn a fact.

## Def. XI.12 - Design System

A *design system* is the shared set of visual tokens, components and interaction patterns a front-end is built from. It is named here only to state that it is **out of scope** for [Book XI](README.md); this book governs how front-ends consume contracts, not how they look.
