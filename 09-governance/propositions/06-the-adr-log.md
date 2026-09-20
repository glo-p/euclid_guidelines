# Prop. X.6 - The ADR log

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every ADR (Def. X.3) MUST live in [`09-governance/adr/`](../adr/README.md) as a file named `ADR-nnnn-<kebab-title>.md`, numbered sequentially from `ADR-0001`, and MUST be listed in [`adr/README.md`](../adr/README.md) with its number, title, status and date. An ADR MUST NOT be deleted or renumbered; a decision that is reversed is recorded by a new ADR that marks the old one `Superseded by ADR-nnnn`. The first entry, [`ADR-0001`](../adr/ADR-0001-adopt-euclidean-guidelines.md), records the adoption of these guidelines and the canonical stack.

## Given

* Def. X.1, Def. X.3
* Post. X.2
* CN 8
* Cor. I.4.1, Prop. I.4, Prop. X.1

## Demonstration

By CN 8 a decision that is not recorded is not architecture, and Prop. I.4 makes the ADR the record; a record that cannot be found is not a record, so the ADRs are in one known folder with one index. Post. X.2 makes git history the complete record of change, which holds only if no file is deleted or renumbered; Cor. I.4.1 already applies that rule to guideline identifiers, and ADR numbers are identifiers of the same kind. Prop. X.1 makes every change carry an ADR, so the log grows only through accepted or rejected proposals and is therefore complete. ∎ Q.E.D.

## Corollaries

* **Cor. X.6.1** - Every `Accepted` ADR is referenced from at least one guideline it affected, in a `Supersedes` row or a Scholium, so the derivation can be traced in both directions.
* **Cor. X.6.2** - Exception ADRs (Prop. X.2) are part of the same sequence; there is one number space.

## Construction

* Folder `09-governance/adr/` with `README.md` index and `ADR-nnnn-*.md` files from `templates/adr.md`.
* CI `adr-lint`: filename pattern, contiguous numbering, header table fields, status vocabulary, `Superseded by` target exists, index row present and matching.
* CI `adr-index-sync`: regenerates the index table from the files and fails if the committed index differs.
* `git log --diff-filter=D -- 09-governance/adr/` checked in CI to fail any deletion.

## Conformance

CI checks `adr-lint` and `adr-index-sync` are required status checks on the guidelines repository.

## Scholium

The index is generated but committed, so that a reader on a laptop with no CI sees the same list. A rejected ADR is as valuable as an accepted one: it records why not.
