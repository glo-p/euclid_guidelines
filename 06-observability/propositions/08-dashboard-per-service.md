# Prop. VI.8 - One dashboard per service, generated from a standard template

| | |
|---|---|
| **Level** | SHOULD |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every service SHOULD have exactly one primary dashboard (Def. VI.15), generated from the company template from the service's contracts and `slo.yaml`, showing SLO status and error budget, RED metrics per operation and per consumer, consumer lag and DLQ depth, readiness, recent deployments and active alerts. Teams MAY add further dashboards; the primary one is not hand-edited.

## Given

Def. VI.15, CN 5, CN 7, Post. VI.2, Post. VI.3, Prop. VI.3, Prop. VI.4, Prop. VI.5.

## Demonstration

The standard metrics (Prop. VI.3), SLOs (Prop. VI.4) and health shape (Prop. VI.5) mean every service exposes the same signals, and by CN 5 one layout for one set of signals is enough; a second layout is a cost without benefit. A generated dashboard exists from the first deployment, before the team has had an incident to motivate building one, which by CN 7 is the moment it is needed. A responder from another team (Post. VI.3 puts the page with the owner, but incidents cross teams) can read any service's dashboard because they all read the same way. ∎ Q.E.D.

## Corollaries

* **Cor. VI.8.1** - The dashboard URL is recorded in the service catalogue entry and in every runbook header (Prop. VI.6).
* **Cor. VI.8.2** - A template change regenerates every service's dashboard; teams do not fork the template.

## Construction

* Terraform module `company-service-dashboard` producing `datadog_dashboard_json` from the OpenAPI document (operation list), event subscriptions and `slo.yaml`.
* Template panels: SLO summary widgets, `http.server.request.duration` by `operationId`, `messaging.process.duration` by event type, SQS age and DLQ depth (Prop. III.8), `service.health.check` status, deployment markers from the CI pipeline via the Datadog Deployment Tracking API.
* Datadog Service Catalog `dashboards` link; Datadog Teams for ownership.

## Conformance

Terraform plan check that the module is invoked once per service; drift detection that the generated dashboard has not been hand-edited (`datadog_dashboard_json` diff).

## Scholium

SHOULD until the template exists in Terraform and has been exercised by three services; then the principals may raise it to MUST.
