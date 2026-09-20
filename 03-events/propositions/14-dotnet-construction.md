# Prop. III.14 - The .NET construction of a conforming producer and consumer

| | |
|---|---|
| **Level** | SHOULD |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

To construct a publisher and a consumer satisfying [Prop. III.1](01-envelope.md) to [III.13](13-archive-and-replay.md) from a single starting point. Every service that publishes or consumes SHOULD use `Company.Platform.Messaging` and the `company-consumer` template (`dotnet new company-consumer`), and SHOULD host consumers on AWS Lambda with an SQS event source ([Prop. VIII.3](../../08-infrastructure-aws/propositions/03-compute-choice.md)) or, where the service already runs on ECS, as a hosted `BackgroundService` using the same pipeline. MassTransit MAY be used as the transport abstraction provided the platform pipeline behaviours are installed.

## Given

[Post. I.2](../../01-foundations/postulates.md#Post.%20I.2%20-%20Language), [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification), [Post. I.7](../../01-foundations/postulates.md#Post.%20I.7%20-%20Contract%20First), [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape), [Prop. III.1](01-envelope.md) to [III.13](13-archive-and-replay.md), [Prop. VIII.3](../../08-infrastructure-aws/propositions/03-compute-choice.md).

## Demonstration (construction)

Let `Company.Platform.Messaging` contain:

| Component | Discharges |
|---|---|
| `CloudEventFactory`, `EventEnvelope` validation, EventBridge mapping (`detail-type`/`source`) | III.1, III.2 |
| Generated payload records from catalogue packages; architecture test isolating `Domain` from `Contracts` | III.3, III.4 |
| `Outbox` (EF Core table, `IOutbox`, relay `BackgroundService`/Lambda, `PutEvents` batching) | III.5 |
| `Inbox` behaviour wrapping every `IEventHandler<T>` | III.6 |
| `LastSequenceStore` and ordering behaviour; `subject`/`sequence` set by the factory from the aggregate | III.7 |
| `PermanentFailureException` → DLQ; transient → visibility backoff; batch item failure reporting for Lambda | III.8 |
| Terraform modules `eventbridge-publisher`, `sqs-consumer`, `sqs-command-queue`, `eventbridge-bus` | III.9, III.11, III.13 |
| Schema validation on receive; `x-data-classification` propagation to the envelope | III.4, III.10 |
| `ICommandSender`; command queue consumer identical to event consumer | III.11 |
| `Company.Platform.Messaging.Testing` harnesses | III.12 |
| `isReplay` flag from `replay-name` | III.13 |
| OpenTelemetry: consumer span linked to producer span via `traceparent`; RED metrics per event type | VI.2, VI.3 |

Each row is checked by the conformance section of the proposition it discharges; therefore a service built from the template and the package, unmodified in those components, satisfies III.1 to III.13. ∎ Q.E.F.

## Construction

Publisher (inside a request handler):

```csharp
public async Task<Order> PlaceAsync(PlaceOrder cmd, CancellationToken ct)
{
    var order = Order.Place(cmd);                       // raises domain event OrderPlaced
    db.Orders.Add(order);
    outbox.Add(events.Create(                           // maps to integration event
        type: EventTypes.Orders.OrderPlacedV1,
        subject: order.Id, sequence: order.Version,
        data: OrderPlacedV1.From(order)));
    await db.SaveChangesAsync(ct);                       // one transaction: state + outbox
    return order;
}
```

Consumer (Lambda, SQS event source, partial batch responses enabled):

```csharp
public sealed class OrderPlacedHandler : IEventHandler<OrderPlacedV1>
{
    public async Task HandleAsync(CloudEvent<OrderPlacedV1> evt, ConsumeContext ctx, CancellationToken ct)
    {
        if (evt.Data.Total.Currency is not ("GBP" or "EUR"))
            throw new PermanentFailureException("unsupported-currency");
        await invoices.CreateDraftAsync(evt.Data, evt.Subject, evt.Sequence, ct);   // upsert keyed by subject
    }
}
// Program.cs
builder.Services.AddCompanyMessaging(o => o.Consumer("billing-orders-consumer"))
    .AddHandler<OrderPlacedV1, OrderPlacedHandler>();
```

The pipeline (validate envelope → validate schema → inbox claim → ordering → handler → commit → ack) is applied by `AddCompanyMessaging`; handlers contain only business logic.

## Conformance

Template repository CI runs every [Book III](../README.md) conformance check; services run them against themselves; template version is recorded in `conformance.json`.

## Scholium

Lambda is the default host for consumers because scaling, retries, partial-batch failure and DLQ are all provided by the SQS event source mapping. ECS-hosted consumers exist for services that already run there and where a cold start on a hot path is unacceptable; they use the same pipeline and a long-polling `BackgroundService`.
