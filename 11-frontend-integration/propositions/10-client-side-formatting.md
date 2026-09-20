# Prop. XI.10 - Money, dates and identifiers are formatted client-side from canonical shapes

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

An API MUST return money, temporal values and identifiers only in their canonical shared shapes ([Prop. IV.3](../../04-shared-schemas/propositions/03-money.md), [Prop. IV.4](../../04-shared-schemas/propositions/04-temporal.md), [Prop. IV.2](../../04-shared-schemas/propositions/02-identifier.md)) and MUST NOT return pre-formatted display strings for them. A front-end MUST format these values for display from the canonical shape, in the user's locale and time zone, using the formatting helpers of `@company/contracts` or the platform's own internationalisation facilities.

## Given

* [Def. I.8](../../01-foundations/definitions.md#Def.%20I.8%20-%20Representation), [Def. I.13](../../01-foundations/definitions.md#Def.%20I.13%20-%20Shared%20Schema), [Def. I.18](../../01-foundations/definitions.md#Def.%20I.18%20-%20Identifier), [Def. XI.1](../definitions.md#Def.%20XI.1%20-%20Front-end)
* [Post. XI.1](../postulates.md#Post.%20XI.1%20-%20Mixed%20Technology), [Post. XI.4](../postulates.md#Post.%20XI.4%20-%20Common%20Language)
* [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape)
* [Prop. II.11](../../02-api-guidelines/propositions/11-json-conventions.md), [Prop. IV.2](../../04-shared-schemas/propositions/02-identifier.md), [Prop. IV.3](../../04-shared-schemas/propositions/03-money.md), [Prop. IV.4](../../04-shared-schemas/propositions/04-temporal.md), [Prop. XI.2](02-shared-schema-types.md)

## Demonstration

The shared schemas fix one shape for each concept: Money as a decimal string with an ISO 4217 currency ([Prop. IV.3](../../04-shared-schemas/propositions/03-money.md)), timestamps as RFC 3339 UTC ([Prop. IV.4](../../04-shared-schemas/propositions/04-temporal.md)), identifiers as ULID strings ([Prop. IV.2](../../04-shared-schemas/propositions/02-identifier.md)). A display string is a second shape for the same concept, which [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape) forbids, and it embeds a locale and time zone the producer cannot know: the same representation ([Def. I.8](../../01-foundations/definitions.md#Def.%20I.8%20-%20Representation)) is served to users in different places. Formatting is therefore a property of the front-end, the only party that knows the user. Because the canonical shapes come from one package ([Prop. XI.2](02-shared-schema-types.md)) and TypeScript is common ([Post. XI.4](../postulates.md#Post.%20XI.4%20-%20Common%20Language)), the helpers that format them can be shared once and behave identically in every front-end. ∎ Q.E.D.

## Corollaries

* **Cor. XI.10.1** - Arithmetic on Money in a front-end is performed on the decimal string with a decimal library, never on a JavaScript `number`; display rounding happens last.
* **Cor. XI.10.2** - Plain dates ([Prop. IV.4](../../04-shared-schemas/propositions/04-temporal.md)) are displayed without time-zone conversion; timestamps are converted to the user's zone. Conflating the two is a defect.

## Construction

* Helpers: `@company/contracts/format` exporting `formatMoney(money, locale)`, `formatTimestamp(ts, locale, timeZone)`, `formatDate(date, locale)`, `formatPeriod(period, locale)`, `shortId(ulid)`; implemented on `Intl.NumberFormat` and `Intl.DateTimeFormat`, no framework dependency.
* Decimal arithmetic: `decimal.js` or `big.js` pinned in the package; Money values never pass through `parseFloat`.
* Time zone: the user's zone from the browser (`Intl.DateTimeFormat().resolvedOptions() .timeZone`) or the user's profile; never assumed to be UTC or the server's.
* Identifiers: displayed verbatim or abbreviated for layout by a shared helper; never parsed for meaning ([Def. I.18](../../01-foundations/definitions.md#Def.%20I.18%20-%20Identifier)).
* Input: forms submit canonical shapes; parsing user input into Money or dates uses the package's `parseMoney` and `parseDate` helpers.

## Conformance

OpenAPI linter rule (Spectral) on every contract: no property named `*Display`, `*Formatted`, `*Text` typed as string whose sibling is a Money, Timestamp, Date or Identifier; no string property with a `format` of `currency-display` or similar. Front-end lint rule: `parseFloat`, `Number(...)` and `toFixed` forbidden on values typed `Money`.

## Scholium

Producers are tempted to add `amountDisplay: "£1,234.56"` because one front-end asked for it. The next front-end is in a different locale, and the field is now a bug. The canonical shape plus a shared formatter costs each front-end one import. This proposition binds APIs as well as front-ends; it is listed here because the reason for it lives on the front-end side.
