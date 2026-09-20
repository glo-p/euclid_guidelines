# Prop. XI.1 - Front-ends consume APIs through generated clients

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A front-end MUST call an API only through a generated client (Def. XI.6) produced from that API's published OpenAPI document. Each API's client MUST be published as one npm package whose major version equals the API's URI major version (Prop. II.7). Hand-written request or response code against a Book II API MUST NOT exist in a front-end.

## Given

* Def. I.2, Def. I.4, Def. XI.1, Def. XI.6
* Post. I.5, Post. I.7, Post. XI.1, Post. XI.3, Post. XI.4
* CN 3
* Prop. II.1, Prop. II.7, Prop. IV.1

## Demonstration

By Prop. II.1 every API is described by an OpenAPI document before it is implemented, and by Post. I.7 a client can be generated from that document. A front-end is a consumer (Def. XI.1, Def. I.4) and is conforming only if it relies on nothing outside the contract (CN 3); a generated client can by construction rely on nothing else, whereas hand-written code can and does. Front-ends are of mixed technology (Post. XI.1), so the client must be in the one language all of them consume (Post. XI.4), which is TypeScript on npm. Consumers lag producers indefinitely (Post. I.5), so the client package must carry the major version of the API it binds to (Prop. II.7), allowing two majors to coexist during migration. ∎ Q.E.D.

## Corollaries

* **Cor. XI.1.1** - The generated client is regenerated and republished by the API's owning team on every contract change; the front-end team never edits it.
* **Cor. XI.1.2** - A BFF (Def. XI.4) is an API and therefore also has a generated client; its front-end consumes it the same way.

## Construction

* Source: the API's OpenAPI 3.1 document from the contract repository (Prop. II.1).
* Generator: `openapi-typescript` for types plus `openapi-fetch` for the runtime, or `@hey-api/openapi-ts`; one generator chosen company-wide and pinned in the catalogue (Prop. IV.1). Generated output uses `fetch` only, so no framework is assumed.
* Package name: `@company/api-<service>-v<major>`; version `<major>.<minor>.<patch>` where `<major>` equals the API's URI major.
* Shared types (Money, Page, ProblemDetails, Identifier) are imported from `@company/contracts`, not duplicated into the client (Prop. XI.2).
* Registry: private npm registry on AWS CodeArtifact; publish from the API's CI on every merged contract change.
* The client accepts a `fetch` implementation and base URL as parameters so that traceparent (Prop. XI.8) and Idempotency-Key (Prop. XI.12) can be injected by middleware.

## Conformance

CI check in every front-end repository: a dependency rule (`eslint-plugin-import` `no-restricted-imports` or `dependency-cruiser`) forbids direct calls to `fetch`, `XMLHttpRequest` or `axios` against any host in the API domain list outside the generated client package; and a package audit confirms every `@company/api-*` dependency's major matches a currently published API major.

## Scholium

The rule is about provenance, not about a particular generator. A front-end that wraps the generated client in its own repository pattern, hooks or stores is conforming; the wrapping calls the client and the client calls the API. What is forbidden is a hand-maintained description of an API that drifts silently from the contract. Server-rendered front-ends calling APIs from their own server process are front-ends for the purpose of this book and use the same package.
