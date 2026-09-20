# Prop. IV.5 - Errors are Problem Details with `traceId` and `errors`

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

The shape of every error body (Prop. II.4) and of every `Operation.error` (Prop. IV.10) MUST be [`problem-details.json`](../schemas/shared/v1/problem-details.json): RFC 9457's `type`, `title`, `status`, `detail`, `instance`, plus the required extension `traceId` (32 hex characters, the W3C trace-id) and the optional extension `errors[]` of `{ field?, code, message }`. `type` MUST be under `https://problems.company.com/` and registered (Cor. IV.1.3). Additional extension members MAY be added by an API provided they are declared in its contract and do not collide with names in this schema.

## Given

Def. II.8, Def. IV.5, Post. II.1, Post. II.5, CN 3, CN 5, CN 7, Prop. II.4.

## Demonstration

Prop. II.4 establishes that every error is a Problem; this proposition fixes the one shape (CN 5) and the two extensions. `traceId` is required because a consumer that cannot quote a trace gives support nothing to find (CN 7). `errors[]` is defined here rather than per API because every front-end (Post. II.5) needs to place a message next to a field, and it needs one way to do so. `code` is machine-readable so that the front-end may localise (Prop. XI.5) without parsing `message`. The extension mechanism is RFC 9457's own (Post. II.1), and declaring extensions in the contract is CN 3. ∎ Q.E.D.

## Corollaries

* **Cor. IV.5.1** - `errors[].field` uses the JSON path of the *request* body as the client sent it, in camelCase with bracket indices (`lines[0].quantity`).
* **Cor. IV.5.2** - A Problem never carries `Money`, `Actor` or other domain data in extensions; it describes the failure, not the state.
* **Cor. IV.5.3** - The registry entry for a `type` states the `status` it is used with; using one `type` with two statuses is a defect.

## Construction

C#: `Microsoft.AspNetCore.Mvc.ProblemDetails` is used directly; `Company.Contracts.Shared` adds `ProblemTypes` (constants), `ProblemDetailsExtensions.WithTraceId()`, `.WithErrors(IEnumerable<FieldError>)`, and a `FieldError(string? Field, string Code, string Message)` record. TypeScript: `interface ProblemDetails` plus `isProblemDetails(x): x is ProblemDetails` guard and the `problemMessages` map.

## Conformance

Catalogue CI validates examples; Spectral `operation-4xx-problem` (Prop. II.4) enforces the `$ref`. Registry CI: every `type` referenced in any OpenAPI document in the estate has a page in `problems/`.
