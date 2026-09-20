# Prop. VIII.2 - Account topology and the central event bus

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Each environment (Def. VIII.4) MUST be one OU (Def. VIII.3) containing one account per team (Post. VIII.2), one central bus account and one shared-services account. Every integration event MUST be published to the central EventBridge bus of its environment and every consumer queue MUST be fed from that bus by a rule targeting the consumer's own account (Prop. III.9). Team-to-team EventBridge or SQS access that bypasses the central bus is prohibited.

## Given

* Def. I.17, Def. VIII.1, Def. VIII.3, Def. VIII.4
* Post. I.3, Post. I.9, Post. VIII.1, Post. VIII.2
* Prop. III.9, Prop. III.1
* CN 8

## Demonstration

Post. VIII.2 places each team in its own account per environment, and accounts are isolated by default (Def. VIII.1), so any cross-team interaction requires an explicit policy on both sides. Post. I.3 permits interaction only through contracts, and the asynchronous contract is the event on the bus (Post. I.9, Prop. III.1). A central bus per environment is the one point where a policy can be applied once for all teams; a mesh of bus-to-bus permissions would be N squared policies that no one can read, which by CN 8 is architecture that does not exist. Prop. III.9 fixes the topology; this proposition fixes the accounts it runs in. ∎ Q.E.D.

## Corollaries

* **Cor. VIII.2.1** - A team account publishes to the central bus via a cross-account target and receives via a rule in the central bus account whose target is a bus in the team account; the team's SQS queues subscribe to its own bus.
* **Cor. VIII.2.2** - The shared-services account holds the artefact registries, state buckets, private DNS zones and the Terraform module registry, and nothing owned by a team.

## Construction

* AWS Organizations OU tree: `root / environments / {dev,test,prod} / {team-*, bus, shared}` plus `root / sandbox`.
* AWS Control Tower account factory (Account Factory for Terraform) creating team accounts from a template with baseline VPC, Config, CloudTrail and tags.
* Central bus account: one custom EventBridge bus per environment with a resource policy allowing `events:PutEvents` from the environment OU; EventBridge Archive (Prop. III.13) and Schema Registry (Prop. III.4) in the same account.
* Team account: a local bus receiving forwarded events, rules to SQS queues with DLQs (Prop. III.8); a Terraform module `eventbridge-subscription` that creates both halves.
* Shared-services account: ECR, CodeArtifact (NuGet), S3 for Terraform state and modules, Route 53 private zones (Prop. VIII.11).

## Conformance

AWS Config custom rule that every EventBridge rule in a team account targets only that account's own resources or the central bus; SCP denying `events:PutPermission` and `sqs:AddPermission` to non-central accounts for principals outside the same account.

## Scholium

The topology is a star. Stars are easy to draw, audit and replace; meshes are none of those.
