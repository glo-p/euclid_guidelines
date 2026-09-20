# Prop. IV.1 - The catalogue is one repository and reaches teams as packages

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

All shared schemas, event schemas, the problem type registry and the context register MUST live in the catalogue repository (Def. IV.1). Its CI MUST, on every merge to main: validate every schema and every example; diff every schema against the previous release and fail on a breaking change within a major (Def. I.14); generate C# and TypeScript types (Post. IV.3); publish packages (Def. IV.3); publish schemas to `https://schemas.company.com/` and to the EventBridge Schema Registry; and publish a machine-readable index (`index.json`) listing every schema, its `$id`, lifecycle stage, owner and dependants. Services MUST consume catalogue concepts only through the packages, pinned to a version, and MUST NOT vendor or hand-write catalogue types.

## Given

Def. I.13, Def. I.24, Def. IV.1 to IV.4, Post. I.6, Post. I.7, Post. IV.2, Post. IV.3, CN 2, CN 4, CN 5, CN 8.

## Demonstration

There is one catalogue by definition (Def. I.24); two would mean two shapes for one concept (CN 5). A schema that reaches a team as a document is re-typed by hand and drifts (Post. I.6); a schema that reaches a team as a generated type in a package cannot drift from the schema (Post. IV.3). Because a published schema is owed (CN 2) and compatibility is compositional (CN 4), the check that a change is compatible must run on the schema diff, centrally, before anything is published. The index makes the set of contracts enumerable, which CN 8 says is the architecture. ∎ Q.E.D.

## Corollaries

* **Cor. IV.1.1** - The package version is semantic over the *catalogue*: a new schema or a compatible field is a minor bump; a fix to an example or description is a patch; the major is fixed at 1 because breaking changes are new `$id`s (Post. IV.2).
* **Cor. IV.1.2** - `index.json` is the input to the conformance index (Prop. X.3), the consumer registry (Prop. IX.4) and the lifecycle report (Prop. IX.8).
* **Cor. IV.1.3** - The problem registry (Cor. II.4.2) is `problems/*.md` in the catalogue and is served at `https://problems.company.com/`.

## Construction

```
company/contracts/
  schemas/shared/v1/*.json
  schemas/<context>/v<major>/*.json
  problems/<slug>.md
  contexts.json                 # bounded context register
  gen/csharp/  gen/typescript/  # generated, committed for review, never edited
  .github/workflows/release.yml # validate -> diff -> generate -> pack -> publish -> index
```

Packages: `Company.Contracts.Shared` (C# records with `System.Text.Json` converters: `Ulid`, `Money`, `Timestamp`, `Page<T>`, `PageInfo`, `ProblemDetails`, `ResourceMetadata`, `Actor`, `CloudEvent<T>`, `Operation`, `Health`, `AuditEvent`, `ProblemTypes` constants, `Cursor` helpers); `@company/contracts` (TypeScript types, `zod` schemas as an open question, `problemMessages` map for Prop. XI.5).

## Conformance

The catalogue CI itself. In services: architecture test that no type named like a catalogue concept is declared outside the package (Prop. II.17, III.14); `dotnet list package` shows `Company.Contracts.Shared` at a version no more than two minors behind latest (reported by the conformance index).
