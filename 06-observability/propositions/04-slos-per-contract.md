# Prop. VI.4 - Every service declares SLOs for each published contract

| | |
|---|---|
| **Level** | SHOULD |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every service SHOULD declare, in its repository as machine-readable configuration, at least an availability SLO and a latency SLO ([Def. VI.8](../definitions.md#Def.%20VI.8%20-%20SLO)) for each published API contract, and a freshness SLO for each published event type, each with a window and an error budget policy ([Def. VI.9](../definitions.md#Def.%20VI.9%20-%20Error%20Budget)). SLOs SHOULD be computed from the RED metrics of [Prop. VI.3](03-red-metrics.md).

## Given

[Def. VI.7](../definitions.md#Def.%20VI.7%20-%20SLI), [Def. VI.8](../definitions.md#Def.%20VI.8%20-%20SLO), [Def. VI.9](../definitions.md#Def.%20VI.9%20-%20Error%20Budget), [Def. I.2](../../01-foundations/definitions.md#Def.%20I.2%20-%20Contract), [CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed), [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [Post. VI.3](../postulates.md#Post.%20VI.3%20-%20Owned%20On-call), [Prop. VI.3](03-red-metrics.md).

## Demonstration

A published contract is owed ([CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed)), but the contract describes shape, not reliability; by [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit) a consumer may not rely on what is not stated, so a producer that states no SLO has promised nothing and a consumer can plan for nothing. An SLO makes the owed reliability explicit and the error budget makes the trade-off between change and stability a number the owning team ([Post. VI.3](../postulates.md#Post.%20VI.3%20-%20Owned%20On-call)) controls. Computing SLOs from the standard metrics ([Prop. VI.3](03-red-metrics.md)) means every service's objective is comparable and needs no bespoke instrumentation. ∎ Q.E.D.

## Corollaries

* **Cor. VI.4.1** - An exhausted error budget pauses feature deployment for that contract until the budget recovers or an exception is recorded; the policy is in the same file as the SLO.
* **Cor. VI.4.2** - SLOs are referenced from the OpenAPI document's `info` or a linked `x-slo` extension so that consumers find them with the contract.

## Construction

* `slo.yaml` in the service repository following the company schema; Terraform `datadog_service_level_objective` resources generated from it.
* Datadog SLOs of type `metric` over `http.server.request.duration` and consumer histograms; burn-rate monitors at 2 % and 5 % per hour feeding [Prop. VI.6](06-alerts-on-symptoms-with-runbooks.md).
* Service catalogue entry (Datadog Service Catalog) linking SLOs, team and runbooks.

## Conformance

CI policy: repository of a service with a published contract contains a valid `slo.yaml`; Terraform plan diff check that every declared SLO exists in Datadog.

## Scholium

SHOULD rather than MUST because a new service has no traffic to measure; the recording requirement of [Prop. I.2](../../01-foundations/method.md#Prop.%20I.2%20-%20Levels%20and%20their%20obligations) forces the team to say when it will declare them.
