# Prop. XI.1 - Front-ends consume APIs through generated clients

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A front-end MUST call an API only through a generated client ([Def. XI.6](../definitions.md#Def.%20XI.6%20-%20Generated%20Client)) produced from that API's published OpenAPI document. Each API's client MUST be published as one npm package whose major version equals the API's URI major version ([Prop. II.7](../../02-api-guidelines/propositions/07-versioning.md)). Hand-written request or response code against a [Book II](../../02-api-guidelines/README.md) API MUST NOT exist in a front-end.

## Given

* [Def. I.2](../../01-foundations/definitions.md#Def.%20I.2%20-%20Contract), [Def. I.4](../../01-foundations/definitions.md#Def.%20I.4%20-%20Consumer), [Def. XI.1](../definitions.md#Def.%20XI.1%20-%20Front-end), [Def. XI.6](../definitions.md#Def.%20XI.6%20-%20Generated%20Client)
* [Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution), [Post. I.7](../../01-foundations/postulates.md#Post.%20I.7%20-%20Contract%20First), [Post. XI.1](../postulates.md#Post.%20XI.1%20-%20Mixed%20Technology), [Post. XI.3](../postulates.md#Post.%20XI.3%20-%20Contracts%20Only), [Post. XI.4](../postulates.md#Post.%20XI.4%20-%20Common%20Language)
* [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit)
* [Prop. II.1](../../02-api-guidelines/propositions/01-contract-first.md), [Prop. II.7](../../02-api-guidelines/propositions/07-versioning.md), [Prop. IV.1](../../04-shared-schemas/propositions/01-catalogue-and-distribution.md)

## Demonstration

By [Prop. II.1](../../02-api-guidelines/propositions/01-contract-first.md) every API is described by an OpenAPI document before it is implemented, and by [Post. I.7](../../01-foundations/postulates.md#Post.%20I.7%20-%20Contract%20First) a client can be generated from that document. A front-end is a consumer ([Def. XI.1](../definitions.md#Def.%20XI.1%20-%20Front-end), [Def. I.4](../../01-foundations/definitions.md#Def.%20I.4%20-%20Consumer)) and is conforming only if it relies on nothing outside the contract ([CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit)); a generated client can by construction rely on nothing else, whereas hand-written code can and does. Front-ends are of mixed technology ([Post. XI.1](../postulates.md#Post.%20XI.1%20-%20Mixed%20Technology)), so the client must be in the one language all of them consume ([Post. XI.4](../postulates.md#Post.%20XI.4%20-%20Common%20Language)), which is TypeScript on npm. Consumers lag producers indefinitely ([Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution)), so the client package must carry the major version of the API it binds to ([Prop. II.7](../../02-api-guidelines/propositions/07-versioning.md)), allowing two majors to coexist during migration. ∎ Q.E.D.

## Corollaries

* **Cor. XI.1.1** - The generated client is regenerated and republished by the API's owning team on every contract change; the front-end team never edits it.
* **Cor. XI.1.2** - A BFF ([Def. XI.4](../definitions.md#Def.%20XI.4%20-%20Backend-for-Frontend%20%28BFF%29)) is an API and therefore also has a generated client; its front-end consumes it the same way.

## Construction

* Source: the API's OpenAPI 3.1 document from the contract repository ([Prop. II.1](../../02-api-guidelines/propositions/01-contract-first.md)).
* Generator: `openapi-typescript` for types plus `openapi-fetch` for the runtime, or `@hey-api/openapi-ts`; one generator chosen company-wide and pinned in the catalogue ([Prop. IV.1](../../04-shared-schemas/propositions/01-catalogue-and-distribution.md)). Generated output uses `fetch` only, so no framework is assumed.
* Package name: `@company/api-<service>-v<major>`; version `<major>.<minor>.<patch>` where `<major>` equals the API's URI major.
* Shared types (Money, Page, ProblemDetails, Identifier) are imported from `@company/contracts`, not duplicated into the client ([Prop. XI.2](02-shared-schema-types.md)).
* Registry: private npm registry on AWS CodeArtifact; publish from the API's CI on every merged contract change.
* The client accepts a `fetch` implementation and base URL as parameters so that traceparent ([Prop. XI.8](08-traceparent-and-rum.md)) and Idempotency-Key ([Prop. XI.12](12-client-side-idempotency-keys.md)) can be injected by middleware.

## Conformance

CI check in every front-end repository: a dependency rule (`eslint-plugin-import` `no-restricted-imports` or `dependency-cruiser`) forbids direct calls to `fetch`, `XMLHttpRequest` or `axios` against any host in the API domain list outside the generated client package; and a package audit confirms every `@company/api-*` dependency's major matches a currently published API major.

## Scholium

The rule is about provenance, not about a particular generator. A front-end that wraps the generated client in its own repository pattern, hooks or stores is conforming; the wrapping calls the client and the client calls the API. What is forbidden is a hand-maintained description of an API that drifts silently from the contract. Server-rendered front-ends calling APIs from their own server process are front-ends for the purpose of this book and use the same package.
