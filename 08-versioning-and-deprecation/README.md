# Book IX - Versioning and Deprecation

Book IX governs the lifecycle of **any** contract (Def. I.2): an OpenAPI document, an event type, or a shared schema. It says how a contract is introduced, how it may change without breaking anyone, how a breaking change is introduced beside the old one, how consumers are told, and how a version is finally removed. The books that own each kind of contract (II, III, IV) say how the mechanics look in their medium; this book says what the mechanics are for and in which order they happen.

Depth: **Scaffold**. Every proposition is `Draft` until its machine check exists (Prop. I.3).

## Contents

| File | Contents |
|---|---|
| [`definitions.md`](definitions.md) | `Def. IX.1` to `Def. IX.9`: version, major version, minor change, lifecycle stage, deprecation notice, sunset date, migration guide, consumer registry, traffic evidence. |
| [`postulates.md`](postulates.md) | `Post. IX.1` to `Post. IX.3`: at most two majors live at once; minimum deprecation periods; consumers are enumerable. |
| [`propositions/`](propositions/) | `Prop. IX.1` to `Prop. IX.8`. |

## Definitions

| Id | Term | Summary |
|---|---|---|
| Def. IX.1 | Version | A named, immutable revision of a contract. |
| Def. IX.2 | Major Version | The integer a consumer selects; changes only on a breaking change. |
| Def. IX.3 | Minor Change | A compatible change published within a major version. |
| Def. IX.4 | Lifecycle Stage | One of Proposed, Active, Deprecated, Sunset, Retired, with precise meanings. |
| Def. IX.5 | Deprecation Notice | The dated, machine-readable declaration that a version is Deprecated. |
| Def. IX.6 | Sunset Date | The date after which a Deprecated version is no longer owed. |
| Def. IX.7 | Migration Guide | The document that takes a consumer from major N to major N+1. |
| Def. IX.8 | Consumer Registry | The enumerated set of consumers of each contract version. |
| Def. IX.9 | Traffic Evidence | Telemetry that shows whether a version is still used. |

## Postulates

| Id | Summary |
|---|---|
| Post. IX.1 | At most two major versions of one contract are Active or Deprecated at the same time. |
| Post. IX.2 | Minimum deprecation period: 6 months internal, 12 months external. |
| Post. IX.3 | The consumers of any contract version can be enumerated. |

## Propositions

| Id | Title | Level | Summary |
|---|---|---|---|
| [Prop. IX.1](propositions/01-only-compatible-changes-within-a-major.md) | Only compatible changes within a major version | MUST | A major version admits only compatible changes, verified by contract diff in CI. |
| [Prop. IX.2](propositions/02-breaking-change-is-a-new-major-side-by-side.md) | A breaking change is a new major version published side by side | MUST | A breaking change creates major N+1 and leaves major N untouched and served. |
| [Prop. IX.3](propositions/03-deprecation-is-declared-in-the-contract.md) | Deprecation is declared in the contract and notified to consumers | MUST | Deprecation lives in the contract artefact and reaches every registered consumer. |
| [Prop. IX.4](propositions/04-producers-know-their-consumers.md) | Producers know their consumers | MUST | A consumer registry is populated from API Gateway client identities and EventBridge rules. |
| [Prop. IX.5](propositions/05-sunset-on-evidence-retirement-removes.md) | Sunset on evidence; retirement removes the contract | MUST | Sunset requires 30 days of zero traffic; retirement deletes the version. |
| [Prop. IX.6](propositions/06-shared-schemas-follow-the-same-lifecycle.md) | Shared schemas follow the same lifecycle | MUST | Shared schemas are versioned by `$id` URI and pass through the same stages. |
| [Prop. IX.7](propositions/07-every-major-ships-with-a-migration-guide.md) | Every new major ships with a migration guide | MUST | Major N+1 is not Active until its migration guide exists; an adapter SHOULD exist. |
| [Prop. IX.8](propositions/08-lifecycle-stage-is-machine-readable.md) | The lifecycle stage of every contract is machine-readable | MUST | The catalogue records the stage of every contract version in a queryable form. |

## Reading order

Definitions, then postulates, then IX.1 and IX.2 (how a contract changes), then IX.3 to IX.5 (how a version leaves), then IX.6 to IX.8 (where the facts are kept).

## Open questions

Decisions the principal engineers still need to make. Each is resolved only by an ADR (Prop. X.7).

1. **Deprecation periods.** Post. IX.2 states 6 months internal and 12 months external. Confirm the numbers, and decide whether a contract with no external consumers may use the shorter period automatically or only by declaration.
2. **Zero-traffic window.** Prop. IX.5 requires 30 days of zero traffic before Sunset. Confirm 30 days, and decide whether the window is measured in `prod` only or in every environment (Def. I.21).
3. **Consumer registry store.** Prop. IX.4 needs a store. Candidates: a table in the catalogue repository (Def. I.24), a DynamoDB table populated by a scheduled Lambda, or the AWS Service Catalog AppRegistry. Choose one.
4. **Client identity for internal API consumers.** API Gateway API keys identify external consumers. For service-to-service calls the identity is the access token subject (Book V). Decide which claim is the canonical consumer identifier.
5. **Event consumers outside the account.** EventBridge rule inventory covers rules in the bus's account. Decide how cross-account event bus targets are enumerated.
6. **Adapter obligation.** Prop. IX.7 makes an adapter a SHOULD. Decide whether a producer-side adapter (serve N from N+1) is required for APIs, for events, or neither.
7. **Retired contract artefacts.** Decide whether a Retired version is deleted from the catalogue or kept under a `retired/` path for audit.
8. **Pre-release versions.** Decide whether the `Proposed` stage is served in any environment or only exists in the repository.
