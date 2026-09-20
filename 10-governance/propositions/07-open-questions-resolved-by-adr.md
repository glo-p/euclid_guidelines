# Prop. X.7 - Open questions are tracked per book and resolved only by ADR

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every open question ([Def. X.9](../definitions.md#Def.%20X.9%20-%20Open%20Question)) MUST be listed, numbered, under the heading "Open questions" in the `README.md` of the book that needs it, with the proposition(s) it blocks. An open question MUST be removed from that list only by a pull request that contains the ADR ([Def. X.3](../definitions.md#Def.%20X.3%20-%20ADR)) resolving it, accepted under [Prop. X.1](01-change-by-pull-request-with-adr.md), and that pull request MUST update the propositions the answer unblocks. A proposition that depends on an open question MUST remain `Draft`.

## Given

* [Def. X.3](../definitions.md#Def.%20X.3%20-%20ADR), [Def. X.9](../definitions.md#Def.%20X.9%20-%20Open%20Question)
* [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification), [Post. X.2](../postulates.md#Post.%20X.2%20-%20Guidelines%20Live%20in%20Git)
* [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts)
* [Prop. I.3](../../01-foundations/method.md#Prop.%20I.3%20-%20Every%20MUST%20has%20a%20machine%20check), [Prop. X.1](01-change-by-pull-request-with-adr.md), [Prop. X.6](06-the-adr-log.md)

## Demonstration

By [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit) what is not written does not exist; an unwritten open question is a rule that each team answers differently, which is the absence of a guideline. Listing it in the book's README makes the gap explicit and [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts) makes the listing part of the architecture. A question that is closed by a conversation is not closed by [Post. X.2](../postulates.md#Post.%20X.2%20-%20Guidelines%20Live%20in%20Git), which recognises only merged pull requests; and [Prop. X.1](01-change-by-pull-request-with-adr.md) says such a pull request carries an ADR, which [Prop. X.6](06-the-adr-log.md) then keeps. [Prop. I.3](../../01-foundations/method.md#Prop.%20I.3%20-%20Every%20MUST%20has%20a%20machine%20check) already withholds `Accepted` from a MUST that cannot be checked; a MUST that cannot be stated because a question is open cannot be checked either, so it stays `Draft`. By [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification) the link between question and proposition is enforced by CI. ∎ Q.E.D.

## Corollaries

* **Cor. X.7.1** - A guild ([Prop. X.5](05-guilds-own-books.md)) may reorder or reword open questions without an ADR; it may not remove one.
* **Cor. X.7.2** - The set of all open questions across all READMEs is the backlog of the principal engineer group.

## Construction

* Fixed README section format: `## Open questions`, an ordered list, each item ending with `Blocks: Prop. B.n[, ...]`.
* CI `open-questions-lint`: parses every `*/README.md`; fails if a listed item removed in the diff is not matched by a new `adr/ADR-*.md` in the same pull request whose `Guidelines affected` row names the book; fails if a proposition named in `Blocks:` has status other than `Draft`.
* Workflow `open-questions-sync`: mirrors each item to a GitHub issue labelled `open-question:<book>` for the guild board ([Prop. X.5](05-guilds-own-books.md)) and closes the issue when the item is removed.
* ADR template row `Resolves open question | <book> #n`.

## Conformance

CI check `open-questions-lint` required on the guidelines repository; weekly governance report listing open questions by book and age.

## Scholium

The scaffold books were written with their open questions on purpose. Writing "we have not decided" is better than writing a number nobody agreed to; the number can be changed by ADR, and the reader knows which numbers are provisional.
