# Prop. III.1 - Every message is a CloudEvent

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Every event and command MUST be a CloudEvents 1.0 message in JSON structured format conforming to the shared schema `EventEnvelope` (Prop. IV.8). The following attributes are required beyond the CloudEvents minimum:

| Attribute | Kind | Value |
|---|---|---|
| `specversion` | core | `"1.0"` |
| `id` | core | ULID, unique per event; the idempotency key of the consumer (Prop. III.6) |
| `source` | core | `/<context>/<service>` e.g. `/orders/orders-api` |
| `type` | core | per Prop. III.2, e.g. `com.company.orders.order.placed.v1` |
| `time` | core | RFC 3339 UTC, the time the fact was committed |
| `subject` | core | the ordering key (Def. III.15): the aggregate's identifier |
| `datacontenttype` | core | `application/json` |
| `dataschema` | core | absolute `$id` of the registered payload schema (Prop. III.4) |
| `data` | core | the payload |
| `traceparent`, `tracestate` | extension | W3C Trace Context (Def. III.17) |
| `causationid` | extension | `id` of the message that caused this one, or absent for a root cause |
| `sequence` | extension | per Def. III.16, string of digits |
| `tenantid` | extension | tenant (Def. I.23), or absent for platform events |
| `dataclassification` | extension | highest classification present in `data` (Prop. V.5) |

On EventBridge the message is placed in `detail` whole; `detail-type` MUST equal `type` and `source` MUST equal the envelope `source`, so that rules can match without inspecting `detail`.

## Given

Def. I.11, Def. I.22, Def. III.3, Def. III.15, III.16, III.17, Post. III.1, Post. III.4, CN 3, CN 5, CN 7, Prop. IV.8, Prop. VI.2.

## Demonstration

A message carries envelope and payload separately (Def. I.11). Every consumer must be able to identify, deduplicate, order, trace and validate a message before it can interpret the payload; those five needs correspond to `id`, `subject`+`sequence`, `traceparent`, and `dataschema`. One envelope for all messages is CN 5; CloudEvents is the existing standard with the attributes we need and library support in .NET, so inventing our own would violate CN 5 twice. Since the transport does not order (Post. III.4), the order must be in the envelope (`subject`, `sequence`). Since the event is invisible to operators without correlation (CN 7), `traceparent` is required. Duplicating `type` and `source` into EventBridge's own fields lets rules route without parsing, which is what keeps the bus cheap. ∎ Q.E.D.

## Corollaries

* **Cor. III.1.1** - Consumers MUST ignore unknown envelope extensions (CN 6).
* **Cor. III.1.2** - `id` is minted by the publisher at outbox-write time (Prop. III.5), so a re-published outbox row has the same `id`.
* **Cor. III.1.3** - Commands (Prop. III.11) use the same envelope with `type` in the imperative form.

## Construction

```json
{
  "specversion": "1.0",
  "id": "01J8QJ4Z9V6K3P2M1N0RSTVWXY",
  "source": "/orders/orders-api",
  "type": "com.company.orders.order.placed.v1",
  "time": "2026-09-20T14:03:00.000Z",
  "subject": "01J8QJ2A7H5F4D3C2B1A0ZYXWV",
  "datacontenttype": "application/json",
  "dataschema": "https://schemas.company.com/orders/v1/order-placed.json",
  "traceparent": "00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01",
  "causationid": "01J8QJ4X1B2C3D4E5F6G7H8J9K",
  "sequence": "7",
  "tenantid": "01J8Q000000000000000TENANT",
  "dataclassification": "internal",
  "data": { "orderId": "01J8QJ2A7H5F4D3C2B1A0ZYXWV", "total": { "amount": "120.00", "currency": "GBP" } }
}
```

.NET: `CloudNative.CloudEvents` + `CloudNative.CloudEvents.SystemTextJson`; the platform package (`Company.Platform.Messaging`) exposes `CloudEventFactory.Create<TData>(...)` which fills every required attribute from ambient context (`Activity.Current`, the tenant accessor, the aggregate).

## Conformance

JSON Schema validation of the whole message against `EventEnvelope` in the publisher's unit tests and in the consumer's inbound pipeline (Post. III.7). EventBridge rule on the bus routes any event whose `detail` fails the envelope schema to a platform DLQ and alerts (Prop. VIII.2).
