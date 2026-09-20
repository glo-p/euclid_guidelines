# Book IX - Postulates

Postulates of [Book IX](README.md) are facts we accept about how versions live in **our** estate. They could be otherwise elsewhere. If one changes, re-examine every proposition that cites it.

---

## Post. IX.1 - Two Majors at Most

For any one contract, at most two major versions ([Def. IX.2](definitions.md#Def.%20IX.2%20-%20Major%20Version)) are in the Active or Deprecated stage at the same time. A third major cannot become Active until the oldest has reached Sunset. *(Rationale: each live major is a surface that is owed, tested, monitored and secured; the cost is linear in the number of live majors and the benefit of a third is nil, since the deprecation period of the second is enough for any consumer to move.)*

## Post. IX.2 - Minimum Deprecation Period

The interval between the date of deprecation and the sunset date ([Def. IX.6](definitions.md#Def.%20IX.6%20-%20Sunset%20Date)) is at least **6 months** where every registered consumer ([Def. IX.8](definitions.md#Def.%20IX.8%20-%20Consumer%20Registry)) is internal, and at least **12 months** where any registered consumer is external to the company. *(The numbers are an open question recorded in the [Book IX](README.md) README; the existence of a minimum is not.)*

## Post. IX.3 - Consumers Are Enumerable

The consumers of any contract version can be listed by machine. Synchronous consumers are identified by the client identity that API Gateway attaches to every request (the API key or the access token subject, [Book V](../05-security/README.md)). Asynchronous consumers are identified by the EventBridge rules and SQS queue targets that match the event type. A consumer that reaches a contract by any other route is outside the contract and is not owed ([CN 3](../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit)). *(This is [Post. I.6](../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification) applied to consumption: a consumer that is not enumerated by machine will be forgotten within a year.)*
