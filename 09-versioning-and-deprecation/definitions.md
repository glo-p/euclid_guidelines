# Book IX - Definitions

These definitions fix the vocabulary of contract lifecycle used in [Book IX](README.md) and cited by Books II, III, IV and X. They are not rules. Where a definition refines one in [Book I](../01-foundations/README.md) it says so.

---

## Def. IX.1 - Version

A *version* is a named, immutable revision of a contract ([Def. I.2](../01-foundations/definitions.md#Def.%20I.2%20-%20Contract)). Once published, the content of a version never changes; a correction is a new version. A version is identified by its contract name, its major version ([Def. IX.2](definitions.md#Def.%20IX.2%20-%20Major%20Version)) and, within the major, a monotonically increasing minor revision.

## Def. IX.2 - Major Version

The *major version* is the positive integer that a consumer ([Def. I.4](../01-foundations/definitions.md#Def.%20I.4%20-%20Consumer)) selects when it binds to a contract: the path segment `/v2` of an API ([Prop. II.7](../02-api-guidelines/propositions/07-versioning.md)), the suffix `.v2` of an event type ([Prop. III.2](../03-events/propositions/02-naming.md)), the path segment of a shared schema `$id` ([Prop. IV.1](../04-shared-schemas/propositions/01-catalogue-and-distribution.md)). Two versions with the same major are substitutable for a conforming consumer ([CN 1](../01-foundations/common-notions.md#CN%201%20-%20Substitutability)). The major changes if and only if a breaking change ([Def. I.14](../01-foundations/definitions.md#Def.%20I.14%20-%20Breaking%20Change)) is introduced.

## Def. IX.3 - Minor Change

A *minor change* is a compatible change ([Def. I.15](../01-foundations/definitions.md#Def.%20I.15%20-%20Compatible%20Change)) published within an existing major version. A minor change produces a new version ([Def. IX.1](definitions.md#Def.%20IX.1%20-%20Version)) but not a new major version. Consumers do not select minor revisions; they receive them.

## Def. IX.4 - Lifecycle Stage

The *lifecycle stage* of a version is exactly one of the following, in this order, and a version moves only forwards:

* **Proposed.** The version exists in the repository and catalogue but is not owed. It may change or be withdrawn without notice. No consumer may bind to it in `prod`.
* **Active.** The version is published and owed ([CN 2](../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed)). It accepts minor changes ([Def. IX.3](definitions.md#Def.%20IX.3%20-%20Minor%20Change)). New consumers may bind to it.
* **Deprecated.** The version is still owed and still served unchanged, but a deprecation notice ([Def. IX.5](definitions.md#Def.%20IX.5%20-%20Deprecation%20Notice)) and a sunset date ([Def. IX.6](definitions.md#Def.%20IX.6%20-%20Sunset%20Date)) have been published. New consumers MUST NOT bind to it. No minor change is made to it except a security fix.
* **Sunset.** The sunset date has passed and traffic evidence ([Def. IX.9](definitions.md#Def.%20IX.9%20-%20Traffic%20Evidence)) shows no consumer. The version is no longer owed. It may still be deployed but may be removed at any moment without further notice.
* **Retired.** The version has been removed from every environment and its artefacts are marked retired in the catalogue. The identifier is never reused ([Cor. I.4.1](../01-foundations/method.md#Prop.%20I.4%20-%20Change%20is%20by%20ADR%20against%20the%20foundations)).

## Def. IX.5 - Deprecation Notice

A *deprecation notice* is the machine-readable declaration, carried in or alongside the contract artefact itself, that a version has entered the Deprecated stage. It carries: the date of deprecation, the sunset date ([Def. IX.6](definitions.md#Def.%20IX.6%20-%20Sunset%20Date)), the identifier of the successor major version, and a link to the migration guide ([Def. IX.7](definitions.md#Def.%20IX.7%20-%20Migration%20Guide)). Its medium is fixed by [Prop. IX.3](propositions/03-deprecation-is-declared-in-the-contract.md).

## Def. IX.6 - Sunset Date

The *sunset date* is the calendar date, stated in the deprecation notice, on and after which the producer ([Def. I.3](../01-foundations/definitions.md#Def.%20I.3%20-%20Producer)) may cease to honour a Deprecated version. It is at least the minimum deprecation period ([Post. IX.2](postulates.md#Post.%20IX.2%20-%20Minimum%20Deprecation%20Period)) after the date of deprecation. It may be moved later; it is never moved earlier.

## Def. IX.7 - Migration Guide

A *migration guide* is the document, kept beside the contract in its repository, that enumerates every breaking change ([Def. I.14](../01-foundations/definitions.md#Def.%20I.14%20-%20Breaking%20Change)) between major N and major N+1 of one contract and states, for each, what a consumer must do. A migration guide that omits a breaking change is defective.

## Def. IX.8 - Consumer Registry

The *consumer registry* is the enumerated, queryable set of consumers ([Def. I.4](../01-foundations/definitions.md#Def.%20I.4%20-%20Consumer)) of each Active or Deprecated version, each identified by a stable consumer identity and its owning team ([Def. I.17](../01-foundations/definitions.md#Def.%20I.17%20-%20Team)) or external organisation. It is populated by machine from observed bindings ([Prop. IX.4](propositions/04-producers-know-their-consumers.md)) and is the sole source of the recipients of a deprecation notice.

## Def. IX.9 - Traffic Evidence

*Traffic evidence* is the RED metric series ([Prop. VI.3](../06-observability/propositions/03-red-metrics.md)) for one contract version, labelled by consumer identity, over a stated window. A version has *zero traffic* over a window if its request count and message delivery count are both zero for every consumer for the whole window.
