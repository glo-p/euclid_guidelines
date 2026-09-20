# Prop. VIII.6 - Mandatory tags on every resource

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every taggable AWS resource MUST carry the tags `service`, `team`, `environment`, `cost-centre` and `data-classification`, with values drawn from the catalogue's controlled lists. Tag keys and values MUST be enforced by AWS Organizations tag policies and by pipeline policy; a resource without them MUST NOT be created.

## Given

* Def. I.1, Def. I.17, Def. VIII.4, Def. VIII.13
* Post. I.6, Post. VIII.1
* Prop. V.5, Prop. VIII.10
* CN 7, CN 8

## Demonstration

A resource is owned by exactly one service and team (Def. I.1, Def. I.17) and exists in exactly one environment (Def. VIII.4); tags (Def. VIII.13) are the only universal place to record those facts on the resource itself, and by CN 8 a fact not recorded is not architecture. Cost allocation (Prop. VIII.10) and classification (Prop. V.5) need the same carrier. By CN 7 an untagged resource cannot be attributed and therefore cannot be operated. Post. I.6 requires enforcement by machine; the organisation (Post. VIII.1) provides tag policies, and the pipeline provides a check before apply. ∎ Q.E.D.

## Corollaries

* **Cor. VIII.6.1** - `data-classification` on a store is the highest classification of any field it holds (Prop. V.5).
* **Cor. VIII.6.2** - Tags are set once by the Terraform provider `default_tags` block and are never hand-edited.

## Construction

* AWS Organizations tag policy attached at the environment OUs, enforcing keys and case; SCP `deny-create-without-tags` using `aws:RequestTag` and `aws:TagKeys` conditions on `Create*` and `RunInstances` families.
* Terraform: `provider "aws" { default_tags {...} }` populated from the service manifest; a shared module `tags` producing the map.
* Pipeline: OPA/Conftest or Sentinel policy on the Terraform plan JSON rejecting any resource missing a mandatory tag.
* AWS Config rule `required-tags` with the five keys; AWS Resource Explorer view for untagged resources.
* Cost allocation tags activated in the billing console for `service`, `team`, `cost-centre`.

## Conformance

AWS Config managed rule `required-tags`; pipeline policy `mandatory-tags`; tag policy compliance report from AWS Organizations, reviewed weekly.

## Scholium

Five tags is the minimum that lets cost, ownership and classification be answered without opening a repository. Teams may add more; they may not omit these.
