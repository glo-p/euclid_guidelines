# Prop. VIII.3 - Compute tier decision table

| | |
|---|---|
| **Level** | SHOULD |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A workload (Def. VIII.5) SHOULD run on the compute tier (Def. VIII.10) the following table names for its kind. Amazon EKS MUST NOT be used without a recorded exception (Def. I.20).

| Workload kind | Default tier | Reason for the default |
|---|---|---|
| Synchronous HTTP API (Book II) | ECS on Fargate | Steady concurrency, warm .NET runtime, predictable latency. |
| Event consumer (Book III) | Lambda, SQS trigger | Bursty, scales to zero, batch semantics match SQS. |
| Command consumer (Prop. III.11) | Lambda, SQS trigger | As above. |
| Scheduled job | Lambda, EventBridge Scheduler | Short, infrequent, no host to keep. |
| Long-running job over 15 minutes | ECS Fargate task or Step Functions | Lambda time limit. |
| Anything requiring Kubernetes semantics | EKS, by exception only | Operational cost of a cluster is paid by the whole company. |

## Given

* Def. I.20, Def. VIII.10
* Post. I.1, Post. I.2, Post. VIII.4
* Prop. II.17, Prop. III.14, Prop. VIII.8

## Demonstration

Post. I.1 prefers managed services, which orders the tiers by how much host management they remove: Lambda, then Fargate, then EKS. Post. VIII.4 provides one pipeline template per tier, so every additional tier a team uses is a cost paid by the central pipeline owners. Each workload kind has a load shape; matching it to the tier whose billing and scaling model fits that shape is the entire content of the table, and the .NET constructions of Prop. II.17 and Prop. III.14 already target those tiers on Post. I.2's runtime. EKS removes the least management and its cost is shared, so it is reserved for a recorded exception. ∎ Q.E.D.

## Corollaries

* **Cor. VIII.3.1** - A service MAY run its API on Fargate and its consumers on Lambda; the compute tier is per workload, not per service.
* **Cor. VIII.3.2** - A deviation from a SHOULD row is recorded in the service repository per Prop. I.2; a use of EKS requires an ADR.

## Construction

* Fargate: Terraform module `fargate-service` (ECS cluster, service, task definition, ALB target group behind API Gateway VPC link, autoscaling on RED metrics of Prop. VI.3), image in ECR, .NET `Microsoft.NET.Sdk.Web` chiselled or `runtime-deps` base image.
* Lambda: Terraform module `lambda-sqs-consumer` (function, event source mapping with batch size and partial batch response, DLQ, alarms), .NET `Amazon.Lambda.AspNetCoreServer` not used for consumers; `Amazon.Lambda.SQSEvents` with `Amazon.Lambda.RuntimeSupport` and ReadyToRun or Native AOT where the SDK permits.
* Scheduled: EventBridge Scheduler targeting Lambda, with a `schedule` module.
* Long-running: `fargate-task` module invoked by Step Functions `ecs:runTask.sync`.

## Conformance

Pipeline policy check that a workload's Terraform uses only the sanctioned modules for its declared kind (from the service manifest `workload.kind`); SCP denying `eks:CreateCluster` outside accounts listed in an exception ADR.

## Scholium

The table is short by design. A long decision tree is a sign that the tiers overlap; the right fix is to remove a tier, not to add a branch.
