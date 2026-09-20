# Book VII - Data Ownership

Book VII fixes who owns each piece of data, where it may be copied, how copies are fed and rebuilt, how long data lives, where it lives, and how identifiers are minted. It rests on [Post. I.3](../01-foundations/postulates.md#Post.%20I.3%20-%20Autonomy) (autonomy) and refines it into rules about stores.

**Depth:** Scaffold. Every proposition is `Draft`.

## Definitions

| Item | Summary |
|---|---|
| [`definitions.md`](definitions.md) | [`Def. VII.1`](definitions.md#Def.%20VII.1%20-%20System%20of%20Record) - [`Def. VII.12`](definitions.md#Def.%20VII.12%20-%20Expand%2FContract). |
| [Def. VII.1](definitions.md#Def.%20VII.1%20-%20System%20of%20Record) System of record | The single service whose store holds authoritative state for a concept. |
| [Def. VII.2](definitions.md#Def.%20VII.2%20-%20Data%20Owner) Data owner | The team owning the system of record; decides schema, retention, classification. |
| [Def. VII.3](definitions.md#Def.%20VII.3%20-%20Read%20Model) Read model | A derived, disposable, eventually consistent copy held by another service. |
| [Def. VII.4](definitions.md#Def.%20VII.4%20-%20Projection) Projection | The deterministic function from events to a read model. |
| [Def. VII.5](definitions.md#Def.%20VII.5%20-%20Replica) Replica | An engine-maintained copy of a database; as private as its primary. |
| [Def. VII.6](definitions.md#Def.%20VII.6%20-%20Reporting%20Store) Reporting store | A cross-context store for analytics, fed only by events or CDC. |
| [Def. VII.7](definitions.md#Def.%20VII.7%20-%20Reference%20Data) Reference data | Externally defined value sets nobody owns as a business concept. |
| [Def. VII.8](definitions.md#Def.%20VII.8%20-%20Master%20Data) Master data | Business concepts many services need and one owns. |
| [Def. VII.9](definitions.md#Def.%20VII.9%20-%20Retention%20Period) Retention period | Time after a trigger by which data must be deleted or anonymised, in every copy. |
| [Def. VII.10](definitions.md#Def.%20VII.10%20-%20Data%20Residency) Data residency | The regions in which a tenant's data may be stored and processed. |
| [Def. VII.11](definitions.md#Def.%20VII.11%20-%20Migration) Migration | A versioned, ordered, tool-applied change to a store's structure. |
| [Def. VII.12](definitions.md#Def.%20VII.12%20-%20Expand%2FContract) Expand/contract | Add the new shape, move the code, remove the old shape, in separate deployments. |

## Postulates

| Item | Summary |
|---|---|
| [`postulates.md`](postulates.md) | [`Post. VII.1`](postulates.md#Post.%20VII.1%20-%20One%20System%20of%20Record) - [`Post. VII.4`](postulates.md#Post.%20VII.4%20-%20Default%20Stores). |
| [Post. VII.1](postulates.md#Post.%20VII.1%20-%20One%20System%20of%20Record) One system of record | Every aggregate has exactly one. |
| [Post. VII.2](postulates.md#Post.%20VII.2%20-%20Private%20Database) Private database | A database, its replicas and backups are private to the service (from [Post. I.3](../01-foundations/postulates.md#Post.%20I.3%20-%20Autonomy)). |
| [Post. VII.3](postulates.md#Post.%20VII.3%20-%20Reporting%20Is%20Fed%2C%20Not%20Read) Reporting is fed, not read | Analytics take events or CDC, never direct database reads. |
| [Post. VII.4](postulates.md#Post.%20VII.4%20-%20Default%20Stores) Default stores | SQL Server or PostgreSQL on RDS/Aurora; DynamoDB for key-value. Open question. |

## Propositions

| Prop. | File | Level | Summary |
|---|---|---|---|
| VII.1 | [`01-one-database-per-service.md`](propositions/01-one-database-per-service.md) | MUST | One database per service; cross-service access blocked by network and IAM. |
| VII.2 | [`02-system-of-record-in-catalogue.md`](propositions/02-system-of-record-in-catalogue.md) | MUST | The catalogue names one system of record per concept; every other copy is a read model. |
| VII.3 | [`03-read-models-from-events.md`](propositions/03-read-models-from-events.md) | MUST | Read models are projections of integration events and are rebuildable from the archive. |
| VII.4 | [`04-reference-data-distribution.md`](propositions/04-reference-data-distribution.md) | MUST | Reference data ships as a versioned dataset with its shared schema. |
| VII.5 | [`05-retention-and-deletion.md`](propositions/05-retention-and-deletion.md) | MUST | Retention per classification applied to every copy; erasure propagated by event. |
| VII.6 | [`06-backward-compatible-migrations.md`](propositions/06-backward-compatible-migrations.md) | MUST | Migrations are compatible with the running version; expand/contract; no downtime. |
| VII.7 | [`07-analytics-via-data-platform.md`](propositions/07-analytics-via-data-platform.md) | MUST | Analytics on S3 + Glue + Athena fed by events or CDC; never production replicas. |
| VII.8 | [`08-data-residency.md`](propositions/08-data-residency.md) | MUST | A tenant's data lives in the tenant's home region, in every copy. |
| VII.9 | [`09-ulids-from-system-of-record.md`](propositions/09-ulids-from-system-of-record.md) | MUST | Identifiers are ULIDs minted by the system of record; surrogate keys stay internal. |

## Open questions

Decisions the principal engineers still need to make before items leave `Draft`:

1. **[Post. VII.4](postulates.md#Post.%20VII.4%20-%20Default%20Stores), default relational engine.** Keep both SQL Server and PostgreSQL as defaults, or name one for new services. Licensing cost and Aurora feature parity are the deciding factors.
2. **[Post. VII.4](postulates.md#Post.%20VII.4%20-%20Default%20Stores), DynamoDB.** Default key-value store for all new services, or a sanctioned option only where the access pattern is known in advance.
3. **[Prop. VII.3](propositions/03-read-models-from-events.md), rebuild drill cadence.** Once per environment per release, or a scheduled quarterly drill; and whether the drill is a promotion gate.
4. **[Prop. VII.5](propositions/05-retention-and-deletion.md), erasure deadline.** The number of days within which every copy must have honoured `Subject.ErasureRequested`, and whether the archive uses crypto-shredding or field-level tokenisation.
5. **[Prop. VII.7](propositions/07-analytics-via-data-platform.md), Redshift.** Whether Amazon Redshift Serverless is added as a curated layer, and the volume or latency threshold that triggers it.
6. **[Prop. VII.7](propositions/07-analytics-via-data-platform.md), CDC.** Whether change data capture (DMS, Aurora zero-ETL) is a sanctioned feed or an exception, given that events are the preferred contract.
7. **[Prop. VII.8](propositions/08-data-residency.md), region set.** The list of approved regions and whether a tenant may be homed in more than one for disaster recovery.
8. **[Prop. VII.6](propositions/06-backward-compatible-migrations.md), migration execution.** Pipeline step before rotation versus application start-up; the former is assumed here.
9. **[Def. VII.2](definitions.md#Def.%20VII.2%20-%20Data%20Owner) vs [Def. I.17](../01-foundations/definitions.md#Def.%20I.17%20-%20Team).** Whether "data owner" needs to exist as a separate term or is always identical to the owning team of the system of record.
