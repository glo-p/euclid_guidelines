# Prop. IX.4 - Producers know their consumers

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every producer ([Def. I.3](../../01-foundations/definitions.md#Def.%20I.3%20-%20Producer)) MUST be able to list the consumers of each Active or Deprecated version of each of its contracts from the consumer registry ([Def. IX.8](../definitions.md#Def.%20IX.8%20-%20Consumer%20Registry)). The registry MUST be populated by machine, at least daily, from the client identities that API Gateway attaches to requests and from the inventory of EventBridge rules and their SQS targets that match each event type. A consumer absent from the registry is not owed a deprecation notice.

## Given

* [Def. I.3](../../01-foundations/definitions.md#Def.%20I.3%20-%20Producer), [Def. I.4](../../01-foundations/definitions.md#Def.%20I.4%20-%20Consumer), [Def. IX.8](../definitions.md#Def.%20IX.8%20-%20Consumer%20Registry), [Def. IX.9](../definitions.md#Def.%20IX.9%20-%20Traffic%20Evidence)
* [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification), [Post. IX.3](../postulates.md#Post.%20IX.3%20-%20Consumers%20Are%20Enumerable)
* [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated)
* [Prop. II.12](../../02-api-guidelines/propositions/12-authentication-and-authorisation.md), [Prop. III.9](../../03-events/propositions/09-topology.md), [Prop. VI.3](../../06-observability/propositions/03-red-metrics.md)

## Demonstration

Deprecation ([Prop. IX.3](03-deprecation-is-declared-in-the-contract.md)) and sunset ([Prop. IX.5](05-sunset-on-evidence-retirement-removes.md)) both require the producer to know who depends on a version; without that knowledge the minimum period of [Post. IX.2](../postulates.md#Post.%20IX.2%20-%20Minimum%20Deprecation%20Period) has no recipients and the zero-traffic test has no denominator. [Post. IX.3](../postulates.md#Post.%20IX.3%20-%20Consumers%20Are%20Enumerable) says consumers are enumerable through the ingress identity of [Prop. II.12](../../02-api-guidelines/propositions/12-authentication-and-authorisation.md) and the topology of [Prop. III.9](../../03-events/propositions/09-topology.md), and [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated) says that what is not observed is absent, so the enumeration must be built from observed bindings, not from a form that teams fill in. By [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification) a registry maintained by hand decays within a year; therefore it is populated by a scheduled machine process. A consumer that reaches a contract by a route the registry cannot see is relying on something outside the contract ([CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit)) and is not conforming. ∎ Q.E.D.

## Corollaries

* **Cor. IX.4.1** - Every synchronous request carries a consumer identity; anonymous access to an internal API is not conforming.
* **Cor. IX.4.2** - The registry is the join key for traffic evidence ([Def. IX.9](../definitions.md#Def.%20IX.9%20-%20Traffic%20Evidence)): RED metrics ([Prop. VI.3](../../06-observability/propositions/03-red-metrics.md)) are labelled by the same consumer identity.

## Construction

* API Gateway access logs to CloudWatch Logs with `$context.identity.apiKeyId`, `$context.authorizer.claims.sub` (or `client_id`), stage and resource path; a CloudWatch Logs Insights query aggregates distinct identities per `/v{N}` prefix.
* Scheduled Lambda `consumer-registry-collector` (EventBridge Scheduler, daily) that runs the query, calls `events:ListRules` and `events:ListTargetsByRule` per bus, resolves SQS queue ARNs to owning accounts and teams via resource tags (`team`, `service`), and upserts rows into the registry.
* Registry store: DynamoDB table `contract-consumers` keyed by `(contract, major)` with a set of `{consumerId, team, kind: api|event, lastSeen}` (store choice is an open question in the README).
* Metric label: `Company.Observability` enriches RED metrics with `consumer_id` taken from the same claim, so [Prop. IX.5](05-sunset-on-evidence-retirement-removes.md) can query by consumer.
* Read model: `GET /contracts/{name}/v{major}/consumers` on the platform catalogue API.

## Conformance

AWS Config custom rule asserting API Gateway access logging with the identity fields is enabled on every stage; catalogue lint failing any Active contract with no registry entry older than 24 hours.

## Scholium

The registry records who *is* consuming, not who *claims* they will. A team that intends to consume a contract but has not yet sent a request is not in the registry and will not receive a notice; this is deliberate. Cross-account event targets are an open question.
