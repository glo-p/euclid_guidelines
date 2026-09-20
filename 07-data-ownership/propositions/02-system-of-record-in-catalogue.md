# Prop. VII.2 - The system of record is named in the catalogue

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

For every business concept, the catalogue (Def. I.24) MUST name exactly one system of record (Def. VII.1) and its data owner (Def. VII.2). Every other store holding that concept MUST be declared a read model (Def. VII.3) and MUST NOT be presented to any consumer as authoritative.

## Given

* Def. I.6, Def. I.24, Def. VII.1, Def. VII.2, Def. VII.3
* Post. VII.1
* CN 3, CN 5, CN 8

## Demonstration

Post. VII.1 gives each aggregate exactly one system of record, but a fact that is not written down does not exist for a consumer (CN 3) and is not architecture (CN 8). Therefore the assignment must be recorded, and the only place shared by every team is the catalogue (Def. I.24). Once one store is authoritative, every other store is by definition a copy (Def. VII.3); leaving a copy's status implicit invites two "truths" for one concept, which CN 5 identifies as cost without benefit. ∎ Q.E.D.

## Corollaries

* **Cor. VII.2.1** - A concept with no entry in the catalogue has no system of record and MUST NOT be written by any service until one is assigned.
* **Cor. VII.2.2** - Moving a concept's system of record between services is a breaking change to the concept and is performed under Book IX.

## Construction

* A `systems-of-record.yaml` file in the catalogue repository (Prop. IV.1) listing concept, owning service, owning team, schema reference and classification.
* A CI check in the catalogue that every shared schema (Prop. IV.1) for a business concept references an entry in that file.
* The service's own OpenAPI document (Prop. II.1) declares in `info.x-system-of-record` the concepts it is authoritative for; read-model endpoints declare `x-read-model-of` naming the source concept.

## Conformance

Catalogue CI job `sor-registry-check` fails if a concept appears twice, has no owner, or is referenced by an OpenAPI `x-system-of-record` extension in more than one service contract.

## Scholium

The catalogue entry is small. Its value is that the question "who owns customer address" has one answer that a machine can look up.
