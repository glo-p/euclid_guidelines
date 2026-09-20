# Prop. XI.10 - Money, dates and identifiers are formatted client-side from canonical shapes

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

An API MUST return money, temporal values and identifiers only in their canonical shared shapes (Prop. IV.3, Prop. IV.4, Prop. IV.2) and MUST NOT return pre-formatted display strings for them. A front-end MUST format these values for display from the canonical shape, in the user's locale and time zone, using the formatting helpers of `@company/contracts` or the platform's own internationalisation facilities.

## Given

* Def. I.8, Def. I.13, Def. I.18, Def. XI.1
* Post. XI.1, Post. XI.4
* CN 3, CN 5
* Prop. II.11, Prop. IV.2, Prop. IV.3, Prop. IV.4, Prop. XI.2

## Demonstration

The shared schemas fix one shape for each concept: Money as a decimal string with an ISO 4217 currency (Prop. IV.3), timestamps as RFC 3339 UTC (Prop. IV.4), identifiers as ULID strings (Prop. IV.2). A display string is a second shape for the same concept, which CN 5 forbids, and it embeds a locale and time zone the producer cannot know: the same representation (Def. I.8) is served to users in different places. Formatting is therefore a property of the front-end, the only party that knows the user. Because the canonical shapes come from one package (Prop. XI.2) and TypeScript is common (Post. XI.4), the helpers that format them can be shared once and behave identically in every front-end. ∎ Q.E.D.

## Corollaries

* **Cor. XI.10.1** - Arithmetic on Money in a front-end is performed on the decimal string with a decimal library, never on a JavaScript `number`; display rounding happens last.
* **Cor. XI.10.2** - Plain dates (Prop. IV.4) are displayed without time-zone conversion; timestamps are converted to the user's zone. Conflating the two is a defect.

## Construction

* Helpers: `@company/contracts/format` exporting `formatMoney(money, locale)`, `formatTimestamp(ts, locale, timeZone)`, `formatDate(date, locale)`, `formatPeriod(period, locale)`, `shortId(ulid)`; implemented on `Intl.NumberFormat` and `Intl.DateTimeFormat`, no framework dependency.
* Decimal arithmetic: `decimal.js` or `big.js` pinned in the package; Money values never pass through `parseFloat`.
* Time zone: the user's zone from the browser (`Intl.DateTimeFormat().resolvedOptions() .timeZone`) or the user's profile; never assumed to be UTC or the server's.
* Identifiers: displayed verbatim or abbreviated for layout by a shared helper; never parsed for meaning (Def. I.18).
* Input: forms submit canonical shapes; parsing user input into Money or dates uses the package's `parseMoney` and `parseDate` helpers.

## Conformance

OpenAPI linter rule (Spectral) on every contract: no property named `*Display`, `*Formatted`, `*Text` typed as string whose sibling is a Money, Timestamp, Date or Identifier; no string property with a `format` of `currency-display` or similar. Front-end lint rule: `parseFloat`, `Number(...)` and `toFixed` forbidden on values typed `Money`.

## Scholium

Producers are tempted to add `amountDisplay: "£1,234.56"` because one front-end asked for it. The next front-end is in a different locale, and the field is now a bug. The canonical shape plus a shared formatter costs each front-end one import. This proposition binds APIs as well as front-ends; it is listed here because the reason for it lives on the front-end side.
