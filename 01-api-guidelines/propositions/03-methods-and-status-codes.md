# Prop. II.3 - Methods and status codes carry their HTTP meaning

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Operations MUST use the method whose RFC 9110 semantics match the operation, and MUST return the status codes in the table below for the conditions listed. An operation MUST NOT return `200` with a body that describes a failure. `PATCH` bodies MUST be JSON Merge Patch (RFC 7386) with media type `application/merge-patch+json`.

| Method | Meaning | Safe | Idempotent | Success codes |
|---|---|---|---|---|
| `GET` | read | yes | yes | `200`; `304` on conditional |
| `HEAD` | read headers | yes | yes | `200` |
| `POST` | create item in collection, or perform an action | no | no (unless Idempotency-Key, Prop. II.8) | `201` + `Location` on create; `200` for an action with a result; `202` for long-running (Prop. II.15) |
| `PUT` | replace whole representation | no | yes | `200` with body, or `204` |
| `PATCH` | partial update | no | no | `200` with body |
| `DELETE` | remove | no | yes | `204`; `200` if a body is returned |

| Condition | Code |
|---|---|
| Malformed request (syntax, schema validation) | `400` |
| No or invalid credentials | `401` |
| Valid credentials, not permitted | `403` |
| Resource does not exist, or exists in another tenant | `404` |
| Method not supported on this path | `405` |
| Unacceptable `Accept` header | `406` |
| Conflict with current state (duplicate, state machine violation) | `409` |
| `If-Match` failed | `412` |
| Unsupported `Content-Type` | `415` |
| Well-formed but semantically invalid (business rule) | `422` |
| Rate limited | `429` |
| Unhandled server error | `500` |
| Upstream dependency failed | `502` / `504` |
| Deliberately unavailable (maintenance, shedding) | `503` |

## Given

Post. II.1, Post. I.4, Def. I.16, Def. II.8, CN 1, CN 3.

## Demonstration

Post. II.1 fixes the meaning of methods and codes; a producer that uses them otherwise is not speaking HTTP, and generic clients, caches, the edge and observability tooling (which all assume HTTP) will misbehave. Under Post. I.4 a client must decide whether to retry; the safe/idempotent columns are exactly the information it needs, so the method must be honest about them (Def. I.16). Returning `200` for a failure hides the failure from every layer that reads status codes, contradicting CN 3. Two partial-update grammars would violate CN 5, so one is chosen; Merge Patch is chosen because it is the representation itself with fields omitted, which any consumer can produce without a library. ∎ Q.E.D.

## Corollaries

* **Cor. II.3.1** - `404` is returned for a resource that exists but belongs to another tenant, never `403`, so that existence is not disclosed across tenants (Book V).
* **Cor. II.3.2** - `422` is for requests that pass schema validation but fail a business rule; `400` is for requests that fail schema validation. The Problem `type` (Prop. II.4) distinguishes the rule.
* **Cor. II.3.3** - `GET` and `DELETE` never have request bodies.

## Construction

* ASP.NET Core: enable `ProblemDetails` services (Prop. II.4) so every listed condition produces the right code with a Problem body.
* Model validation → `400` via `ValidationProblemDetails`; business-rule failures → `422` via a domain exception mapped in a single exception filter.
* Merge Patch: `System.Text.Json` deserialise into a `JsonElement`/dictionary and apply to the current state; or use the `Morcatko.AspNetCore.JsonMergePatch` package.

## Conformance

Spectral rules: `operation-success-codes` (each method's allowed success set), `operation-4xx-problem` (every 4xx/5xx response has `application/problem+json`), `patch-merge-patch-media-type`, `get-no-request-body`, `post-create-returns-201`.

## Scholium

The one genuinely debatable row is `POST` for actions. It is the least worst option: `PUT` is idempotent, which an action often is not; inventing a method is not HTTP. See Prop. II.2 for the sub-resource form that keeps it honest.
