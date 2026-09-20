# Prop. XI.4 - Browser authentication uses OIDC authorisation code with PKCE

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A browser client MUST authenticate through the OIDC authorisation code flow with PKCE against the identity provider ([Post. V.1](../../05-security/postulates.md#Post.%20V.1%20-%20One%20Identity%20Provider)). Access tokens MUST be held either in process memory or in an `httpOnly`, `Secure`, `SameSite` cookie issued by the front-end's BFF ([Prop. XI.3](03-backend-for-frontend.md)). Tokens MUST NOT be written to `localStorage`, `sessionStorage`, IndexedDB or any storage readable by script. Native clients MUST use the same flow with the operating system's secure store as token storage.

## Given

* [Def. V.2](../../05-security/definitions.md#Def.%20V.2%20-%20Identity%20Provider), [Def. V.3](../../05-security/definitions.md#Def.%20V.3%20-%20Access%20Token), [Def. XI.2](../definitions.md#Def.%20XI.2%20-%20Browser%20Client), [Def. XI.3](../definitions.md#Def.%20XI.3%20-%20Native%20Client), [Def. XI.7](../definitions.md#Def.%20XI.7%20-%20Session), [Def. XI.8](../definitions.md#Def.%20XI.8%20-%20Token%20Storage)
* [Post. V.1](../../05-security/postulates.md#Post.%20V.1%20-%20One%20Identity%20Provider), [Post. XI.2](../postulates.md#Post.%20XI.2%20-%20Untrusted%20Clients), [Post. XI.3](../postulates.md#Post.%20XI.3%20-%20Contracts%20Only)
* [Prop. V.1](../../05-security/propositions/01-every-api-behind-the-edge.md), [Prop. V.2](../../05-security/propositions/02-coarse-scopes-fine-authorisation.md), [Prop. XI.3](03-backend-for-frontend.md)

## Demonstration

Front-ends are untrusted ([Post. XI.2](../postulates.md#Post.%20XI.2%20-%20Untrusted%20Clients)), so a front-end cannot hold a client secret; the authorisation code flow with PKCE is the OIDC flow designed for a public client, and by [Post. V.1](../../05-security/postulates.md#Post.%20V.1%20-%20One%20Identity%20Provider) the identity provider is the only issuer. The token is validated at the edge ([Prop. V.1](../../05-security/propositions/01-every-api-behind-the-edge.md)) and scoped coarsely there ([Prop. V.2](../../05-security/propositions/02-coarse-scopes-fine-authorisation.md)), so the front-end's only duty is to present it unaltered and not to leak it. Any storage readable by script is readable by any script the page executes, including injected script; therefore web storage is not acceptable token storage ([Def. XI.8](../definitions.md#Def.%20XI.8%20-%20Token%20Storage)). Process memory is unreadable across page loads, and an `httpOnly` cookie is unreadable by script at all; a BFF ([Prop. XI.3](03-backend-for-frontend.md)) is the only party that can set one. Hence the two permitted storages and nothing else. ∎ Q.E.D.

## Corollaries

* **Cor. XI.4.1** - Implicit flow and resource owner password credentials MUST NOT be used; they are incompatible with the demonstration above.
* **Cor. XI.4.2** - A front-end holding tokens in memory refreshes them through the identity provider's refresh mechanism or silent re-authentication; it does not persist a refresh token in the browser.

## Construction

* Identity provider: the company OIDC provider ([Post. V.1](../../05-security/postulates.md#Post.%20V.1%20-%20One%20Identity%20Provider)); the front-end is registered as a public client with PKCE required and exact redirect URIs per environment.
* Memory pattern: a framework-neutral OIDC library (`oidc-client-ts`) configured for authorisation code with PKCE; tokens kept in an in-memory store; the generated client ([Prop. XI.1](01-generated-clients.md)) receives a token provider function.
* Cookie pattern: the BFF ([Prop. XI.3](03-backend-for-frontend.md)) performs the code exchange server-side, holds tokens in its session store, and sets `Set-Cookie: session=<opaque>; HttpOnly; Secure; SameSite=Lax; Path=/`. The BFF attaches the access token to outbound calls. CSRF protection by `SameSite` plus an anti-forgery token on state-changing requests.
* Native pattern: `AppAuth` (iOS, Android) or the platform equivalent with the system browser; tokens in Keychain or Keystore.
* Edge: Amazon API Gateway JWT authoriser ([Prop. V.1](../../05-security/propositions/01-every-api-behind-the-edge.md)); for the cookie pattern the BFF is the JWT bearer and the browser never sees the token.

## Conformance

CI check in every browser client repository: a lint rule forbids `localStorage`, `sessionStorage` and `indexedDB` access from any module that imports the OIDC library or the generated client; a security test (Playwright or equivalent) logs in and asserts that no token-shaped value appears in web storage. Identity provider configuration check: no client registered with implicit or password grant enabled.

## Scholium

The choice between memory and cookie is the front-end team's, and it is the main reason a team adopts a BFF. Memory is simpler and needs no server; the cost is a re-login or silent renewal on every full page load. The cookie pattern survives reloads and keeps the token out of the browser entirely; the cost is running a BFF. Both are conforming. Refresh token rotation and lifetime policy are [Book V](../../05-security/README.md) matters and are not restated here.
