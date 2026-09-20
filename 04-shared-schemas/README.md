# Book IV - Shared Schemas

Book IV is the catalogue: the small set of concepts that every API and every event expresses in exactly one shape, and the mechanism by which that shape reaches every team as a package rather than as a document. It is one of the three *detailed* books. The schemas in [`schemas/`](schemas/) are the normative artefacts; the propositions explain and derive them.

## Contents

| File | Contents |
|---|---|
| [`definitions.md`](definitions.md) | `Def. IV.1` - `Def. IV.8`: catalogue, schema id, package, generated type, open enumeration, … |
| [`postulates.md`](postulates.md) | `Post. IV.1` - `Post. IV.5`: JSON Schema 2020-12, `$id` URIs, generation targets, … |
| [`schemas/`](schemas/README.md) | The JSON Schema documents, `shared/v1/*.json`. |

### Propositions

| # | Title | Level | Schema | File |
|---|---|---|---|---|
| IV.1 | The catalogue is one repository and reaches teams as packages | MUST | | [`01-catalogue-and-distribution.md`](propositions/01-catalogue-and-distribution.md) |
| IV.2 | Identifiers are ULID strings | MUST | `identifier.json` | [`02-identifier.md`](propositions/02-identifier.md) |
| IV.3 | Money is a decimal string and an ISO 4217 currency | MUST | `money.json` | [`03-money.md`](propositions/03-money.md) |
| IV.4 | Temporal values are RFC 3339 strings | MUST | `timestamp.json`, `date.json`, `duration.json`, `period.json` | [`04-temporal.md`](propositions/04-temporal.md) |
| IV.5 | Errors are Problem Details with `traceId` and `errors` | MUST | `problem-details.json` | [`05-problem-details.md`](propositions/05-problem-details.md) |
| IV.6 | Collections are a `Page` with `PageInfo` | MUST | `page.json`, `page-info.json` | [`06-page.md`](propositions/06-page.md) |
| IV.7 | Resources carry `ResourceMetadata`; actions carry an `Actor` | MUST | `resource-metadata.json`, `actor.json` | [`07-resource-metadata-and-actor.md`](propositions/07-resource-metadata-and-actor.md) |
| IV.8 | Messages are an `EventEnvelope` | MUST | `event-envelope.json` | [`08-event-envelope.md`](propositions/08-event-envelope.md) |
| IV.9 | Enumerations are open unless proven closed | MUST | (pattern) | [`09-open-enumerations.md`](propositions/09-open-enumerations.md) |
| IV.10 | Long-running work is an `Operation` | MUST | `operation.json` | [`10-operation.md`](propositions/10-operation.md) |
| IV.11 | Health is a `Health` report | MUST | `health.json` | [`11-health.md`](propositions/11-health.md) |
| IV.12 | Security-relevant actions are an `AuditEvent` | MUST | `audit-event.json` | [`12-audit-event.md`](propositions/12-audit-event.md) |

## How a team uses the catalogue

1. Reference a schema by absolute `$id` from an OpenAPI document or an event schema (`$ref: "https://schemas.company.com/shared/v1/money.json"`).
2. Add the package: `Company.Contracts.Shared` (NuGet) or `@company/contracts` (npm).
3. Use the generated type (`Money`, `Page<T>`, `ProblemDetails`, `Ulid`, …). Never declare a local type for a catalogue concept (CN 5).
4. To propose a new shared concept or a change: pull request to the catalogue repository with the schema, examples, and an ADR when the change is a new major (Book IX, Book X).

## Open questions

* Whether `Money.amount` should be a decimal string (current) or an integer in minor units. See the scholium of IV.3 for the argument; the decision is recorded in ADR-0001 only implicitly and deserves its own ADR.
* Whether to add `Address`, `PersonName` and `ContactPoint` as shared schemas in v1 or wait for two concrete consumers (Cor. III.10.1 applied to schemas). Current text: wait.
* Whether TypeScript generation targets `zod` schemas as well as plain types.
