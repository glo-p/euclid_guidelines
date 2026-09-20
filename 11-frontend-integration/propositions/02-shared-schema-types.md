# Prop. XI.2 - Shared schema types come only from @company/contracts

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A front-end MUST obtain the types and validators for every shared schema ([Def. I.13](../../01-foundations/definitions.md#Def.%20I.13%20-%20Shared%20Schema)) from the `@company/contracts` npm package ([Prop. IV.1](../../04-shared-schemas/propositions/01-catalogue-and-distribution.md)) and MUST NOT declare its own type for Money, Page, PageInfo, ProblemDetails, Identifier, or any other concept in the catalogue ([Def. I.24](../../01-foundations/definitions.md#Def.%20I.24%20-%20Catalogue)).

## Given

* [Def. I.12](../../01-foundations/definitions.md#Def.%20I.12%20-%20Schema), [Def. I.13](../../01-foundations/definitions.md#Def.%20I.13%20-%20Shared%20Schema), [Def. I.24](../../01-foundations/definitions.md#Def.%20I.24%20-%20Catalogue), [Def. XI.1](../definitions.md#Def.%20XI.1%20-%20Front-end)
* [Post. XI.3](../postulates.md#Post.%20XI.3%20-%20Contracts%20Only), [Post. XI.4](../postulates.md#Post.%20XI.4%20-%20Common%20Language)
* [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape)
* [Prop. IV.1](../../04-shared-schemas/propositions/01-catalogue-and-distribution.md), [Prop. IV.2](../../04-shared-schemas/propositions/02-identifier.md), [Prop. IV.3](../../04-shared-schemas/propositions/03-money.md), [Prop. IV.5](../../04-shared-schemas/propositions/05-problem-details.md), [Prop. IV.6](../../04-shared-schemas/propositions/06-page.md)

## Demonstration

Shared schemas are owned by the platform and reused unchanged by every team ([Def. I.13](../../01-foundations/definitions.md#Def.%20I.13%20-%20Shared%20Schema)); by [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape) a second shape for the same concept is a defect. [Prop. IV.1](../../04-shared-schemas/propositions/01-catalogue-and-distribution.md) publishes the catalogue as TypeScript types generated from the JSON Schema, and by [Post. XI.4](../postulates.md#Post.%20XI.4%20-%20Common%20Language) every front-end can consume them. A front-end that declares its own `Money` or `Page` type has, by [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape), created a second shape, and the two will diverge the first time the catalogue changes. Therefore the only conforming source is the package, and Money, Page, ProblemDetails and Identifier are parsed, validated and rendered by the same code in every front-end. ∎ Q.E.D.

## Corollaries

* **Cor. XI.2.1** - Generated clients ([Prop. XI.1](01-generated-clients.md)) reference `@company/contracts` types by import rather than inlining them, so the generator is configured to map the shared `$ref`s to that package.
* **Cor. XI.2.2** - Formatting helpers for shared types ([Prop. XI.10](10-client-side-formatting.md)) live alongside the types in `@company/contracts` so that a front-end never re-implements them.

## Construction

* Package: `@company/contracts` from the catalogue ([Prop. IV.1](../../04-shared-schemas/propositions/01-catalogue-and-distribution.md)), containing TypeScript types, JSON Schema 2020-12 documents and compiled validators for each shared schema.
* Validation at runtime: `ajv` (2020-12 dialect) compiled validators shipped in the package, used at the front-end boundary where a payload is untrusted or user-edited.
* Generator mapping: `openapi-typescript` `--redocly` or equivalent `$ref` override so that `Money`, `Page`, `PageInfo`, `ProblemDetails`, `Identifier`, `Timestamp`, `Date` and `Period` resolve to `@company/contracts` exports.
* Versioning: the package follows the catalogue's own version ([Prop. IV.1](../../04-shared-schemas/propositions/01-catalogue-and-distribution.md)); front-ends pin a major and take minors automatically.

## Conformance

CI check in every front-end repository: a lint rule forbids type or interface declarations whose name matches a catalogue schema name outside `node_modules`, and `dependency-cruiser` forbids importing shared type names from any module other than `@company/contracts`.

## Scholium

The package is framework-neutral by construction: types and pure functions only, no components. Front-end teams may build framework-specific components on top of it; those components are the team's own and are not shared through this package. Whether the package should also carry a small set of framework-neutral formatting helpers is settled in [Prop. XI.10](10-client-side-formatting.md).
