# Book VII - Definitions

These definitions refine or extend Book I for the subject of data ownership. They are not rules. Where a term is already fixed in Book I it is cited, not restated.

---

**Def. VII.1 - System of Record.** The *system of record* for a business concept is the single service (Def. I.1) whose store holds the authoritative state of the aggregates (Def. I.6) that represent that concept. A write that does not pass through the system of record does not change the concept; it changes a copy.

**Def. VII.2 - Data Owner.** The *data owner* of a business concept is the team (Def. I.17) that owns its system of record. The data owner decides the schema, the retention period, the classification of each field and who may receive the data.

**Def. VII.3 - Read Model.** A *read model* is a store, held by any service other than the system of record, that contains a copy of some part of a concept shaped for a particular query. A read model is derived, disposable and eventually consistent; it is never written to directly by a user of the platform.

**Def. VII.4 - Projection.** A *projection* is the deterministic function that builds or updates a read model from a sequence of events (Def. I.9). Applying the same events in the same order to an empty read model yields the same read model.

**Def. VII.5 - Replica.** A *replica* is a byte-for-byte or row-for-row copy of a service's database maintained by the database engine (for example an Aurora reader instance or a cross-region read replica). A replica belongs to the same service as its primary and is subject to the same privacy as the primary (Post. VII.2).

**Def. VII.6 - Reporting Store.** A *reporting store* is a store, outside every service, that holds data from many bounded contexts (Def. I.5) for analytical or regulatory querying. It is fed only by events or change data capture (Post. VII.3).

**Def. VII.7 - Reference Data.** *Reference data* is a slowly changing, externally defined set of values that many services need and none owns as a business concept: currencies, countries, time zones, units of measure, industry codes.

**Def. VII.8 - Master Data.** *Master data* is a business concept that many services need and exactly one service owns: customer, product, organisation, tenant (Def. I.23). Master data has a system of record (Def. VII.1); reference data does not.

**Def. VII.9 - Retention Period.** The *retention period* of a class of data is the interval, measured from a defined trigger, after which the data owner (Def. VII.2) is obliged to delete or irreversibly anonymise it. A retention period applies to every copy, including read models, reporting stores, archives and backups.

**Def. VII.10 - Data Residency.** *Data residency* is the constraint that data belonging to a tenant (Def. I.23) is stored and processed only within a named set of geographic regions. The set is a property of the tenant, not of the service.

**Def. VII.11 - Migration.** A *migration* (schema migration) is a versioned, ordered, repeatable change to the structure of a service's store, applied by a tool rather than by hand, and recorded in the store so that it is applied exactly once.

**Def. VII.12 - Expand/Contract.** *Expand/contract* is the discipline of making a structural change in two or more deployments: first *expand* the store so that both the old and the new shape are valid, then move the code, then *contract* by removing the old shape only once no running version depends on it.
