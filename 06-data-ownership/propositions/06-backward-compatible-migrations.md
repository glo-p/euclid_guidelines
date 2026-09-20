# Prop. VII.6 - Schema migrations are backward compatible and use expand/contract

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every migration (Def. VII.11) MUST be compatible with the version of the service that is running when it is applied. Structural changes MUST follow expand/contract (Def. VII.12); a single migration MUST NOT both add the new shape and remove the old. Migrations MUST run without taking the service out of rotation.

## Given

* Def. I.14, Def. I.15, Def. VII.11, Def. VII.12
* Post. I.4, Post. I.5, Post. I.6
* Prop. VIII.7, Prop. VIII.8
* CN 4

## Demonstration

Deployments are rolling and independent (Post. I.5), so for some interval both the old and the new version of a service run against one store; a migration that only the new version tolerates breaks the old one during that interval. The store's structure is, to the service's own code, a contract, and the definitions of breaking and compatible change (Def. I.14, Def. I.15) apply: adding is compatible, removing is not until nothing depends on it. CN 4 makes a migration that mixes an add and a remove breaking as a whole; hence the two steps are separate deployments (Def. VII.12). Because any step may fail part way (Post. I.4), each migration is small, ordered and recorded (Def. VII.11), and by Post. I.6 the compatibility is checked by a machine rather than by review. ∎ Q.E.D.

## Corollaries

* **Cor. VII.6.1** - Renaming a column is three deployments: add the new column and dual-write, switch reads and backfill, drop the old column.
* **Cor. VII.6.2** - A migration is part of the artefact that is promoted (Prop. VIII.7); it is never run by hand in any environment above sandbox.

## Construction

* Migration tooling: EF Core migrations (`Microsoft.EntityFrameworkCore`) or `DbUp`/`FluentMigrator` for SQL-first teams; DynamoDB shape changes via versioned item attributes and a backfill Lambda.
* Migrations applied by a dedicated pipeline step or an ECS one-off task before the new version is placed in rotation, using the same container image.
* Long-running backfills as ECS tasks or Step Functions with checkpointing, never inside the migration transaction.
* Online DDL where the engine supports it (PostgreSQL `CREATE INDEX CONCURRENTLY`, SQL Server `ONLINE = ON`); Aurora blue/green deployments for engine version changes.

## Conformance

CI job `migration-compat` that applies the candidate migrations to a database snapshot and runs the previous release's integration test suite against it; a linter rule rejecting `DROP`, `RENAME` or `NOT NULL` without default in a migration that also contains `ADD`.

## Scholium

Expand/contract is slower to write and faster to run. Teams that skip the contract step accumulate dead columns; that is a lesser sin than an outage, and is cleaned up by a quarterly review.
