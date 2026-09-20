# Prop. VIII.4 - Private subnets, VPC endpoints, no public databases

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Compute and data resources MUST be placed in private subnets with no route to an internet gateway. Access to AWS services from those subnets MUST use VPC endpoints (Def. VIII.12). No database, cache or queue MAY be reachable from the public internet. Outbound internet access, where required, MUST pass through a centrally managed egress with logging.

## Given

* Def. VIII.11, Def. VIII.12
* Post. I.1, Post. I.6
* Prop. V.3, Prop. VII.1, Prop. VIII.5
* CN 7

## Demonstration

Ingress is defined as a single point (Def. VIII.11, Prop. VIII.5); a compute or data resource with a public address is a second ingress that no proposition governs. Prop. VII.1 requires a database to be reachable by exactly one service, which is impossible if it is reachable by the internet. AWS services can be reached privately (Def. VIII.12, Post. I.1), so the public route is not needed for them, and removing it means that service-to-service credentials (Prop. V.3) cannot be replayed from outside. Post. I.6 requires the placement to be checked by policy, and CN 7 requires that whatever egress remains is logged. ∎ Q.E.D.

## Corollaries

* **Cor. VIII.4.1** - A Lambda function that touches a database or a VPC endpoint is VPC-attached; one that touches only AWS APIs over the public endpoint set MAY be unattached, but SHOULD be attached for uniformity.
* **Cor. VIII.4.2** - Developers reach private resources through AWS Systems Manager Session Manager or a client VPN, never through a bastion with a public IP.

## Construction

* Baseline VPC per account from the landing zone: three private subnets (compute), three isolated subnets (data), no public subnets by default; IPAM-allocated CIDRs.
* Gateway endpoints for S3 and DynamoDB; interface endpoints (PrivateLink) for EventBridge, SQS, Secrets Manager, KMS, ECR (api and dkr), CloudWatch Logs, STS, SSM; endpoint policies restricting to the organisation.
* Central egress VPC in the shared-services account via AWS Transit Gateway, with AWS Network Firewall and flow logs to the log archive account.
* Security groups: database SG admits only the service compute SG (Prop. VII.1); NACLs left permissive, SGs carry the policy.
* RDS `PubliclyAccessible = false`; ElastiCache and SQS reached only via endpoints.

## Conformance

AWS Config managed rules `rds-instance-public-access-check`, `lambda-inside-vpc`, `subnet-auto-assign-public-ip-disabled`, `vpc-default-security-group-closed`, `ec2-instance-no-public-ip`; a custom rule that every VPC in an environment OU has the mandatory endpoint set.

## Scholium

Private networking is cheaper to get right at account creation than afterwards. The landing zone baseline exists so that no team has to decide this.
