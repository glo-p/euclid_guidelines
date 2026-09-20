# Book IV - Definitions

---

## Def. IV.1 - Catalogue Repository

The *catalogue repository* is the single git repository (`company/contracts`) that holds every shared schema ([Def. I.13](../01-foundations/definitions.md#Def.%20I.13%20-%20Shared%20Schema)), every event schema ([Prop. III.4](../03-events/propositions/04-schemas-and-registry.md)), the problem type registry ([Cor. II.4.2](../02-api-guidelines/propositions/04-problem-details.md#Corollaries)), the context register ([Book VII](../07-data-ownership/README.md)), and the CI that publishes packages from them. It is the catalogue of [Def. I.24](../01-foundations/definitions.md#Def.%20I.24%20-%20Catalogue) made concrete.

## Def. IV.2 - Schema Id

The *schema id* is the absolute URI in a schema's `$id`, of the form `https://schemas.company.com/<segment>/v<major>/<name>.json`, where `<segment>` is `shared` for [Book IV](README.md) schemas or a bounded-context name for event schemas. The id is the schema's identity; the file path mirrors it.

## Def. IV.3 - Package

A *package* is a versioned artefact generated from the catalogue for one language: `Company.Contracts.Shared` and `Company.Contracts.<Context>` on NuGet; `@company/contracts` and `@company/contracts-<context>` on npm. A package's semantic version tracks the catalogue's; its major never changes because schema majors are in the `$id`.

## Def. IV.4 - Generated Type

A *generated type* is the C# record or TypeScript type produced from a schema by the catalogue CI. Generated types are the only permitted in-code representation of a catalogue concept.

## Def. IV.5 - Shape

The *shape* of a concept is the set of its fields, their types, constraints and meanings, as fixed by its schema. Two concepts with the same shape but different meanings are different schemas; one concept with two shapes is a defect ([CN 5](../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape)).

## Def. IV.6 - Open Enumeration

(Refines [Def. II.16](../02-api-guidelines/definitions.md#Def.%20II.16%20-%20Open%20Enumeration).) An *open enumeration* is a string type whose schema lists known values under `x-known-values` and declares no `enum` constraint, so that validation accepts unknown values. A *closed enumeration* uses `enum`.

## Def. IV.7 - Example

An *example* is a JSON value listed in a schema's `examples` array that validates against the schema and is used as a test fixture by every consumer of the schema ([Cor. III.12.1](../03-events/propositions/12-consumer-contracts-and-testing.md#Corollaries)).

## Def. IV.8 - Extension Keyword

An *extension keyword* is a schema keyword beginning `x-` that carries catalogue metadata (`x-lifecycle`, `x-event-type`, `x-data-classification`, `x-known-values`) and is ignored by validators.
