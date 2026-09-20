# Prop. III.9 - One bus per environment; consumers own their queues

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Each environment MUST have exactly one custom EventBridge bus, `company-<env>`, in the central bus account ([Prop. VIII.2](../../08-infrastructure-aws/propositions/02-account-topology-and-central-bus.md)). Publishers MUST put events only on that bus, from their own account, via a cross-account resource policy that permits `PutEvents` with a `source` constrained to the publisher's registered sources. Subscribers MUST create their rules on that bus (or on a local bus fed by a forwarding rule) targeting a queue in their own account. A publisher MUST NOT target a subscriber's queue directly, and a subscriber MUST NOT read from a queue it does not own. Rules MUST match on `detail-type` (and optionally `source`), never on payload content alone.

## Given

[Def. III.6](../definitions.md#Def.%20III.6%20-%20Publisher) to [III.10](../definitions.md#Def.%20III.10%20-%20Queue), [Post. I.3](../../01-foundations/postulates.md#Post.%20I.3%20-%20Autonomy), [Post. III.1](../postulates.md#Post.%20III.1%20-%20The%20Bus), [Post. III.2](../postulates.md#Post.%20III.2%20-%20The%20Buffer), [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts), [Prop. III.1](01-envelope.md), [Prop. VIII.2](../../08-infrastructure-aws/propositions/02-account-topology-and-central-bus.md).

## Demonstration

Teams interact only through contracts ([Post. I.3](../../01-foundations/postulates.md#Post.%20I.3%20-%20Autonomy)); an event type is the contract ([Def. III.5](../definitions.md#Def.%20III.5%20-%20Event%20Type)). If a publisher wrote to a subscriber's queue it would know its subscribers and be coupled to their deployment, which is the coupling [Post. I.3](../../01-foundations/postulates.md#Post.%20I.3%20-%20Autonomy) forbids. A single bus per environment is the one place where "who publishes what" and "who listens to what" are both visible, which [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts) says is the architecture itself. Matching on `detail-type` uses the envelope attribute the contract guarantees ([CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [Prop. III.1](01-envelope.md)) rather than payload internals that may change compatibly. Ownership of the queue by the subscriber places the buffer, its DLQ and its alarms with the team that is paged for them. ∎ Q.E.D.

## Corollaries

* **Cor. III.9.1** - The bus resource policy is the publisher registry: a service may publish a `source` only if the policy lists it, and the policy is IaC ([Prop. VIII.1](../../08-infrastructure-aws/propositions/01-everything-is-iac.md)).
* **Cor. III.9.2** - The set of rules on the bus is the consumer registry ([Prop. IX.4](../../09-versioning-and-deprecation/propositions/04-producers-know-their-consumers.md)); it is exported nightly to the catalogue.
* **Cor. III.9.3** - Non-AWS targets (partners, SaaS) are reached through an EventBridge API destination owned by an integration service, never by giving a third party a queue.
* **Cor. III.9.4** - Cross-region or cross-environment delivery does not exist; environments are isolated ([Def. I.21](../../01-foundations/definitions.md#Def.%20I.21%20-%20Environment)).

## Construction

```
Account: platform-bus-<env>
  EventBridge bus: company-<env>
    resource policy: allow PutEvents from team accounts, condition events:source in [registered]
    archive: company-<env>-archive (Prop. III.13)
    rule: envelope-invalid -> platform DLQ + alarm
Account: team-orders-<env>
  publisher: orders-api -> PutEvents(company-<env>)          (via outbox relay)
Account: team-billing-<env>
  rule on company-<env>: detail-type prefix "com.company.orders.order." -> SQS billing-orders-consumer
  SQS: billing-orders-consumer (+ DLQ)  <- consumed by billing-worker
```

Terraform: `company/eventbridge-publisher` (registers sources, IAM for PutEvents) and `company/sqs-consumer` (rule + queue + DLQ + alarms) modules.

## Conformance

AWS Config custom rule: every EventBridge rule target is an SQS queue in the same account as the rule's owner tag; every `PutEvents` IAM grant is to `company-<env>` only.
