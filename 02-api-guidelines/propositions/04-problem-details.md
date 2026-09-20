# Prop. II.4 - Every error is a Problem Details document

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Every response with status `4xx` or `5xx` MUST have media type `application/problem+json` and a body conforming to the shared schema `ProblemDetails` (Prop. IV.5). The `type` member MUST be an absolute URI under `https://problems.company.com/` that is stable across versions and listed in the API's contract. The `traceId` extension MUST be present. Validation errors MUST list each failing field in the `errors` extension. The body MUST NOT contain stack traces, internal identifiers, SQL, or the names of internal systems.

## Given

Def. II.8, Post. II.1, Post. II.5, CN 3, CN 5, CN 7, Prop. IV.5, Prop. II.3.

## Demonstration

Consumers we do not deploy (Post. II.5) must handle errors we have not yet thought of; they can only do so if every error has one shape (CN 5) and a machine-readable discriminator that does not depend on prose (Def. II.8). RFC 9457 already provides that shape and its `type` URI is the discriminator; re-inventing it would violate CN 5. The `type` must be listed in the contract, otherwise by CN 3 the consumer cannot rely on it. Under CN 7 an error the operator cannot correlate with a trace is invisible, hence `traceId`. Internal detail in an error body is disclosure to a party we do not control (Post. II.5) and therefore excluded. ∎ Q.E.D.

## Corollaries

* **Cor. II.4.1** - The set of `type` URIs an operation may return is part of its contract; adding one is a compatible change, removing one is breaking (Def. I.14).
* **Cor. II.4.2** - Each `type` URI resolves to a human-readable page in the problem registry describing the condition, the status code, and the remedy. The registry lives in the catalogue repository (Book IV).
* **Cor. II.4.3** - `title` is constant per `type`; `detail` may vary per occurrence and is safe to display to an end user; neither is localised by the API (localisation is a front-end concern, Book XI).

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

Domain errors: one `DomainException(ProblemType type, string detail)` hierarchy mapped by a single `IExceptionHandler` to `Results.Problem(...)`. `ProblemTypes` is a static class in `Company.Contracts.Shared` (Prop. IV.1) so URIs are shared, not retyped.

## Conformance

Spectral `operation-4xx-problem` (every error response uses the shared `$ref` and `application/problem+json`). Integration test in the project template asserts that an unhandled exception yields `500` + Problem with `traceId` and no `exception` member outside `Development`.

## Scholium

Common `type` values seeded in the registry: `validation-failed`, `not-found`, `conflict`, `precondition-failed`, `unauthorised`, `forbidden`, `rate-limited`, `unsupported-media-type`, `internal`, `upstream-unavailable`, `business-rule-violated` (with `errors[].code` naming the rule). APIs add domain-specific types under their own segment, e.g. `https://problems.company.com/orders/order-already-shipped`.
