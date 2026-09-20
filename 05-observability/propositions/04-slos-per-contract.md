# Prop. VI.4 - Every service declares SLOs for each published contract

| | |
|---|---|
| **Level** | SHOULD |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every service SHOULD declare, in its repository as machine-readable configuration, at least an availability SLO and a latency SLO (Def. VI.8) for each published API contract, and a freshness SLO for each published event type, each with a window and an error budget policy (Def. VI.9). SLOs SHOULD be computed from the RED metrics of Prop. VI.3.

## Given

Def. VI.7, Def. VI.8, Def. VI.9, Def. I.2, CN 2, CN 3, Post. VI.3, Prop. VI.3.

## Demonstration

A published contract is owed (CN 2), but the contract describes shape, not reliability; by CN 3 a consumer may not rely on what is not stated, so a producer that states no SLO has promised nothing and a consumer can plan for nothing. An SLO makes the owed reliability explicit and the error budget makes the trade-off between change and stability a number the owning team (Post. VI.3) controls. Computing SLOs from the standard metrics (Prop. VI.3) means every service's objective is comparable and needs no bespoke instrumentation. ∎ Q.E.D.

## Corollaries

* **Cor. VI.4.1** - An exhausted error budget pauses feature deployment for that contract until the budget recovers or an exception is recorded; the policy is in the same file as the SLO.
* **Cor. VI.4.2** - SLOs are referenced from the OpenAPI document's `info` or a linked `x-slo` extension so that consumers find them with the contract.

## Construction

* `slo.yaml` in the service repository following the company schema; Terraform `datadog_service_level_objective` resources generated from it.
* Datadog SLOs of type `metric` over `http.server.request.duration` and consumer histograms; burn-rate monitors at 2 % and 5 % per hour feeding Prop. VI.6.
* Service catalogue entry (Datadog Service Catalog) linking SLOs, team and runbooks.

## Conformance

CI policy: repository of a service with a published contract contains a valid `slo.yaml`; Terraform plan diff check that every declared SLO exists in Datadog.

## Scholium

SHOULD rather than MUST because a new service has no traffic to measure; the recording requirement of Prop. I.2 forces the team to say when it will declare them.
