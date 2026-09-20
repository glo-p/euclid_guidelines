# Prop. X.5 - Each book has a guild

| | |
|---|---|
| **Level** | SHOULD |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Each of Books II to XI SHOULD have exactly one guild ([Def. X.8](../definitions.md#Def.%20X.8%20-%20Guild)), named in that book's `README.md`, which owns the book's open questions ([Def. X.9](../definitions.md#Def.%20X.9%20-%20Open%20Question)), drafts the proposals ([Def. X.2](../definitions.md#Def.%20X.2%20-%20Proposal%20%28RFC%29)) that resolve them, and reviews every proposal touching the book before it reaches the principal engineer group ([Def. X.7](../definitions.md#Def.%20X.7%20-%20Principal%20Engineer%20Group)). A guild SHOULD include at least one engineer from every team that produces a contract governed by the book.

## Given

* [Def. X.2](../definitions.md#Def.%20X.2%20-%20Proposal%20%28RFC%29), [Def. X.7](../definitions.md#Def.%20X.7%20-%20Principal%20Engineer%20Group), [Def. X.8](../definitions.md#Def.%20X.8%20-%20Guild), [Def. X.9](../definitions.md#Def.%20X.9%20-%20Open%20Question)
* [Post. I.3](../../01-foundations/postulates.md#Post.%20I.3%20-%20Autonomy), [Post. X.1](../postulates.md#Post.%20X.1%20-%20Three%20Principals%20Accept)
* [Cor. I.4.2](../../01-foundations/method.md#Prop.%20I.4%20-%20Change%20is%20by%20ADR%20against%20the%20foundations)
* [Prop. X.1](01-change-by-pull-request-with-adr.md), [Prop. X.7](07-open-questions-resolved-by-adr.md)

## Demonstration

[Post. X.1](../postulates.md#Post.%20X.1%20-%20Three%20Principals%20Accept) concentrates acceptance in three people, and three people cannot draft and research every proposal for ten books while also doing their other work; so drafting must be distributed. [Post. I.3](../../01-foundations/postulates.md#Post.%20I.3%20-%20Autonomy) says teams are autonomous and interact through contracts, so the people who know whether a proposed rule fits a contract are the teams that produce those contracts; a guild is the smallest group that contains them. [Cor. I.4.2](../../01-foundations/method.md#Prop.%20I.4%20-%20Change%20is%20by%20ADR%20against%20the%20foundations) keeps acceptance with the principals, so a guild proposes and reviews but never accepts, which is exactly [Def. X.8](../definitions.md#Def.%20X.8%20-%20Guild). [Prop. X.7](07-open-questions-resolved-by-adr.md) requires every open question to be closed by an ADR, and an ADR needs an author; the guild is that author by default. ∎ Q.E.D.

## Corollaries

* **Cor. X.5.1** - A proposal touching a book that has a guild is not put to the principal engineer group until the guild has reviewed it or 10 working days have passed.

## Construction

* GitHub team per guild `@company/guild-<book-slug>` listed as a reviewer (not code owner) in `CODEOWNERS` for that book's folder.
* `README.md` of each book: a "Guild" row naming the team, its lead and its meeting cadence.
* Open questions issue label `open-question:<book>` and a GitHub Project board per guild, generated from the README section by the `open-questions-sync` workflow ([Prop. X.7](07-open-questions-resolved-by-adr.md)).
* Guild charter file `10-governance/guilds/<book-slug>.md` (added when a guild is formed) listing members and the open questions they hold.

## Conformance

CI check `guild-review-requested` adds the guild team as reviewer on any pull request touching its book; a book README without a "Guild" row is reported by the weekly governance report.

## Scholium

This is a SHOULD because a book with few contracts and no open questions ([Book I](../../01-foundations/README.md), once stable) may not need a guild. The point is ownership of questions, not meetings. Membership rules are an open question.
