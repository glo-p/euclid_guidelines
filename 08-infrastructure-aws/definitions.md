# Book VIII - Definitions

These definitions fix the vocabulary of infrastructure on AWS. They are not rules. Where [Book I](../01-foundations/README.md) already fixes a term it is refined here and the refinement is marked.

---

## Def. VIII.1 - AWS Account

An *AWS account* is the unit of isolation, billing and IAM scope in AWS. Resources in different accounts cannot reach one another unless a policy on each side permits it.

## Def. VIII.2 - Landing Zone

The *landing zone* is the pre-configured, centrally owned set of accounts, guardrails, logging and networking into which every workload account is enrolled. It is the platform's foundation, not a team's.

## Def. VIII.3 - Organisational Unit

An *organisational unit* (OU) is a node in the AWS Organizations tree to which accounts are attached and to which service control policies and tag policies are applied. Guardrails are inherited down the tree.

## Def. VIII.4 - Environment (refines Def. I.21)

An *environment* is a complete, isolated deployment of the platform, realised as one OU containing one account per team, one central bus account and one shared-services account, all in the same region set. Contracts are identical across environments; only configuration differs.

## Def. VIII.5 - Workload

A *workload* is the set of AWS resources that together run one service ([Def. I.1](../01-foundations/definitions.md#Def.%20I.1%20-%20Service)) in one environment: compute, store, queues, roles, alarms. A workload is owned by the service's team and lives in that team's account.

## Def. VIII.6 - Infrastructure as Code

*Infrastructure as code* (IaC) is the practice of declaring every resource in version-controlled source that is applied by a tool, so that the source is the only cause of the resource's existence and configuration.

## Def. VIII.7 - Module

A *module* is a versioned, reusable unit of IaC that encapsulates one building block (a queue with its DLQ and alarms, a Fargate service with its role and log group) behind declared inputs and outputs.

## Def. VIII.8 - Pipeline

A *pipeline* is the automated sequence that takes a commit through build, test, packaging, and deployment to each environment in order. It is the only path by which code or infrastructure reaches an environment above sandbox.

## Def. VIII.9 - Artefact

An *artefact* is the immutable, uniquely identified output of a build (a container image digest, a Lambda zip hash, a Terraform module version, a NuGet package version). The same artefact is deployed to every environment.

## Def. VIII.10 - Compute Tier

A *compute tier* is one of the sanctioned ways to run code: **AWS Lambda** (functions, event-driven, per-invocation billing), **Amazon ECS on AWS Fargate** (long-running containers without host management), or **Amazon EKS** (Kubernetes, host or Fargate backed). Each has a default use fixed by [Prop. VIII.3](propositions/03-compute-choice.md).

## Def. VIII.11 - Ingress

*Ingress* is the point at which traffic from outside a workload's private network enters it. For APIs the ingress is Amazon API Gateway ([Prop. VIII.5](propositions/05-api-gateway-single-ingress.md)); for events it is the EventBridge bus ([Prop. VIII.2](propositions/02-account-topology-and-central-bus.md)).

## Def. VIII.12 - VPC Endpoint

A *VPC endpoint* is a private path from a VPC to an AWS service (gateway endpoints for S3 and DynamoDB, interface endpoints via AWS PrivateLink for the rest) that does not traverse the public internet.

## Def. VIII.13 - Tag

A *tag* is a key-value label attached to an AWS resource. Tags are the only cross-cutting metadata AWS offers on every resource type and are therefore the carrier of ownership, environment, cost and classification ([Prop. VIII.6](propositions/06-mandatory-tags.md)).
