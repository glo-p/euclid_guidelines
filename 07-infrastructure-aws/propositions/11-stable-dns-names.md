# Prop. VIII.11 - Stable, environment-scoped DNS names for every service

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every service MUST be reachable in every environment by a DNS name of the form `<service>.<env>.<internal-domain>` for private traffic and by its path prefix under `api.<env>.<company-domain>` (Prop. VIII.5) for API traffic. These names MUST NOT change when the service is redeployed, rescaled or moved between accounts or compute tiers. Names MUST be served from Amazon Route 53 private hosted zones shared across the environment's accounts.

## Given

* Def. I.1, Def. VIII.4, Def. VIII.5
* Post. I.8, Post. VIII.2
* Prop. VIII.2, Prop. VIII.5, Prop. VIII.7
* CN 1, CN 3

## Demonstration

Post. I.8 states that a stable, environment-scoped name is possible; this proposition fixes its form. By CN 1 a consumer must not be able to distinguish two conforming implementations, so the name cannot encode the account, the compute tier or the deployment; only the service and the environment (Def. VIII.4) may appear. Accounts are isolated (Post. VIII.2), so the zone must be shared to every account in the environment OU for the name to resolve everywhere. By CN 3 a name that is not declared is not relied upon, so the name is declared in the service manifest and created by the pipeline (Prop. VIII.7), never by hand. ∎ Q.E.D.

## Corollaries

* **Cor. VIII.11.1** - Configuration refers to other services only by these names; account ids, ALB hostnames and function ARNs do not appear in configuration.
* **Cor. VIII.11.2** - Moving a service between compute tiers (Prop. VIII.3) is invisible to its consumers.

## Construction

* Route 53 private hosted zone `<env>.<internal-domain>` in the shared-services account, associated with every VPC in the environment OU (cross-account VPC association via `AssociateVPCWithHostedZone` authorisations, or Route 53 Profiles).
* Terraform module `private-dns-name` creating an alias record to the service's internal ALB, or a CNAME to the API Gateway private endpoint, from the service manifest; records are owned by the service's Terraform state, not by the zone's.
* Public: `api.<env>.<company-domain>` in a public hosted zone (Prop. VIII.5), ACM certificates validated by DNS.
* Route 53 Resolver inbound and outbound endpoints in the shared-services account for on-premises or VPN resolution.
* Service discovery for Fargate-to-Fargate calls via AWS Cloud Map namespaces mapped into the same private zone where used.

## Conformance

Pipeline policy that the service manifest declares `dns.name` and that a Route 53 record for it exists after apply; AWS Config custom rule that every VPC in an environment OU is associated with the environment's private zone; architecture test that no configuration value matches an account id, `*.elb.amazonaws.com` or `arn:aws:lambda` pattern.

## Scholium

A stable name is the cheapest abstraction in the platform. Everything that hides behind it can be replaced without a single consumer noticing.
