# Book X - Definitions

These definitions fix the vocabulary of governance used in [Book X](README.md) and cited by every other book when it names an ADR, an exception or an open question. They are not rules. Where a definition refines one in [Book I](../01-foundations/README.md) it says so.

---

## Def. X.1 - Guideline

A *guideline* is any item of Books I to XI that carries an identifier: a definition, a postulate, a common notion, a proposition, or a corollary. The guidelines are exactly the contents of the repository named in [Post. X.2](postulates.md#Post.%20X.2%20-%20Guidelines%20Live%20in%20Git); nothing outside it is a guideline.

## Def. X.2 - Proposal (RFC)

A *proposal* is a pull request against the guidelines repository that asks for a guideline ([Def. X.1](definitions.md#Def.%20X.1%20-%20Guideline)) to be added, changed, deprecated, or for an open question ([Def. X.9](definitions.md#Def.%20X.9%20-%20Open%20Question)) to be resolved. A proposal contains exactly one ADR ([Def. X.3](definitions.md#Def.%20X.3%20-%20ADR)) plus the guideline changes that ADR describes. It is open to any employee.

## Def. X.3 - ADR

An *architecture decision record* (ADR) is a document in [`adr/`](adr/README.md), written from [`templates/adr.md`](../templates/adr.md), that records one decision: its context, the decision, its consequences, and a demonstration showing that every guideline it touches still follows from the definitions, postulates and common notions, or stating which of those it changes. An ADR has a permanent number `ADR-nnnn` and exactly one status: `Proposed`, `Accepted`, `Rejected`, or `Superseded by ADR-nnnn`.

## Def. X.4 - Exception

*(Refines [Def. I.20](../01-foundations/definitions.md#Def.%20I.20%20-%20Exception).)* An *exception* is an ADR ([Def. X.3](definitions.md#Def.%20X.3%20-%20ADR)) that grants one named service ([Def. I.1](../01-foundations/definitions.md#Def.%20I.1%20-%20Service)) a deviation from one named MUST or SHOULD proposition, states the reason, states the compensating control, and carries an expiry date at most 12 months after acceptance. An exception binds only the service it names. It is listed in the exception register ([Prop. X.2](propositions/02-exception-process.md)) and referenced from that service's conformance manifest ([Def. X.5](definitions.md#Def.%20X.5%20-%20Conformance%20Manifest)).

## Def. X.5 - Conformance Manifest

A *conformance manifest* is the file `conformance.json` at the root of a service repository ([Post. X.3](postulates.md#Post.%20X.3%20-%20Every%20Service%20Has%20a%20Repository)) that lists every MUST proposition of Books II to XI applicable to that service and, for each, one of the statuses `conformant`, `exception ADR-nnnn`, or `non-conformant`, together with the evidence reference for `conformant` entries. Its shape is fixed by a JSON Schema in the catalogue ([Def. I.24](../01-foundations/definitions.md#Def.%20I.24%20-%20Catalogue)).

## Def. X.6 - Architecture Review

An *architecture review* is a recorded examination of a proposed design against the guidelines, held before the design is built, producing either a finding of conformance, a list of guidelines the design would violate, or a proposal ([Def. X.2](definitions.md#Def.%20X.2%20-%20Proposal%20%28RFC%29)) to change a guideline. It is triggered by the events of [Prop. X.4](propositions/04-architecture-review-by-event.md) and by nothing else.

## Def. X.7 - Principal Engineer Group

The *principal engineer group* is the three principal engineers named in the repository `CODEOWNERS` file. It is the only body that can move an ADR to `Accepted` or `Rejected` ([Cor. I.4.2](../01-foundations/method.md#Prop.%20I.4%20-%20Change%20is%20by%20ADR%20against%20the%20foundations), [Post. X.1](postulates.md#Post.%20X.1%20-%20Three%20Principals%20Accept)).

## Def. X.8 - Guild

A *guild* is a named, cross-team group of engineers that owns one book ([Def. X.1](definitions.md#Def.%20X.1%20-%20Guideline)): it keeps the book's open questions ([Def. X.9](definitions.md#Def.%20X.9%20-%20Open%20Question)), drafts the proposals ([Def. X.2](definitions.md#Def.%20X.2%20-%20Proposal%20%28RFC%29)) that resolve them, and reviews proposals against that book before the principal engineer group decides. A guild proposes; it does not accept.

## Def. X.9 - Open Question

An *open question* is a decision that a book needs in order for one of its propositions to leave `Draft`, that has not yet been taken, and that is listed under the heading "Open questions" in that book's `README.md`. An open question is closed only by an accepted ADR ([Prop. X.7](propositions/07-open-questions-resolved-by-adr.md)).
