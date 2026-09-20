# Prop. III.9 - One bus per environment; consumers own their queues

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Each environment MUST have exactly one custom EventBridge bus, `company-<env>`, in the central bus account (Prop. VIII.2). Publishers MUST put events only on that bus, from their own account, via a cross-account resource policy that permits `PutEvents` with a `source` constrained to the publisher's registered sources. Subscribers MUST create their rules on that bus (or on a local bus fed by a forwarding rule) targeting a queue in their own account. A publisher MUST NOT target a subscriber's queue directly, and a subscriber MUST NOT read from a queue it does not own. Rules MUST match on `detail-type` (and optionally `source`), never on payload content alone.

## Given

Def. III.6 to III.10, Post. I.3, Post. III.1, Post. III.2, CN 3, CN 8, Prop. III.1, Prop. VIII.2.

## Demonstration

Teams interact only through contracts (Post. I.3); an event type is the contract (Def. III.5). If a publisher wrote to a subscriber's queue it would know its subscribers and be coupled to their deployment, which is the coupling Post. I.3 forbids. A single bus per environment is the one place where "who publishes what" and "who listens to what" are both visible, which CN 8 says is the architecture itself. Matching on `detail-type` uses the envelope attribute the contract guarantees (CN 3, Prop. III.1) rather than payload internals that may change compatibly. Ownership of the queue by the subscriber places the buffer, its DLQ and its alarms with the team that is paged for them. ∎ Q.E.D.

## Corollaries

* **Cor. III.9.1** - The bus resource policy is the publisher registry: a service may publish a `source` only if the policy lists it, and the policy is IaC (Prop. VIII.1).
* **Cor. III.9.2** - The set of rules on the bus is the consumer registry (Prop. IX.4); it is exported nightly to the catalogue.
* **Cor. III.9.3** - Non-AWS targets (partners, SaaS) are reached through an EventBridge API destination owned by an integration service, never by giving a third party a queue.
* **Cor. III.9.4** - Cross-region or cross-environment delivery does not exist; environments are isolated (Def. I.21).

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
