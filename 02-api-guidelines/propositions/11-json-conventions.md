# Prop. II.11 - JSON representation conventions

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Representations MUST follow these conventions:

1. Field names are camelCase ASCII (`purchaseOrderId`), never snake_case or PascalCase.
2. Identifiers are strings conforming to `Identifier` ([Prop. IV.2](../../04-shared-schemas/propositions/02-identifier.md)); never numbers.
3. Timestamps are strings in RFC 3339 with UTC (`Z`) and millisecond precision: `2026-09-20T14:03:00.000Z`; dates without time are `YYYY-MM-DD`; durations are ISO 8601 (`PT15M`). All conform to [Prop. IV.4](../../04-shared-schemas/propositions/04-temporal.md).
4. Monetary values use the shared `Money` object ([Prop. IV.3](../../04-shared-schemas/propositions/03-money.md)); never a bare number.
5. Enumerations are strings in kebab-case (`awaiting-payment`); never integers.
6. Absent and `null` are distinct: `null` on output means "known to be empty"; absent means "not applicable or not selected". On input, `null` in a Merge Patch clears a field.
7. Booleans are booleans, never `"true"`, `0/1` or `"Y"`.
8. Arrays are never `null`; an empty array is `[]`.
9. Unknown fields on input MUST be ignored, not rejected ([CN 6](../../01-foundations/common-notions.md#CN%206%20-%20The%20Producer%20Pays%20for%20Stability%3B%20the%20Consumer%20Pays%20for%20Tolerance)), unless the operation declares `additionalProperties: false` for a security reason.
10. Numbers that are not counts or measurements (account numbers, postcodes, percentages with a fixed scale) are strings.
11. Top-level bodies are objects, never arrays or scalars.

## Given

[Post. II.3](../postulates.md#Post.%20II.3%20-%20JSON%20Is%20the%20Representation), [Post. II.5](../postulates.md#Post.%20II.5%20-%20Consumers%20We%20Do%20Not%20Deploy), [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape), [CN 6](../../01-foundations/common-notions.md#CN%206%20-%20The%20Producer%20Pays%20for%20Stability%3B%20the%20Consumer%20Pays%20for%20Tolerance), [Prop. IV.2](../../04-shared-schemas/propositions/02-identifier.md), [IV.3](../../04-shared-schemas/propositions/03-money.md), [IV.4](../../04-shared-schemas/propositions/04-temporal.md), [IV.9](../../04-shared-schemas/propositions/09-open-enumerations.md).

## Demonstration

Each rule removes a choice that would otherwise be made differently by different teams, which [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape) identifies as pure cost. Rules 2, 3, 4 and 5 defer to shared schemas so that a consumer parses each concept once. Rule 6 makes a semantic distinction the contract can rely on ([CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit)) and makes Merge Patch ([Prop. II.3](03-methods-and-status-codes.md)) well defined. Rule 9 is the consumer's half of [CN 6](../../01-foundations/common-notions.md#CN%206%20-%20The%20Producer%20Pays%20for%20Stability%3B%20the%20Consumer%20Pays%20for%20Tolerance) and is what allows compatible change ([Def. I.15](../../01-foundations/definitions.md#Def.%20I.15%20-%20Compatible%20Change)) to be safe. Rule 10 exists because JavaScript consumers ([Post. II.5](../postulates.md#Post.%20II.5%20-%20Consumers%20We%20Do%20Not%20Deploy)) lose precision on large integers and leading zeros. Rule 11 leaves room to add `pageInfo` or `metadata` later without a breaking change. ∎ Q.E.D.

## Corollaries

* **Cor. II.11.1** - JSON number is used only for `integer` counts and for measurements with declared units, and never for money.
* **Cor. II.11.2** - Time zones of *display* are a front-end concern ([Prop. XI.10](../../11-frontend-integration/propositions/10-client-side-formatting.md)); APIs never emit local times.

## Construction

`System.Text.Json` options in the project template:

```csharp
new JsonSerializerOptions(JsonSerializerDefaults.Web)
{
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
    DictionaryKeyPolicy = JsonNamingPolicy.CamelCase,
    DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull,   // absent vs null: opt in with [JsonInclude]
    NumberHandling = JsonNumberHandling.Strict,
    Converters = { new JsonStringEnumConverter(JsonNamingPolicy.KebabCaseLower),
                   new UlidJsonConverter(), new MoneyJsonConverter(), new Rfc3339Converter() }
};
```

`UnmappedMemberHandling` is left at `Skip` (rule 9). Converters ship in `Company.Contracts.Shared`.

## Conformance

Spectral: `property-names-camel-case`, `no-integer-enums`, `date-time-format` (string fields named `*At` have `format: date-time`), `money-is-shared-ref`, `id-fields-are-strings`, `top-level-object`. Serializer options are asserted by a unit test in the template.
