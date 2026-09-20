# Prop. IX.7 - Every new major ships with a migration guide

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A new major version (Def. IX.2) MUST NOT enter the Active stage (Def. IX.4) until a migration guide (Def. IX.7) exists beside the contract in its repository and is linked from the catalogue entry. The guide MUST enumerate every breaking change (Def. I.14) reported by the contract diff between the last minor of major N and the first minor of major N+1, with the consumer action for each. Where the old representation can be derived from the new by a pure function, the producer SHOULD publish that adapter as part of the shared package or the producer itself.

## Given

* Def. I.14, Def. IX.2, Def. IX.4, Def. IX.7
* Post. I.5, Post. I.7
* CN 3, CN 6
* Prop. IX.1, Prop. IX.2

## Demonstration

A new major exists only because of breaking changes (Prop. IX.2), each of which by Def. I.14 will make a lagging consumer (Post. I.5) fail unless the consumer acts. By CN 3 the consumer can know what to do only from what is written; the migration guide is that writing, and its content is exactly the diff that Prop. IX.1's tooling already produces, so completeness is checkable. CN 6 places the cost of stability on the producer, and an adapter that maps the new shape to the old is the cheapest form of that stability, which Post. I.7 makes possible to generate from the two contracts. ∎ Q.E.D.

## Corollaries

* **Cor. IX.7.1** - A migration guide is versioned with the contract and is itself immutable once the major is Active; corrections are appended, not rewritten.
* **Cor. IX.7.2** - The deprecation notice of major N (Prop. IX.3) links to this guide; therefore major N cannot be deprecated before major N+1 is Active.

## Construction

* File `MIGRATION-v{N}-to-v{N+1}.md` beside the contract, generated as a skeleton by the contract diff tool (Prop. IX.1) with one section per breaking change.
* Catalogue entry field `migrationGuide` (URL into the repository at the release tag).
* Adapter, API: producer-side mapping in the .NET service so `/v{N}` is served from the `v{N+1}` domain model; or a `Company.Contracts.Adapters` NuGet with `IContractAdapter<TOld, TNew>` for consumer-side use.
* Adapter, events: `IEventUpcaster` in `Company.Messaging` that translates `.v1` payloads to `.v2` on the consumer side, registered per event type.
* CI check `migration-guide-present`: a catalogue transition to `active` for a major greater than 1 requires the guide file and a section count equal to the diff's breaking change count.

## Conformance

CI check `migration-guide-present` in the catalogue repository; catalogue lint rejecting `active` for major N+1 without a `migrationGuide` link.

## Scholium

Whether an adapter should be a MUST for some media is an open question in the README. The guide is the minimum; the adapter is the courtesy that makes the minimum rarely needed.
