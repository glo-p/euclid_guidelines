# Prop. III.12 - Producers and consumers are held to the schema by tests

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

A publisher MUST have a test that validates every event it can emit against the registered schema, using the schema's examples as fixtures. A consumer MUST have a test that deserialises every registered example of every event type it subscribes to and processes it to a success or a permanent failure with a reason, and MUST have a test that delivers each example twice and out of order (Prop. III.6, III.7). Both sets of tests MUST run in CI against the catalogue package version pinned in the service and MUST run again, automatically, when the catalogue publishes a compatible change to a schema the service depends on.

## Given

Post. I.5, Post. I.6, Post. III.7, CN 2, CN 4, CN 6, Prop. III.4, III.6, III.7.

## Demonstration

The schema is the contract, and Post. I.6 says an unchecked contract drifts. The publisher's check proves it emits what it promised (CN 2). The consumer's check proves it tolerates what was promised, including additions it has not seen (CN 6), and proves the two properties this Book requires of it (idempotence, order tolerance). Because producers and consumers evolve independently (Post. I.5), a compatible change on one side must be re-verified against the other, which is why the catalogue triggers downstream runs; CN 4 makes a compatible change safe in principle and the test makes it safe in fact. ∎ Q.E.D.

## Corollaries

* **Cor. III.12.1** - Schema examples are the shared fixtures; a schema without examples is rejected by catalogue CI.
* **Cor. III.12.2** - A consumer that needs a field not present in any example has found a gap in the contract and raises it with the publisher rather than inventing test data.

## Construction

`Company.Platform.Messaging.Testing` provides `SchemaFixtures.For<TEvent>()` (loads examples from the catalogue package), `ConsumerHarness` (delivers twice, reversed, and with unknown extra fields) and `PublisherHarness` (captures outbox writes and validates). Catalogue CI publishes a "dependents" list and triggers each dependent's pipeline with a `contract-recheck` job.

## Conformance

CI jobs `publisher-contract-tests` and `consumer-contract-tests` present in every service that has an outbox or a queue; listed in `conformance.json` (Prop. X.3).
