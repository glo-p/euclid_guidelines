# Prop. II.4 - Every error is a Problem Details document

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Every response with status `4xx` or `5xx` MUST have media type `application/problem+json` and a body conforming to the shared schema `ProblemDetails` ([Prop. IV.5](../../04-shared-schemas/propositions/05-problem-details.md)). The `type` member MUST be an absolute URI under `https://problems.company.com/` that is stable across versions and listed in the API's contract. The `traceId` extension MUST be present. Validation errors MUST list each failing field in the `errors` extension. The body MUST NOT contain stack traces, internal identifiers, SQL, or the names of internal systems.

## Given

[Def. II.8](../definitions.md#Def.%20II.8%20-%20Problem), [Post. II.1](../postulates.md#Post.%20II.1%20-%20HTTP%20Is%20Authoritative), [Post. II.5](../postulates.md#Post.%20II.5%20-%20Consumers%20We%20Do%20Not%20Deploy), [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape), [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated), [Prop. IV.5](../../04-shared-schemas/propositions/05-problem-details.md), [Prop. II.3](03-methods-and-status-codes.md).

## Demonstration

Consumers we do not deploy ([Post. II.5](../postulates.md#Post.%20II.5%20-%20Consumers%20We%20Do%20Not%20Deploy)) must handle errors we have not yet thought of; they can only do so if every error has one shape ([CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape)) and a machine-readable discriminator that does not depend on prose ([Def. II.8](../definitions.md#Def.%20II.8%20-%20Problem)). RFC 9457 already provides that shape and its `type` URI is the discriminator; re-inventing it would violate [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape). The `type` must be listed in the contract, otherwise by [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit) the consumer cannot rely on it. Under [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated) an error the operator cannot correlate with a trace is invisible, hence `traceId`. Internal detail in an error body is disclosure to a party we do not control ([Post. II.5](../postulates.md#Post.%20II.5%20-%20Consumers%20We%20Do%20Not%20Deploy)) and therefore excluded. ∎ Q.E.D.

## Corollaries

* **Cor. II.4.1** - The set of `type` URIs an operation may return is part of its contract; adding one is a compatible change, removing one is breaking ([Def. I.14](../../01-foundations/definitions.md#Def.%20I.14%20-%20Breaking%20Change)).
* **Cor. II.4.2** - Each `type` URI resolves to a human-readable page in the problem registry describing the condition, the status code, and the remedy. The registry lives in the catalogue repository ([Book IV](../../04-shared-schemas/README.md)).
* **Cor. II.4.3** - `title` is constant per `type`; `detail` may vary per occurrence and is safe to display to an end user; neither is localised by the API (localisation is a front-end concern, [Book XI](../../11-frontend-integration/README.md)).

## Construction

Example body:

```json
{
  "type": "https://problems.company.com/validation-failed",
  "title": "The request failed validation.",
  "status": 400,
  "detail": "Two fields are invalid.",
  "instance": "/v1/orders",
  "traceId": "4bf92f3577b34da6a3ce929d0e0e4736",
  "errors": [
    { "field": "lines[0].quantity", "code": "range", "message": "Must be at least 1." },
    { "field": "currency", "code": "invalid-currency", "message": "'XXX' is not an ISO 4217 code." }
  ]
}
```

ASP.NET Core:

```csharp
builder.Services.AddProblemDetails(o => o.CustomizeProblemDetails = ctx =>
{
    ctx.ProblemDetails.Extensions["traceId"] = Activity.Current?.TraceId.ToString()
        ?? ctx.HttpContext.TraceIdentifier;
    ctx.ProblemDetails.Type ??= ProblemTypes.ForStatus(ctx.ProblemDetails.Status);
});
app.UseExceptionHandler();   // maps unhandled -> 500 problem, no stack trace outside Development
app.UseStatusCodePages();    // maps bare 404/405 -> problem
```

Domain errors: one `DomainException(ProblemType type, string detail)` hierarchy mapped by a single `IExceptionHandler` to `Results.Problem(...)`. `ProblemTypes` is a static class in `Company.Contracts.Shared` ([Prop. IV.1](../../04-shared-schemas/propositions/01-catalogue-and-distribution.md)) so URIs are shared, not retyped.

## Conformance

Spectral `operation-4xx-problem` (every error response uses the shared `$ref` and `application/problem+json`). Integration test in the project template asserts that an unhandled exception yields `500` + Problem with `traceId` and no `exception` member outside `Development`.

## Scholium

Common `type` values seeded in the registry: `validation-failed`, `not-found`, `conflict`, `precondition-failed`, `unauthorised`, `forbidden`, `rate-limited`, `unsupported-media-type`, `internal`, `upstream-unavailable`, `business-rule-violated` (with `errors[].code` naming the rule). APIs add domain-specific types under their own segment, e.g. `https://problems.company.com/orders/order-already-shipped`.
