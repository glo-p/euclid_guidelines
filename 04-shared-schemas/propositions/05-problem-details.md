# Prop. IV.5 - Errors are Problem Details with `traceId` and `errors`

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

The shape of every error body ([Prop. II.4](../../02-api-guidelines/propositions/04-problem-details.md)) and of every `Operation.error` ([Prop. IV.10](10-operation.md)) MUST be [`problem-details.json`](../schemas/shared/v1/problem-details.json): RFC 9457's `type`, `title`, `status`, `detail`, `instance`, plus the required extension `traceId` (32 hex characters, the W3C trace-id) and the optional extension `errors[]` of `{ field?, code, message }`. `type` MUST be under `https://problems.company.com/` and registered ([Cor. IV.1.3](01-catalogue-and-distribution.md#Corollaries)). Additional extension members MAY be added by an API provided they are declared in its contract and do not collide with names in this schema.

## Given

[Def. II.8](../../02-api-guidelines/definitions.md#Def.%20II.8%20-%20Problem), [Def. IV.5](../definitions.md#Def.%20IV.5%20-%20Shape), [Post. II.1](../../02-api-guidelines/postulates.md#Post.%20II.1%20-%20HTTP%20Is%20Authoritative), [Post. II.5](../../02-api-guidelines/postulates.md#Post.%20II.5%20-%20Consumers%20We%20Do%20Not%20Deploy), [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape), [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated), [Prop. II.4](../../02-api-guidelines/propositions/04-problem-details.md).

## Demonstration

[Prop. II.4](../../02-api-guidelines/propositions/04-problem-details.md) establishes that every error is a Problem; this proposition fixes the one shape ([CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape)) and the two extensions. `traceId` is required because a consumer that cannot quote a trace gives support nothing to find ([CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated)). `errors[]` is defined here rather than per API because every front-end ([Post. II.5](../../02-api-guidelines/postulates.md#Post.%20II.5%20-%20Consumers%20We%20Do%20Not%20Deploy)) needs to place a message next to a field, and it needs one way to do so. `code` is machine-readable so that the front-end may localise ([Prop. XI.5](../../11-frontend-integration/propositions/05-problem-details-user-messages.md)) without parsing `message`. The extension mechanism is RFC 9457's own ([Post. II.1](../../02-api-guidelines/postulates.md#Post.%20II.1%20-%20HTTP%20Is%20Authoritative)), and declaring extensions in the contract is [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit). ∎ Q.E.D.

## Corollaries

* **Cor. IV.5.1** - `errors[].field` uses the JSON path of the *request* body as the client sent it, in camelCase with bracket indices (`lines[0].quantity`).
* **Cor. IV.5.2** - A Problem never carries `Money`, `Actor` or other domain data in extensions; it describes the failure, not the state.
* **Cor. IV.5.3** - The registry entry for a `type` states the `status` it is used with; using one `type` with two statuses is a defect.

## Construction

C#: `Microsoft.AspNetCore.Mvc.ProblemDetails` is used directly; `Company.Contracts.Shared` adds `ProblemTypes` (constants), `ProblemDetailsExtensions.WithTraceId()`, `.WithErrors(IEnumerable<FieldError>)`, and a `FieldError(string? Field, string Code, string Message)` record. TypeScript: `interface ProblemDetails` plus `isProblemDetails(x): x is ProblemDetails` guard and the `problemMessages` map.

## Conformance

Catalogue CI validates examples; Spectral `operation-4xx-problem` ([Prop. II.4](../../02-api-guidelines/propositions/04-problem-details.md)) enforces the `$ref`. Registry CI: every `type` referenced in any OpenAPI document in the estate has a page in `problems/`.
