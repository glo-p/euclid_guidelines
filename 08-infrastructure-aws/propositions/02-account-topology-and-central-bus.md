# Prop. VIII.2 - Account topology and the central event bus

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Each environment ([Def. VIII.4](../definitions.md#Def.%20VIII.4%20-%20Environment%20%28refines%20Def.%20I.21%29)) MUST be one OU ([Def. VIII.3](../definitions.md#Def.%20VIII.3%20-%20Organisational%20Unit)) containing one account per team ([Post. VIII.2](../postulates.md#Post.%20VIII.2%20-%20Account%20per%20Team%20per%20Environment)), one central bus account and one shared-services account. Every integration event MUST be published to the central EventBridge bus of its environment and every consumer queue MUST be fed from that bus by a rule targeting the consumer's own account ([Prop. III.9](../../03-events/propositions/09-topology.md)). Team-to-team EventBridge or SQS access that bypasses the central bus is prohibited.

## Given

* [Def. I.17](../../01-foundations/definitions.md#Def.%20I.17%20-%20Team), [Def. VIII.1](../definitions.md#Def.%20VIII.1%20-%20AWS%20Account), [Def. VIII.3](../definitions.md#Def.%20VIII.3%20-%20Organisational%20Unit), [Def. VIII.4](../definitions.md#Def.%20VIII.4%20-%20Environment%20%28refines%20Def.%20I.21%29)
* [Post. I.3](../../01-foundations/postulates.md#Post.%20I.3%20-%20Autonomy), [Post. I.9](../../01-foundations/postulates.md#Post.%20I.9%20-%20Transport), [Post. VIII.1](../postulates.md#Post.%20VIII.1%20-%20Organisation%20and%20Landing%20Zone), [Post. VIII.2](../postulates.md#Post.%20VIII.2%20-%20Account%20per%20Team%20per%20Environment)
* [Prop. III.9](../../03-events/propositions/09-topology.md), [Prop. III.1](../../03-events/propositions/01-envelope.md)
* [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts)

## Demonstration

[Post. VIII.2](../postulates.md#Post.%20VIII.2%20-%20Account%20per%20Team%20per%20Environment) places each team in its own account per environment, and accounts are isolated by default ([Def. VIII.1](../definitions.md#Def.%20VIII.1%20-%20AWS%20Account)), so any cross-team interaction requires an explicit policy on both sides. [Post. I.3](../../01-foundations/postulates.md#Post.%20I.3%20-%20Autonomy) permits interaction only through contracts, and the asynchronous contract is the event on the bus ([Post. I.9](../../01-foundations/postulates.md#Post.%20I.9%20-%20Transport), [Prop. III.1](../../03-events/propositions/01-envelope.md)). A central bus per environment is the one point where a policy can be applied once for all teams; a mesh of bus-to-bus permissions would be N squared policies that no one can read, which by [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts) is architecture that does not exist. [Prop. III.9](../../03-events/propositions/09-topology.md) fixes the topology; this proposition fixes the accounts it runs in. ∎ Q.E.D.

## Corollaries

* **Cor. VIII.2.1** - A team account publishes to the central bus via a cross-account target and receives via a rule in the central bus account whose target is a bus in the team account; the team's SQS queues subscribe to its own bus.
* **Cor. VIII.2.2** - The shared-services account holds the artefact registries, state buckets, private DNS zones and the Terraform module registry, and nothing owned by a team.

## Construction

* AWS Organizations OU tree: `root / environments / {dev,test,prod} / {team-*, bus, shared}` plus `root / sandbox`.
* AWS Control Tower account factory (Account Factory for Terraform) creating team accounts from a template with baseline VPC, Config, CloudTrail and tags.
* Central bus account: one custom EventBridge bus per environment with a resource policy allowing `events:PutEvents` from the environment OU; EventBridge Archive ([Prop. III.13](../../03-events/propositions/13-archive-and-replay.md)) and Schema Registry ([Prop. III.4](../../03-events/propositions/04-schemas-and-registry.md)) in the same account.
* Team account: a local bus receiving forwarded events, rules to SQS queues with DLQs ([Prop. III.8](../../03-events/propositions/08-retries-and-dead-letters.md)); a Terraform module `eventbridge-subscription` that creates both halves.
* Shared-services account: ECR, CodeArtifact (NuGet), S3 for Terraform state and modules, Route 53 private zones ([Prop. VIII.11](11-stable-dns-names.md)).

## Conformance

AWS Config custom rule that every EventBridge rule in a team account targets only that account's own resources or the central bus; SCP denying `events:PutPermission` and `sqs:AddPermission` to non-central accounts for principals outside the same account.

## Scholium

The topology is a star. Stars are easy to draw, audit and replace; meshes are none of those.
