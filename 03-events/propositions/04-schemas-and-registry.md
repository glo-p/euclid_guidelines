# Prop. III.4 - Every event type has a registered schema

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Every event type MUST have a JSON Schema 2020-12 document for its `data`, stored in the catalogue repository ([Def. I.24](../../01-foundations/definitions.md#Def.%20I.24%20-%20Catalogue)) under `schemas/<context>/v<major>/<fact>.json`, with an absolute `$id`, an `x-event-type`, an `x-publisher` (service name), an `x-lifecycle` ([Book IX](../../09-versioning-and-deprecation/README.md) stage), and examples. CI MUST publish the schema to the EventBridge Schema Registry and to the `Company.Contracts.<Context>` NuGet and `@company/contracts-<context>` npm packages. Within a major, only compatible changes ([Def. I.15](../../01-foundations/definitions.md#Def.%20I.15%20-%20Compatible%20Change)) are permitted, verified by schema diff in CI. Payloads MUST use shared schemas ([Book IV](../../04-shared-schemas/README.md)) by `$ref` for shared concepts.

## Given

[Def. I.2](../../01-foundations/definitions.md#Def.%20I.2%20-%20Contract), [Def. I.12](../../01-foundations/definitions.md#Def.%20I.12%20-%20Schema), [Def. I.14](../../01-foundations/definitions.md#Def.%20I.14%20-%20Breaking%20Change), [Def. I.15](../../01-foundations/definitions.md#Def.%20I.15%20-%20Compatible%20Change), [Def. I.24](../../01-foundations/definitions.md#Def.%20I.24%20-%20Catalogue), [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification), [Post. I.7](../../01-foundations/postulates.md#Post.%20I.7%20-%20Contract%20First), [Post. III.7](../postulates.md#Post.%20III.7%20-%20Schema%20Validation%20Is%20the%20Check), [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 4](../../01-foundations/common-notions.md#CN%204%20-%20Compatibility%20Is%20Compositional), [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape), [Prop. IV.1](../../04-shared-schemas/propositions/01-catalogue-and-distribution.md), [Prop. IX.1](../../09-versioning-and-deprecation/propositions/01-only-compatible-changes-within-a-major.md).

## Demonstration

An event type is a contract and a contract is machine-readable by definition ([Def. I.2](../../01-foundations/definitions.md#Def.%20I.2%20-%20Contract), [Def. I.12](../../01-foundations/definitions.md#Def.%20I.12%20-%20Schema)). [Post. I.7](../../01-foundations/postulates.md#Post.%20I.7%20-%20Contract%20First) lets us write it first and generate types for both sides. [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification) says an unchecked shape drifts, so the schema must be what tests and inbound validation check ([Post. III.7](../postulates.md#Post.%20III.7%20-%20Schema%20Validation%20Is%20the%20Check)). Compatibility is compositional ([CN 4](../../01-foundations/common-notions.md#CN%204%20-%20Compatibility%20Is%20Compositional)) and decidable by diffing schema documents, which is why the schema, not the C# type, is the source. Shared concepts appear by reference ([CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape)). One registry, one location and one naming rule mean a consumer can find any schema from the `dataschema` attribute alone ([CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit)). ∎ Q.E.D.

## Corollaries

* **Cor. III.4.1** - `data` schemas declare `additionalProperties: true` (the default) so that additive change is compatible and consumers tolerate it ([CN 6](../../01-foundations/common-notions.md#CN%206%20-%20The%20Producer%20Pays%20for%20Stability%3B%20the%20Consumer%20Pays%20for%20Tolerance)).
* **Cor. III.4.2** - Enumerations in payloads are open ([Prop. IV.9](../../04-shared-schemas/propositions/09-open-enumerations.md)) unless the publisher can demonstrate the set is closed by law or by nature.
* **Cor. III.4.3** - The generated C# record for an event payload is the only type a publisher serialises and the only type a consumer deserialises.

## Construction

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://schemas.company.com/orders/v1/order-placed.json",
  "title": "OrderPlaced",
  "x-event-type": "com.company.orders.order.placed.v1",
  "x-publisher": "orders-api",
  "x-lifecycle": "active",
  "type": "object",
  "required": ["orderId", "customerId", "total", "placedAt", "lines"],
  "properties": {
    "orderId":    { "$ref": "https://schemas.company.com/shared/v1/identifier.json" },
    "customerId": { "$ref": "https://schemas.company.com/shared/v1/identifier.json" },
    "total":      { "$ref": "https://schemas.company.com/shared/v1/money.json" },
    "placedAt":   { "$ref": "https://schemas.company.com/shared/v1/timestamp.json" },
    "lines": { "type": "array", "items": { "$ref": "#/$defs/line" } }
  },
  "$defs": { "line": { "type": "object", "required": ["sku", "quantity", "unitPrice"], "properties": {
    "sku": { "type": "string" }, "quantity": { "type": "integer", "minimum": 1 },
    "unitPrice": { "$ref": "https://schemas.company.com/shared/v1/money.json" } } } },
  "examples": [ { "orderId": "01J8QJ2A7H5F4D3C2B1A0ZYXWV", "customerId": "01J8Q000000000000000CSTM01",
    "total": { "amount": "120.00", "currency": "GBP" }, "placedAt": "2026-09-20T14:03:00.000Z",
    "lines": [ { "sku": "ABC-1", "quantity": 2, "unitPrice": { "amount": "60.00", "currency": "GBP" } } ] } ]
}
```

Catalogue CI: `ajv compile` → `json-schema-diff` (breaking gate) → `aws schemas put-schema` (registry `company.<env>`) → code generation (`NJsonSchema` for C#, `json-schema-to-typescript`) → package publish.

## Conformance

Catalogue CI as above. Publisher unit test: every outbox write validates `data` against the schema resolved from `dataschema`. Consumer pipeline: validate on receive, reject to DLQ with reason `schema-invalid`.
