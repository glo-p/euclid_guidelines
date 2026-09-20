# Prop. X.1 - Change is a pull request carrying an ADR

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Any change to a guideline ([Def. X.1](../definitions.md#Def.%20X.1%20-%20Guideline)) MUST be made as a proposal ([Def. X.2](../definitions.md#Def.%20X.2%20-%20Proposal%20%28RFC%29)): one pull request to the guidelines repository containing exactly one ADR ([Def. X.3](../definitions.md#Def.%20X.3%20-%20ADR)) whose Demonstration section shows, per [Prop. I.4](../../01-foundations/method.md#Prop.%20I.4%20-%20Change%20is%20by%20ADR%20against%20the%20foundations), that every guideline citing the changed items still holds or is amended in the same pull request. The pull request MUST NOT be merged until two of the three principal engineers ([Def. X.7](../definitions.md#Def.%20X.7%20-%20Principal%20Engineer%20Group)) have approved it, and merging MUST set the ADR status to `Accepted`.

## Given

* [Def. X.1](../definitions.md#Def.%20X.1%20-%20Guideline), [Def. X.2](../definitions.md#Def.%20X.2%20-%20Proposal%20%28RFC%29), [Def. X.3](../definitions.md#Def.%20X.3%20-%20ADR), [Def. X.7](../definitions.md#Def.%20X.7%20-%20Principal%20Engineer%20Group)
* [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification), [Post. X.1](../postulates.md#Post.%20X.1%20-%20Three%20Principals%20Accept), [Post. X.2](../postulates.md#Post.%20X.2%20-%20Guidelines%20Live%20in%20Git)
* [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts)
* [Prop. I.1](../../01-foundations/method.md#Prop.%20I.1%20-%20Every%20rule%20is%20a%20proposition%20with%20a%20demonstration), [Prop. I.4](../../01-foundations/method.md#Prop.%20I.4%20-%20Change%20is%20by%20ADR%20against%20the%20foundations), [Cor. I.4.2](../../01-foundations/method.md#Prop.%20I.4%20-%20Change%20is%20by%20ADR%20against%20the%20foundations)

## Demonstration

By [Post. X.2](../postulates.md#Post.%20X.2%20-%20Guidelines%20Live%20in%20Git) a guideline is in force only when merged, so a pull request is the only mechanism by which a change can happen at all. [Prop. I.4](../../01-foundations/method.md#Prop.%20I.4%20-%20Change%20is%20by%20ADR%20against%20the%20foundations) already requires that a change be an ADR carrying the enumeration of consequences, and [Prop. I.1](../../01-foundations/method.md#Prop.%20I.1%20-%20Every%20rule%20is%20a%20proposition%20with%20a%20demonstration) requires that any new rule be demonstrated; placing the ADR in the same pull request as the change keeps the record and the change inseparable, which [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts) requires. [Post. X.1](../postulates.md#Post.%20X.1%20-%20Three%20Principals%20Accept) and [Cor. I.4.2](../../01-foundations/method.md#Prop.%20I.4%20-%20Change%20is%20by%20ADR%20against%20the%20foundations) fix who may accept and how many, and by [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification) that rule must be enforced by the repository rather than by memory, so branch protection carries the quorum. ∎ Q.E.D.

## Corollaries

* **Cor. X.1.1** - A pull request that changes a guideline file but contains no ADR is rejected by CI, not by a reviewer.
* **Cor. X.1.2** - A rejected proposal keeps its ADR, merged with status `Rejected`, so that the reasoning is not lost and the question is not reopened without new facts.

## Construction

* GitHub branch protection on the default branch: two required approvals from the `CODEOWNERS` team `@company/principal-engineers`, dismiss stale reviews, no administrator bypass.
* `CODEOWNERS` mapping every path except `*/README.md` "Open questions" sections and `adr/` drafts to the principal engineer team.
* CI workflow `guideline-change-guard`: fails when files under `00-*` to `10-*` change and no new or modified `adr/ADR-*.md` file is in the diff.
* CI workflow `adr-lint`: validates the ADR header table, required sections, a `Guidelines affected` row listing every changed identifier, and a Demonstration section that is non-empty.
* Merge action sets `Status | Accepted` in the ADR and appends the row to `adr/README.md` ([Prop. X.6](06-the-adr-log.md)).
* Pull request template `.github/PULL_REQUEST_TEMPLATE/proposal.md`.

## Conformance

CI checks `guideline-change-guard` and `adr-lint` are required status checks; branch protection rules exported by `gh api` and diffed against the expected policy in a weekly job.

## Scholium

The quorum is two of three so that one absent principal does not stop the company. It is not one of three so that no single person can change what every team is bound by.
