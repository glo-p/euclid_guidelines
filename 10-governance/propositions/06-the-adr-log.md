# Prop. X.6 - The ADR log

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every ADR ([Def. X.3](../definitions.md#Def.%20X.3%20-%20ADR)) MUST live in [`10-governance/adr/`](../adr/README.md) as a file named `ADR-nnnn-<kebab-title>.md`, numbered sequentially from `ADR-0001`, and MUST be listed in [`adr/README.md`](../adr/README.md) with its number, title, status and date. An ADR MUST NOT be deleted or renumbered; a decision that is reversed is recorded by a new ADR that marks the old one `Superseded by ADR-nnnn`. The first entry, [`ADR-0001`](../adr/ADR-0001-adopt-euclidean-guidelines.md), records the adoption of these guidelines and the canonical stack.

## Given

* [Def. X.1](../definitions.md#Def.%20X.1%20-%20Guideline), [Def. X.3](../definitions.md#Def.%20X.3%20-%20ADR)
* [Post. X.2](../postulates.md#Post.%20X.2%20-%20Guidelines%20Live%20in%20Git)
* [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts)
* [Cor. I.4.1](../../01-foundations/method.md#Prop.%20I.4%20-%20Change%20is%20by%20ADR%20against%20the%20foundations), [Prop. I.4](../../01-foundations/method.md#Prop.%20I.4%20-%20Change%20is%20by%20ADR%20against%20the%20foundations), [Prop. X.1](01-change-by-pull-request-with-adr.md)

## Demonstration

By [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts) a decision that is not recorded is not architecture, and [Prop. I.4](../../01-foundations/method.md#Prop.%20I.4%20-%20Change%20is%20by%20ADR%20against%20the%20foundations) makes the ADR the record; a record that cannot be found is not a record, so the ADRs are in one known folder with one index. [Post. X.2](../postulates.md#Post.%20X.2%20-%20Guidelines%20Live%20in%20Git) makes git history the complete record of change, which holds only if no file is deleted or renumbered; [Cor. I.4.1](../../01-foundations/method.md#Prop.%20I.4%20-%20Change%20is%20by%20ADR%20against%20the%20foundations) already applies that rule to guideline identifiers, and ADR numbers are identifiers of the same kind. [Prop. X.1](01-change-by-pull-request-with-adr.md) makes every change carry an ADR, so the log grows only through accepted or rejected proposals and is therefore complete. ∎ Q.E.D.

## Corollaries

* **Cor. X.6.1** - Every `Accepted` ADR is referenced from at least one guideline it affected, in a `Supersedes` row or a Scholium, so the derivation can be traced in both directions.
* **Cor. X.6.2** - Exception ADRs ([Prop. X.2](02-exception-process.md)) are part of the same sequence; there is one number space.

## Construction

* Folder `10-governance/adr/` with `README.md` index and `ADR-nnnn-*.md` files from `templates/adr.md`.
* CI `adr-lint`: filename pattern, contiguous numbering, header table fields, status vocabulary, `Superseded by` target exists, index row present and matching.
* CI `adr-index-sync`: regenerates the index table from the files and fails if the committed index differs.
* `git log --diff-filter=D -- 10-governance/adr/` checked in CI to fail any deletion.

## Conformance

CI checks `adr-lint` and `adr-index-sync` are required status checks on the guidelines repository.

## Scholium

The index is generated but committed, so that a reader on a laptop with no CI sees the same list. A rejected ADR is as valuable as an accepted one: it records why not.
