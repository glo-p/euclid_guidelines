# ADR-0001: Adopt Euclidean guidelines and the canonical stack

| | |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-09-20 |
| **Deciders** | The three principal engineers ([Def. X.7](../definitions.md#Def.%20X.7%20-%20Principal%20Engineer%20Group)) |
| **Guidelines affected** | Books I to XI (creation); [Post. I.1](../../01-foundations/postulates.md#Post.%20I.1%20-%20Cloud), [Post. I.2](../../01-foundations/postulates.md#Post.%20I.2%20-%20Language), [Post. I.9](../../01-foundations/postulates.md#Post.%20I.9%20-%20Transport); [Prop. I.1](../../01-foundations/method.md#Prop.%20I.1%20-%20Every%20rule%20is%20a%20proposition%20with%20a%20demonstration) to [Prop. I.4](../../01-foundations/method.md#Prop.%20I.4%20-%20Change%20is%20by%20ADR%20against%20the%20foundations); [Prop. II.1](../../02-api-guidelines/propositions/01-contract-first.md), [Prop. II.5](../../02-api-guidelines/propositions/05-cursor-pagination.md); [Prop. III.1](../../03-events/propositions/01-envelope.md), [Prop. III.9](../../03-events/propositions/09-topology.md); [Prop. IV.2](../../04-shared-schemas/propositions/02-identifier.md) |

## Context

The company runs autonomous teams on AWS with .NET back-ends and mixed front-ends. Teams have interacted through contracts in practice but without a shared, written statement of what a contract must look like, how it may change, or who decides. Previous attempts at guidelines were lists of preferences; each was argued about until it was ignored. Three principal engineers were asked to produce a set of guidelines that teams can follow without a committee and can dispute without a fight.

The forces are: autonomy of teams ([Post. I.3](../../01-foundations/postulates.md#Post.%20I.3%20-%20Autonomy)); independent evolution of producers and consumers ([Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution)); the observed fact that unverified rules decay ([Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification)); and a stack that is already mostly uniform ([Post. I.1](../../01-foundations/postulates.md#Post.%20I.1%20-%20Cloud), [Post. I.2](../../01-foundations/postulates.md#Post.%20I.2%20-%20Language), [Post. I.9](../../01-foundations/postulates.md#Post.%20I.9%20-%20Transport)) but has never been declared canonical.

## Decision

The principal engineers decide:

1. **Form.** The guidelines are written in the manner of Euclid's *Elements*: a small set of *definitions*, *postulates* and *common notions* in [Book I](../../01-foundations/README.md), and *propositions* in Books II to XI, each with a demonstration that derives it from those foundations. No rule exists without a demonstration ([Prop. I.1](../../01-foundations/method.md#Prop.%20I.1%20-%20Every%20rule%20is%20a%20proposition%20with%20a%20demonstration)). Disagreement is directed at a definition, postulate or common notion, not at the rule.

2. **Synchronous style.** REST/JSON over HTTP, described contract-first in OpenAPI 3.1, is the sole synchronous integration style between teams ([Post. I.9](../../01-foundations/postulates.md#Post.%20I.9%20-%20Transport), [Prop. II.1](../../02-api-guidelines/propositions/01-contract-first.md)). Errors use RFC 9457 Problem Details.

3. **Asynchronous transport.** Amazon EventBridge is the bus and Amazon SQS the consumer buffer for all asynchronous communication between teams ([Post. I.9](../../01-foundations/postulates.md#Post.%20I.9%20-%20Transport), [Prop. III.9](../../03-events/propositions/09-topology.md)). Messages carry a CloudEvents 1.0 envelope ([Prop. III.1](../../03-events/propositions/01-envelope.md)) and payloads are described in JSON Schema 2020-12.

4. **Canonical primitives.** Cursor pagination is the only pagination style ([Prop. II.5](../../02-api-guidelines/propositions/05-cursor-pagination.md)). The ULID is the canonical identifier ([Def. I.18](../../01-foundations/definitions.md#Def.%20I.18%20-%20Identifier), [Prop. IV.2](../../04-shared-schemas/propositions/02-identifier.md)).

5. **Scope.** All eleven books are created now: Books I to IV at detailed depth ([Book I](../../01-foundations/README.md) complete), Books V to XI at scaffold depth. A scaffold book has full definitions, postulates, and propositions with statement, demonstration, construction outline and conformance check, all at status `Draft`, with its open questions listed in its README ([Prop. X.7](../propositions/07-open-questions-resolved-by-adr.md)). Scaffold propositions are promoted to `Accepted` by later ADRs as their machine checks are built ([Prop. I.3](../../01-foundations/method.md#Prop.%20I.3%20-%20Every%20MUST%20has%20a%20machine%20check)).

6. **Governance.** This ADR is the first entry of the ADR log ([Prop. X.6](../propositions/06-the-adr-log.md)). All later changes follow [Prop. X.1](../propositions/01-change-by-pull-request-with-adr.md): a pull request with an ADR, accepted by two of the three principal engineers.

## Consequences

Easier: any two teams can integrate by reading the contract and Books II to IV, with no meeting. Reviewers point at a proposition number instead of an opinion. New services start from the same shape, so shared tooling (linters, contract diff, conformance manifests) pays off across the estate.

Harder: existing services that use other transports (gRPC, Kafka, GraphQL, WebSockets), offset pagination, or non-ULID identifiers must either migrate or hold an exception ADR ([Prop. X.2](../propositions/02-exception-process.md)) with an expiry. A migration inventory is the first task of each guild ([Prop. X.5](../propositions/05-guilds-own-books.md)). Every team must add `conformance.json` ([Prop. X.3](../propositions/03-conformance-manifest.md)) within one release cycle of that proposition leaving `Draft`.

Migrated by when: the scaffold books carry open questions; each book's guild proposes ADRs to close them. No calendar deadline is set by this ADR; the deadline for each migration is set by the ADR that promotes the relevant proposition to `Accepted`.

## Demonstration

This ADR creates the foundations rather than changing them, so the demonstration is that the foundations are consistent with each other and with the decisions above.

Decision 1 is [Prop. I.1](../../01-foundations/method.md#Prop.%20I.1%20-%20Every%20rule%20is%20a%20proposition%20with%20a%20demonstration), which rests on [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts) and [Post. I.3](../../01-foundations/postulates.md#Post.%20I.3%20-%20Autonomy): autonomous teams follow only rules whose derivation they can see, and what is not written does not exist. Decisions 2 and 3 are the content of [Post. I.9](../../01-foundations/postulates.md#Post.%20I.9%20-%20Transport), a postulate: they are facts about our estate accepted without proof, and they are recorded here so the acceptance has a date and deciders. Decision 4 restates [Def. I.18](../../01-foundations/definitions.md#Def.%20I.18%20-%20Identifier) and the canonical forms chosen in Books II and IV; by [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape) one concept has one shape, so a single pagination style and a single identifier form follow once any is chosen. Decision 5 follows from [Prop. I.3](../../01-foundations/method.md#Prop.%20I.3%20-%20Every%20MUST%20has%20a%20machine%20check): a MUST without a machine check is `Draft`, so a scaffold book is one whose checks are not yet built, and nothing else. Decision 6 is [Cor. I.4.2](../../01-foundations/method.md#Prop.%20I.4%20-%20Change%20is%20by%20ADR%20against%20the%20foundations) and [Post. X.1](../postulates.md#Post.%20X.1%20-%20Three%20Principals%20Accept). No definition, postulate or common notion is changed by this ADR; every proposition in Books I to XI cites only items that exist after it. ∎
