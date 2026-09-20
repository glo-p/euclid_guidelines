# Book VII - Postulates

Postulates are things we accept as true about our world without proof. Those below are specific to data ownership; [Book I](../01-foundations/README.md) postulates apply throughout.

---

## Post. VII.1 - One System of Record

Every aggregate ([Def. I.6](../01-foundations/definitions.md#Def.%20I.6%20-%20Aggregate)) has exactly one system of record ([Def. VII.1](definitions.md#Def.%20VII.1%20-%20System%20of%20Record)). There is no aggregate that two services both write authoritatively.

## Post. VII.2 - Private Database

A service's database, including every replica ([Def. VII.5](definitions.md#Def.%20VII.5%20-%20Replica)), backup and snapshot of it, is private to that service. This follows from [Post. I.3](../01-foundations/postulates.md#Post.%20I.3%20-%20Autonomy) (teams interact only through contracts) and is stated here so that it can be cited directly: a database is not a contract.

## Post. VII.3 - Reporting Is Fed, Not Read

Reporting stores ([Def. VII.6](definitions.md#Def.%20VII.6%20-%20Reporting%20Store)) and analytics are fed by integration events or by change data capture from the system of record. They are never fed by a direct read of a service database or its replica.

## Post. VII.4 - Default Stores

The default relational stores are SQL Server and PostgreSQL on Amazon RDS or Amazon Aurora. The default key-value store is Amazon DynamoDB. Any other store is a recorded exception ([Def. I.20](../01-foundations/definitions.md#Def.%20I.20%20-%20Exception)).

> **Open question.** Whether to keep both SQL Server and PostgreSQL as defaults, or to
> name one; and whether DynamoDB is the default for new services or only a sanctioned
> option. To be decided by the principal engineers before this postulate leaves Draft.
