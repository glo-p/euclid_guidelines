# Prop. IX.4 - Producers know their consumers

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every producer (Def. I.3) MUST be able to list the consumers of each Active or Deprecated version of each of its contracts from the consumer registry (Def. IX.8). The registry MUST be populated by machine, at least daily, from the client identities that API Gateway attaches to requests and from the inventory of EventBridge rules and their SQS targets that match each event type. A consumer absent from the registry is not owed a deprecation notice.

## Given

* Def. I.3, Def. I.4, Def. IX.8, Def. IX.9
* Post. I.6, Post. IX.3
* CN 3, CN 7
* Prop. II.12, Prop. III.9, Prop. VI.3

## Demonstration

Deprecation (Prop. IX.3) and sunset (Prop. IX.5) both require the producer to know who depends on a version; without that knowledge the minimum period of Post. IX.2 has no recipients and the zero-traffic test has no denominator. Post. IX.3 says consumers are enumerable through the ingress identity of Prop. II.12 and the topology of Prop. III.9, and CN 7 says that what is not observed is absent, so the enumeration must be built from observed bindings, not from a form that teams fill in. By Post. I.6 a registry maintained by hand decays within a year; therefore it is populated by a scheduled machine process. A consumer that reaches a contract by a route the registry cannot see is relying on something outside the contract (CN 3) and is not conforming. ∎ Q.E.D.

## Corollaries

* **Cor. IX.4.1** - Every synchronous request carries a consumer identity; anonymous access to an internal API is not conforming.
* **Cor. IX.4.2** - The registry is the join key for traffic evidence (Def. IX.9): RED metrics (Prop. VI.3) are labelled by the same consumer identity.

## Construction

* API Gateway access logs to CloudWatch Logs with `$context.identity.apiKeyId`, `$context.authorizer.claims.sub` (or `client_id`), stage and resource path; a CloudWatch Logs Insights query aggregates distinct identities per `/v{N}` prefix.
* Scheduled Lambda `consumer-registry-collector` (EventBridge Scheduler, daily) that runs the query, calls `events:ListRules` and `events:ListTargetsByRule` per bus, resolves SQS queue ARNs to owning accounts and teams via resource tags (`team`, `service`), and upserts rows into the registry.
* Registry store: DynamoDB table `contract-consumers` keyed by `(contract, major)` with a set of `{consumerId, team, kind: api|event, lastSeen}` (store choice is an open question in the README).
* Metric label: `Company.Observability` enriches RED metrics with `consumer_id` taken from the same claim, so Prop. IX.5 can query by consumer.
* Read model: `GET /contracts/{name}/v{major}/consumers` on the platform catalogue API.

## Conformance

AWS Config custom rule asserting API Gateway access logging with the identity fields is enabled on every stage; catalogue lint failing any Active contract with no registry entry older than 24 hours.

## Scholium

The registry records who *is* consuming, not who *claims* they will. A team that intends to consume a contract but has not yet sent a request is not in the registry and will not receive a notice; this is deliberate. Cross-account event targets are an open question.
