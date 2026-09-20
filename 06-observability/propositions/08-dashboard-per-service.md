# Prop. VI.8 - One dashboard per service, generated from a standard template

| | |
|---|---|
| **Level** | SHOULD |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every service SHOULD have exactly one primary dashboard ([Def. VI.15](../definitions.md#Def.%20VI.15%20-%20Dashboard)), generated from the company template from the service's contracts and `slo.yaml`, showing SLO status and error budget, RED metrics per operation and per consumer, consumer lag and DLQ depth, readiness, recent deployments and active alerts. Teams MAY add further dashboards; the primary one is not hand-edited.

## Given

[Def. VI.15](../definitions.md#Def.%20VI.15%20-%20Dashboard), [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape), [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated), [Post. VI.2](../postulates.md#Post.%20VI.2%20-%20Backend), [Post. VI.3](../postulates.md#Post.%20VI.3%20-%20Owned%20On-call), [Prop. VI.3](03-red-metrics.md), [Prop. VI.4](04-slos-per-contract.md), [Prop. VI.5](05-health-and-readiness-endpoints.md).

## Demonstration

The standard metrics ([Prop. VI.3](03-red-metrics.md)), SLOs ([Prop. VI.4](04-slos-per-contract.md)) and health shape ([Prop. VI.5](05-health-and-readiness-endpoints.md)) mean every service exposes the same signals, and by [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape) one layout for one set of signals is enough; a second layout is a cost without benefit. A generated dashboard exists from the first deployment, before the team has had an incident to motivate building one, which by [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated) is the moment it is needed. A responder from another team ([Post. VI.3](../postulates.md#Post.%20VI.3%20-%20Owned%20On-call) puts the page with the owner, but incidents cross teams) can read any service's dashboard because they all read the same way. ∎ Q.E.D.

## Corollaries

* **Cor. VI.8.1** - The dashboard URL is recorded in the service catalogue entry and in every runbook header ([Prop. VI.6](06-alerts-on-symptoms-with-runbooks.md)).
* **Cor. VI.8.2** - A template change regenerates every service's dashboard; teams do not fork the template.

## Construction

* Terraform module `company-service-dashboard` producing `datadog_dashboard_json` from the OpenAPI document (operation list), event subscriptions and `slo.yaml`.
* Template panels: SLO summary widgets, `http.server.request.duration` by `operationId`, `messaging.process.duration` by event type, SQS age and DLQ depth ([Prop. III.8](../../03-events/propositions/08-retries-and-dead-letters.md)), `service.health.check` status, deployment markers from the CI pipeline via the Datadog Deployment Tracking API.
* Datadog Service Catalog `dashboards` link; Datadog Teams for ownership.

## Conformance

Terraform plan check that the module is invoked once per service; drift detection that the generated dashboard has not been hand-edited (`datadog_dashboard_json` diff).

## Scholium

SHOULD until the template exists in Terraform and has been exercised by three services; then the principals may raise it to MUST.
