# Prop. IX.3 - Deprecation is declared in the contract and notified to consumers

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A version enters the Deprecated stage (Def. IX.4) only by a deprecation notice (Def. IX.5) recorded in the contract artefact itself: for an API, the OpenAPI `deprecated: true` flag on the operations concerned together with the `Deprecation` (RFC 9745) and `Sunset` (RFC 8594) response headers on every response of that major; for an event type, the `status: deprecated` field of its registry entry; for a shared schema, a `$comment` and an `x-lifecycle` object in the schema. The producer MUST send the notice to every consumer in the consumer registry (Def. IX.8) on the date of deprecation, and the sunset date carried by every medium MUST be identical.

## Given

* Def. I.2, Def. IX.4, Def. IX.5, Def. IX.6, Def. IX.8
* Post. IX.2, Post. IX.3
* CN 3
* Prop. II.7, Prop. II.10, Prop. III.4, Prop. IV.1

## Demonstration

By CN 3 what is not in the contract does not exist for the consumer; a deprecation announced only in a chat channel or wiki is therefore not a deprecation. The contract (Def. I.2) is the one artefact every conforming consumer already reads, so the notice belongs there, and Books II, III and IV each provide a field for it. Because the sunset date (Def. IX.6) is what the consumer plans against, it must agree across headers, registry and schema, or the consumer cannot know which one is owed. Consumers are enumerable (Post. IX.3) and recorded (Def. IX.8), so direct notification is possible and, given the minimum period of Post. IX.2 starts on the notice date, necessary. ∎ Q.E.D.

## Corollaries

* **Cor. IX.3.1** - A Deprecated API version responds with `Deprecation` and `Sunset` headers on every response, including error responses, so that a consumer that only ever sees failures is still informed.
* **Cor. IX.3.2** - The notice carries the successor major and the migration guide link; a notice without a successor is invalid, since a consumer cannot be asked to move to nowhere.

## Construction

* OpenAPI 3.1: `deprecated: true` on each operation of the major; `x-lifecycle: {stage: deprecated, deprecatedOn, sunsetOn, successor, migrationGuide}` at document level.
* ASP.NET Core middleware `DeprecationHeadersMiddleware` in `Company.Web.Contracts` emitting `Deprecation: @<unix-seconds>` (RFC 9745), `Sunset: <HTTP-date>` (RFC 8594) and `Link: <guide>; rel="deprecation"` for every route under the deprecated prefix, configured from the OpenAPI `x-lifecycle` block.
* EventBridge Schema Registry: schema tag `lifecycle=deprecated`, `sunset=<date>`; the same fields in the repository-side registry file of Prop. III.4.
* JSON Schema: `"$comment": "Deprecated 2026-09-20, sunset 2027-09-20, successor /schemas/v2/money.json"` plus `"x-lifecycle": {...}` matching the OpenAPI shape.
* Notification: scheduled Lambda `contract-lifecycle-notifier` reads the catalogue (Prop. IX.8) and the consumer registry (Prop. IX.4), emits event `company.platform.contract-deprecated.v1` on the platform bus and sends e-mail via Amazon SES to each registered consumer's owning team.
* Spectral rule `deprecated-requires-lifecycle`: an operation with `deprecated: true` must sit in a document with a complete `x-lifecycle` block.

## Conformance

Spectral rule `deprecated-requires-lifecycle`; contract test asserting the presence and agreement of `Deprecation` and `Sunset` headers on a Deprecated API major; catalogue lint asserting that every Deprecated entry has a notice with all fields of Def. IX.5.

## Scholium

RFC 9745 replaced the earlier `Deprecation` header draft; its value is an `@`-prefixed Unix timestamp, not an HTTP-date, while RFC 8594 `Sunset` uses an HTTP-date. Both are emitted; the middleware owns the formatting so no team writes it by hand.
