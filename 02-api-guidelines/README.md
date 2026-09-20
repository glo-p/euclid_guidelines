# Book II - API Guidelines

Book II governs every synchronous interface a service exposes: how it is named, how it behaves, how it fails, how it grows, and how it is built in .NET. It is one of the three *detailed* books; every proposition here is meant to be applied as written.

## Contents

| File                               | Contents                                                                                                                                            |
| ---------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`definitions.md`](definitions.md) | [`Def. II.1`](definitions.md#Def.%20II.1%20-%20API) - [`Def. II.18`](definitions.md#Def.%20II.18%20-%20Sub-resource): API, operation, collection, page, cursor, problem, edge, envelope, …                                                    |
| [`postulates.md`](postulates.md)   | [`Post. II.1`](postulates.md#Post.%20II.1%20-%20HTTP%20Is%20Authoritative) - [`Post. II.6`](postulates.md#Post.%20II.6%20-%20The%20Linter%20Is%20the%20Check): HTTP semantics are authoritative, OpenAPI is the contract, JSON is the representation, the edge is the only ingress, … |

### Propositions

| # | Title | Level | File |
|---|---|---|---|
| II.1 | The contract is written first and is the source of truth | MUST | [`01-contract-first.md`](propositions/01-contract-first.md) |
| II.2 | Resources are plural nouns in kebab-case paths | MUST | [`02-resource-naming.md`](propositions/02-resource-naming.md) |
| II.3 | Methods and status codes carry their HTTP meaning | MUST | [`03-methods-and-status-codes.md`](propositions/03-methods-and-status-codes.md) |
| II.4 | Every error is a Problem Details document | MUST | [`04-problem-details.md`](propositions/04-problem-details.md) |
| II.5 | Collections are paginated by opaque cursor | MUST | [`05-cursor-pagination.md`](propositions/05-cursor-pagination.md) |
| II.6 | Filtering, sorting and field selection use one grammar | SHOULD | [`06-filtering-and-sorting.md`](propositions/06-filtering-and-sorting.md) |
| II.7 | Only the major version is in the URI | MUST | [`07-versioning.md`](propositions/07-versioning.md) |
| II.8 | Non-idempotent operations accept an Idempotency-Key | MUST | [`08-idempotency.md`](propositions/08-idempotency.md) |
| II.9 | Updates are conditional on an entity tag | SHOULD | [`09-optimistic-concurrency.md`](propositions/09-optimistic-concurrency.md) |
| II.10 | Standard headers and response metadata | MUST | [`10-headers-and-metadata.md`](propositions/10-headers-and-metadata.md) |
| II.11 | JSON representation conventions | MUST | [`11-json-conventions.md`](propositions/11-json-conventions.md) |
| II.12 | Authentication at the edge, authorisation in the service | MUST | [`12-authentication-and-authorisation.md`](propositions/12-authentication-and-authorisation.md) |
| II.13 | Rate limits are declared and signalled | SHOULD | [`13-rate-limiting.md`](propositions/13-rate-limiting.md) |
| II.14 | Reads are cacheable and conditional | SHOULD | [`14-caching.md`](propositions/14-caching.md) |
| II.15 | Long-running work is an operation resource | MUST | [`15-long-running-operations.md`](propositions/15-long-running-operations.md) |
| II.16 | Every service exposes health and readiness | MUST | [`16-health-endpoints.md`](propositions/16-health-endpoints.md) |
| II.17 | The .NET construction of a conforming API | Q.E.F. | [`17-dotnet-construction.md`](propositions/17-dotnet-construction.md) |

## Reading order

II.1, II.2, II.3, II.4 and II.11 are the core; a service that satisfies those five is recognisably "ours". II.5 to II.10 make it robust under [Post. I.4](../01-foundations/postulates.md#Post.%20I.4%20-%20Unreliable%20Network) and [Post. I.5](../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution). II.12 to II.16 make it operable. II.17 shows how to get all of it from one project template.

## Open questions

* Whether to permit `PATCH` with JSON Merge Patch (RFC 7386) only, or also JSON Patch (RFC 6902). Current text: Merge Patch only (see II.3).
* Whether bulk operations (`POST /things:batch`) deserve their own proposition. Current text: covered by a scholium under II.15.
* Whether the company Spectral ruleset is published from this repository or from the catalogue repository ([Book IV](../04-shared-schemas/README.md)). Current text: this repository, under `rulesets/`.
