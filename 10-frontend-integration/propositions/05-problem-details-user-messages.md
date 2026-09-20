# Prop. XI.5 - Errors are mapped from Problem Details type URIs through a shared map

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A front-end MUST derive user-facing error messages from the `type` URI of the Problem Details response (Prop. II.4) through the shared, versioned error map published in `@company/contracts` (Prop. IV.1). A front-end MUST NOT branch on `title`, `detail` or free text, and MUST render a generic message for any `type` absent from the map rather than fail.

## Given

* Def. I.2, Def. I.13, Def. XI.1
* Post. XI.1, Post. XI.4
* CN 3, CN 5, CN 6
* Prop. II.4, Prop. IV.5, Prop. XI.2

## Demonstration

By Prop. II.4 every error is a Problem Details document whose `type` URI is stable and documented, and by Prop. IV.5 its shape is a shared schema. The `type` is the only field of an error that is contract (CN 3); `title` and `detail` are prose and may change without notice. A front-end that branches on prose therefore relies on something outside the contract and is not conforming. Each front-end writing its own map from `type` to message would produce as many shapes for one concept as there are front-ends, which CN 5 forbids; the map is therefore shared, and by Post. XI.4 it is shipped as TypeScript. An unknown `type` is an unknown value, and by CN 6 the consumer ignores what it does not understand rather than failing. ∎ Q.E.D.

## Corollaries

* **Cor. XI.5.1** - Adding a new `type` to an API is a compatible change (Def. I.15); the front-end shows the generic message until the map is updated, and nothing breaks.
* **Cor. XI.5.2** - Validation problems carrying a per-field `errors` extension (Prop. IV.5) are mapped per field by the same mechanism, keyed on the field's problem `type`.

## Construction

* Map location: `@company/contracts/problems`, a JSON document keyed by `type` URI with a message key per entry, plus TypeScript types; versioned with the catalogue.
* Message text: the map carries message keys, not prose; each front-end resolves keys through its own localisation resources so that language and tone remain local.
* Runtime: a pure function `problemToMessageKey(problem: ProblemDetails): MessageKey` exported from the package; the generated client (Prop. XI.1) surfaces non-2xx responses as typed `ProblemDetails` for it to consume.
* Registration: an API team adds its `type` URIs to the map in the same pull request that adds them to the contract; the catalogue CI validates that every URI resolves to a documented problem page.
* Telemetry: unknown `type` values are recorded as a client-side warning (Prop. XI.8) so the map can be completed.

## Conformance

CI check in every front-end repository: a lint rule forbids string comparison against `.title` or `.detail` of a `ProblemDetails` value; contract test in the catalogue asserts every `type` URI declared in any published OpenAPI document has a map entry or an explicit `generic` marker.

## Scholium

The map carries keys rather than sentences so that a single package serves all locales and all brands without becoming a localisation bottleneck. A front-end may of course show `detail` to the user when the map says it is safe to do so; it may not decide what happened by reading it.
