# Book VIII - Postulates

Postulates are things we accept as true about our world without proof. Those below are specific to infrastructure; Post. I.1 (all workloads in AWS, managed services preferred) and Post. I.8 (stable names) apply throughout and are cited directly by propositions.

---

**Post. VIII.1 - Organisation and Landing Zone.** All accounts belong to one AWS Organization, provisioned and governed through a landing zone (Def. VIII.2) built with AWS Control Tower. No account exists outside it.

**Post. VIII.2 - Account per Team per Environment.** Each team (Def. I.17) has one AWS account (Def. VIII.1) per environment (Def. VIII.4). All of a team's workloads (Def. VIII.5) in that environment live in that account.

> **Open question.** Whether large teams may, or must, split to one account per workload
> per environment, and what threshold (service count, blast radius, quota pressure)
> triggers the split.

**Post. VIII.3 - Terraform.** Infrastructure as code (Def. VIII.6) is written in Terraform (HCL). Modules (Def. VIII.7) are Terraform modules.

> **Open question.** Whether AWS CDK (in C#) is permitted alongside Terraform for
> service-local resources, given Post. I.2, and if so where the boundary lies.

**Post. VIII.4 - One Pipeline System.** All teams use one CI/CD system, with one pipeline template per compute tier, operated centrally. A team may extend a template; it may not replace it.
