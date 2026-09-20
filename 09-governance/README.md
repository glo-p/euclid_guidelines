# Book X - Governance

Book X says how these guidelines themselves change, how a team departs from them, how we know which services conform, when a design is reviewed, and who owns the questions nobody has answered yet. It is deliberately the smallest point of central control in the company: the principal engineers accept changes; everyone else proposes them.

Depth: **Scaffold**. Every proposition is `Draft` until its machine check exists (Prop. I.3).

## Contents

| File | Contents |
|---|---|
| [`definitions.md`](definitions.md) | `Def. X.1` to `Def. X.9`: guideline, proposal, ADR, exception, conformance manifest, architecture review, principal engineer group, guild, open question. |
| [`postulates.md`](postulates.md) | `Post. X.1` to `Post. X.3`: who accepts changes; guidelines live in git; every service has a repository. |
| [`propositions/`](propositions/) | `Prop. X.1` to `Prop. X.7`. |
| [`adr/`](adr/README.md) | The ADR log: every accepted, rejected and superseded decision. |

## Definitions

| Id | Term | Summary |
|---|---|---|
| Def. X.1 | Guideline | Any item of Books I to XI with an identifier. |
| Def. X.2 | Proposal (RFC) | A pull request that asks for a guideline to be added or changed. |
| Def. X.3 | ADR | The record of one decision, with a demonstration. |
| Def. X.4 | Exception | Refines Def. I.20: an ADR granting a time-bound deviation to a named service. |
| Def. X.5 | Conformance Manifest | The `conformance.json` in a service repository. |
| Def. X.6 | Architecture Review | A review of a design against the guidelines, triggered by an event. |
| Def. X.7 | Principal Engineer Group | The three principal engineers who accept changes. |
| Def. X.8 | Guild | The cross-team group that owns one book. |
| Def. X.9 | Open Question | A decision a book needs that has not yet been taken. |

## Postulates

| Id | Summary |
|---|---|
| Post. X.1 | The three principal engineers accept changes; nobody else does. |
| Post. X.2 | The guidelines live in one git repository and change only by pull request. |
| Post. X.3 | Every service has exactly one repository. |

## Propositions

| Id | Title | Level | Summary |
|---|---|---|---|
| [Prop. X.1](propositions/01-change-by-pull-request-with-adr.md) | Change is a pull request carrying an ADR | MUST | Any change to a guideline is a PR with an ADR and a demonstration, accepted by two of three principals. |
| [Prop. X.2](propositions/02-exception-process.md) | Exceptions expire and are registered | MUST | An exception is an ADR with an expiry of at most 12 months, re-reviewed at expiry, listed in the register. |
| [Prop. X.3](propositions/03-conformance-manifest.md) | Every service publishes a conformance manifest | MUST | `conformance.json` lists each MUST proposition with a status; CI publishes it to the central index. |
| [Prop. X.4](propositions/04-architecture-review-by-event.md) | Architecture review is triggered by events | MUST | Review happens on new service, contract, external dependency, data store or major version; never by calendar. |
| [Prop. X.5](propositions/05-guilds-own-books.md) | Each book has a guild | SHOULD | A guild owns a book's open questions and proposes its ADRs. |
| [Prop. X.6](propositions/06-the-adr-log.md) | The ADR log | MUST | Every decision is an ADR in `adr/`, numbered, indexed and never deleted. |
| [Prop. X.7](propositions/07-open-questions-resolved-by-adr.md) | Open questions are tracked per book and resolved only by ADR | MUST | Each README lists its open questions; only an ADR closes one. |

## Reading order

Definitions, then postulates, then X.1 and X.6 (how a decision is made and kept), then X.2 and X.3 (how a service departs from or proves conformance to the rules), then X.4, X.5 and X.7.

## Open questions

Decisions the principal engineers still need to make. Each is resolved only by an ADR (Prop. X.7).

1. **Maximum exception length.** Prop. X.2 fixes 12 months. Confirm, and decide whether an exception may be renewed more than once.
2. **Conformance index store.** Prop. X.3 needs a central index. Candidates: an S3 bucket plus Athena, a DynamoDB table with a small read API, or the catalogue repository itself. Choose one.
3. **Manifest granularity.** Decide whether `conformance.json` covers only MUST propositions or also SHOULD deviations, which Prop. I.2 requires to be recorded in the repository anyway.
4. **Review quorum.** Prop. X.4 says who triggers a review; decide who must attend (principal plus guild lead of each affected book is the proposal) and the maximum turnaround.
5. **Guild membership.** Decide whether guild membership is voluntary, nominated by teams, or one representative per team, and how a guild lead is chosen.
6. **Principal unavailability.** Post. X.1 assumes three principals. Decide the rule when one is absent for more than two weeks, and whether a named deputy may accept.
7. **RFC discussion venue.** Decide whether a proposal is discussed on the pull request only, or in a GitHub Discussion linked from the pull request.
8. **Non-conformance escalation.** Decide what happens when a service reports `non-conformant` for a MUST with no exception for more than one release cycle.
