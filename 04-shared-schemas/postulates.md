# Book IV - Postulates

---

## Post. IV.1 - JSON Schema 2020-12

Every schema is a JSON Schema draft 2020-12 document. No other schema language is used for JSON payloads.

## Post. IV.2 - Identity by URI

A schema is identified by its `$id` ([Def. IV.2](definitions.md#Def.%20IV.2%20-%20Schema%20Id)) and nothing else; two documents with the same `$id` are the same schema, and a change to the `$id` is a different schema.

## Post. IV.3 - Generation

C# (via NJsonSchema, `System.Text.Json` attributes, records) and TypeScript (via `json-schema-to-typescript`) types are generated from schemas by the catalogue CI. Generated code is never edited by hand.

## Post. IV.4 - Validation Is Available Everywhere

A JSON Schema validator is available in every runtime we use (`JsonSchema.Net` in .NET, `ajv` in TypeScript) and is fast enough to run on every message and, where required, every request.

## Post. IV.5 - Small Catalogue

The catalogue holds concepts that at least two bounded contexts express identically. It is not a domain model; a concept owned by one context lives in that context's schemas.
