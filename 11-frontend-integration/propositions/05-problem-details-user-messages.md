# Prop. XI.5 - Errors are mapped from Problem Details type URIs through a shared map

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A front-end MUST derive user-facing error messages from the `type` URI of the Problem Details response ([Prop. II.4](../../02-api-guidelines/propositions/04-problem-details.md)) through the shared, versioned error map published in `@company/contracts` ([Prop. IV.1](../../04-shared-schemas/propositions/01-catalogue-and-distribution.md)). A front-end MUST NOT branch on `title`, `detail` or free text, and MUST render a generic message for any `type` absent from the map rather than fail.

## Given

* [Def. I.2](../../01-foundations/definitions.md#Def.%20I.2%20-%20Contract), [Def. I.13](../../01-foundations/definitions.md#Def.%20I.13%20-%20Shared%20Schema), [Def. XI.1](../definitions.md#Def.%20XI.1%20-%20Front-end)
* [Post. XI.1](../postulates.md#Post.%20XI.1%20-%20Mixed%20Technology), [Post. XI.4](../postulates.md#Post.%20XI.4%20-%20Common%20Language)
* [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape), [CN 6](../../01-foundations/common-notions.md#CN%206%20-%20The%20Producer%20Pays%20for%20Stability%3B%20the%20Consumer%20Pays%20for%20Tolerance)
* [Prop. II.4](../../02-api-guidelines/propositions/04-problem-details.md), [Prop. IV.5](../../04-shared-schemas/propositions/05-problem-details.md), [Prop. XI.2](02-shared-schema-types.md)

## Demonstration

By [Prop. II.4](../../02-api-guidelines/propositions/04-problem-details.md) every error is a Problem Details document whose `type` URI is stable and documented, and by [Prop. IV.5](../../04-shared-schemas/propositions/05-problem-details.md) its shape is a shared schema. The `type` is the only field of an error that is contract ([CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit)); `title` and `detail` are prose and may change without notice. A front-end that branches on prose therefore relies on something outside the contract and is not conforming. Each front-end writing its own map from `type` to message would produce as many shapes for one concept as there are front-ends, which [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape) forbids; the map is therefore shared, and by [Post. XI.4](../postulates.md#Post.%20XI.4%20-%20Common%20Language) it is shipped as TypeScript. An unknown `type` is an unknown value, and by [CN 6](../../01-foundations/common-notions.md#CN%206%20-%20The%20Producer%20Pays%20for%20Stability%3B%20the%20Consumer%20Pays%20for%20Tolerance) the consumer ignores what it does not understand rather than failing. ∎ Q.E.D.

## Corollaries

* **Cor. XI.5.1** - Adding a new `type` to an API is a compatible change ([Def. I.15](../../01-foundations/definitions.md#Def.%20I.15%20-%20Compatible%20Change)); the front-end shows the generic message until the map is updated, and nothing breaks.
* **Cor. XI.5.2** - Validation problems carrying a per-field `errors` extension ([Prop. IV.5](../../04-shared-schemas/propositions/05-problem-details.md)) are mapped per field by the same mechanism, keyed on the field's problem `type`.

## Construction

* Map location: `@company/contracts/problems`, a JSON document keyed by `type` URI with a message key per entry, plus TypeScript types; versioned with the catalogue.
* Message text: the map carries message keys, not prose; each front-end resolves keys through its own localisation resources so that language and tone remain local.
* Runtime: a pure function `problemToMessageKey(problem: ProblemDetails): MessageKey` exported from the package; the generated client ([Prop. XI.1](01-generated-clients.md)) surfaces non-2xx responses as typed `ProblemDetails` for it to consume.
* Registration: an API team adds its `type` URIs to the map in the same pull request that adds them to the contract; the catalogue CI validates that every URI resolves to a documented problem page.
* Telemetry: unknown `type` values are recorded as a client-side warning ([Prop. XI.8](08-traceparent-and-rum.md)) so the map can be completed.

## Conformance

CI check in every front-end repository: a lint rule forbids string comparison against `.title` or `.detail` of a `ProblemDetails` value; contract test in the catalogue asserts every `type` URI declared in any published OpenAPI document has a map entry or an explicit `generic` marker.

## Scholium

The map carries keys rather than sentences so that a single package serves all locales and all brands without becoming a localisation bottleneck. A front-end may of course show `detail` to the user when the map says it is safe to do so; it may not decide what happened by reading it.
