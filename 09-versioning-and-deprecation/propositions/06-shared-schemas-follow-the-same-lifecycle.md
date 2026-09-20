# Prop. IX.6 - Shared schemas follow the same lifecycle

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A shared schema ([Def. I.13](../../01-foundations/definitions.md#Def.%20I.13%20-%20Shared%20Schema)) MUST pass through the lifecycle stages of [Def. IX.4](../definitions.md#Def.%20IX.4%20-%20Lifecycle%20Stage) under Props. IX.1 to IX.5 and IX.7 to IX.8 exactly as an API or event type does. Its major version ([Def. IX.2](../definitions.md#Def.%20IX.2%20-%20Major%20Version)) MUST be the major version segment of its `$id` URI as fixed by [Prop. IV.1](../../04-shared-schemas/propositions/01-catalogue-and-distribution.md), and its consumers ([Def. IX.8](../definitions.md#Def.%20IX.8%20-%20Consumer%20Registry)) MUST be enumerated as the set of contracts whose published artefacts reference that `$id`, plus the packages that embed it.

## Given

* [Def. I.2](../../01-foundations/definitions.md#Def.%20I.2%20-%20Contract), [Def. I.13](../../01-foundations/definitions.md#Def.%20I.13%20-%20Shared%20Schema), [Def. I.24](../../01-foundations/definitions.md#Def.%20I.24%20-%20Catalogue), [Def. IX.2](../definitions.md#Def.%20IX.2%20-%20Major%20Version), [Def. IX.4](../definitions.md#Def.%20IX.4%20-%20Lifecycle%20Stage), [Def. IX.8](../definitions.md#Def.%20IX.8%20-%20Consumer%20Registry)
* [Post. IX.3](../postulates.md#Post.%20IX.3%20-%20Consumers%20Are%20Enumerable)
* [CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed), [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape)
* [Prop. IV.1](../../04-shared-schemas/propositions/01-catalogue-and-distribution.md)

## Demonstration

A shared schema is a contract ([Def. I.2](../../01-foundations/definitions.md#Def.%20I.2%20-%20Contract), [Def. I.13](../../01-foundations/definitions.md#Def.%20I.13%20-%20Shared%20Schema)) and is therefore owed ([CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed)); the propositions of this book apply to contracts without regard to medium, so they apply to it. [Prop. IV.1](../../04-shared-schemas/propositions/01-catalogue-and-distribution.md) already fixes the major version in the `$id` path, which gives [Def. IX.2](../definitions.md#Def.%20IX.2%20-%20Major%20Version) its spelling for this medium. The consumers of a schema are not request senders but the contracts and packages that reference it, which are all in the catalogue ([Def. I.24](../../01-foundations/definitions.md#Def.%20I.24%20-%20Catalogue)) and so are enumerable ([Post. IX.3](../postulates.md#Post.%20IX.3%20-%20Consumers%20Are%20Enumerable)) by a reference scan rather than by traffic. By [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape), a team that copies a shared schema to escape its lifecycle has created a second shape for one concept, which is a defect; so the lifecycle must be followed, not avoided. ∎ Q.E.D.

## Corollaries

* **Cor. IX.6.1** - Traffic evidence ([Def. IX.9](../definitions.md#Def.%20IX.9%20-%20Traffic%20Evidence)) for a shared schema is the count of Active or Deprecated contracts referencing its `$id`; it is zero when no such contract remains.
* **Cor. IX.6.2** - A new major of a shared schema forces no new major of the contracts that reference it until those contracts adopt it; a contract may reference `v1` of a schema while `v2` exists.

## Construction

* Catalogue repository layout `schemas/v{N}/<name>.json` with `$id` `https://schemas.company.example/v{N}/<name>.json` ([Prop. IV.1](../../04-shared-schemas/propositions/01-catalogue-and-distribution.md)).
* Reference scanner in the catalogue CI: walks every published OpenAPI document and event schema in the EventBridge Schema Registry, collects `$ref` targets, and writes the consumer registry entries of kind `schema-ref`.
* NuGet `Company.Contracts.Shared` and npm `@company/contracts`: types for each major in a versioned namespace; Deprecated majors carry `[Obsolete]` (C#) and `@deprecated` JSDoc with the sunset date, generated from `x-lifecycle`.
* Package manager signals: NuGet `deprecation` metadata and `npm deprecate` on the package version that last carries a Retired major.
* Same `x-lifecycle` block and Spectral/JSON Schema lint as [Prop. IX.3](03-deprecation-is-declared-in-the-contract.md).

## Conformance

Catalogue lint asserting every shared schema has an `x-lifecycle` block with a stage of [Def. IX.4](../definitions.md#Def.%20IX.4%20-%20Lifecycle%20Stage) and a `$id` whose major matches its path; reference scanner output present and under 24 hours old.

## Scholium

`[Obsolete]` and `@deprecated` are the shared-schema analogue of the `Deprecation` header: a consumer meets the notice in the tool it already uses.
