# ADR-0001: Adopt Euclidean guidelines and the canonical stack

| | |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-09-20 |
| **Deciders** | The three principal engineers (Def. X.7) |
| **Guidelines affected** | Books I to XI (creation); Post. I.1, Post. I.2, Post. I.9; Prop. I.1 to Prop. I.4; Prop. II.1, Prop. II.5; Prop. III.1, Prop. III.9; Prop. IV.2 |

## Context

The company runs autonomous teams on AWS with .NET back-ends and mixed front-ends. Teams have interacted through contracts in practice but without a shared, written statement of what a contract must look like, how it may change, or who decides. Previous attempts at guidelines were lists of preferences; each was argued about until it was ignored. Three principal engineers were asked to produce a set of guidelines that teams can follow without a committee and can dispute without a fight.

The forces are: autonomy of teams (Post. I.3); independent evolution of producers and consumers (Post. I.5); the observed fact that unverified rules decay (Post. I.6); and a stack that is already mostly uniform (Post. I.1, Post. I.2, Post. I.9) but has never been declared canonical.

## Decision

The principal engineers decide:

1. **Form.** The guidelines are written in the manner of Euclid's *Elements*: a small set of *definitions*, *postulates* and *common notions* in Book I, and *propositions* in Books II to XI, each with a demonstration that derives it from those foundations. No rule exists without a demonstration (Prop. I.1). Disagreement is directed at a definition, postulate or common notion, not at the rule.

2. **Synchronous style.** REST/JSON over HTTP, described contract-first in OpenAPI 3.1, is the sole synchronous integration style between teams (Post. I.9, Prop. II.1). Errors use RFC 9457 Problem Details.

3. **Asynchronous transport.** Amazon EventBridge is the bus and Amazon SQS the consumer buffer for all asynchronous communication between teams (Post. I.9, Prop. III.9). Messages carry a CloudEvents 1.0 envelope (Prop. III.1) and payloads are described in JSON Schema 2020-12.

4. **Canonical primitives.** Cursor pagination is the only pagination style (Prop. II.5). The ULID is the canonical identifier (Def. I.18, Prop. IV.2).

5. **Scope.** All eleven books are created now: Books I to IV at detailed depth (Book I complete), Books V to XI at scaffold depth. A scaffold book has full definitions, postulates, and propositions with statement, demonstration, construction outline and conformance check, all at status `Draft`, with its open questions listed in its README (Prop. X.7). Scaffold propositions are promoted to `Accepted` by later ADRs as their machine checks are built (Prop. I.3).

6. **Governance.** This ADR is the first entry of the ADR log (Prop. X.6). All later changes follow Prop. X.1: a pull request with an ADR, accepted by two of the three principal engineers.

## Consequences

Easier: any two teams can integrate by reading the contract and Books II to IV, with no meeting. Reviewers point at a proposition number instead of an opinion. New services start from the same shape, so shared tooling (linters, contract diff, conformance manifests) pays off across the estate.

Harder: existing services that use other transports (gRPC, Kafka, GraphQL, WebSockets), offset pagination, or non-ULID identifiers must either migrate or hold an exception ADR (Prop. X.2) with an expiry. A migration inventory is the first task of each guild (Prop. X.5). Every team must add `conformance.json` (Prop. X.3) within one release cycle of that proposition leaving `Draft`.

Migrated by when: the scaffold books carry open questions; each book's guild proposes ADRs to close them. No calendar deadline is set by this ADR; the deadline for each migration is set by the ADR that promotes the relevant proposition to `Accepted`.

## Demonstration

This ADR creates the foundations rather than changing them, so the demonstration is that the foundations are consistent with each other and with the decisions above.

Decision 1 is Prop. I.1, which rests on CN 3, CN 8 and Post. I.3: autonomous teams follow only rules whose derivation they can see, and what is not written does not exist. Decisions 2 and 3 are the content of Post. I.9, a postulate: they are facts about our estate accepted without proof, and they are recorded here so the acceptance has a date and deciders. Decision 4 restates Def. I.18 and the canonical forms chosen in Books II and IV; by CN 5 one concept has one shape, so a single pagination style and a single identifier form follow once any is chosen. Decision 5 follows from Prop. I.3: a MUST without a machine check is `Draft`, so a scaffold book is one whose checks are not yet built, and nothing else. Decision 6 is Cor. I.4.2 and Post. X.1. No definition, postulate or common notion is changed by this ADR; every proposition in Books I to XI cites only items that exist after it. ∎
