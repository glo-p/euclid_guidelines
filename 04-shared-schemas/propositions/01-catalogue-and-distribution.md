# Prop. IV.1 - The catalogue is one repository and reaches teams as packages

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

All shared schemas, event schemas, the problem type registry and the context register MUST live in the catalogue repository ([Def. IV.1](../definitions.md#Def.%20IV.1%20-%20Catalogue%20Repository)). Its CI MUST, on every merge to main: validate every schema and every example; diff every schema against the previous release and fail on a breaking change within a major ([Def. I.14](../../01-foundations/definitions.md#Def.%20I.14%20-%20Breaking%20Change)); generate C# and TypeScript types ([Post. IV.3](../postulates.md#Post.%20IV.3%20-%20Generation)); publish packages ([Def. IV.3](../definitions.md#Def.%20IV.3%20-%20Package)); publish schemas to `https://schemas.company.com/` and to the EventBridge Schema Registry; and publish a machine-readable index (`index.json`) listing every schema, its `$id`, lifecycle stage, owner and dependants. Services MUST consume catalogue concepts only through the packages, pinned to a version, and MUST NOT vendor or hand-write catalogue types.

## Given

[Def. I.13](../../01-foundations/definitions.md#Def.%20I.13%20-%20Shared%20Schema), [Def. I.24](../../01-foundations/definitions.md#Def.%20I.24%20-%20Catalogue), [Def. IV.1](../definitions.md#Def.%20IV.1%20-%20Catalogue%20Repository) to [IV.4](../definitions.md#Def.%20IV.4%20-%20Generated%20Type), [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification), [Post. I.7](../../01-foundations/postulates.md#Post.%20I.7%20-%20Contract%20First), [Post. IV.2](../postulates.md#Post.%20IV.2%20-%20Identity%20by%20URI), [Post. IV.3](../postulates.md#Post.%20IV.3%20-%20Generation), [CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed), [CN 4](../../01-foundations/common-notions.md#CN%204%20-%20Compatibility%20Is%20Compositional), [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape), [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts).

## Demonstration

There is one catalogue by definition ([Def. I.24](../../01-foundations/definitions.md#Def.%20I.24%20-%20Catalogue)); two would mean two shapes for one concept ([CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape)). A schema that reaches a team as a document is re-typed by hand and drifts ([Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification)); a schema that reaches a team as a generated type in a package cannot drift from the schema ([Post. IV.3](../postulates.md#Post.%20IV.3%20-%20Generation)). Because a published schema is owed ([CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed)) and compatibility is compositional ([CN 4](../../01-foundations/common-notions.md#CN%204%20-%20Compatibility%20Is%20Compositional)), the check that a change is compatible must run on the schema diff, centrally, before anything is published. The index makes the set of contracts enumerable, which [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts) says is the architecture. ∎ Q.E.D.

## Corollaries

* **Cor. IV.1.1** - The package version is semantic over the *catalogue*: a new schema or a compatible field is a minor bump; a fix to an example or description is a patch; the major is fixed at 1 because breaking changes are new `$id`s ([Post. IV.2](../postulates.md#Post.%20IV.2%20-%20Identity%20by%20URI)).
* **Cor. IV.1.2** - `index.json` is the input to the conformance index ([Prop. X.3](../../10-governance/propositions/03-conformance-manifest.md)), the consumer registry ([Prop. IX.4](../../09-versioning-and-deprecation/propositions/04-producers-know-their-consumers.md)) and the lifecycle report ([Prop. IX.8](../../09-versioning-and-deprecation/propositions/08-lifecycle-stage-is-machine-readable.md)).
* **Cor. IV.1.3** - The problem registry ([Cor. II.4.2](../../02-api-guidelines/propositions/04-problem-details.md#Corollaries)) is `problems/*.md` in the catalogue and is served at `https://problems.company.com/`.

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

Packages: `Company.Contracts.Shared` (C# records with `System.Text.Json` converters: `Ulid`, `Money`, `Timestamp`, `Page<T>`, `PageInfo`, `ProblemDetails`, `ResourceMetadata`, `Actor`, `CloudEvent<T>`, `Operation`, `Health`, `AuditEvent`, `ProblemTypes` constants, `Cursor` helpers); `@company/contracts` (TypeScript types, `zod` schemas as an open question, `problemMessages` map for [Prop. XI.5](../../11-frontend-integration/propositions/05-problem-details-user-messages.md)).

## Conformance

The catalogue CI itself. In services: architecture test that no type named like a catalogue concept is declared outside the package ([Prop. II.17](../../02-api-guidelines/propositions/17-dotnet-construction.md), [III.14](../../03-events/propositions/14-dotnet-construction.md)); `dotnet list package` shows `Company.Contracts.Shared` at a version no more than two minors behind latest (reported by the conformance index).
