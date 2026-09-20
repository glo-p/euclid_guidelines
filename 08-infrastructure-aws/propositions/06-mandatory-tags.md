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

* [Def. I.1](../../01-foundations/definitions.md#Def.%20I.1%20-%20Service), [Def. I.17](../../01-foundations/definitions.md#Def.%20I.17%20-%20Team), [Def. VIII.4](../definitions.md#Def.%20VIII.4%20-%20Environment%20%28refines%20Def.%20I.21%29), [Def. VIII.13](../definitions.md#Def.%20VIII.13%20-%20Tag)
* [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification), [Post. VIII.1](../postulates.md#Post.%20VIII.1%20-%20Organisation%20and%20Landing%20Zone)
* [Prop. V.5](../../05-security/propositions/05-data-classification-in-contracts.md), [Prop. VIII.10](10-cost-allocation.md)
* [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated), [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts)

## Demonstration

A resource is owned by exactly one service and team ([Def. I.1](../../01-foundations/definitions.md#Def.%20I.1%20-%20Service), [Def. I.17](../../01-foundations/definitions.md#Def.%20I.17%20-%20Team)) and exists in exactly one environment ([Def. VIII.4](../definitions.md#Def.%20VIII.4%20-%20Environment%20%28refines%20Def.%20I.21%29)); tags ([Def. VIII.13](../definitions.md#Def.%20VIII.13%20-%20Tag)) are the only universal place to record those facts on the resource itself, and by [CN 8](../../01-foundations/common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts) a fact not recorded is not architecture. Cost allocation ([Prop. VIII.10](10-cost-allocation.md)) and classification ([Prop. V.5](../../05-security/propositions/05-data-classification-in-contracts.md)) need the same carrier. By [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated) an untagged resource cannot be attributed and therefore cannot be operated. [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification) requires enforcement by machine; the organisation ([Post. VIII.1](../postulates.md#Post.%20VIII.1%20-%20Organisation%20and%20Landing%20Zone)) provides tag policies, and the pipeline provides a check before apply. ∎ Q.E.D.

## Corollaries

* **Cor. VIII.6.1** - `data-classification` on a store is the highest classification of any field it holds ([Prop. V.5](../../05-security/propositions/05-data-classification-in-contracts.md)).
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
