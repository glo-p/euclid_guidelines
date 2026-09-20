# Prop. IV.3 - Money is a decimal string and an ISO 4217 currency

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Every monetary value in a contract MUST conform to [`money.json`](../schemas/shared/v1/money.json): an object `{ amount, currency }` where `amount` is a decimal string with a scale no greater than the currency's ISO 4217 minor units and `currency` is the ISO 4217 alphabetic code. A bare number MUST NOT represent money. Arithmetic across currencies MUST NOT be implied by a contract; a conversion is a separate concept with a rate, a source and a time. Rounding, where an operation performs it, MUST be stated in the contract as *half-even* unless the domain (tax law, for example) requires otherwise, and the rule MUST be named.

## Given

[Def. IV.5](../definitions.md#Def.%20IV.5%20-%20Shape), [Post. II.3](../../02-api-guidelines/postulates.md#Post.%20II.3%20-%20JSON%20Is%20the%20Representation), [Post. II.5](../../02-api-guidelines/postulates.md#Post.%20II.5%20-%20Consumers%20We%20Do%20Not%20Deploy), [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape), [Prop. II.11](../../02-api-guidelines/propositions/11-json-conventions.md).

## Demonstration

Money has one shape everywhere ([CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape)). JSON numbers are IEEE doubles in every JavaScript consumer ([Post. II.5](../../02-api-guidelines/postulates.md#Post.%20II.5%20-%20Consumers%20We%20Do%20Not%20Deploy)) and `0.1 + 0.2 ≠ 0.3` there; a string survives the round trip exactly and maps to `decimal` in .NET and to arbitrary-precision libraries in TypeScript. An amount without a currency is not a quantity of money, so the two travel together. Scale is a property of the currency, so it is validated against the code rather than fixed at two. Rounding and conversion change values; by [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit) they must be declared, not assumed. ∎ Q.E.D.

## Corollaries

* **Cor. IV.3.1** - Percentages, rates and quantities are not `Money`; they are decimal strings with their own declared scale and unit.
* **Cor. IV.3.2** - A `Money` with `amount: "0"` and a `Money` that is absent mean different things ([Prop. II.11](../../02-api-guidelines/propositions/11-json-conventions.md) rule 6); "free" is `"0.00"`.
* **Cor. IV.3.3** - Sorting or filtering by `amount` ([Prop. II.6](../../02-api-guidelines/propositions/06-filtering-and-sorting.md)) is defined only within one `currency`.

## Construction

```csharp
public readonly record struct Money(decimal Amount, Currency Currency)
{
    public static Money Parse(string amount, string currency) { /* validates scale vs Currency.MinorUnits */ }
    public Money Add(Money other) => Currency == other.Currency ? this with { Amount = Amount + other.Amount }
                                     : throw new CurrencyMismatchException(Currency, other.Currency);
    public Money Round(MidpointRounding mode = MidpointRounding.ToEven) => this with { Amount = decimal.Round(Amount, Currency.MinorUnits, mode) };
}
// JSON: { "amount": "120.00", "currency": "GBP" } via MoneyJsonConverter (writes Amount.ToString("F{minor}", Invariant))
```

`Currency` is a value object backed by the ISO 4217 reference dataset ([Prop. VII.4](../../07-data-ownership/propositions/04-reference-data-distribution.md)). TypeScript: `interface Money { amount: string; currency: string }` with helpers built on `big.js`; formatting via `Intl.NumberFormat` ([Prop. XI.10](../../11-frontend-integration/propositions/10-client-side-formatting.md)).

## Conformance

Spectral `money-is-shared-ref` (properties named `*Amount`, `*Price`, `*Total`, `*Cost`, `*Fee` and any property with `format: money` must `$ref` `money.json`; no `type: number` property may have such a name). Catalogue CI validates examples.

## Scholium

The alternative of integer minor units (`12000` for £120.00) is exact and common (Stripe). It was not chosen because the scale is then implicit in the currency and every consumer must look it up before displaying anything, and because a three-decimal currency next to a zero-decimal one is a recurring source of bugs. The decimal string carries its own scale. This deserves its own ADR; see the Book's open questions.
