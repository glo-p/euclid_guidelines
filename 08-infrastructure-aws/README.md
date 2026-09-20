# Book VIII - Infrastructure on AWS

Book VIII fixes how the platform is laid out in AWS: accounts, networking, compute tiers, infrastructure as code, pipelines, environments, tags, cost and names. It rests on [Post. I.1](../01-foundations/postulates.md#Post.%20I.1%20-%20Cloud) (all workloads in AWS, managed services preferred) and [Post. I.8](../01-foundations/postulates.md#Post.%20I.8%20-%20Stable%20Names) (stable names), and on the autonomy of [Post. I.3](../01-foundations/postulates.md#Post.%20I.3%20-%20Autonomy) realised as one account per team.

**Depth:** Scaffold. Every proposition is `Draft`.

## Definitions

| Item | Summary |
|---|---|
| [`definitions.md`](definitions.md) | [`Def. VIII.1`](definitions.md#Def.%20VIII.1%20-%20AWS%20Account) - [`Def. VIII.13`](definitions.md#Def.%20VIII.13%20-%20Tag). |
| [Def. VIII.1](definitions.md#Def.%20VIII.1%20-%20AWS%20Account) AWS account | The unit of isolation, billing and IAM scope. |
| [Def. VIII.2](definitions.md#Def.%20VIII.2%20-%20Landing%20Zone) Landing zone | The centrally owned baseline every account is enrolled in. |
| [Def. VIII.3](definitions.md#Def.%20VIII.3%20-%20Organisational%20Unit) Organisational unit | A node in the Organizations tree carrying inherited guardrails. |
| [Def. VIII.4](definitions.md#Def.%20VIII.4%20-%20Environment%20%28refines%20Def.%20I.21%29) Environment | Refines [Def. I.21](../01-foundations/definitions.md#Def.%20I.21%20-%20Environment): one OU with one account per team, a bus account and a shared-services account. |
| [Def. VIII.5](definitions.md#Def.%20VIII.5%20-%20Workload) Workload | The resources that run one service in one environment. |
| [Def. VIII.6](definitions.md#Def.%20VIII.6%20-%20Infrastructure%20as%20Code) Infrastructure as code | Resources exist only because version-controlled source says so. |
| [Def. VIII.7](definitions.md#Def.%20VIII.7%20-%20Module) Module | A versioned, reusable IaC building block with declared inputs and outputs. |
| [Def. VIII.8](definitions.md#Def.%20VIII.8%20-%20Pipeline) Pipeline | The only path from commit to environment. |
| [Def. VIII.9](definitions.md#Def.%20VIII.9%20-%20Artefact) Artefact | The immutable, digest-identified output of one build. |
| [Def. VIII.10](definitions.md#Def.%20VIII.10%20-%20Compute%20Tier) Compute tier | Lambda, ECS Fargate or EKS. |
| [Def. VIII.11](definitions.md#Def.%20VIII.11%20-%20Ingress) Ingress | The single point where outside traffic enters a workload. |
| [Def. VIII.12](definitions.md#Def.%20VIII.12%20-%20VPC%20Endpoint) VPC endpoint | A private path from a VPC to an AWS service. |
| [Def. VIII.13](definitions.md#Def.%20VIII.13%20-%20Tag) Tag | The universal carrier of ownership, environment, cost and classification. |

## Postulates

| Item | Summary |
|---|---|
| [`postulates.md`](postulates.md) | [`Post. VIII.1`](postulates.md#Post.%20VIII.1%20-%20Organisation%20and%20Landing%20Zone) - [`Post. VIII.4`](postulates.md#Post.%20VIII.4%20-%20One%20Pipeline%20System); [Post. I.1](../01-foundations/postulates.md#Post.%20I.1%20-%20Cloud) and [Post. I.8](../01-foundations/postulates.md#Post.%20I.8%20-%20Stable%20Names) apply throughout. |
| [Post. VIII.1](postulates.md#Post.%20VIII.1%20-%20Organisation%20and%20Landing%20Zone) Organisation and landing zone | One AWS Organization, governed through Control Tower. |
| [Post. VIII.2](postulates.md#Post.%20VIII.2%20-%20Account%20per%20Team%20per%20Environment) Account per team per environment | Open question: per workload. |
| [Post. VIII.3](postulates.md#Post.%20VIII.3%20-%20Terraform) Terraform | The IaC tool. Open question: CDK. |
| [Post. VIII.4](postulates.md#Post.%20VIII.4%20-%20One%20Pipeline%20System) One pipeline system | One CI/CD system, one template per compute tier. |

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

1. **[Post. VIII.2](postulates.md#Post.%20VIII.2%20-%20Account%20per%20Team%20per%20Environment), account granularity.** One account per team per environment, or per workload per environment; and the threshold that forces a split.
2. **[Post. VIII.3](postulates.md#Post.%20VIII.3%20-%20Terraform), CDK.** Whether AWS CDK in C# is permitted for service-local resources alongside Terraform, and where the boundary lies.
3. **[Post. VIII.4](postulates.md#Post.%20VIII.4%20-%20One%20Pipeline%20System), pipeline system.** AWS CodePipeline/CodeBuild versus GitHub Actions with OIDC federation; the choice fixes the template format in [Prop. VIII.8](propositions/08-pipelines.md).
4. **[Prop. VIII.5](propositions/05-api-gateway-single-ingress.md), domain layout.** One central `api.<env>.<domain>` with cross-account base path mappings, or one domain per team aliased under it.
5. **[Prop. VIII.5](propositions/05-api-gateway-single-ingress.md), HTTP API versus REST API.** HTTP API is assumed as default; REST API is required for some WAF, usage plan and request validation features.
6. **[Prop. VIII.9](propositions/09-shared-terraform-modules.md), registry.** Terraform Cloud private registry versus a self-hosted S3-backed registry in the shared-services account.
7. **[Prop. VIII.4](propositions/04-private-networking.md), egress.** Central egress via Transit Gateway and Network Firewall versus per-account NAT with endpoint policies only; the former is assumed.
8. **[Prop. VIII.1](propositions/01-everything-is-iac.md), drift response.** Whether drift in `prod` auto-reverts by a pipeline apply or only alerts; one working day is the placeholder.
9. **[Prop. VIII.3](propositions/03-compute-choice.md), Native AOT.** Whether Lambda consumers default to Native AOT on the current LTS, given SDK and reflection constraints.
10. **Region set.** The list of approved regions, shared with [Prop. VII.8](../07-data-ownership/propositions/08-data-residency.md), and whether every environment exists in every region.
