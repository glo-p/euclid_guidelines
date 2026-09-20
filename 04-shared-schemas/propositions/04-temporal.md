# Prop. IV.4 - Temporal values are RFC 3339 strings

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

An instant MUST conform to [`timestamp.json`](../schemas/shared/v1/timestamp.json) (RFC 3339, emitted in UTC with `Z` and millisecond precision). A calendar date MUST conform to [`date.json`](../schemas/shared/v1/date.json). A duration MUST conform to [`duration.json`](../schemas/shared/v1/duration.json) (days and time components only). An interval MUST conform to [`period.json`](../schemas/shared/v1/period.json) (half-open, `[start, end)`). Contracts MUST NOT use Unix epoch numbers, local times without offset, or `.NET`-style `/Date(…)/`. Fields holding an instant SHOULD be named `<verb>At` (`createdAt`, `expiresAt`); fields holding a date SHOULD be named `<noun>Date` (`invoiceDate`, `dueDate`).

## Given

Post. II.3, Post. II.5, CN 3, CN 5, Prop. II.11, Prop. XI.10.

## Demonstration

One shape per temporal concept is CN 5, and the four concepts are genuinely different: an instant is a point on the global timeline, a date is a label that is the same in every zone, a duration is a length, a period is a pair of instants. Conflating them (a date sent as midnight UTC) breaks the moment a consumer is in another zone (Post. II.5). RFC 3339 is the profile of ISO 8601 that every runtime parses without options. UTC on output makes two timestamps comparable as strings; display in local time is the front-end's job (Prop. XI.10). Naming conventions make the kind visible without opening the schema (CN 3). ∎ Q.E.D.

## Corollaries

* **Cor. IV.4.1** - Inputs MAY carry any offset; the producer normalises to UTC and echoes UTC. A producer that needs the original zone (a calendar appointment) stores and exposes an IANA zone name in a separate field (`timeZone: "Europe/London"`).
* **Cor. IV.4.2** - Months and years are not durations; a "monthly" period is expressed as a `Period` per occurrence or a recurrence rule, never as `P1M`.
* **Cor. IV.4.3** - `Period.end` null means open-ended; a `Period` never has `end < start`.

## Construction

.NET: `DateTimeOffset` for instants (converter forces UTC and `fffZ`), `DateOnly` for dates, `TimeSpan` for durations (converter to ISO 8601), `Period` record in `Company.Contracts.Shared`. Never `DateTime` in a contract type. TypeScript: strings at the boundary; `Temporal` or `date-fns` inside.

## Conformance

Spectral: `timestamp-fields-format` (properties named `*At` are `$ref` timestamp), `date-fields-format` (`*Date` are `$ref` date), `no-epoch-numbers` (no integer property named `*At`, `*Time`, `*Timestamp`).
