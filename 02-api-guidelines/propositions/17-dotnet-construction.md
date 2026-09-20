# Prop. II.17 - The .NET construction of a conforming API

| | |
|---|---|
| **Level** | SHOULD |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

To construct an API that satisfies [Prop. II.1](01-contract-first.md) to [II.16](16-health-endpoints.md) from a single starting point. Every new .NET API SHOULD be created from the `company-api` template (`dotnet new company-api`) and reference `Company.Contracts.Shared` ([Prop. IV.1](../../04-shared-schemas/propositions/01-catalogue-and-distribution.md)) and `Company.Platform.AspNetCore`. The template is the *construction* of this Book; a service that satisfies the propositions by other means is conforming, but bears the cost of proving it.

## Given

[Post. I.2](../../01-foundations/postulates.md#Post.%20I.2%20-%20Language), [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification), [Post. I.7](../../01-foundations/postulates.md#Post.%20I.7%20-%20Contract%20First), [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape), [Prop. II.1](01-contract-first.md) to [II.16](16-health-endpoints.md).

## Demonstration (construction)

Let the template contain the following, each item discharging the proposition named:

| Component | Discharges |
|---|---|
| `contracts/openapi.yaml`, `.spectral.yaml`, CI jobs `contract-lint`, `contract-diff` | II.1, II.2, II.3, II.7 (gate) |
| `Company.Platform.AspNetCore.AddCompanyApi()` registering: ProblemDetails + `traceId` + exception mapping | II.4 |
| `Page<T>`, `PageInfo`, `Cursor` from `Company.Contracts.Shared` and a `KeysetPaging` helper | II.5 |
| `FilterBinder`, `SortBinder` | II.6 |
| `Asp.Versioning.Http` with URL segment reader and deprecation-header middleware | II.7 |
| `IdempotencyMiddleware` (Redis or table backed) | II.8 |
| `ETagEndpointFilter` and EF Core concurrency token convention | II.9 |
| OpenTelemetry tracing with `traceparent` echo; `ResourceMetadata` on every DTO | II.10 |
| `JsonSerializerOptions` preset with shared converters | II.11 |
| `AddJwtBearer` wired to the IdP, scope policies, resource authorisation handlers | II.12 |
| `AddRateLimiter` with client-id partition and `OnRejected` Problem | II.13 |
| `CacheControlEndpointFilter` defaulting to `private, no-cache`; `304` support | II.14 |
| `Operation` resource, outbox row and completion event scaffolding | II.15 |
| `/health/live`, `/health/ready` via HealthChecks | II.16 |
| Architecture tests (`NetArchTest`/`ArchUnitNET`) asserting no local copy of a catalogue schema, no endpoint outside `/v{n}` except health | [Cor. II.1.1](01-contract-first.md#Corollaries), II.16 |

Each row is checked by the conformance section of the proposition it discharges, and the template's own test suite runs those checks. Therefore a service created from the template and left unmodified in those components satisfies II.1 to II.16. ∎ Q.E.F.

## Construction

```
dotnet new install Company.Templates
dotnet new company-api -n Company.Orders --context orders
```

Produces:

```
Company.Orders/
  contracts/openapi.yaml
  src/Company.Orders.Api/        Program.cs  -> builder.AddCompanyApi(); app.UseCompanyApi();
  src/Company.Orders.Domain/
  src/Company.Orders.Infrastructure/
  tests/Company.Orders.ContractTests/   (generated from openapi.yaml, runs against TestServer)
  tests/Company.Orders.ArchitectureTests/
  infra/                          (Terraform module call, Prop. VIII.9)
  conformance.json                (Prop. X.3)
```

Minimal endpoint using the platform package:

```csharp
group.MapGet("/orders", async (
        [AsParameters] PageRequest page, [AsParameters] OrderFilter filter,
        IOrderReader reader, CancellationToken ct)
    => Results.Ok(await reader.ListAsync(page, filter, ct)))       // returns Page<OrderDto>
   .RequireScope("orders:read")
   .WithName("ListOrders");

group.MapPost("/orders", async (CreateOrder cmd, IOrderWriter writer, CancellationToken ct) =>
{
    var order = await writer.CreateAsync(cmd, ct);
    return Results.Created($"/v1/orders/{order.Id}", order);
}).RequireScope("orders:write").RequireIdempotencyKey();
```

## Conformance

The template repository has its own CI running every [Book II](../README.md) conformance check; a service's CI runs them against the service. Template version is recorded in `conformance.json`.

## Scholium

The template is versioned and services are expected to take template upgrades the way they take NuGet upgrades: regularly and with a diff to read. A service that pins an old template for more than two releases is reported by the conformance index ([Prop. X.3](../../10-governance/propositions/03-conformance-manifest.md)).
