# Shared schemas

Normative JSON Schema 2020-12 documents. In the catalogue repository these live at the same paths and are served at `https://schemas.company.com/shared/v1/<name>.json`.

| File | Concept | Proposition |
|---|---|---|
| `shared/v1/identifier.json` | ULID identifier | IV.2 |
| `shared/v1/money.json` | Monetary amount with currency | IV.3 |
| `shared/v1/timestamp.json` | RFC 3339 UTC instant | IV.4 |
| `shared/v1/date.json` | Calendar date | IV.4 |
| `shared/v1/duration.json` | ISO 8601 duration | IV.4 |
| `shared/v1/period.json` | Half-open interval of instants | IV.4 |
| `shared/v1/problem-details.json` | RFC 9457 error | IV.5 |
| `shared/v1/page-info.json` | Cursor pagination info | IV.6 |
| `shared/v1/page.json` | Collection envelope | IV.6 |
| `shared/v1/resource-metadata.json` | Created/updated/version | IV.7 |
| `shared/v1/actor.json` | Who performed an action | IV.7 |
| `shared/v1/event-envelope.json` | CloudEvents envelope with company extensions | IV.8 |
| `shared/v1/operation.json` | Long-running operation | IV.10 |
| `shared/v1/health.json` | Health/readiness report | IV.11 |
| `shared/v1/audit-event.json` | Audit event payload | IV.12 |

Validate all examples locally:

```
npx ajv-cli validate --spec=draft2020 -c ajv-formats -r "shared/v1/*.json" -d "shared/v1/*.json" --all-errors
```
