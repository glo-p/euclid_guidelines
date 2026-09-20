# Prop. IX.7 - Every new major ships with a migration guide

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A new major version ([Def. IX.2](../definitions.md#Def.%20IX.2%20-%20Major%20Version)) MUST NOT enter the Active stage ([Def. IX.4](../definitions.md#Def.%20IX.4%20-%20Lifecycle%20Stage)) until a migration guide ([Def. IX.7](../definitions.md#Def.%20IX.7%20-%20Migration%20Guide)) exists beside the contract in its repository and is linked from the catalogue entry. The guide MUST enumerate every breaking change ([Def. I.14](../../01-foundations/definitions.md#Def.%20I.14%20-%20Breaking%20Change)) reported by the contract diff between the last minor of major N and the first minor of major N+1, with the consumer action for each. Where the old representation can be derived from the new by a pure function, the producer SHOULD publish that adapter as part of the shared package or the producer itself.

## Given

* [Def. I.14](../../01-foundations/definitions.md#Def.%20I.14%20-%20Breaking%20Change), [Def. IX.2](../definitions.md#Def.%20IX.2%20-%20Major%20Version), [Def. IX.4](../definitions.md#Def.%20IX.4%20-%20Lifecycle%20Stage), [Def. IX.7](../definitions.md#Def.%20IX.7%20-%20Migration%20Guide)
* [Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution), [Post. I.7](../../01-foundations/postulates.md#Post.%20I.7%20-%20Contract%20First)
* [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 6](../../01-foundations/common-notions.md#CN%206%20-%20The%20Producer%20Pays%20for%20Stability%3B%20the%20Consumer%20Pays%20for%20Tolerance)
* [Prop. IX.1](01-only-compatible-changes-within-a-major.md), [Prop. IX.2](02-breaking-change-is-a-new-major-side-by-side.md)

## Demonstration

A new major exists only because of breaking changes ([Prop. IX.2](02-breaking-change-is-a-new-major-side-by-side.md)), each of which by [Def. I.14](../../01-foundations/definitions.md#Def.%20I.14%20-%20Breaking%20Change) will make a lagging consumer ([Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution)) fail unless the consumer acts. By [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit) the consumer can know what to do only from what is written; the migration guide is that writing, and its content is exactly the diff that [Prop. IX.1](01-only-compatible-changes-within-a-major.md)'s tooling already produces, so completeness is checkable. [CN 6](../../01-foundations/common-notions.md#CN%206%20-%20The%20Producer%20Pays%20for%20Stability%3B%20the%20Consumer%20Pays%20for%20Tolerance) places the cost of stability on the producer, and an adapter that maps the new shape to the old is the cheapest form of that stability, which [Post. I.7](../../01-foundations/postulates.md#Post.%20I.7%20-%20Contract%20First) makes possible to generate from the two contracts. ∎ Q.E.D.

## Corollaries

* **Cor. IX.7.1** - A migration guide is versioned with the contract and is itself immutable once the major is Active; corrections are appended, not rewritten.
* **Cor. IX.7.2** - The deprecation notice of major N ([Prop. IX.3](03-deprecation-is-declared-in-the-contract.md)) links to this guide; therefore major N cannot be deprecated before major N+1 is Active.

## Construction

* File `MIGRATION-v{N}-to-v{N+1}.md` beside the contract, generated as a skeleton by the contract diff tool ([Prop. IX.1](01-only-compatible-changes-within-a-major.md)) with one section per breaking change.
* Catalogue entry field `migrationGuide` (URL into the repository at the release tag).
* Adapter, API: producer-side mapping in the .NET service so `/v{N}` is served from the `v{N+1}` domain model; or a `Company.Contracts.Adapters` NuGet with `IContractAdapter<TOld, TNew>` for consumer-side use.
* Adapter, events: `IEventUpcaster` in `Company.Messaging` that translates `.v1` payloads to `.v2` on the consumer side, registered per event type.
* CI check `migration-guide-present`: a catalogue transition to `active` for a major greater than 1 requires the guide file and a section count equal to the diff's breaking change count.

## Conformance

CI check `migration-guide-present` in the catalogue repository; catalogue lint rejecting `active` for major N+1 without a `migrationGuide` link.

## Scholium

Whether an adapter should be a MUST for some media is an open question in the README. The guide is the minimum; the adapter is the courtesy that makes the minimum rarely needed.
