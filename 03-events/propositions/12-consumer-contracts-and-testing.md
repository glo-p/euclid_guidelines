# Prop. III.12 - Producers and consumers are held to the schema by tests

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

A publisher MUST have a test that validates every event it can emit against the registered schema, using the schema's examples as fixtures. A consumer MUST have a test that deserialises every registered example of every event type it subscribes to and processes it to a success or a permanent failure with a reason, and MUST have a test that delivers each example twice and out of order ([Prop. III.6](06-idempotent-consumers.md), [III.7](07-ordering.md)). Both sets of tests MUST run in CI against the catalogue package version pinned in the service and MUST run again, automatically, when the catalogue publishes a compatible change to a schema the service depends on.

## Given

[Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution), [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification), [Post. III.7](../postulates.md#Post.%20III.7%20-%20Schema%20Validation%20Is%20the%20Check), [CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed), [CN 4](../../01-foundations/common-notions.md#CN%204%20-%20Compatibility%20Is%20Compositional), [CN 6](../../01-foundations/common-notions.md#CN%206%20-%20The%20Producer%20Pays%20for%20Stability%3B%20the%20Consumer%20Pays%20for%20Tolerance), [Prop. III.4](04-schemas-and-registry.md), [III.6](06-idempotent-consumers.md), [III.7](07-ordering.md).

## Demonstration

The schema is the contract, and [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification) says an unchecked contract drifts. The publisher's check proves it emits what it promised ([CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed)). The consumer's check proves it tolerates what was promised, including additions it has not seen ([CN 6](../../01-foundations/common-notions.md#CN%206%20-%20The%20Producer%20Pays%20for%20Stability%3B%20the%20Consumer%20Pays%20for%20Tolerance)), and proves the two properties this Book requires of it (idempotence, order tolerance). Because producers and consumers evolve independently ([Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution)), a compatible change on one side must be re-verified against the other, which is why the catalogue triggers downstream runs; [CN 4](../../01-foundations/common-notions.md#CN%204%20-%20Compatibility%20Is%20Compositional) makes a compatible change safe in principle and the test makes it safe in fact. ∎ Q.E.D.

## Corollaries

* **Cor. III.12.1** - Schema examples are the shared fixtures; a schema without examples is rejected by catalogue CI.
* **Cor. III.12.2** - A consumer that needs a field not present in any example has found a gap in the contract and raises it with the publisher rather than inventing test data.

## Construction

`Company.Platform.Messaging.Testing` provides `SchemaFixtures.For<TEvent>()` (loads examples from the catalogue package), `ConsumerHarness` (delivers twice, reversed, and with unknown extra fields) and `PublisherHarness` (captures outbox writes and validates). Catalogue CI publishes a "dependents" list and triggers each dependent's pipeline with a `contract-recheck` job.

## Conformance

CI jobs `publisher-contract-tests` and `consumer-contract-tests` present in every service that has an outbox or a queue; listed in `conformance.json` ([Prop. X.3](../../10-governance/propositions/03-conformance-manifest.md)).
