# Book VIII - Infrastructure on AWS

Book VIII fixes how the platform is laid out in AWS: accounts, networking, compute tiers, infrastructure as code, pipelines, environments, tags, cost and names. It rests on Post. I.1 (all workloads in AWS, managed services preferred) and Post. I.8 (stable names), and on the autonomy of Post. I.3 realised as one account per team.

**Depth:** Scaffold. Every proposition is `Draft`.

## Definitions

| Item | Summary |
|---|---|
| [`definitions.md`](definitions.md) | `Def. VIII.1` - `Def. VIII.13`. |
| Def. VIII.1 AWS account | The unit of isolation, billing and IAM scope. |
| Def. VIII.2 Landing zone | The centrally owned baseline every account is enrolled in. |
| Def. VIII.3 Organisational unit | A node in the Organizations tree carrying inherited guardrails. |
| Def. VIII.4 Environment | Refines Def. I.21: one OU with one account per team, a bus account and a shared-services account. |
| Def. VIII.5 Workload | The resources that run one service in one environment. |
| Def. VIII.6 Infrastructure as code | Resources exist only because version-controlled source says so. |
| Def. VIII.7 Module | A versioned, reusable IaC building block with declared inputs and outputs. |
| Def. VIII.8 Pipeline | The only path from commit to environment. |
| Def. VIII.9 Artefact | The immutable, digest-identified output of one build. |
| Def. VIII.10 Compute tier | Lambda, ECS Fargate or EKS. |
| Def. VIII.11 Ingress | The single point where outside traffic enters a workload. |
| Def. VIII.12 VPC endpoint | A private path from a VPC to an AWS service. |
| Def. VIII.13 Tag | The universal carrier of ownership, environment, cost and classification. |

## Postulates

| Item | Summary |
|---|---|
| [`postulates.md`](postulates.md) | `Post. VIII.1` - `Post. VIII.4`; Post. I.1 and Post. I.8 apply throughout. |
| Post. VIII.1 Organisation and landing zone | One AWS Organization, governed through Control Tower. |
| Post. VIII.2 Account per team per environment | Open question: per workload. |
| Post. VIII.3 Terraform | The IaC tool. Open question: CDK. |
| Post. VIII.4 One pipeline system | One CI/CD system, one template per compute tier. |

## Propositions

| Prop. | File | Level | Summary |
|---|---|---|---|
| VIII.1 | [`01-everything-is-iac.md`](propositions/01-everything-is-iac.md) | MUST | Everything is IaC; no console changes outside sandbox; drift detected. |
| VIII.2 | [`02-account-topology-and-central-bus.md`](propositions/02-account-topology-and-central-bus.md) | MUST | One OU per environment; central bus account; teams publish to and subscribe from it. |
| VIII.3 | [`03-compute-choice.md`](propositions/03-compute-choice.md) | SHOULD | Fargate for APIs, Lambda for consumers and jobs, EKS by exception. |
| VIII.4 | [`04-private-networking.md`](propositions/04-private-networking.md) | MUST | Private subnets, VPC endpoints, no public databases, central egress. |
| VIII.5 | [`05-api-gateway-single-ingress.md`](propositions/05-api-gateway-single-ingress.md) | MUST | API Gateway is the only ingress; one custom domain per environment. |
| VIII.6 | [`06-mandatory-tags.md`](propositions/06-mandatory-tags.md) | MUST | Five mandatory tags, enforced by tag policies and pipeline. |
| VIII.7 | [`07-identical-environments.md`](propositions/07-identical-environments.md) | MUST | Same source, same artefact by digest; only configuration differs. |
| VIII.8 | [`08-pipelines.md`](propositions/08-pipelines.md) | MUST | Trunk-based; build once; contract tests and scanning gate promotion. |
| VIII.9 | [`09-shared-terraform-modules.md`](propositions/09-shared-terraform-modules.md) | MUST | Shared modules in a private registry, semantically versioned, pinned. |
| VIII.10 | [`10-cost-allocation.md`](propositions/10-cost-allocation.md) | MUST | Cost per service from tags; budgets and alerts per account and service. |
| VIII.11 | [`11-stable-dns-names.md`](propositions/11-stable-dns-names.md) | MUST | `<service>.<env>.<internal-domain>` in Route 53 private zones; never changes. |

## Open questions

Decisions the principal engineers still need to make before items leave `Draft`:

1. **Post. VIII.2, account granularity.** One account per team per environment, or per workload per environment; and the threshold that forces a split.
2. **Post. VIII.3, CDK.** Whether AWS CDK in C# is permitted for service-local resources alongside Terraform, and where the boundary lies.
3. **Post. VIII.4, pipeline system.** AWS CodePipeline/CodeBuild versus GitHub Actions with OIDC federation; the choice fixes the template format in Prop. VIII.8.
4. **Prop. VIII.5, domain layout.** One central `api.<env>.<domain>` with cross-account base path mappings, or one domain per team aliased under it.
5. **Prop. VIII.5, HTTP API versus REST API.** HTTP API is assumed as default; REST API is required for some WAF, usage plan and request validation features.
6. **Prop. VIII.9, registry.** Terraform Cloud private registry versus a self-hosted S3-backed registry in the shared-services account.
7. **Prop. VIII.4, egress.** Central egress via Transit Gateway and Network Firewall versus per-account NAT with endpoint policies only; the former is assumed.
8. **Prop. VIII.1, drift response.** Whether drift in `prod` auto-reverts by a pipeline apply or only alerts; one working day is the placeholder.
9. **Prop. VIII.3, Native AOT.** Whether Lambda consumers default to Native AOT on the current LTS, given SDK and reflection constraints.
10. **Region set.** The list of approved regions, shared with Prop. VII.8, and whether every environment exists in every region.
